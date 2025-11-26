SW-BS.AS#show running-config 
Building configuration...

Current configuration : 1459 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname SW-BS.AS
!
!
!
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
!
interface FastEthernet0/2
 description PC_BS_AS
 switchport access vlan 30
 switchport mode access
 spanning-tree portfast
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
!
interface FastEthernet0/23
!
interface FastEthernet0/24
 description WAN2_to_ISP_LOCAL_G0/2
 switchport access vlan 200
 switchport mode access
!
interface GigabitEthernet0/1
 description WAN1_to_ISP_LOCAL_G0/1
 switchport access vlan 100
 switchport mode access
!
interface GigabitEthernet0/2
 description Trunk_to_Router_BSAS_G0/1
 switchport trunk allowed vlan 30,100,200
 switchport mode trunk
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


SW-BS.AS#  