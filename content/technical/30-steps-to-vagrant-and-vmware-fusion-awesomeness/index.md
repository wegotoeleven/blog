---
title: "30 Steps to Vagrant and VMWare Fusion Awesomeness"
date: 2020-03-02
lastmod: 2022-08-29
categories: ["technical"]
tags: ["macadmin", "virtualisation"]
---

Vagrant is pretty cool, but of late I've been having real issues finding a Vagrant base box that works with VMware Fusion OOTB. My blogging workflow (used to) involve(s) using a Vagrant box with Jekyll to generate a static site on my local Mac, and then uploading to an S3 bucket. The Vagrant Jekyll boxes weren't up to scratch, so I've decided to figure out how to create my own.

This guide covers how to create the box and add it to your list of boxes, for use in specifying within a Vagrantfile. It doesn't cover Vagrant fundementals, so I assume that you'll have some basic knowledge in how it works before embarking on this magical quest.
1. Open VMware Fusion
2. Select “Install from disc or image” and click "Continue"
    ![][image-1]
3. Locate the ISO for the distribution you want to install, then select “Open”, and click "Continue". 
    ![][image-2]
4. Unselect the checkbox for “Use Easy Install”
    ![][image-3]
5. Choose “Customize Settings”, and enter a name for the VMware file (`name/OSarch`, i.e. `wegotoeleven/trusty64`) and select “Save"
    ![][image-4]
6. Customise the VM’s settings
	1. Turn on Shared Folders
        ![][image-5]
	2. Set memory to desired size, at least 1024MB
        ![][image-6]
	3. Change the Networking to "Share with my Mac" (aka NAT)
        ![][image-7]
	4. Change the disk size to 40 GB and deselect “Split into 2GB files”
        ![][image-8]
	5. Turn off the sound card
        ![][image-9]
	6. Expand "Advanced USB options" and select "Remove USB Controller"
        ![][image-10]
	7. Untick "Share Mac printers with Linux"
        ![][image-11]
7. Boot the VM, install the Guest OS and customise the following settings when prompted:
    ![][image-12]
8. Host Name: Same as the second part of the name of the VMware file (i.e. trusty64)
    ![][image-13]
9. Full Name: `vagrant`
    ![][image-14]
10. User: `vagrant`
    ![][image-15]
11. Password: `vagrant`
    ![][image-16]
12. Once setup and at the command prompt, login and change the Root Password. When asked, set to `vagrant`
	
	```bash
	$ sudo passwd root
	```
13. Update the OS because updates
	
	```bash
	$ sudo apt update && sudo apt upgrade -y
	```
14. Update the Sudoers file with the following to allow the vagrant user sudo rights without having to authenticate with a password
	
	```
	#Defaults !visiblepw
	Defaults env_keep="SSH_AUTH_SOCK"
	```
15. Test. Is the following command outputs the current directory instead of an error, all is good
	
	```bash
	$ sudo pwd
	```
16. Restart VM
	
	```bash
	$ sudo shutdown -r now
	```
17. Make ssh folder
	
	```bash
	$ mkdir ~/.ssh
	```
18. Set permissions
	
	```bash
	$ chmod 700 ~/.ssh
	```
19. Download Vagrant authorised keys
	
	```bash
	$ wget --no-check-certificate https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub -O ~/.ssh/authorized_keys
	```
20. Set permissions on authorised keys file
	
	```bash
	$ chmod 600 ~/.ssh/authorized_keys && chown -R vagrant ~/.ssh
	```
21. Install OpenSSH
	
	```bash
	$ sudo apt install openssh-server -y
	```
22. Edit the SSH config file
	
	```bash
	$ sudo nano /etc/ssh/sshd_config
	```
23. Restart SSH service
	
	```bash
	$ sudo service ssh restart
	```
24. Install VMware Tools. Begin by mounting the VMware Tools by selecting Virtual Machines \> Install VMware Tools from the menu bar menu
	
	```bash
	$ sudo mkdir -p /mnt/cdrom
	$ sudo mount /dev/cdrom /mnt/cdrom
	$ tar xzvf /mnt/cdrom/VMwareTools-9.2.2-893683.tar.gz -C /tmp
	$ cd /tmp/vmware-tools-distrib/
	$ sudo ./vmware-install.pl\
	```
25. Wipe the free space on the VM to stop fragmentation
	
	```bash
	$ sudo dd if=/dev/zero of=/EMPTY bs=1M
	$ sudo rm -f /EMPTY
	```
26. Shutdown the VM
	
	```bash
	$ sudo shutdown -h now
	```
27. Navigate to the location of your vmwarevm. By default this location is `~/Virtual Machines/`.
	
	```bash
	$ cd ~/Virtual Machines/wegotoeleven:xenial64.vmwarevm
	```
	Note: Any forward slashes (`/`) will be translated into colons if the name of the VMware file contains a forward slash
28. Create a file named `metadata.json` and enter the following contents:
	
	```json
	{
	  "provider": "vmware_fusion"
	}
	```
29. Create a file named `Vagrantfile` and enter the following contents.
	
	```ruby
	# -*- mode: ruby -*-
	# vi: set ft=ruby
	
	Vagrant.configure("2") do |config|
	  config.vm.provider :vmware_fusion do |v, override|
	    v.gui = false
	  end
	end
	```

30. We next want to optimize the box to reduce it’s size:
	
	```
	$ /Applications/VMware\ Fusion.app/Contents/Library/vmware-vdiskmanager -d Virtual\ Disk.vmdk
	$ /Applications/VMware\ Fusion.app/Contents/Library/vmware-vdiskmanager -k Virtual\ Disk.vmdk
	```
31. Finally compress the box:
	
	```bash
	$ tar cvzf package.box ./
	```
32. Add the box to Vagrant
	
	```bash
	$ vagrant box add wegotoeleven/xenial64 package.box
	```

I couldn't have done this without the following blog posts:
- [creating-a-custom-box-from-scratch][link-1]
- [building-a-vagrant-box][link-2]

And yeah. I know I said 30. 

[image-1]:	images/2015-09-28-01.png
[image-2]:	images/2015-09-28-02.png
[image-3]:	images/2015-09-28-03.png
[image-4]:	images/2015-09-28-04.png
[image-5]:	images/2015-09-28-05.png
[image-6]:	images/2015-09-28-06.png
[image-7]:	images/2015-09-28-07.png
[image-8]:	images/2015-09-28-08.png
[image-9]:	images/2015-09-28-09.png
[image-10]:	images/2015-09-28-10.png
[image-11]:	images/2015-09-28-11.png
[image-12]:	images/2015-09-28-12.png
[image-13]:	images/2015-09-28-13.png
[image-14]:	images/2015-09-28-14.png
[image-15]:	images/2015-09-28-15.png
[image-16]:	images/2015-09-28-16.png
[link-1]:   https://www.skoblenick.com/vagrant/creating-a-custom-box-from-scratch
[link-2]:   https://blog.engineyard.com/2014/building-a-vagrant-box
