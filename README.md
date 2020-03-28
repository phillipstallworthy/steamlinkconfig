# Steam Link
Raspberry to Windows 10 SteamLink config

## Wot?
Raspberry pi connected to SteamLink to Windows 10 gaming PC (PC)
Wifi - too slow.
Both connected to modem by network cable - still slow (10 Mbps)
Network cable runs between them for Steam, wifi to the modem. Challenge.

## Both connected to the modem
Network cable from Pi and Windows to modem.
Make sure all config files on Pi are default, IE dhcpcd.config
Retore Windows ethernet configuration to auto, dhcp.
Turn off wifi on both
Ta-da! It just works. Not very well, only gets 10 Mbs, but does work.
Note. Modem was caching static IP that I set up earlier, fixed by modem reboot.

## Cable between them.

###ethernet cable.
Connect the pi to the pc with an ethernet cable. This needs to be a simple straight RG45 cable. No crossover required. (not sure why, modern network cards just switch modes I guess)
Make sure the green and orange LED's go on.

### static ip on Pi
ifconfig on the PI show the WIFI connected to wlan0 with an IP of 192.168.1.11
So I decided to use 192.168.2.2 as my static ip.
Graphical method:
Right click the wifi symbol and select Wireless and Wired network settings.
Configure inteface eth0 
IPv4 address = 192.168.2.2/24 (the 24 is CIDR notation for a subnet of 255.255.255.0, which means the first 3 digits are for network, the last digit for the host)
I selected Auotmatically configure empty options and Disable IPv6 - not sure if this is important or not.

###Brain dump

Do I need the metrics?

cat /etc/dhcpcd.conf
#Phill Pot, 21 Mar 2020, static ip, rounter is win PC

interface wlan0
metric 301

interface wlan1
metric 300

interface eth0
hostname 
clientid 
persistent 
option rapid_commit
option domain_name_servers, domain_name, domain_search, host_name
option classless_static_routes
option interface_mtu
require dhcp_server_identifier
slaac private
noipv6 
inform 192.168.2.2/24
metric 200

Step 3 - static route on the PI
Maybe there is a more genric way of doing it, but i did
route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    301    0        0 wlan0
192.168.1.0     0.0.0.0         255.255.255.0   U     301    0        0 wlan0
All routes go the wlan0 - IE the wifi. Not what we want. Wifi is not fast enough for SteamLink.

ping 192.168.2.1 works, but it goes via the WIFI router. How do I know??

$ ip route get  192.168.2.1 
192.168.2.1 dev wlan0 src 192.168.2.2 uid 1000

so add a route:
sudo route add -host 192.168.2.1  dev eth0

route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    301    0        0 wlan0
192.168.1.0     0.0.0.0         255.255.255.0   U     301    0        0 wlan0
192.168.2.1     0.0.0.0         255.255.255.255 UH    0      0        0 eth0

route to the host 192.168.2.1 is not via the eth0 interface. 

Again, there may be anther way (metrics??), but this does work.

### Add a static IP to windows


### turn off the windows firewall

return route from windows to PI?? tracert command
do we need a route on windows?? dunno how to do that?



### Pi commands

* `$ sudo service dhcpcd status` **"status of  dhcp, where did it get the ip from?"**
* `$ tail -n100 /var/log/daemon.log`
* `$ tail -n100 /var/log/syslog`
* `$ tail -n100 /var/log/kern.log`
* `$ tail -n100 /var/log/auth.log`
* `$ cat etc/os-release` **"version of pi OS"**
* `$ cmd` **"doc"**


### Got info
https://en.wikipedia.org/wiki/IP_address