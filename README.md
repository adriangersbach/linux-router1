# linux-router1


# PI-BAS0001

## Install

```
sudo apt-get install bridge-utils
sudo apt-get install vlan
sudo apt-get install quagga
```



Edit /etc/network/interfaces.d/vlans
```
# VLAN 200 Interface
auto eth0.200
iface eth0.200 inet manual
    vlan-raw-device eth0

# VLAN 400 Interface
auto eth0.400
iface eth0.400 inet manual
    vlan-raw-device eth0

# VLAN 410 Interface
auto eth0.410
iface eth0.410 inet manual
    vlan-raw-device eth0

# VLAN 420 Interface
auto eth0.420
iface eth0.420 inet manual
    vlan-raw-device eth0
```
Edit /etc/network/interfaces.d/bridges
```
# br 40 Interface
auto br40
iface br40 inet manual
    bridge_ports eth0.40 tap40
    bridge_stp off       # disable Spanning Tree Protocol
```

Edit /etc/dhcpcd.conf
```
# Static IP configuration for VLAN 200
interface eth0.200
static ip_address=10.200.2.5/16
static routers=10.200.2.1

# Static IP configuration for br 40
interface br40
static ip_address=10.40.2.5/16
static routers=10.40.2.1
```

Edit /etc/quagga/daemons
```
nano /etc/quagga/daemons

zebra=yes
ospfd=yes
```

Edit /etc/quagga/ospfd.conf
```
sudo nano /etc/quagga/ospfd.conf

hostname pi-bas0001
!
password zebra
enable password password
!
interface eth0
!
interface eth0.300
!
interface tun0
 ip ospf cost 50
 ip ospf priority 0
!
interface lo
!
router ospf
 ospf router-id 10.10.0.10
 network 192.168.21.0/24 area 0.0.0.0
 network 192.168.27.0/24 area 0.0.0.0
 network 10.0.101.0/30 area 0.0.0.0
!
line vty
 no login
!
no exec-timeout
```

Edit /etc/quagga/zebra.conf
```
sudo nano /etc/quagga/zebra.conf

!
password zebra
enable password password
!
interface eth0
link-detect
ip address 192.168.21.5/24
!
interface eth0.300
link-detect
ip address 192.168.27.5/24
!
interface tun0
link-detect
ip address 10.0.101.1/30
!
interface lo
link-detect
!
no log trap
log stdout
!
line vty
no login
!
no exec-timeout
```

```
sudo /etc/init.d/quagga start
```
```
show running-config
show ip ospf interface
show ip ospf database
show ip ospf border-routers
```

iptables
```
iptables -I FORWARD -m physdev --physdev-out tap40 -p udp --dport 67:68 -j DROP
iptables -I FORWARD -m physdev --physdev-in tap40 -p udp --dport 67:68 -j DROP
iptables -I INPUT -m physdev --physdev-in tap40 -p udp --dport 67:68 -j DROP
```



# PI-BAS0002


## Links
https://www.sbprojects.net/projects/raspberrypi/vlan.php
https://superuser.com/questions/1141983/how-to-stop-dhcp-traffic-via-openvpn-bridge
https://www.networkworld.com/article/2225768/cisco-subnet/dual-protocol-routing-with-raspberry-pi.html
