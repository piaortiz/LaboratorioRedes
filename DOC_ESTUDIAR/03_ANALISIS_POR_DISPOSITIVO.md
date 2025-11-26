# 🔧 ANÁLISIS DETALLADO POR DISPOSITIVO

Este documento analiza la configuración de cada dispositivo de la red.

---

## 📖 Índice
1. [Router Buenos Aires](#router-bsas)
2. [Router Córdoba](#router-cordoba)
3. [Router Mendoza](#router-mendoza)
4. [ISP Local](#isp-local)
5. [ISP Internacional](#isp-internacional)
6. [Switches de Distribución](#switches-distribucion)
7. [Switches de Acceso](#switches-acceso)
8. [Switch OSPF Backup](#switch-backup)

---

## 🔷 1. Router Buenos Aires {#router-bsas}

### **Rol en la Red**
- **Gateway principal** hacia Internet
- **Punto de salida NAT** para toda la red
- **Inyector de ruta por defecto** en OSPF
- **Hub central** de la topología OSPF

### **Interfaces Configuradas**

#### **GigabitEthernet0/1 (Trunk a SW-BUENOSAIRES)**

**Subinterfaz 0/1.30 - LAN Buenos Aires**
```
interface GigabitEthernet0/1.30
 description LAN_BSAS_VLAN30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip nat inside
```
- **VLAN:** 30
- **Red:** 192.168.30.0/24
- **Gateway:** .1
- **NAT:** Inside (tráfico a traducir)
- **OSPF:** Passive interface (no envía Hellos)

**Subinterfaz 0/1.100 - WAN1 a ISP Local**
```
interface GigabitEthernet0/1.100
 description WAN1_VLAN100_to_ISP_LOCAL
 encapsulation dot1Q 100
 ip address 42.25.25.1 255.255.255.248
 ip nat outside
```
- **VLAN:** 100
- **Red:** 42.25.25.0/29
- **IP:** .1 (ISP Local: .2)
- **NAT:** Outside (interfaz de salida primaria)

**Subinterfaz 0/1.200 - WAN2 a ISP Local**
```
interface GigabitEthernet0/1.200
 description WAN2_VLAN200_to_ISP_LOCAL
 encapsulation dot1Q 200
 ip address 43.26.26.1 255.255.255.248
 ip nat outside
```
- **VLAN:** 200
- **Red:** 43.26.26.0/29
- **IP:** .1 (ISP Local: .2)
- **NAT:** Outside (interfaz de salida backup)

#### **GigabitEthernet0/2 (Trunk a SW_OSPF_BACKUP)**

**Subinterfaz 0/2.1000 - OSPF Backup**
```
interface GigabitEthernet0/2.1000
 description BACKUP_OSPF_VLAN1000
 encapsulation dot1Q 1000
 ip address 172.20.10.1 255.255.255.248
 ip ospf cost 50
 ip ospf priority 100
```
- **VLAN:** 1000
- **Red:** 172.20.10.0/29
- **Costo OSPF:** 50 (alto, para ser ruta de backup)
- **Prioridad OSPF:** 100 (será DR - Designated Router)

#### **GigabitEthernet0/0/0 - P2P a Córdoba**
```
interface GigabitEthernet0/0/0
 description P2P_to_CORDOBA
 ip address 10.10.1.9 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
```
- **Red:** 10.10.1.8/30
- **IP:** .9 (Córdoba: .10)
- **Tipo OSPF:** Point-to-Point (no requiere DR/BDR)
- **Costo:** 10 (ruta preferida)

#### **GigabitEthernet0/1/0 - P2P a Mendoza**
```
interface GigabitEthernet0/1/0
 description P2P_to_MENDOZA
 ip address 10.10.1.1 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
```
- **Red:** 10.10.1.0/30
- **IP:** .1 (Mendoza: .2)
- **Tipo OSPF:** Point-to-Point
- **Costo:** 10 (ruta preferida)

### **Configuración OSPF**

```
router ospf 1
 router-id 1.1.1.1
 log-adjacency-changes
 passive-interface GigabitEthernet0/1.30
 network 192.168.30.0 0.0.0.255 area 0
 network 10.10.1.8 0.0.0.3 area 0
 network 10.10.1.0 0.0.0.3 area 0
 network 172.20.10.0 0.0.0.7 area 0
 default-information originate
```

**Análisis:**
- **Router ID:** 1.1.1.1 (identificador único)
- **Passive Interface:** VLAN 30 (no enviar OSPF a usuarios)
- **Redes anunciadas:**
  - 192.168.30.0/24 (LAN Buenos Aires)
  - 10.10.1.8/30 (P2P a Córdoba)
  - 10.10.1.0/30 (P2P a Mendoza)
  - 172.20.10.0/29 (OSPF Backup)
- **Default-information originate:** Inyecta ruta por defecto (0.0.0.0/0) en OSPF

### **Configuración NAT**

```
ip nat inside source list 10 interface GigabitEthernet0/1.100 overload
ip nat inside source list 11 interface GigabitEthernet0/1.200 overload
```

**ACL 10 (NAT por WAN1):**
```
access-list 10 permit 192.168.30.0 0.0.0.255
access-list 10 permit 192.168.10.0 0.0.0.255
access-list 10 permit 192.168.20.0 0.0.0.255
access-list 10 permit 192.168.44.0 0.0.0.255
access-list 10 permit 192.168.55.0 0.0.0.255
access-list 10 permit 192.168.70.0 0.0.0.255
```

**ACL 11 (NAT por WAN2):**
```
access-list 11 permit 192.168.30.0 0.0.0.255
access-list 11 permit 192.168.10.0 0.0.0.255
access-list 11 permit 192.168.20.0 0.0.0.255
access-list 11 permit 192.168.44.0 0.0.0.255
access-list 11 permit 192.168.55.0 0.0.0.255
access-list 11 permit 192.168.70.0 0.0.0.255
```

**Análisis:**
- **Overload (PAT):** Múltiples hosts comparten una IP pública
- **Redundancia:** Dos ACLs idénticas para dos interfaces de salida
- **Redes traducidas:** Todas las LANs internas

### **Rutas Estáticas**

```
ip route 192.168.100.0 255.255.255.248 42.25.25.2
ip route 0.0.0.0 0.0.0.0 42.25.25.2
ip route 192.168.100.8 255.255.255.248 42.25.25.2
ip route 0.0.0.0 0.0.0.0 43.26.26.2 5
ip route 192.168.100.0 255.255.255.248 43.26.26.2 5
ip route 192.168.100.8 255.255.255.248 43.26.26.2 5
```

**Análisis:**
- **Rutas primarias (AD=1):**
  - Ruta por defecto vía WAN1 (42.25.25.2)
  - Rutas a servidores DNS (192.168.100.0/29) y Web (192.168.100.8/29)
- **Rutas backup (AD=5):**
  - Mismas rutas vía WAN2 (43.26.26.2)
  - Solo se activan si WAN1 falla

### **Funciones Clave**

✅ **Gateway a Internet** para toda la red  
✅ **NAT/PAT** con redundancia  
✅ **Hub OSPF** central  
✅ **Inyección de ruta por defecto**  
✅ **Redundancia WAN** con failover automático  

---

## 🔷 2. Router Córdoba {#router-cordoba}

### **Rol en la Red**
- **Gateway** para sucursal Córdoba
- **Aplicación de ACLs** para seguridad FTP
- **Nodo OSPF** intermedio

### **Interfaces Configuradas**

#### **GigabitEthernet0/0 (Trunk a SW_OSPF_BACKUP)**

**Subinterfaz 0/0.1000 - OSPF Backup**
```
interface GigabitEthernet0/0.1000
 description BACKUP_OSPF_VLAN1000
 encapsulation dot1Q 1000
 ip address 172.20.10.2 255.255.255.248
 ip ospf cost 50
 ip ospf priority 50
```
- **Red:** 172.20.10.0/29
- **IP:** .2
- **Costo:** 50 (ruta de backup)
- **Prioridad:** 50 (puede ser BDR)

#### **GigabitEthernet0/2 (Trunk a SW-CORE-DIS-CORD)**

**Subinterfaz 0/2.10 - Usuarios Córdoba**
```
interface GigabitEthernet0/2.10
 description Gateway_VLAN10_USUARIOS
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
```
- **VLAN:** 10
- **Red:** 192.168.10.0/24
- **Gateway:** .1
- **OSPF:** Passive interface

**Subinterfaz 0/2.20 - Servidores Córdoba**
```
interface GigabitEthernet0/2.20
 description Gateway_VLAN20_SERVIDORES
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip access-group FTP_BLOCK out
```
- **VLAN:** 20
- **Red:** 192.168.20.0/24
- **Gateway:** .1
- **ACL:** FTP_BLOCK (salida)
- **OSPF:** Passive interface

#### **GigabitEthernet0/0/0 - P2P a Buenos Aires**
```
interface GigabitEthernet0/0/0
 description P2P_to_BSAS
 ip address 10.10.1.10 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
 ip access-group FTP_BLOCK_P2P in
```
- **Red:** 10.10.1.8/30
- **IP:** .10 (BS.AS: .9)
- **ACL:** FTP_BLOCK_P2P (entrada)

#### **GigabitEthernet0/1/0 - P2P a Mendoza**
```
interface GigabitEthernet0/1/0
 description P2P_to_MENDOZA
 ip address 10.10.1.17 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
```
- **Red:** 10.10.1.16/30
- **IP:** .17 (Mendoza: .18)

### **Configuración OSPF**

```
router ospf 1
 router-id 2.2.2.2
 log-adjacency-changes
 passive-interface GigabitEthernet0/2.10
 passive-interface GigabitEthernet0/2.20
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 10.10.1.8 0.0.0.3 area 0
 network 10.10.1.16 0.0.0.3 area 0
 network 172.20.10.0 0.0.0.7 area 0
```

**Análisis:**
- **Router ID:** 2.2.2.2
- **Passive Interfaces:** VLANs 10 y 20
- **Redes anunciadas:**
  - 192.168.10.0/24 (Usuarios)
  - 192.168.20.0/24 (Servidores)
  - 10.10.1.8/30 (P2P a BS.AS)
  - 10.10.1.16/30 (P2P a Mendoza)
  - 172.20.10.0/29 (Backup)

### **ACLs de Seguridad**

#### **FTP_BLOCK (Interfaz VLAN 20)**
```
ip access-list extended FTP_BLOCK
 permit ip host 192.168.30.10 host 192.168.20.10
 deny ip any host 192.168.20.10
 permit ip any any
```

**Lógica:**
1. **Permitir:** Solo PC de Buenos Aires (192.168.30.10) puede acceder al servidor (192.168.20.10)
2. **Denegar:** Todo lo demás hacia el servidor
3. **Permitir:** Resto del tráfico

#### **FTP_BLOCK_P2P (Interfaz P2P a BS.AS)**
```
ip access-list extended FTP_BLOCK_P2P
 deny tcp 192.168.44.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.55.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.70.0 0.0.0.255 host 192.168.20.10 eq ftp
 permit ip any any
```

**Lógica:** Bloquear FTP desde Mendoza (VLANs 44, 55, 70) al servidor de archivos

### **Ruta Estática**

```
ip route 0.0.0.0 0.0.0.0 10.10.1.9
```

**Análisis:**
- **Ruta por defecto** apunta a Buenos Aires (10.10.1.9)
- **Backup:** Si falla, OSPF aprende ruta por defecto inyectada por BS.AS

### **Funciones Clave**

✅ **Gateway** para VLANs 10 y 20  
✅ **Control de acceso FTP** mediante ACLs  
✅ **Enrutamiento OSPF** entre sucursales  
✅ **Ruta de backup** por VLAN 1000  

---

## 🔷 3. Router Mendoza {#router-mendoza}

### **Rol en la Red**
- **Gateway** para sucursal Mendoza
- **DHCP Relay** para WiFi
- **Nodo OSPF** intermedio

### **Interfaces Configuradas**

#### **GigabitEthernet0/1 (Trunk a SW_OSPF_BACKUP)**

**Subinterfaz 0/1.1000 - OSPF Backup**
```
interface GigabitEthernet0/1.1000
 description BACKUP_OSPF_VLAN1000
 encapsulation dot1Q 1000
 ip address 172.20.10.3 255.255.255.248
 ip ospf cost 50
 ip ospf priority 50
```
- **Red:** 172.20.10.0/29
- **IP:** .3
- **Costo:** 50
- **Prioridad:** 50

#### **GigabitEthernet0/2 (Trunk a SW-CORE-DIS-MEND)**

**Subinterfaz 0/2.44 - WiFi Internos**
```
interface GigabitEthernet0/2.44
 description WiFi_INTERNOS_VLAN44
 encapsulation dot1Q 44
 ip address 192.168.44.1 255.255.255.0
 ip helper-address 192.168.70.10
```
- **VLAN:** 44
- **Red:** 192.168.44.0/24
- **DHCP Relay:** 192.168.70.10
- **OSPF:** Passive interface

**Subinterfaz 0/2.55 - WiFi Invitados**
```
interface GigabitEthernet0/2.55
 description WiFi_INVITADOS_VLAN55
 encapsulation dot1Q 55
 ip address 192.168.55.1 255.255.255.0
 ip helper-address 192.168.70.10
```
- **VLAN:** 55
- **Red:** 192.168.55.0/24
- **DHCP Relay:** 192.168.70.10
- **OSPF:** Passive interface

**Subinterfaz 0/2.70 - Management**
```
interface GigabitEthernet0/2.70
 description MANAGEMENT_VLAN70_NATIVE
 encapsulation dot1Q 70 native
 ip address 192.168.70.1 255.255.255.0
```
- **VLAN:** 70 (Nativa)
- **Red:** 192.168.70.0/24
- **OSPF:** Passive interface

#### **GigabitEthernet0/0/0 - P2P a Buenos Aires**
```
interface GigabitEthernet0/0/0
 description P2P_to_BSAS
 ip address 10.10.1.2 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
```
- **Red:** 10.10.1.0/30
- **IP:** .2 (BS.AS: .1)

#### **GigabitEthernet0/1/0 - P2P a Córdoba**
```
interface GigabitEthernet0/1/0
 description P2P_to_CORDOBA
 ip address 10.10.1.18 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
```
- **Red:** 10.10.1.16/30
- **IP:** .18 (Córdoba: .17)

### **Configuración OSPF**

```
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
```

**Análisis:**
- **Router ID:** 3.3.3.3
- **Passive Interfaces:** VLANs 44, 55, 70
- **Redes anunciadas:**
  - 192.168.44.0/24 (WiFi Internos)
  - 192.168.55.0/24 (WiFi Invitados)
  - 192.168.70.0/24 (Management)
  - 10.10.1.0/30 (P2P a BS.AS)
  - 10.10.1.16/30 (P2P a Córdoba)
  - 172.20.10.0/29 (Backup)

### **Ruta Estática**

```
ip route 0.0.0.0 0.0.0.0 10.10.1.1
```

**Análisis:** Ruta por defecto apunta a Buenos Aires (10.10.1.1)

### **Funciones Clave**

✅ **Gateway** para VLANs 44, 55, 70  
✅ **DHCP Relay** para WiFi  
✅ **Enrutamiento OSPF**  
✅ **Segregación WiFi** (Internos vs Invitados)  

---

## 🔷 4. ISP Local {#isp-local}

### **Rol en la Red**
- **Proveedor de Internet** local
- **Punto de agregación** de enlaces WAN
- **Enrutamiento** hacia ISP Internacional

### **Interfaces**

#### **GigabitEthernet0/0 - Uplink a ISP Internacional**
```
interface GigabitEthernet0/0
 description Internet_uplink
 ip address 164.25.0.2 255.255.255.248
```
- **Red:** 164.25.0.0/29
- **IP:** .2 (ISP Internacional: .1)

#### **GigabitEthernet0/1 - WAN1 desde Buenos Aires**
```
interface GigabitEthernet0/1
 description WAN1_from_BSAS
 ip address 42.25.25.2 255.255.255.248
```
- **Red:** 42.25.25.0/29
- **IP:** .2 (BS.AS: .1)

#### **GigabitEthernet0/2 - WAN2 desde Buenos Aires**
```
interface GigabitEthernet0/2
 description WAN2_from_BSAS
 ip address 43.26.26.2 255.255.255.248
```
- **Red:** 43.26.26.0/29
- **IP:** .2 (BS.AS: .1)

### **Rutas Estáticas**

**Rutas hacia redes internas (vía WAN1 - primarias):**
```
ip route 192.168.30.0 255.255.255.0 42.25.25.1
ip route 192.168.10.0 255.255.255.0 42.25.25.1
ip route 192.168.20.0 255.255.255.0 42.25.25.1
ip route 192.168.44.0 255.255.255.0 42.25.25.1
ip route 192.168.55.0 255.255.255.0 42.25.25.1
ip route 192.168.70.0 255.255.255.0 42.25.25.1
ip route 172.20.10.0 255.255.255.248 42.25.25.1
```

**Rutas hacia redes internas (vía WAN2 - backup, AD=5):**
```
ip route 192.168.30.0 255.255.255.0 43.26.26.1 5
ip route 192.168.10.0 255.255.255.0 43.26.26.1 5
ip route 192.168.20.0 255.255.255.0 43.26.26.1 5
ip route 192.168.44.0 255.255.255.0 43.26.26.1 5
ip route 192.168.55.0 255.255.255.0 43.26.26.1 5
ip route 192.168.70.0 255.255.255.0 43.26.26.1 5
ip route 172.20.10.0 255.255.255.248 43.26.26.1 5
```

**Rutas hacia Internet y servidores públicos:**
```
ip route 0.0.0.0 0.0.0.0 164.25.0.1
ip route 192.168.100.0 255.255.255.248 164.25.0.1
ip route 192.168.100.8 255.255.255.248 164.25.0.1
ip route 45.162.20.10 255.255.255.255 164.25.0.1
ip route 1.1.1.1 255.255.255.255 164.25.0.1
```

### **Funciones Clave**

✅ **Agregación de WAN** (WAN1 + WAN2)  
✅ **Redundancia** con failover automático  
✅ **Enrutamiento** hacia Internet  
✅ **Rutas estáticas** a todas las redes internas  

---

## 🔷 5. ISP Internacional {#isp-internacional}

### **Rol en la Red**
- **Proveedor de Internet** internacional
- **NAT estático** para servidores públicos
- **Hosting** de servidores DNS y Web

### **Interfaces**

#### **GigabitEthernet0/0 - Hacia ISP Local**
```
interface GigabitEthernet0/0
 description Hacia_ISP_LOCAL_G0/0
 ip address 164.25.0.1 255.255.255.248
 ip nat outside
```
- **Red:** 164.25.0.0/29
- **IP:** .1
- **NAT:** Outside

#### **GigabitEthernet0/1 (Trunk a SW-MS-CORE)**

**Subinterfaz 0/1.100 - DNS Server**
```
interface GigabitEthernet0/1.100
 description Hacia_SW_MS_CORE_VLAN100_DNS
 encapsulation dot1Q 100
 ip address 192.168.100.1 255.255.255.248
 ip nat inside
```
- **VLAN:** 100
- **Red:** 192.168.100.0/29
- **NAT:** Inside

**Subinterfaz 0/1.101 - Web Server**
```
interface GigabitEthernet0/1.101
 description Hacia_SW_MS_CORE_VLAN101_WEB
 encapsulation dot1Q 101
 ip address 192.168.100.14 255.255.255.248
 ip nat inside
```
- **VLAN:** 101
- **Red:** 192.168.100.8/29 (gateway: .14)
- **NAT:** Inside

### **NAT Estático**

```
ip nat inside source static 192.168.100.2 1.1.1.1
ip nat inside source static 192.168.100.9 45.162.20.10
```

**Análisis:**
- **DNS Server:** 192.168.100.2 → 1.1.1.1 (IP pública)
- **Web Server:** 192.168.100.9 → 45.162.20.10 (IP pública)

### **Rutas Estáticas**

```
ip route 192.168.30.0 255.255.255.0 164.25.0.2
ip route 42.25.25.0 255.255.255.248 164.25.0.2
ip route 43.26.26.0 255.255.255.248 164.25.0.2
ip route 192.168.10.0 255.255.255.0 164.25.0.2
ip route 192.168.20.0 255.255.255.0 164.25.0.2
ip route 172.20.10.0 255.255.255.248 164.25.0.2
ip route 192.168.44.0 255.255.255.0 164.25.0.2
ip route 192.168.55.0 255.255.255.0 164.25.0.2
ip route 192.168.70.0 255.255.255.0 164.25.0.2
```

**Análisis:** Rutas a todas las redes internas vía ISP Local (164.25.0.2)

### **Funciones Clave**

✅ **NAT estático** para servidores públicos  
✅ **Hosting** de DNS y Web  
✅ **Gateway** a Internet  
✅ **Enrutamiento** hacia redes internas  

---

## 🔷 6. Switches de Distribución {#switches-distribucion}

### **SW-CORE-DIS-CORD (Córdoba)**

#### **Configuración STP**
```
spanning-tree mode pvst
spanning-tree vlan 10,20 priority 4096
```
- **Modo:** PVST (Per-VLAN Spanning Tree)
- **Prioridad:** 4096 → **Root Bridge** para VLANs 10 y 20

#### **Interfaces**

**GigabitEthernet0/1 - Trunk a Router Córdoba**
```
interface GigabitEthernet0/1
 description Trunk_to_Router_CORDOBA_Gig0/2
 switchport trunk allowed vlan 10,20
 switchport mode trunk
```

**GigabitEthernet0/2 - Trunk a SW-ACC-CORD**
```
interface GigabitEthernet0/2
 description Trunk_to_ACC-CORDO
 switchport trunk allowed vlan 10,20
 switchport mode trunk
```

#### **Banner de Seguridad**
```
banner motd ^C
*************************************************
*     ACCESO NO AUTORIZADO PROHIBIDO           *
*     Switch DIS-CORD - Sitio Cordoba Core     *
*     Toda actividad es monitoreada            *
*************************************************
^C
```

### **SW-CORE-DIS-MEND (Mendoza)**

#### **Configuración STP**
```
spanning-tree mode pvst
spanning-tree vlan 44,55,70 priority 24576
```
- **Prioridad:** 24576 → **Root Bridge** para VLANs 44, 55, 70

#### **Interfaces**

**FastEthernet0/24 - Trunk a WLC (Wireless LAN Controller)**
```
interface FastEthernet0/24
 description TRUNK_to_WLC-MENDOZA_Gi1
 switchport trunk native vlan 70
 switchport trunk allowed vlan 44,55,70
 switchport mode trunk
```

**GigabitEthernet0/1 - Trunk a SW-ACC-MEND**
```
interface GigabitEthernet0/1
 description TRUNK_to_SW-ACC-MEND_G0/1
 switchport trunk native vlan 70
 switchport trunk allowed vlan 44,55,70
 switchport mode trunk
```

**GigabitEthernet0/2 - Trunk a Router Mendoza**
```
interface GigabitEthernet0/2
 description TRUNK_to_ROUTER_MENDOZA_G0/2
 switchport trunk native vlan 70
 switchport trunk allowed vlan 44,55,70
 switchport mode trunk
```

---

## 🔷 7. Switches de Acceso {#switches-acceso}

### **SW-ACC-CORD (Córdoba)**

#### **Configuración STP**
```
spanning-tree mode pvst
```
- **Prioridad:** 32768 (default) → Non-root

#### **Puertos de Acceso**

**FastEthernet0/1 - PC Usuario**
```
interface FastEthernet0/1
 description PC2-VLAN10
 switchport access vlan 10
 switchport mode access
 spanning-tree portfast
```

**FastEthernet0/2 - Servidor de Archivos**
```
interface FastEthernet0/2
 description FILE_SERVER_INTERNAL_VLAN20
 switchport access vlan 20
 switchport mode access
 ip access-group FTP_BLOCK in
 spanning-tree portfast
```

#### **ACL FTP_BLOCK**
```
ip access-list extended FTP_BLOCK
 deny tcp 192.168.10.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.20.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.44.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.55.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.70.0 0.0.0.255 host 192.168.20.10 eq ftp
 permit ip any any
```

**Análisis:** Bloqueo de FTP a nivel de switch (defensa en profundidad)

### **SW-ACC-MEND (Mendoza)**

#### **Configuración STP**
```
spanning-tree mode pvst
spanning-tree vlan 44,55,70 priority 49152
```
- **Prioridad:** 49152 → Non-root (mayor que DIS-MEND)

#### **Puertos de Acceso**

**FastEthernet0/1 - PC Admin**
```
interface FastEthernet0/1
 description PC-ADMIN-MENDOZA
 switchport access vlan 70
 switchport mode access
 spanning-tree portfast
```

**FastEthernet0/22 - DHCP Server**
```
interface FastEthernet0/22
 description DHCP-SERVER-MENDOZA
 switchport access vlan 70
 switchport mode access
 spanning-tree portfast
```

**FastEthernet0/23 - RADIUS Server**
```
interface FastEthernet0/23
 description RADIUS-SERVER-MENDOZA
 switchport access vlan 70
 switchport mode access
 spanning-tree portfast
```

**FastEthernet0/24 - Trunk a Access Point**
```
interface FastEthernet0/24
 description TRUNK_to_AP-01-MENDOZA_Gi0
 switchport trunk native vlan 70
 switchport trunk allowed vlan 44,55,70
 switchport mode trunk
```

---

## 🔷 8. Switch OSPF Backup {#switch-backup}

### **SW_OSPF_BACKUP**

#### **Rol:** Proporcionar ruta de backup OSPF entre los tres routers

#### **Configuración STP**
```
spanning-tree mode pvst
spanning-tree vlan 1000 priority 4096
```
- **Prioridad:** 4096 → Root Bridge para VLAN 1000

#### **Interfaces**

**FastEthernet0/22 - Trunk a Router Buenos Aires**
```
interface FastEthernet0/22
 description To_Router_BSAS_Gig0/2
 switchport trunk allowed vlan 1000
 switchport mode trunk
```

**FastEthernet0/23 - Trunk a Router Mendoza**
```
interface FastEthernet0/23
 description To_Router_MENDOZA_Gig0/2
 switchport trunk allowed vlan 1000
 switchport mode trunk
```

**FastEthernet0/24 - Trunk a Router Córdoba**
```
interface FastEthernet0/24
 description To_Router_CORDOBA_Fa0/24
 switchport trunk allowed vlan 1000
 switchport mode trunk
```

#### **Funciones Clave**

✅ **Ruta de backup OSPF** (VLAN 1000)  
✅ **Redundancia** ante falla de enlaces P2P  
✅ **Costo elevado** (50) para ser secundario  

---

## 🔷 9. SW-BUENOSAIRES

### **Rol:** Switch de acceso para Buenos Aires y conexión a ISPs

#### **Puertos de Acceso**

**FastEthernet0/2 - PC Buenos Aires**
```
interface FastEthernet0/2
 description PC_BS_AS
 switchport access vlan 30
 switchport mode access
 spanning-tree portfast
```

**FastEthernet0/24 - WAN2 a ISP Local**
```
interface FastEthernet0/24
 description WAN2_to_ISP_LOCAL_G0/2
 switchport access vlan 200
 switchport mode access
```

**GigabitEthernet0/1 - WAN1 a ISP Local**
```
interface GigabitEthernet0/1
 description WAN1_to_ISP_LOCAL_G0/1
 switchport access vlan 100
 switchport mode access
```

#### **Trunk a Router Buenos Aires**

**GigabitEthernet0/2**
```
interface GigabitEthernet0/2
 description Trunk_to_Router_BSAS_G0/1
 switchport trunk allowed vlan 30,100,200
 switchport mode trunk
```

---

## 🔷 10. SW-MS-CORE

### **Rol:** Switch para servidores públicos (DNS y Web)

#### **Puertos de Acceso**

**FastEthernet0/1 - Web Server**
```
interface FastEthernet0/1
 description To_WEB_SERVER
 switchport access vlan 101
 switchport mode access
```

**FastEthernet0/2 - DNS Server**
```
interface FastEthernet0/2
 description To_DNS_SERVER
 switchport access vlan 100
 switchport mode access
```

#### **Trunk a ISP Internacional**

**GigabitEthernet0/1**
```
interface GigabitEthernet0/1
 description Uplink_to_ISP_INTERNACIONAL
 switchport trunk allowed vlan 100-101
 switchport mode trunk
```

---

## 📊 Resumen de Dispositivos

| Dispositivo | Función Principal | Tecnologías Clave |
|-------------|------------------|-------------------|
| Router BS.AS | Gateway, NAT, OSPF Hub | NAT/PAT, OSPF, Redundancia WAN |
| Router CORDOBA | Gateway, ACLs | OSPF, ACLs extendidas |
| Router MENDOZA | Gateway, DHCP Relay | OSPF, DHCP Relay |
| ISP_LOCAL | Proveedor local | Enrutamiento estático, Redundancia |
| ISP_INTERNACIONAL | Proveedor internacional | NAT estático |
| SW-CORE-DIS-CORD | Distribución Córdoba | STP Root, VLANs |
| SW-CORE-DIS-MEND | Distribución Mendoza | STP Root, VLANs |
| SW-ACC-CORD | Acceso Córdoba | ACLs, PortFast |
| SW-ACC-MEND | Acceso Mendoza | PortFast, Trunk a AP |
| SW_OSPF_BACKUP | Backup OSPF | STP Root VLAN 1000 |
| SW-BUENOSAIRES | Acceso Buenos Aires | VLANs, Trunk |
| SW-MS-CORE | Servidores públicos | VLANs |

---

**Este análisis detalla la configuración y función de cada dispositivo en la red.**
