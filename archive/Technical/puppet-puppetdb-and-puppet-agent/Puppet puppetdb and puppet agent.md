---
title: "Puppet, PuppetDB and Puppet Agent"
category: technical
published: 2017-05-03
modified: 2022-08-29
tags: [stateless, management]
---

I like the idea of desired state management systems. You define a set of criteria, and your devices will indefinitely match that criteria, returning to the "Desired State" if required. I've done this a few times with Jamf Pro, using smart groups, extension attributes and policies to "mimic" the functionality of a desired state management system ([which was nice][1]) but I wanted to really get to grips with a system that's explicitly designed for this purpose. Enter the king of "Desired State" - _[Puppet][2]_.

In an effort to get to know Puppet, I need to build out some infrastructure. Yeah, I know there's a [learning VM][3] (that's really awesome, by the way), but I learn by experience. What better way than to build my own.

## Assumptions

I'm not going to get into the nitty gritty of what puppet actually is and does, and I'm going to assume that if you're reading this post, you:

1. Understand what Desired State is
2. Understand (roughly) what Puppet is and can do
3. Are poor / nuts, and would rather build and spin up your own servers, rather than using Puppet's [Enterprise version][4] (which is still totally viable, but I'm a masochist)

For the purposes of this example, I will setup 3 VMs and use the following defaults:

* VMware Fusion 8.5.6 to host the VMs
	* Network set to custom "vmnet2" network
		* NAT
		* "Connect Host Mac to this network"
		* Provide DHCP, subnet 10.0.1.0/24
* Puppet Server
	* Ubuntu 16.04
	* 2 x CPU Cores
	* 3072MB Memory
	* 50GB HDD (thin provisioned)
	* Hostname `puppet-server`
	* IP Address `10.0.1.129`
* Puppet Database Server
	* Ubuntu 16.04
	* 2 x CPU Cores
	* 3072MB Memory
	* 50GB HDD (thin provisioned)
	* Hostname `puppet-db`
	* IP Address `10.0.1.130`
* Puppet Test Client
	* macOS 10.12.4
	* 2 x CPU Cores
	* 8192MB Memory
	* 50GB HDD (thin provisioned)
	* Hostname `puppet-client`
	* IP Address `10.0.1.128`

Obviously, if you're following along with me, feel free to change any of the above defaults, but please make sure you're changing them in the instructions below.

## Network Configuration

1. Spin up your boxes with the above config, and set each device up to allow access via SSH (cause it's easier like)
1. Rename both servers by editing the hostname file on each device, and adding the desired hostname

	```bash
	$ sudo vi /etc/hostname
	```

2. In addition, add the hostname into the hosts file, pointing to the loopback (`127.0.0.1`) address under the `localhost` entry

	```bash
	$ sudo vi /etc/hosts
	```

	```
	127.0.0.1       localhost
	127.0.0.1       puppet-db
	```

3. In an environment without proper DNS, do the following:
	4. Set each device with a static IP address. I'm using the IP addresses specified above.

		```bash
		$ sudo vi /etc/network/interfaces
		```
		```
		# The primary network interface
				auto eth0
				iface eth0 inet static
				address 10.0.0.41
				netmask 255.255.255.0
				network 10.0.0.0
				broadcast 10.0.0.255
				gateway 10.0.0.1
				dns-nameservers 10.0.0.1 8.8.8.8
		```

	- Add each device into the each device's hosts file

		```bash
		$ sudo vi /etc/hosts
		```

		```
		10.0.1.129    puppet-server
		10.0.1.130    puppet-db
		10.0.1.128    puppet-client
		```

## Puppet Server Configuration

8. Install the NTP service on the Puppet server, because Puppet uses an NTP source when issuing certificates to it's nodes

	```bash
	$ sudo apt install ntp
	```

6. Add the Puppet repositories to the Puppet Server

	```bash
	$ wget https://apt.puppetlabs.com/puppetlabs-release-pc1-xenial.deb
	$ sudo dpkg -i puppetlabs-release-pc1-xenial.deb
	$ sudo apt update
	```

7. Install `puppetserver` on the Puppet Server (duh)

	```bash
	$ sudo apt install puppetserver
	```

8. In addition, install the "puppetdb-termini" package to enable the Puppet server to talk to the Puppet DB server

	```bash
	$ sudo /opt/puppetlabs/puppet/bin/puppet resource package puppetdb-termini ensure=latest
	```

10. Create the required configuration files on the Puppet server for the connections to the PuppetDB:
	- routes.yaml:

		```bash
		$ sudo vi /etc/puppetlabs/puppet/routes.yaml
		```

		```yaml
		---
		master:
		    facts:
		        terminus: puppetdb
		        cache: yaml
		```

	- puppetdb.conf:

		```bash
		$ sudo vi /etc/puppetlabs/puppet/puppetdb.conf
		```

		```
		[main]
		server_urls = https://puppet-db:8081
		```

20. Edit puppet.conf  


	```bash
	$ sudo vi /etc/puppetlabs/puppet/puppet.conf
	```

	```
	[master]
	storeconfigs = true
	storeconfigs_backend = puppetdb
	```

19. Ensure the permissions are correct on the Puppet configuration folder  

	```bash
	$ sudo chown -R puppet:puppet $(sudo /opt/puppetlabs/puppet/bin/puppet config print confdir)
	```

20. Set `puppetserver` to start automatically when the system starts

	```bash
	$ sudo /opt/puppetlabs/puppet/bin/puppet resource service puppetserver ensure=running enable=true
	```

21. Restart the Server to pickup the above changes.

## Puppet DB Server Configuration

6. Add the Puppet repositories to the Puppet DB Server

	```bash
	$ wget https://apt.puppetlabs.com/puppetlabs-release-pc1-xenial.deb
	$ sudo dpkg -i puppetlabs-release-pc1-xenial.deb
	```

7. Install the Database backend for PuppetDB  

	```bash
	$ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
	$ wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
	$ sudo apt update
	$ sudo apt install postgresql postgresql-contrib
	```

8. Setup PostgreSQL on the Puppet DB Server  

	```bash
	$ sudo -u postgres sh
	```

	```sql
	createuser -DRSP puppetdb
	createdb -E UTF8 -O puppetdb puppetdb
	psql puppetdb -c 'create extension pg_trgm'
	exit
	```

9. Restart the PostgreSQL service

	```bash
	$ sudo service postgresql restart
	```

10. Test PostgreSQL. If this is successful, you'll be dropped into PostgreSQL. Use `\q` to exit.

	```bash
	$ psql -h localhost puppetdb puppetdb
	```

11. To configure PuppetDB to use this database, add the following details to the `puppetdb` database configuration file. In this example, I'm using super secret usernames and passwords. Don't do this if you're planning on setting this up in prod.

	```bash
	$ sudo vi /etc/puppetlabs/puppetdb/conf.d/database.ini
	```

	```
	subname = //localhost:5432/puppetdb
	username = puppetdb
	password = puppetdb
	```

	For the `subname`, replace the hostname with the DB server’s hostname (or `localhost` if the PostgreSQL service is running on the same server), replace the port with the port on which PostgreSQL is listening (usually 5432), and replace the database with the name of the database you’ve created for use with PuppetDB. In addition, uncomment the following sections:  

	```
	classname
	subprotocol
	gc-interval
	```

## Connecting the servers together

20. Install the Puppet Agent on the Puppet DB Server  

	```bash
	$ sudo apt install puppet-agent
	```

21. On the Puppet DB server, set the name of the server that the agent should connect to, replacing the server with the required hostname of the Puppet server

	```bash
	$ sudo vi /etc/puppetlabs/puppet/puppet.conf
	```

	```
	server = puppet-server
	```

22. Enrol the Puppet DB Server into Puppet  

	```bash
	$ sudo /opt/puppetlabs/puppet/bin/puppet agent --test --waitforcert 60
	```

23. On the Puppet Server, check to see whether the Puppet DB server has requested a certificate  

	```bash
	$ sudo /opt/puppetlabs/puppet/bin/puppet cert --list
	```

24. You should see the name of the Puppet DB server, followed by some other stuff. Accept the cert request  

	```bash
	$ sudo /opt/puppetlabs/puppet/bin/puppet cert --sign puppet-db
	```

25. On the Puppet DB server, install the `puppetdb` service  

	```bash
	$ sudo /opt/puppetlabs/puppet/bin/puppet resource package puppetdb ensure=latest
	```

26. Stop the `puppetdb` service and setup SSL  

	```bash
	$ sudo service puppetdb stop && sudo /opt/puppetlabs/bin/puppetdb ssl-setup
	```

27. Ensure the `puppetdb` service runs on reboot  

	```bash
	$ sudo /opt/puppetlabs/puppet/bin/puppet resource service puppetserver ensure=running enable=true
	```

28. Restart the Puppet DB server
29. Check the PuppetDB logs on the Puppet DB server to verify that it's working  

	```bash
	$ sudo cat /var/log/puppetlabs/puppetdb/puppetdb.log
	```

## Enrolling a macOS client

1. Install the puppet agent onto the Puppet Client, by downloading the specific version as per the OS version, from [here][5]
2. Install the download pkg file. When asked, specify the hostname of the server and the client
3. Open Terminal
4. Edit the Puppet configuration file and add the following contents  

	```bash
	$ sudo vi /etc/puppetlabs/puppet/puppet.conf
	```

	```
	[main]
	logdir=/var/log/puppet
	vardir=/var/lib/puppet
	ssldir=/var/lib/puppet/ssl
	#rundir=/var/run/puppet
	factpath=$vardir/lib/facter
	templatedir=$confdir/templates

	[master]
	ssl_client_header = SSL_CLIENT_S_DN
	ssl_client_verify_header = SSL_CLIENT_VERIFY

	[agent]
	server=puppet-server
	certname=puppet-client
	report=true
	pluginsync=true
	```

5.  Request a cert from the Puppet Server  

	```bash
	$ sudo /opt/puppetlabs/puppet/bin/puppet agent --waitforcert 60 --test
	```

6. On the Puppet DB server, run the log file and keep an eye on it  

	```bash
	$ sudo tail -f /var/log/puppetlabs/puppetdb/puppetdb.log
	```

7. Then, on the Puppet Server, check and sign the cert request from `puppet-client`  

	```bash
	$ sudo /opt/puppetlabs/puppet/bin/puppet cert --list
	$ sudo /opt/puppetlabs/puppet/bin/puppet cert --sign puppet-client
	```

[1]:	https://www.youtube.com/watch?v=XOhZgAPn_CU
[2]:	https://puppet.com
[3]:	https://puppet.com/download-learning-vm
[4]:	https://puppet.com/product
[5]:	https://downloads.puppetlabs.com/mac/
