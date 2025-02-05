# Переход на полноценную L3 сеть(архитектура - CLOS) в центре обработки данных предприятия
![alt-dtp](https://github.com/vk1391/OTUS_DC_PROJECT/blob/main/project2.png)
 - В качестве underlay - eBGP
 - сервер - ubuntu-server 22.04 с установленным FRR.
1. Конфигурация оборудования сети CLOS:
- Spine1:
```
service routing protocols model multi-agent
!
spanning-tree mode mstp
!
interface Ethernet1
   no switchport
   ip address 10.1.1.1/30
!
interface Ethernet2
   no switchport
   ip address 10.1.2.1/30
!
interface Ethernet3
   no switchport
   ip address 10.1.3.1/30
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 1.1.1.1/32
!
interface Management1
!
ip routing
!
route-map lo0 permit 10
   match interface Loopback0
!
router bgp 101
   router-id 1.1.1.1
   maximum-paths 3
   neighbor DCI peer group
   neighbor DCI out-delay 0
   neighbor DCI bfd
   neighbor DCI bfd interval 100 min-rx 100 multiplier 3
   neighbor DCI timers 3 9
   neighbor LEAF1 peer group DCI
   neighbor LEAF1 remote-as 102
   neighbor LEAF2 peer group DCI
   neighbor LEAF2 remote-as 103
   neighbor LEAF3 peer group DCI
   neighbor LEAF3 remote-as 104
   neighbor 10.1.1.2 peer group LEAF1
   neighbor 10.1.2.2 peer group LEAF2
   neighbor 10.1.3.2 peer group LEAF3
   redistribute connected route-map lo0
```
- Spine2:
```
service routing protocols model multi-agent
!
spanning-tree mode mstp
!
interface Ethernet1
   no switchport
   ip address 10.2.1.1/30
!
interface Ethernet2
   no switchport
   ip address 10.2.2.1/30
!
interface Ethernet3
   no switchport
   ip address 10.2.3.1/30
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 2.2.2.2/32
!
interface Management1
!
ip routing
!
route-map lo0 permit 10
   match interface Loopback0
!
router bgp 101
   maximum-paths 3
   neighbor DCI peer group
   neighbor DCI out-delay 0
   neighbor DCI bfd
   neighbor DCI bfd interval 100 min-rx 100 multiplier 3
   neighbor DCI timers 3 9
   neighbor LEAF1 peer group DCI
   neighbor LEAF1 remote-as 102
   neighbor LEAF2 peer group DCI
   neighbor LEAF2 remote-as 103
   neighbor LEAF3 peer group DCI
   neighbor LEAF3 remote-as 104
   neighbor 10.2.1.2 peer group LEAF1
   neighbor 10.2.2.2 peer group LEAF2
   neighbor 10.2.3.2 peer group LEAF3
   redistribute connected route-map lo0
!
end
```
- Leaf1:
```
service routing protocols model multi-agent
!
spanning-tree mode mstp
!
interface Ethernet1
   no switchport
   ip address 10.1.1.2/30
!
interface Ethernet2
   no switchport
   ip address 10.2.1.2/30
!
interface Ethernet3
   speed 10full
   no switchport
   ip address 11.0.0.1/30
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 11.11.11.11/32
!
interface Management1
!
ip routing
!
route-map lo0 permit 10
   match interface Loopback0
!
route-map lo0 permit 20
   match interface Ethernet3
!
router bgp 102
   router-id 11.11.11.11
   maximum-paths 2
   neighbor CLOS peer group
   neighbor CLOS bfd
   neighbor CLOS bfd interval 100 min-rx 100 multiplier 3
   neighbor SRV peer group
   neighbor SRV remote-as 105
   neighbor SRV bfd
   neighbor 10.1.1.1 peer group CLOS
   neighbor 10.1.1.1 remote-as 101
   neighbor 10.2.1.1 peer group CLOS
   neighbor 10.2.1.1 remote-as 101
   neighbor 11.0.0.2 peer group SRV
   redistribute connected route-map lo0
   !
   address-family ipv4
      neighbor CLOS activate
      neighbor SRV activate
!
end
```
Leaf2:
```
service routing protocols model multi-agent
!
spanning-tree mode mstp
!
interface Port-Channel2
!
interface Ethernet1
   no switchport
   ip address 10.1.2.2/30
!
interface Ethernet2
   no switchport
   ip address 10.2.2.2/30
!
interface Ethernet3
   speed 10full
   no switchport
   ip address 10.0.0.1/30
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 22.22.22.22/32
!
interface Management1
!
ip routing
!
route-map lo0 permit 10
   match interface Loopback0
!
route-map lo0 permit 20
   match interface Ethernet3
!
router bgp 103
   maximum-paths 2
   neighbor CLOS peer group
   neighbor CLOS bfd
   neighbor CLOS bfd interval 100 min-rx 100 multiplier 3
   neighbor SRV peer group
   neighbor SRV out-delay 0
   neighbor SRV bfd
   neighbor SRV timers 3 9
   neighbor 10.0.0.2 peer group SRV
   neighbor 10.0.0.2 remote-as 105
   neighbor 10.1.2.1 peer group CLOS
   neighbor 10.1.2.1 remote-as 101
   neighbor 10.2.2.1 peer group CLOS
   neighbor 10.2.2.1 remote-as 101
   redistribute connected route-map lo0
   !
   address-family ipv4
      neighbor SRV activate
!
end
```
- Leaf3:
```
service routing protocols model multi-agent
!
spanning-tree mode mstp
!
interface Ethernet1
   no switchport
   ip address 10.1.3.2/30
!
interface Ethernet2
   no switchport
   ip address 10.2.3.2/30
!
interface Ethernet3
   no switchport
   ip address 100.0.0.1/30
!
interface Ethernet4
!
interface Ethernet5
!
interface Ethernet6
!
interface Ethernet7
!
interface Ethernet8
!
interface Loopback0
   ip address 33.33.33.33/32
!
interface Management1
!
ip routing
!
route-map lo0 permit 10
   match interface Loopback0
!
route-map lo0 permit 20
   match interface Ethernet3
!
router bgp 104
   bgp listen range 10.0.0.0/14 peer-group CLOS remote-as 101
   neighbor CLOS peer group
   neighbor CLOS bfd
   neighbor CLOS bfd interval 100 min-rx 100 multiplier 3
   redistribute connected route-map lo0
!
end
```
2. Подготовка сервера
- взят образ ubuntu strver 22.04
- сделаны следующие шаги:
  - rmmod floppy
  - echo 'blacklist floppy' >> /etc/modprobe.d/blacklist.conf
  - dpkg-reconfigure initramfs-tools
  - reboot
- задана скорость на интерфейсах подключенных к коммутатору:
  - ethtool -s ens3 speed 10 duplex full
  - ethtool -s ens4 speed 10 duplex full
- применена следующая конфигурация netplan(меняется ip):
```
network:
  ethernets:
    ens3:
      dhcp4: no
      addresses:
        - 10.0.0.2/30
    ens4:
      dhcp4: no
      addresses:
        - 11.0.0.2/30
    ens5:
      dhcp4: true
    lo0:
      dhcp4: no
      addresses:
        - 105.105.105.105/32
        
  version: 2
```
- установлен FRR,включены модули bgp,bfd(etc/frr/daemons)
- применена следующая конфигурация к FRR:
```
frr version 8.1
frr defaults traditional
hostname ubuntu22-server
log syslog informational
no ipv6 forwarding
service integrated-vtysh-config
!
interface lo
 ip address 105.105.105.105/32 label lo:0
exit
!
router bgp 105
 no bgp ebgp-requires-policy
 neighbor 10.0.0.1 remote-as 103
 neighbor 10.0.0.1 bfd
 neighbor 10.0.0.1 bfd profile srv
 neighbor 10.0.0.1 advertisement-interval 0
 neighbor 10.0.0.1 timers 3 9
 neighbor 11.0.0.1 remote-as 102
 neighbor 11.0.0.1 bfd
 neighbor 11.0.0.1 bfd profile srv
 neighbor 11.0.0.1 advertisement-interval 0
 neighbor 11.0.0.1 timers 3 9
 !
 address-family ipv4 unicast
  redistribute connected
  neighbor 10.0.0.1 soft-reconfiguration inbound
  neighbor 10.0.0.1 route-map lo0 out
  neighbor 11.0.0.1 soft-reconfiguration inbound
  neighbor 11.0.0.1 route-map lo0 out
  maximum-paths 2
 exit-address-family
exit
!
route-map lo0 permit 10
 match interface lo
exit
!
bfd
 profile srv
  transmit-interval 100
  receive-interval 100
 exit
 !
exit
```
3. Проверка:
- таблица соседства bgp spine1:
```
localhost#sh ip bgp sum
BGP summary information for VRF default
Router identifier 1.1.1.1, local AS number 101
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.1.1.2 4 102             2313      2205    0    0 00:12:26 Estab   3      3
  10.1.2.2 4 103             2298      2198    0    0 00:12:33 Estab   1      1
  10.1.3.2 4 104             2414      2214    0    0 01:19:25 Estab   2      2
```
- таблица соседства bgp spine2:
```
localhost#sh ip bgp sum
BGP summary information for VRF default
Router identifier 2.2.2.2, local AS number 101
Neighbor Status Codes: m - Under maintenance
  Neighbor V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  10.2.1.2 4 102            34326     34749    0    0 00:14:36 Estab   3      3
  10.2.2.2 4 103            34203     34639    0    0 00:14:43 Estab   1      1
  10.2.3.2 4 104            34631     34003    0    0 00:18:16 Estab   2      2
```
таблица соседства bgp сервера:
```
ubuntu22-server(config-router)# do sh ip bgp sum

IPv4 Unicast Summary (VRF default):
BGP router identifier 105.105.105.105, local AS number 105 vrf-id 0
BGP table version 214
RIB entries 19, using 3496 bytes of memory
Peers 2, using 1446 KiB of memory

Neighbor        V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
10.0.0.1        4        103       110        84        0    0    0 00:00:07            8        1 N/A
11.0.0.1        4        102      1570      1224        0    0    0 00:17:10            8        1 N/A

```
- таблица маршрутизации сервера:
```
B>* 1.1.1.1/32 [20/0] via 11.0.0.1, ens4, weight 1, 00:17:45
B>* 2.2.2.2/32 [20/0] via 11.0.0.1, ens4, weight 1, 00:17:45
C>* 10.0.0.0/30 is directly connected, ens3, 01:01:32
C>* 11.0.0.0/30 is directly connected, ens4, 01:01:32
B>* 11.11.11.11/32 [20/0] via 11.0.0.1, ens4, weight 1, 00:17:46
B>* 22.22.22.22/32 [20/0] via 10.0.0.1, ens3, weight 1, 00:00:42
B>* 33.33.33.33/32 [20/0] via 11.0.0.1, ens4, weight 1, 00:17:45
B>* 100.0.0.0/30 [20/0] via 11.0.0.1, ens4, weight 1, 00:17:45
C>* 105.105.105.105/32 is directly connected, lo, 01:01:32
```
- проверка работы протокола BGP с обвязкой BFD(отключались по питанию Leaf1 и Leaf2,пинг от ПК до сервера):
```
84 bytes from 105.105.105.105 icmp_seq=458 ttl=61 time=61.720 ms
84 bytes from 105.105.105.105 icmp_seq=459 ttl=61 time=25.902 ms
84 bytes from 105.105.105.105 icmp_seq=460 ttl=61 time=24.037 ms
84 bytes from 105.105.105.105 icmp_seq=461 ttl=61 time=21.461 ms
84 bytes from 105.105.105.105 icmp_seq=462 ttl=61 time=25.136 ms
84 bytes from 105.105.105.105 icmp_seq=463 ttl=61 time=28.982 ms
84 bytes from 105.105.105.105 icmp_seq=464 ttl=61 time=35.033 ms
84 bytes from 105.105.105.105 icmp_seq=465 ttl=61 time=23.588 ms
84 bytes from 105.105.105.105 icmp_seq=466 ttl=61 time=39.454 ms
84 bytes from 105.105.105.105 icmp_seq=467 ttl=61 time=28.363 ms
84 bytes from 105.105.105.105 icmp_seq=468 ttl=61 time=23.357 ms
84 bytes from 105.105.105.105 icmp_seq=469 ttl=61 time=21.474 ms
84 bytes from 105.105.105.105 icmp_seq=470 ttl=61 time=24.655 ms
105.105.105.105 icmp_seq=471 timeout
84 bytes from 105.105.105.105 icmp_seq=472 ttl=61 time=27.550 ms
84 bytes from 105.105.105.105 icmp_seq=473 ttl=61 time=26.904 ms
84 bytes from 105.105.105.105 icmp_seq=474 ttl=61 time=21.967 ms
84 bytes from 105.105.105.105 icmp_seq=475 ttl=61 time=25.748 ms
84 bytes from 105.105.105.105 icmp_seq=476 ttl=61 time=23.743 ms
84 bytes from 105.105.105.105 icmp_seq=477 ttl=61 time=23.919 ms
84 bytes from 105.105.105.105 icmp_seq=478 ttl=61 time=23.688 ms
84 bytes from 105.105.105.105 icmp_seq=479 ttl=61 time=22.977 ms
84 bytes from 105.105.105.105 icmp_seq=480 ttl=61 time=32.858 ms
```
- терялся один пакет.С учетом что лабороторный стенд был построен на Pnetlab, который полностью использовал свой RAM,считаю результат удовлетворительным.
