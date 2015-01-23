# VyOS-IntraCompany-Network
Note my internal network build step in the Intra-Company Network.

#### Network and System configuration

```
(Intra-Company Network : LAN)   
(Intra [DHCP])
 |
(Intra [DNS])
 | IP:172.19.1.1
 |
[host pc]                    | (my internal network) : by virtualbox
 | IP:172.19.1.53            | 
 | GW:172.19.1.254           | 
 |                           |              [DHCP] : (ubuntu14.04 svr)
 |                           |                 | IP:192.168.1.10
 |                           |                 | 
 |                           |              [ProxyDHCP] : (ubuntu14.04 svr)
 |                           |                 | IP:192.168.1.11
 |                           |                 | 
 |                           |              [vBox01] : (CoreOS) guest vm
 |                           |                 | IP:192.168.1.121
 |                           |                 |
 |                           |  *nic2 : eth1   | 
 |                           |  IP:192.168.1.1 | 
 |           -----------=[vyos01]=-------------+ 
 |           *nic1 : eth0    |
 |           IP:172.19.1.20  |
 |    add -> IP:172.19.1.200 |
 |                           | 

```
```
 [vyos01]: router
  mmry: 512 MB
  hdd :   2 GB
  cpu :   1 core
  app : [VyOS 3.13.11-1-amd64-vyos]  (uname -r)
  os  : [Debian 6.0,10] (cat /etc/debian_version)
  iso : vyos-1.1.1-amd64.iso
  
  (networking configuration)
  adapter1 : virt-io, bridge
  adapter0 : virt-io, internal network
```

#### VyOS Install

```
 (LiveCD:vyos-1.1.1-amd64.iso) loading ...
 first login : vyos/vyos
 
 install image (or install system)
 (need answer to a few questions - Enter [Yes] or [Return].)
 
 (finished these questions,then shutdown and eject the boot image.)
 sudo shutdown -h 0
 (eject DVD)
 
 ([vyos01] start. power on.)
```

###### TroubleShooting
**INIT: Id "T0" respawning too fast: disabled for 5 minites**
```
configure
delete system console device ttyS0
commit
save
```

#### VyOS Configuration
*all configuration need [configure]->(set)->[commit]->[save]*

###### hostname
```
set system host-name vyos01
```
###### network interface
```
set interfaces ethernet eth0 address 172.19.1.20/24
set interfaces ethernet eth0 description Intra-Company

set interfaces ethernet eth0 address 192.168.0.1/24
set interfaces ethernet eth0 description Internal
```
###### ssh (for using teraterm)
```
set service ssh port 22
```
###### gateway
```
set system gateway-address 172.19.1.254
```
###### dns
```
set service dns forwarding cache-size 0
set service dns forwarding listen-on eth0
set service dns forwarding address 172.19.1.1
```
**_[at this point : 01]_**  
```
(internal to the router and outer.)  
1. ping from [vbox01] to [vyos01] is OK.  
2. ping from [vbox01] to [hostpc] is Error. (no route.)  
(HostPC to the router and internal nt.)  
3. ping from [hostpc] to [vyos01] is OK.  
4. ping from [hostpc] to [vbox01] is Error. (these 2 network is different.)  
(Router to the hostpc and internal nt.)   
5. ping from [vyos01] to [vbox01] is OK.  
6. ping from [vyos01] to [hostpc] is OK.  
```
###### outbound nat - route from [vbox01] to [hostpc] (and more).
```
set nat source rule 10 outbound-interface eth0
set nat source rule 10 source address 192.168.1.0/24
set nat source rule 10 translation address masquerade
```
⇒2. ping from [vbox01] to [hostpc] is ~~Error~~. - Change to OK!

###### reverse nat - address forwarding
```
set interfaces ethernet eth0 address 172.19.1.200/32
set nat destination rule 20 destination address 172.19.1.200
set nat destination rule 20 inbound-interface eth0
set nat destination rule 20 translation address 192.168.1.121
set nat source rule 20 outbound-interface eth0
set nat source rule 20 source address 192.168.1.121
set nat source rule 20 translation address 172.19.1.200
```
⇒4. ping from [hostpc] to [vbox01] is ~~Error~~. - OK! Translation into internal ip 

###### other configuration
```
(adjust the time) - in Japan
delete system ntp
set system ntp server ntp.jst.mfeed.ad.jp

(set TimeZone)
set system time-zone Asia/Tokyo
```

done!
