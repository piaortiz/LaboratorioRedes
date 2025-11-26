SW-ACC-MEND#
SW-ACC-MEND#
SW-ACC-MEND#
SW-ACC-MEND#
SW-ACC-MEND#show ru
SW-ACC-MEND#show running-config 
Building configuration...

Current configuration : 1727 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname SW-ACC-MEND
!
!
!
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
spanning-tree vlan 44,55,70 priority 49152
!
interface FastEthernet0/1
 description PC-ADMIN-MENDOZA
 switchport access vlan 70
 switchport mode access
 spanning-tree portfast
!
interface FastEthernet0/2
!
interface FastEthernet0/3
!
interface FastEthernet0/4
!
interface FastEthernet0/5
!
interface FastEthernet0/6
!
interface FastEthernet0/7
!
interface FastEthernet0/8
!
interface FastEthernet0/9
!
interface FastEthernet0/10
!
interface FastEthernet0/11
!
interface FastEthernet0/12
!
interface FastEthernet0/13
!
interface FastEthernet0/14
!
interface FastEthernet0/15
!
interface FastEthernet0/16
!
interface FastEthernet0/17
!
interface FastEthernet0/18
!
interface FastEthernet0/19
!
interface FastEthernet0/20
!
interface FastEthernet0/21
!
interface FastEthernet0/22
 description DHCP-SERVER-MENDOZA
 switchport access vlan 70
 switchport mode access
 spanning-tree portfast
!
interface FastEthernet0/23
 description RADIUS-SERVER-MENDOZA
 switchport access vlan 70
 switchport mode access
 spanning-tree portfast
!
interface FastEthernet0/24
 description TRUNK_to_AP-01-MENDOZA_Gi0
 switchport trunk native vlan 70
 switchport trunk allowed vlan 44,55,70
 switchport mode trunk
!
interface GigabitEthernet0/1
 description TRUNK_to_SW-CORE-DIS-MEND_G0/1
 switchport trunk native vlan 70
 switchport trunk allowed vlan 44,55,70
 switchport mode trunk
!
interface GigabitEthernet0/2
!
interface Vlan1
 no ip address
 shutdown
!
!
!
!
line con 0
!
line vty 0 4
 login
line vty 5 15
 login
!
!
!
!
end


SW-ACC-MEND# 