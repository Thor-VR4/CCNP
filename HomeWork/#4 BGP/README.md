# BGP
Лабараторное задание №4.

## Задание
1. [Настроить BGP в Underlay сети, для IP связанности между всеми устройствами NXOS](#chapter-0)
2. [Проверить роботоспособность](#chapter-1)

<a id="chapter-0"></a>
## Настроить BGP в Underlay сети, для IP связанности между всеми устройствами NXOS
Стенд разбит на несколько AS:
1. 65000 - SuperSpine
2. 65001 - Spine в первом POD 
3. 65002 - Spine во втором POD 
4. 65004-7 - Leaf
![alt-текст](https://github.com/Thor-VR4/CCNP/blob/main/HomeWork/%234%20BGP/BGP_underlay.jpg "Стенд №2")

Пример настройки AS65000 на R8:
```
router bgp 65000
 bgp router-id 8.8.8.8
 bgp log-neighbor-changes
 timers bgp 3 9
 neighbor 10.1.0.6 remote-as 65001
 neighbor 10.1.0.8 remote-as 65001
 neighbor 10.1.0.25 remote-as 65002
 neighbor 2A03:0:0:1::1 remote-as 65001
 neighbor 2A03:0:0:1::3 remote-as 65001
 neighbor 2A03:0:0:1::5 remote-as 65002
 !
 address-family ipv4
  network 10.1.0.0 mask 255.255.255.0
  network 172.16.0.8 mask 255.255.255.255
  neighbor 10.1.0.6 activate
  neighbor 10.1.0.8 activate
  neighbor 10.1.0.25 activate
  no neighbor 2A03:0:0:1::1 activate
  no neighbor 2A03:0:0:1::3 activate
  no neighbor 2A03:0:0:1::5 activate
  maximum-paths 2
 exit-address-family
 !
 address-family ipv6
  maximum-paths 2
  network 2A03::8/128
  network 2A03:0:0:1::/64
  neighbor 2A03:0:0:1::1 activate
  neighbor 2A03:0:0:1::3 activate
  neighbor 2A03:0:0:1::5 activate
 exit-address-family
```
Пример настройки AS65001 NXOS1:
```
router bgp 65001
  router-id 1.1.1.1
  timers bgp 3 9
  bestpath as-path multipath-relax
  address-family ipv4 unicast
    network 10.0.1.0/24
    network 172.16.0.1/32
    maximum-paths 2
  address-family ipv6 unicast
    network 2a03:0:0:11::/64
    network 2a03::1/128
	maximum-paths 2
  neighbor 2a03:0:0:1::
    remote-as 65000
    address-family ipv6 unicast
  neighbor 2a03:0:0:11::1
    remote-as 65006
    address-family ipv6 unicast
  neighbor 2a03:0:0:11::3
    remote-as 65004
    address-family ipv6 unicast
  neighbor 2a03:0:0:11::5
    remote-as 65005
    address-family ipv6 unicast
  neighbor 10.0.1.1
    remote-as 65006
    address-family ipv4 unicast
  neighbor 10.0.1.3
    remote-as 65004
    address-family ipv4 unicast
  neighbor 10.0.1.5
    remote-as 65005
    address-family ipv4 unicast
  neighbor 10.1.0.7
    remote-as 65000
    address-family ipv4 unicast
```

Пример настройки AS65004 на NXOS4:
```
router bgp 65004
  router-id 4.4.4.4
  timers bgp 3 9
  address-family ipv4 unicast
    network 172.16.0.4/32
    network 192.168.4.0/24
    maximum-paths 2
  address-family ipv6 unicast
    network 2a03::4/128
    maximum-paths 2
  neighbor 2a03:0:0:11::2
    remote-as 65001
    address-family ipv6 unicast
  neighbor 2a03:0:0:12::2
    remote-as 65001
    address-family ipv6 unicast
  neighbor 10.0.1.2
    remote-as 65001
    address-family ipv4 unicast
  neighbor 10.0.2.14
    remote-as 65001
    address-family ipv4 unicast
```
<a id="chapter-1"></a>
## Проверка роботоспособности

Маршрутная информация на NXOS4:
```
10.0.1.0/24, ubest/mbest: 1/0
    *via 10.0.1.2, [20/0], 00:48:04, bgp-65004, external, tag 65001
10.0.2.0/24, ubest/mbest: 1/0
    *via 10.0.2.14, [20/0], 00:44:31, bgp-65004, external, tag 65001
10.0.3.0/24, ubest/mbest: 2/0
    *via 10.0.1.2, [20/0], 00:56:46, bgp-65004, external, tag 65001
    *via 10.0.2.14, [20/0], 00:56:46, bgp-65004, external, tag 65001
10.1.0.0/24, ubest/mbest: 2/0
    *via 10.0.1.2, [20/0], 00:53:04, bgp-65004, external, tag 65001
    *via 10.0.2.14, [20/0], 00:53:04, bgp-65004, external, tag 65001
172.16.0.1/32, ubest/mbest: 1/0
    *via 10.0.1.2, [20/0], 00:49:43, bgp-65004, external, tag 65001
172.16.0.2/32, ubest/mbest: 1/0
    *via 10.0.2.14, [20/0], 00:46:09, bgp-65004, external, tag 65001
172.16.0.3/32, ubest/mbest: 2/0
    *via 10.0.1.2, [20/0], 00:58:18, bgp-65004, external, tag 65001
    *via 10.0.2.14, [20/0], 00:58:18, bgp-65004, external, tag 65001
172.16.0.5/32, ubest/mbest: 2/0
    *via 10.0.1.2, [20/0], 00:41:41, bgp-65004, external, tag 65001
    *via 10.0.2.14, [20/0], 00:41:41, bgp-65004, external, tag 65001
172.16.0.6/32, ubest/mbest: 2/0
    *via 10.0.1.2, [20/0], 00:43:22, bgp-65004, external, tag 65001
    *via 10.0.2.14, [20/0], 00:43:22, bgp-65004, external, tag 65001
172.16.0.7/32, ubest/mbest: 2/0
    *via 10.0.1.2, [20/0], 01:03:36, bgp-65004, external, tag 65001
    *via 10.0.2.14, [20/0], 01:03:36, bgp-65004, external, tag 65001
172.16.0.8/32, ubest/mbest: 2/0
    *via 10.0.1.2, [20/0], 00:54:23, bgp-65004, external, tag 65001
    *via 10.0.2.14, [20/0], 00:54:22, bgp-65004, external, tag 65001
192.168.5.0/24, ubest/mbest: 2/0
    *via 10.0.1.2, [20/0], 00:40:30, bgp-65004, external, tag 65001
    *via 10.0.2.14, [20/0], 00:40:30, bgp-65004, external, tag 65001
192.168.6.0/24, ubest/mbest: 2/0
    *via 10.0.1.2, [20/0], 00:42:24, bgp-65004, external, tag 65001
    *via 10.0.2.14, [20/0], 00:42:24, bgp-65004, external, tag 65001
192.168.7.0/24, ubest/mbest: 2/0
    *via 10.0.1.2, [20/0], 01:00:35, bgp-65004, external, tag 65001
    *via 10.0.2.14, [20/0], 01:00:35, bgp-65004, external, tag 65001


2a03::1/128, ubest/mbest: 1/0
    *via 2a03:0:0:11::2, Eth1/1, [20/0], 00:49:26, bgp-65004, external, tag 65001
2a03::2/128, ubest/mbest: 1/0
    *via 2a03:0:0:12::2, Eth1/2, [20/0], 00:45:52, bgp-65004, external, tag 65001
2a03::3/128, ubest/mbest: 2/0
    *via 2a03:0:0:11::2, Eth1/1, [20/0], 00:58:21, bgp-65004, external, tag 65001
    *via 2a03:0:0:12::2, Eth1/2, [20/0], 00:58:21, bgp-65004, external, tag 65001
2a03::5/128, ubest/mbest: 2/0
    *via 2a03:0:0:11::2, Eth1/1, [20/0], 00:41:21, bgp-65004, external, tag 65001
    *via 2a03:0:0:12::2, Eth1/2, [20/0], 00:41:21, bgp-65004, external, tag 65001
2a03::6/128, ubest/mbest: 2/0
    *via 2a03:0:0:11::2, Eth1/1, [20/0], 00:43:04, bgp-65004, external, tag 65001
    *via 2a03:0:0:12::2, Eth1/2, [20/0], 00:43:04, bgp-65004, external, tag 65001
2a03::7/128, ubest/mbest: 2/0
    *via 2a03:0:0:11::2, Eth1/1, [20/0], 01:02:29, bgp-65004, external, tag 65001
    *via 2a03:0:0:12::2, Eth1/2, [20/0], 01:02:29, bgp-65004, external, tag 65001
2a03::8/128, ubest/mbest: 2/0
    *via 2a03:0:0:11::2, Eth1/1, [20/0], 00:53:46, bgp-65004, external, tag 65001
    *via 2a03:0:0:12::2, Eth1/2, [20/0], 00:53:46, bgp-65004, external, tag 65001
2a03:0:0:1::/64, ubest/mbest: 2/0
    *via 2a03:0:0:11::2, Eth1/1, [20/0], 00:53:10, bgp-65004, external, tag 65001
    *via 2a03:0:0:12::2, Eth1/2, [20/0], 00:53:10, bgp-65004, external, tag 65001
2a03:0:0:11::/64, ubest/mbest: 1/0
    *via 2a03:0:0:11::2, Eth1/1, [20/0], 00:48:13, bgp-65004, external, tag 65001
2a03:0:0:12::/64, ubest/mbest: 1/0
    *via 2a03:0:0:12::2, Eth1/2, [20/0], 00:44:34, bgp-65004, external, tag 65001
2a03:0:0:23::/64, ubest/mbest: 2/0
    *via 2a03:0:0:11::2, Eth1/1, [20/0], 00:56:52, bgp-65004, external, tag 65001
    *via 2a03:0:0:12::2, Eth1/2, [20/0], 00:56:52, bgp-65004, external, tag 65001
```

Маршрутная информация на R8:
```
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 10 subnets, 3 masks
B        10.0.1.0/24 [20/0] via 10.1.0.6, 00:49:24
B        10.0.2.0/24 [20/0] via 10.1.0.8, 00:45:51
B        10.0.3.0/24 [20/0] via 10.1.0.25, 00:58:07
      172.16.0.0/32 is subnetted, 8 subnets
B        172.16.0.1 [20/0] via 10.1.0.6, 00:51:04
B        172.16.0.2 [20/0] via 10.1.0.8, 00:47:30
B        172.16.0.3 [20/0] via 10.1.0.25, 00:59:39
B        172.16.0.4 [20/0] via 10.1.0.8, 00:41:11
                    [20/0] via 10.1.0.6, 00:41:11
B        172.16.0.5 [20/0] via 10.1.0.8, 00:43:02
                    [20/0] via 10.1.0.6, 00:43:02
B        172.16.0.6 [20/0] via 10.1.0.8, 00:44:42
                    [20/0] via 10.1.0.6, 00:44:42
B        172.16.0.7 [20/0] via 10.1.0.25, 01:04:57
B     192.168.4.0/24 [20/0] via 10.1.0.8, 00:40:23
                     [20/0] via 10.1.0.6, 00:40:23
B     192.168.5.0/24 [20/0] via 10.1.0.8, 00:41:51
                     [20/0] via 10.1.0.6, 00:41:51
B     192.168.6.0/24 [20/0] via 10.1.0.8, 00:43:44
                     [20/0] via 10.1.0.6, 00:43:44
B     192.168.7.0/24 [20/0] via 10.1.0.25, 01:01:56


B   2A03::1/128 [20/0]
     via FE80::1, Ethernet0/0
B   2A03::2/128 [20/0]
     via FE80::2, Ethernet0/1
B   2A03::3/128 [20/0]
     via FE80::3, Ethernet0/2
B   2A03::4/128 [20/0]
     via FE80::1, Ethernet0/0
     via FE80::2, Ethernet0/1
B   2A03::5/128 [20/0]
     via FE80::1, Ethernet0/0
     via FE80::2, Ethernet0/1
B   2A03::6/128 [20/0]
     via FE80::1, Ethernet0/0
     via FE80::2, Ethernet0/1
B   2A03::7/128 [20/0]
     via FE80::3, Ethernet0/2
B   2A03:0:0:11::/64 [20/0]
     via FE80::1, Ethernet0/0
B   2A03:0:0:12::/64 [20/0]
     via FE80::2, Ethernet0/1
B   2A03:0:0:23::/64 [20/0]
     via FE80::3, Ethernet0/2
```

Проверка сввязанности между NXOS4 и NXOS7:
```
NXOS4# ping 172.16.0.7
PING 172.16.0.7 (172.16.0.7): 56 data bytes
64 bytes from 172.16.0.7: icmp_seq=0 ttl=251 time=332.118 ms
64 bytes from 172.16.0.7: icmp_seq=1 ttl=251 time=115.518 ms
64 bytes from 172.16.0.7: icmp_seq=2 ttl=251 time=105.792 ms
64 bytes from 172.16.0.7: icmp_seq=3 ttl=251 time=76.097 ms
64 bytes from 172.16.0.7: icmp_seq=4 ttl=251 time=55.737 ms

--- 172.16.0.7 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 55.737/137.052/332.118 ms



NXOS7# traceroute6  2A03::4
traceroute to 2a03::4 (2a03::4), 30 hops max, 16 byte packets
 1  2a03:0:0:23::  112.675 ms  20.064 ms  9.587 ms
 2  2a03:0:0:1::4  30.095 ms  23.414 ms  40.968 ms
 3  2a03:0:0:1::1  65.531 ms 2a03:0:0:1::3  101.018 ms 2a03:0:0:1::1  21.967 ms
 4  2a03::4  211.225 ms  62.149 ms  64.456 ms
```

