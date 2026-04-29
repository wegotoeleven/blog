---
title: "Jamf Pro's Reported IP Address fun and games"
category: technical
tags: [macadmin, jamf]
---

I had a user with an issue recently where their Mac record in Jamf Pro was not being populated with a correct "Reported IP Address" value when connected to the company's corporate WiFi. The IP address of the WiFi interface would report the correct value on the device and was singable.

For context, this issue:
- Persisted between inventory updates
- Would only ever be present when connecting to WiFi (i.e. the issue abated when disabling WiFi and connecting to Ethernet)
- Would not be resolved by forgetting the SSID and/or deleting and re-adding the network interface
- Would not be affected by renewing the DHCP lease.

Doing some digging, [the documentation](https://learn.jamf.com/bundle/technical-articles/page/Collecting_the_IP_Address_and_Reported_IP_Address_in_Jamf_Pro.html) for the "Reported IP Address" field in Jamf Pro notes that:
> The reported IP address is collected by the jamf binary during a check-in and can be received by executing the following command: 
> 
> ```ifconfig```

While this is technically correct, the documentation doesn't explicitly state where this information is collected from during an inventory; it's actually parsed from a file on disk, `/Library/Preferences/SystemConfiguration/preferences.plist`. Searching though this file for the incorrect IP address highlighted that the user had many, _many_ interfaces (typically caused when moving between desks that have hubs or monitors with build in Ethernet interfaces), and one of these used a Static IP address assignment:
```
IPV4 = {
	Addresses = (
		"10.14.28.1"
	);
	ConfigMethod = Manual;
	...
}
```
Finding and removing the static IP from the above interface immediately fixed the issue the next time that the Mac checked into Jamf Pro.

It does beg the question however, whether the "Reported IP Address" collection process in Jamf Pro could be improved to validate whether the interface is connected before logging it. 
