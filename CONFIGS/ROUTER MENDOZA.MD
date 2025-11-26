MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#
MENDOZA#SHOW RU
MENDOZA#SHOW RUnning-config 
Building configuration...

Current configuration : 2183 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname MENDOZA
!
!
!
!
!
!
!
!
no ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2911/K9 sn FTX1524GCOK-
!
!
!
!
!
!
!
!
!
!
!
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface GigabitEthernet0/1
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/1.1000
 description BACKUP_OSPF_VLAN1000
 encapsulation dot1Q 1000
 ip address 172.20.10.3 255.255.255.248
 ip ospf cost 50
 ip ospf priority 50
!
interface GigabitEthernet0/2
 description Trunk_LAN_MENDOZA_to_SW-CORE-DIS-MEND
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/2.44
 description WiFi_INTERNOS_VLAN44
 encapsulation dot1Q 44
 ip address 192.168.44.1 255.255.255.0
 ip helper-address 192.168.70.10
!
interface GigabitEthernet0/2.55
 description WiFi_INVITADOS_VLAN55
 encapsulation dot1Q 55
 ip address 192.168.55.1 255.255.255.0
 ip helper-address 192.168.70.10
!
interface GigabitEthernet0/2.70
 description MANAGEMENT_VLAN70_NATIVE
 encapsulation dot1Q 70 native
 ip address 192.168.70.1 255.255.255.0
!
interface GigabitEthernet0/0/0
 description P2P_to_BSAS
 ip address 10.10.1.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
!
interface GigabitEthernet0/1/0
 description P2P_to_CORDOBA
 ip address 10.10.1.18 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
!
interface Vlan1
 no ip address
 shutdown
!
router ospf 1
 router-id 3.3.3.3
 log-adjacency-changes
 passive-interface GigabitEthernet0/2.44
 passive-interface GigabitEthernet0/2.55
 passive-interface GigabitEthernet0/2.70
 network 192.168.44.0 0.0.0.255 area 0
 network 192.168.55.0 0.0.0.255 area 0
 network 192.168.70.0 0.0.0.255 area 0
 network 10.10.1.0 0.0.0.3 area 0
 network 10.10.1.16 0.0.0.3 area 0
 network 172.20.10.0 0.0.0.7 area 0
!
ip classless
ip route 0.0.0.0 0.0.0.0 10.10.1.1 
!
ip flow-export version 9
!
!
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end


MENDOZA# 
MENDOZA#