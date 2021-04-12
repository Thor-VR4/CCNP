# VxLAN. Multipod
Лабараторное задание №8.

## Задание
Настроить связанность по технологии Multipod
1. [Настроите BGP peering между Spine в одной зоне и во второй](#chapter-0)
2. [Настроите route-type 2 и route-type 5 между всеми Leaf](#chapter-0)
![alt-текст](https://github.com/Thor-VR4/CCNP/blob/main/HomeWork/%238%20Multipod/Multipod.jpg "Стенд №1")

<a id="chapter-0"></a>
## Настроите BGP peering между Spine в одной зоне и во второй
Настраиваем BGP peering между AS65000 и AS65001.  
Пример настройки на NXOS6:
```
route-map UNCHANGED permit 10
  set ip next-hop unchanged
router bgp 65001
  template peer SPINE
    remote-as 65000
    update-source loopback0
    ebgp-multihop 2
    address-family l2vpn evpn
      send-community
      send-community extended
      route-map UNCHANGED out
  neighbor 172.16.0.1
    inherit peer SPINE
  neighbor 172.16.0.2
    inherit peer SPINE
  vrf R9
    address-family ipv4 unicast
      network 192.168.4.0/24
```
Состояния соседства:
```
NXOS6# show bgp l2vpn evpn summary 
BGP summary information for VRF default, address family L2VPN EVPN
BGP router identifier 172.16.0.6, local AS number 65001
BGP table version is 19, L2VPN EVPN config peers 2, capable peers 2
8 network entries and 12 paths using 2256 bytes of memory
BGP attribute entries [8/1312], BGP AS path entries [1/6]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
172.16.0.1      4 65000      45      32       19    0    0 00:18:56 4         
172.16.0.2      4 65000      44      32       19    0    0 00:17:30 4         
```


<a id="chapter-1"></a>
## Настроите route-type 2 и route-type 5 между всеми Leaf

Для настройки VxLAN в режиме Multipod необходимо задать route-targed метки на LEAF:
```
vrf context R9
  vni 99000
  rd auto
  address-family ipv4 unicast
    route-target import 9999:99000
    route-target import 9999:99000 evpn
    route-target export 9999:99000
    route-target export 9999:99000 evpn
    route-target both auto
    route-target both auto evpn
```
Вывод BGP AF l2vpn на NXOS6:
```
   Network            Next Hop            Metric     LocPrf     Weight Path
Route Distinguisher: 172.16.0.4:5
* e[5]:[0]:[0]:[24]:[192.168.44.0]/224
                      172.17.1.4                                     0 65000 i
*>e                   172.17.1.4                                     0 65000 i

Route Distinguisher: 172.16.0.4:32811
* e[2]:[0]:[0]:[48]:[aabb.cc80.c000]:[32]:[192.168.44.12]/272
                      172.17.1.4                                     0 65000 i
*>e                   172.17.1.4                                     0 65000 i

Route Distinguisher: 172.16.0.5:4
* e[5]:[0]:[0]:[24]:[192.168.44.0]/224
                      172.17.1.4                                     0 65000 i
*>e                   172.17.1.4                                     0 65000 i

Route Distinguisher: 172.16.0.5:32811
* e[2]:[0]:[0]:[48]:[aabb.cc80.c000]:[32]:[192.168.44.12]/272
                      172.17.1.4                                     0 65000 i
*>e                   172.17.1.4                                     0 65000 i

Route Distinguisher: 172.16.0.6:32771    (L2VNI 10004)
*>l[2]:[0]:[0]:[48]:[aabb.cc00.9000]:[0]:[0.0.0.0]/216
                      172.17.0.6                        100      32768 i
*>l[2]:[0]:[0]:[48]:[aabb.cc00.9000]:[32]:[192.168.4.9]/272
                      172.17.0.6                        100      32768 i
*>l[3]:[0]:[32]:[172.17.0.6]/88
                      172.17.0.6                        100      32768 i

Route Distinguisher: 172.16.0.6:3    (L3VNI 99000)
*>l[5]:[0]:[0]:[24]:[192.168.4.0]/224
                      172.17.0.6                        100      32768 i

```

## Проверка роботоспособности

Таблица адресации:
Устройство | VLAN | VNI | IPv4 
--- | --- | --- | --- 
R9 | 4 | 10004 | 192.168.4.9 
R12 | 44 | 10044 | 192.168.44.12

Доступность R9 c R12:
```
R12#ping 192.168.4.9
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.4.9, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 49/130/316 ms
```