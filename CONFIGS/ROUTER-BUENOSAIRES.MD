BS.AS#show running-config 
Building configuration...

Current configuration : 3126 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname BS.AS
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2911/K9 sn FTX15249GL7-
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
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/1.30
 description LAN_BSAS_VLAN30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip nat inside
!
interface GigabitEthernet0/1.100
 description WAN1_VLAN100_to_ISP_LOCAL
 encapsulation dot1Q 100
 ip address 42.25.25.1 255.255.255.248
 ip nat outside
!
interface GigabitEthernet0/1.200
 description WAN2_VLAN200_to_ISP_LOCAL
 encapsulation dot1Q 200
 ip address 43.26.26.1 255.255.255.248
 ip nat outside
!
interface GigabitEthernet0/2
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/2.1000
 description BACKUP_OSPF_VLAN1000
 encapsulation dot1Q 1000
 ip address 172.20.10.1 255.255.255.248
 ip ospf cost 50
 ip ospf priority 100
!
interface GigabitEthernet0/0/0
 description P2P_to_CORDOBA
 ip address 10.10.1.9 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
!
interface GigabitEthernet0/1/0
 description P2P_to_MENDOZA
 ip address 10.10.1.1 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
!
interface Vlan1
 no ip address
 shutdown
!
router ospf 1
 router-id 1.1.1.1
 log-adjacency-changes
 passive-interface GigabitEthernet0/1.30
 network 192.168.30.0 0.0.0.255 area 0
 network 10.10.1.8 0.0.0.3 area 0
 network 10.10.1.0 0.0.0.3 area 0
 network 172.20.10.0 0.0.0.7 area 0
 default-information originate
!
ip nat inside source list 10 interface GigabitEthernet0/1.100 overload
ip nat inside source list 11 interface GigabitEthernet0/1.200 overload
ip classless
ip route 192.168.100.0 255.255.255.248 42.25.25.2 
ip route 0.0.0.0 0.0.0.0 42.25.25.2 
ip route 192.168.100.8 255.255.255.248 42.25.25.2 
ip route 0.0.0.0 0.0.0.0 43.26.26.2 5
ip route 192.168.100.0 255.255.255.248 43.26.26.2 5
ip route 192.168.100.8 255.255.255.248 43.26.26.2 5
!
ip flow-export version 9
!
!
ip access-list standard NAT-INSIDE
 permit 192.168.30.0 0.0.0.255
 permit 192.168.10.0 0.0.0.255
 permit 192.168.20.0 0.0.0.255
 permit 192.168.44.0 0.0.0.255
 permit 192.168.55.0 0.0.0.255
 permit 192.168.70.0 0.0.0.255
access-list 10 permit 192.168.30.0 0.0.0.255
access-list 10 permit 192.168.10.0 0.0.0.255
access-list 10 permit 192.168.20.0 0.0.0.255
access-list 10 permit 192.168.44.0 0.0.0.255
access-list 10 permit 192.168.55.0 0.0.0.255
access-list 10 permit 192.168.70.0 0.0.0.255
access-list 11 permit 192.168.30.0 0.0.0.255
access-list 11 permit 192.168.10.0 0.0.0.255
access-list 11 permit 192.168.20.0 0.0.0.255
access-list 11 permit 192.168.44.0 0.0.0.255
access-list 11 permit 192.168.55.0 0.0.0.255
access-list 11 permit 192.168.70.0 0.0.0.255
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


BS.AS# 