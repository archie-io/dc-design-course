## VxLAN. Аналоги VPC

### Задача:

- Настроить VPC или аналог для работы в Overlay сети
- Подключить клиента двумя линками к разным LEAF
- Настроить агрегированный канал между клиентом и парой LEAF-коммутаторов
- Проверить связанность между клиентами

## Выполнение:

### Схема сети

![](images/1-14.drawio.png)

| HOSTNAME   | LOCAL INTF             | LOCAL IP          | LOCAL AS    | REMOTE AS   | REMOTE IP         | REMOTE INTF            | NEIGHBOR         |
| ---------- | ---------------------- | ----------------- | ----------- | ----------- | ----------------- | ---------------------- | ---------------- |
| SPINE-1    | Ethernet1              | 10.1.1.10/31      | 65000       | 65001       | 10.1.1.11/31      | Ethernet1              | LEAF-1           |
|            | Ethernet2              | 10.1.2.10/31      | 65000       | 65002       | 10.1.2.11/31      | Ethernet1              | LEAF-2           |
|            | Ethernet3              | 10.1.3.10/31      | 65000       | 65003       | 10.1.3.11/31      | Ethernet1              | LEAF-3-1         |
|            | Ethernet4              | 10.1.3.20/31      | 65000       | 65003       | 10.1.3.21/31      | Ethernet1              | LEAF-3-2         |
|            | Loopback1              | 10.0.0.1/32       |             |             |                   |                        |                  |
|            |                        |                   |             |             |                   |                        |                  |
| SPINE-2    | Ethernet1              | 10.2.1.10/31      | 65000       | 65001       | 10.2.1.11/31      | Ethernet2              | LEAF-1           |
|            | Ethernet2              | 10.2.2.10/31      | 65000       | 65002       | 10.2.2.11/31      | Ethernet2              | LEAF-2           |
|            | Ethernet3              | 10.2.3.10/31      | 65000       | 65003       | 10.2.3.11/31      | Ethernet2              | LEAF-3-1         |
|            | Ethernet4              | 10.2.3.20/31      | 65000       | 65003       | 10.2.3.21/31      | Ethernet2              | LEAF-3-2         |
|            | Loopback1              | 10.0.0.2/32       |             |             |                   |                        |                  |
|            |                        |                   |             |             |                   |                        |                  |
| LEAF-1     | Ethernet1              | 10.1.1.11/31      | 65001       | 65000       | 10.1.1.10/31      | Ethernet1              | SPINE-1          |
|            | Ethernet2              | 10.2.1.11/31      | 65001       | 65000       | 10.2.1.10/31      | Ethernet1              | SPINE-2          |
|            | Loopback1              | 10.0.1.1/32       |             |             |                   |                        |                  |
|            | Vxlan1                 | Loopback1         |             |             |                   |                        |                  |
|            | Vlan11                 | **Virtual IP**    |             |             | 192.168.11.11/24  | Ethernet0              | NODE-11-11       |
|            |                        |                   |             |             |                   |                        |                  |
| LEAF-2     | Ethernet1              | 10.1.2.11/31      | 65002       | 65000       | 10.1.2.10/31      | Ethernet2              | SPINE-1          |
|            | Ethernet2              | 10.2.2.11/31      | 65002       | 65000       | 10.2.2.10/31      | Ethernet2              | SPINE-2          |
|            | Loopback1              | 10.0.2.1/32       |             |             |                   |                        |                  |
|            | Vxlan1                 | Loopback1         |             |             |                   |                        |                  |
|            | Vlan11                 | **Virtual IP**    |             |             | 192.168.11.12/24  | Ethernet0              | NODE-11-12       |
|            | Vlan12                 | **Virtual IP**    |             |             | 192.168.12.11/24  | Ethernet0              | NODE-12-11       |
|            |                        |                   |             |             |                   |                        |                  |
| LEAF-3-1   | Ethernet1              | 10.1.3.11/31      | 65003       | 65000       | 10.1.3.10/31      | Ethernet3              | SPINE-1          |
|            | Ethernet2              | 10.2.3.11/31      | 65003       | 65000       | 10.2.3.10/31      | Ethernet3              | SPINE-2          |
|            | ***Vlan4094***         | 10.255.255.1/30   | PeerLink-IP | PeerLink-IP | 10.255.255.2/30   | ***Vlan4094***         | LEAF-3-2         |
|            | ***Vlan4090***         | 10.255.255.201/30 | iBGP        | iBGP        | 10.255.255.202/30 | ***Vlan4090***         | LEAF-3-2         |
|            | ***Port-Channel4094*** |                   | PeerLink    | PeerLink    |                   | ***Port-Channel4094*** | LEAF-3-2         |
|            | Ethernet5              | 10.255.255.101/30 | HeartBeat   | HeartBeat   | 10.255.255.102/30 | Ethernet5              | LEAF-3-2         |
|            | Loopback1              | 10.0.3.1/32       |             |             |                   |                        |                  |
|            | Loopback10             | 10.0.3.10/32      |             |             |                   |                        |                  |
|            | Vxlan1                 | Loopback10        |             |             |                   |                        |                  |
|            | ***Vlan12***           | **Virtual IP**    |             |             | 192.168.12.12/24  | ***Vlan12***           | NODE-12-12       |
|            | ***Port-Channel12***   |                   |             |             |                   | ***Port-Channel12***   | NODE-12-12       |
|            | ***Ethernet12***       |                   |             |             |                   | ***ether1***           | NODE-12-12       |
|            |                        |                   |             |             |                   |                        |                  |
| LEAF-3-2   | Ethernet1              | 10.1.3.21/31      | 65003       | 65000       | 10.1.3.20/31      | Ethernet4              | SPINE-1          |
|            | Ethernet2              | 10.2.3.21/31      | 65003       | 65000       | 10.2.3.20/31      | Ethernet4              | SPINE-2          |
|            | ***Vlan4094***         | 10.255.255.2/30   | PeerLink-IP | PeerLink-IP | 10.255.255.1/30   | ***Vlan4094***         | LEAF-3-1         |
|            | ***Vlan4090***         | 10.255.255.202/30 | iBGP        | iBGP        | 10.255.255.201/30 | ***Vlan4090***         | LEAF-3-1         |
|            | ***Port-Channel4094*** |                   | PeerLink    | PeerLink    |                   | ***Port-Channel4094*** | LEAF-3-1         |
|            | Ethernet5              | 10.255.255.102/30 | HeartBeat   | HeartBeat   | 10.255.255.101/30 | Ethernet5              | LEAF-3-1         |
|            | Loopback1              | 10.0.3.2/32       |             |             |                   |                        |                  |
|            | Loopback10             | 10.0.3.10/32      |             |             |                   |                        |                  |
|            | Vxlan1                 | Loopback10        |             |             |                   |                        |                  |
|            | ***Vlan12***           | **Virtual IP**    |             |             | 192.168.12.12/24  | ***Vlan12***           | NODE-12-12       |
|            | ***Port-Channel12***   |                   |             |             |                   | ***Port-Channel12***   | NODE-12-12       |
|            | ***Ethernet12***       |                   |             |             |                   | ***ether2***           | NODE-12-12       |
|            |                        |                   |             |             |                   |                        |                  |
| NODE-11-11 | Ethernet0              | 192.168.11.11/24  |             |             | **Virtual IP**    | Vlan11                 | LEAF-1           |
|            |                        |                   |             |             |                   |                        |                  |
| NODE-11-12 | Ethernet0              | 192.168.11.12/24  |             |             | **Virtual IP**    | Vlan11                 | LEAF-2           |
|            |                        |                   |             |             |                   |                        |                  |
| NODE-12-11 | Ethernet0              | 192.168.12.11/24  |             |             | **Virtual IP**    | Vlan12                 | LEAF-2           |
|            |                        |                   |             |             |                   |                        |                  |
| NODE-12-12 | ***Vlan12***           | 192.168.12.12/24  |             |             | **Virtual IP**    | ***Vlan12***           | LEAF-3-1/LEAF3-2 |
|            | ***Port-Channel12***   |                   |             |             |                   | ***Port-Channel12***   | LEAF-3-1/LEAF3-2 |
|            | ***ether1***           |                   |             |             |                   | ***Ethernet12***       | LEAF-3-1         |
|            | ***ether2***           |                   |             |             |                   | ***Ethernet12***       | LEAF-3-2         |


|            | **Virtual IP**    |
| ---------- | ----------------- |
| **Vlan11** | 192.168.11.254/24 |
| **Vlan12** | 192.168.11.254/24 |

### Конфигурация оборудования
- #### [LEAF-1](config/LEAF-1.cfg)
- #### [LEAF-2](config/LEAF-2.cfg)

- #### [LEAF-3-1](config/LEAF-3-1.cfg)
```
service routing protocols model multi-agent
!
hostname LEAF-3-1
!
spanning-tree mode rstp
no spanning-tree vlan-id 4090,4094
!
vlan 12
!
vlan 4090,4094
   trunk group MLAGPEER
!
vrf instance HBMLAG
!
vrf instance VRF_COMMON
!
interface Port-Channel12
   switchport trunk allowed vlan 12
   switchport mode trunk
   mlag 12
   spanning-tree portfast
!
interface Port-Channel4094
   switchport mode trunk
   switchport trunk group MLAGPEER
!
interface Ethernet1
   description TO-SPINE-1
   no switchport
   ip address 10.1.3.11/31
   bfd interval 100 min-rx 100 multiplier 3
   bfd echo
!
interface Ethernet2
   description TO-SPINE-2
   no switchport
   ip address 10.2.3.11/31
   bfd interval 100 min-rx 100 multiplier 3
   bfd echo
!
interface Ethernet3
   channel-group 4094 mode active
!
interface Ethernet4
   channel-group 4094 mode active
!
interface Ethernet5
   no switchport
   vrf HBMLAG
   ip address 10.255.255.101/30
!
interface Ethernet12
   switchport access vlan 12
   channel-group 12 mode active
   lacp timer fast
   spanning-tree portfast
!
interface Loopback1
   ip address 10.0.3.1/32
!
interface Loopback10
   ip address 10.0.3.10/32
!
interface Vlan12
   vrf VRF_COMMON
   ip address virtual 192.168.12.254/24
!
interface Vlan4090
   no autostate
   ip address 10.255.255.201/30
!
interface Vlan4094
   no autostate
   ip address 10.255.255.1/30
!
interface Vxlan1
   vxlan source-interface Loopback10
   vxlan virtual-router encapsulation mac-address mlag-system-id
   vxlan udp-port 4789
   vxlan vlan 12 vni 10012
   vxlan vrf VRF_COMMON vni 1000
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing vrf HBMLAG
ip routing vrf VRF_COMMON
!
mlag configuration
   domain-id MLAG_DOMAIN
   heartbeat-interval 1000
   local-interface Vlan4094
   peer-address 10.255.255.2
   peer-address heartbeat 10.255.255.102 vrf HBMLAG
   peer-link Port-Channel4094
   dual-primary detection delay 10 action errdisable all-interfaces
   reload-delay 180
!
route-map RM_REDIST permit 10
   match interface Loopback1
!
route-map RM_REDIST permit 20
   match interface Loopback10
!
route-map next-hop-self-ipv4 permit 10
   match route-type external
   set ip next-hop peer-address
!
route-map next-hop-self-ipv4 permit 20
!
router bgp 65003
   timers bgp 3 9
   distance bgp 20 200 200
   maximum-paths 2 ecmp 2
   neighbor MLAGPEER peer group
   neighbor MLAGPEER remote-as 65003
   neighbor MLAGPEER route-map next-hop-self-ipv4 out
   neighbor MLAGPEER send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE allowas-in 1
   neighbor SPINE rib-in pre-policy retain all
   neighbor SPINE password 7 yVt/SxQ2cHkCSxO5ZSaL8G2JpeqOLEZj
   neighbor SPINE send-community extended
   neighbor 10.1.3.10 peer group SPINE
   neighbor 10.2.3.10 peer group SPINE
   neighbor 10.255.255.202 peer group MLAGPEER
   !
   vlan 12
      rd 10.0.3.1:10012
      route-target both 192.168.12.0:10012
      redistribute learned
   !
   address-family evpn
      neighbor MLAGPEER activate
      neighbor SPINE activate
   !
   address-family ipv4
      neighbor MLAGPEER activate
      neighbor SPINE activate
      redistribute connected route-map RM_REDIST
   !
   vrf VRF_COMMON
      rd 10.0.3.1:1000
      route-target import evpn 1000:1000
      route-target export evpn 1000:1000
      redistribute connected
!
router general
   router-id ipv4 10.0.3.1
!
```

- #### [LEAF-3-2](config/LEAF-3-2.cfg)
```
service routing protocols model multi-agent
!
hostname LEAF-3-2
!
spanning-tree mode rstp
no spanning-tree vlan-id 4090,4094
!
vlan 12
!
vlan 4090,4094
   trunk group MLAGPEER
!
vrf instance HBMLAG
!
vrf instance VRF_COMMON
!
interface Port-Channel12
   switchport trunk allowed vlan 12
   switchport mode trunk
   mlag 12
   spanning-tree portfast
!
interface Port-Channel4094
   switchport mode trunk
   switchport trunk group MLAGPEER
!
interface Ethernet1
   description TO-SPINE-1
   no switchport
   ip address 10.1.3.21/31
   bfd interval 100 min-rx 100 multiplier 3
   bfd echo
!
interface Ethernet2
   description TO-SPINE-2
   no switchport
   ip address 10.2.3.21/31
   bfd interval 100 min-rx 100 multiplier 3
   bfd echo
!
interface Ethernet3
   channel-group 4094 mode active
!
interface Ethernet4
   channel-group 4094 mode active
!
interface Ethernet5
   no switchport
   vrf HBMLAG
   ip address 10.255.255.102/30
!
interface Ethernet12
   switchport access vlan 12
   channel-group 12 mode active
   lacp timer fast
   spanning-tree portfast
!
interface Loopback1
   ip address 10.0.3.2/32
!
interface Loopback10
   ip address 10.0.3.10/32
!
interface Vlan12
   vrf VRF_COMMON
   ip address virtual 192.168.12.254/24
!
interface Vlan4090
   no autostate
   ip address 10.255.255.202/30
!
interface Vlan4094
   no autostate
   ip address 10.255.255.2/30
!
interface Vxlan1
   vxlan source-interface Loopback10
   vxlan virtual-router encapsulation mac-address mlag-system-id
   vxlan udp-port 4789
   vxlan vlan 12 vni 10012
   vxlan vrf VRF_COMMON vni 1000
   vxlan learn-restrict any
!
ip virtual-router mac-address 00:00:00:00:00:01
!
ip routing vrf HBMLAG
ip routing vrf VRF_COMMON
!
mlag configuration
   domain-id MLAG_DOMAIN
   heartbeat-interval 1000
   local-interface Vlan4094
   peer-address 10.255.255.1
   peer-address heartbeat 10.255.255.101 vrf HBMLAG
   peer-link Port-Channel4094
   dual-primary detection delay 10 action errdisable all-interfaces
   reload-delay 180
!
route-map RM_REDIST permit 10
   match interface Loopback1
!
route-map RM_REDIST permit 20
   match interface Loopback10
!
route-map next-hop-self-ipv4 permit 10
   match route-type external
   set ip next-hop peer-address
!
route-map next-hop-self-ipv4 permit 20
!
router bgp 65003
   timers bgp 3 9
   maximum-paths 2 ecmp 2
   neighbor MLAGPEER peer group
   neighbor MLAGPEER remote-as 65003
   neighbor MLAGPEER route-map next-hop-self-ipv4 out
   neighbor MLAGPEER send-community extended
   neighbor SPINE peer group
   neighbor SPINE remote-as 65000
   neighbor SPINE bfd
   neighbor SPINE allowas-in 1
   neighbor SPINE rib-in pre-policy retain all
   neighbor SPINE password 7 yVt/SxQ2cHkCSxO5ZSaL8G2JpeqOLEZj
   neighbor SPINE send-community extended
   neighbor 10.1.3.20 peer group SPINE
   neighbor 10.2.3.20 peer group SPINE
   neighbor 10.255.255.201 peer group MLAGPEER
   !
   vlan 12
      rd 10.0.3.2:10012
      route-target both 192.168.12.0:10012
      redistribute learned
   !
   address-family evpn
      neighbor MLAGPEER activate
      neighbor SPINE activate
   !
   address-family ipv4
      neighbor MLAGPEER activate
      neighbor SPINE activate
      redistribute connected route-map RM_REDIST
   !
   vrf VRF_COMMON
      rd 10.0.3.2:1000
      route-target import evpn 1000:1000
      route-target export evpn 1000:1000
      redistribute connected
!
router general
   router-id ipv4 10.0.3.2
!
```

- #### [NODE-12-12](config/NODE-12-12.cfg)
```
[admin@NODE-12-12] > /interface/bonding/print detail
Flags: X - disabled; R - running
 0  R name="Port-Channel12" mtu=1500 mac-address=0C:D5:56:0F:00:00 arp=enabled
      arp-timeout=auto slaves=ether1,ether2 mode=802.3ad primary=none
      link-monitoring=mii arp-interval=100ms arp-ip-targets="" mii-interval=100ms
      down-delay=0ms up-delay=0ms lacp-rate=1sec transmit-hash-policy=layer-2
      min-links=0

[admin@NODE-12-12] > /interface/vlan/print detail
Flags: X - disabled, R - running
 0 R name="Vlan12" mtu=1496 l2mtu=1496 mac-address=0C:D5:56:0F:00:00 arp=enabled
     arp-timeout=auto loop-protect=default loop-protect-status=off
     loop-protect-send-interval=5s loop-protect-disable-time=5m vlan-id=12
     interface=Port-Channel12 use-service-tag=no mvrp=no

[admin@NODE-12-12] > /ip/address/print detail
Flags: X - disabled, I - invalid; D - dynamic; S - slave
 0     address=192.168.12.12/24 network=192.168.12.0 interface=Vlan12
       actual-interface=Vlan12

[admin@NODE-12-12] > /ip/route/print detail
Flags: D - dynamic; X - disabled, I - inactive, A - active;
c - connect, s - static, r - rip, b - bgp, o - ospf, i - is-is, d - dhcp, v - vpn, m - >
H - hw-offloaded; + - ecmp
 0  As   dst-address=0.0.0.0/0 routing-table=main gateway=192.168.12.254
         immediate-gw=192.168.12.254%Vlan12 distance=1 scope=30 target-scope=10

   DAc   dst-address=192.168.12.0/24 routing-table=main gateway=Vlan12
         immediate-gw=Vlan12 distance=0 scope=10 target-scope=5
         local-address=192.168.12.12%Vlan12
```
---

### Проверка конфигурации LEAF

- #### LEAF-1
```
```

- #### LEAF-3-1
```
```

- #### LEAF-3-2
```
```

- #### NODE-12-12
```
```

### Проверка связности клиентов

- #### NODE-11-11
```
```

- #### NODE-12-12
```
```