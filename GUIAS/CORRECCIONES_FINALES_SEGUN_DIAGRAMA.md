# ✅ CORRECCIONES FINALES SEGÚN DIAGRAMA DE TOPOLOGÍA
**Fecha:** 19/11/2025  
**Estado:** ✅ VERIFICADO CON DIAGRAMAS FÍSICOS

---

## 📐 ANÁLISIS DEL DIAGRAMA

He verificado cuidadosamente los diagramas de topología que compartiste y confirmé las interfaces correctas.

### 🔌 CABLEADO SWITCH SW_OSPF_BACKUP (VLAN 1000)

Según el diagrama:

| Router | Puerto del Switch | Puerto del Router | Subinterfaz | IP en VLAN 1000 |
|--------|-------------------|-------------------|-------------|-----------------|
| **BS.AS** | Fa0/22 | **Gig0/2** | Gig0/2.1000 | 172.20.10.1/29 |
| **CÓRDOBA** | Fa0/24 | **Fa0/24** | Fa0/24.1000 | 172.20.10.2/29 |
| **MENDOZA** | Fa0/23 | **Gig0/2** | Gig0/2.1000 | 172.20.10.3/29 |

> ⚠️ **NOTA IMPORTANTE:** El número del puerto del switch NO tiene que coincidir con el número del puerto del router. Lo que importa es la conexión física del cable.

---

## 🔧 CONFIGURACIONES CORRECTAS

### Router BS.AS
```cisco
interface gig0/2
 description Trunk_to_SW_OSPF_BACKUP_Fa0/22
 no ip address
 no shutdown
exit

interface gig0/2.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.1 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit

router ospf 1
 network 172.20.10.0 0.0.0.7 area 0
exit

interface gig0/2.1000
 ip ospf priority 100
 ip ospf cost 50
exit
```

### Router CÓRDOBA
```cisco
interface fa0/24
 description Trunk_to_SW_OSPF_BACKUP_Fa0/24
 no ip address
 no shutdown
exit

interface fa0/24.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.2 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit

router ospf 1
 network 172.20.10.0 0.0.0.7 area 0
exit

interface fa0/24.1000
 ip ospf priority 50
 ip ospf cost 50
exit
```

### Router MENDOZA
```cisco
interface gig0/2
 description Trunk_to_SW_OSPF_BACKUP_Fa0/23
 no ip address
 no shutdown
exit

interface gig0/2.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.3 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit

router ospf 1
 network 172.20.10.0 0.0.0.7 area 0
exit

interface gig0/2.1000
 ip ospf priority 50
 ip ospf cost 50
exit
```

---

## 🔍 VERIFICACIÓN EN PACKET TRACER

### 1. Verificar interfaces físicas y subinterfaces
```cisco
show ip interface brief
```

**Esperado en BS.AS:**
```
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/2     unassigned      YES NVRAM  up                    up
GigabitEthernet0/2.1000 172.20.10.1    YES NVRAM  up                    up
```

**Esperado en CÓRDOBA:**
```
Interface              IP-Address      OK? Method Status                Protocol
FastEthernet0/24       unassigned      YES NVRAM  up                    up
FastEthernet0/24.1000  172.20.10.2     YES NVRAM  up                    up
```

**Esperado en MENDOZA:**
```
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/2     unassigned      YES NVRAM  up                    up
GigabitEthernet0/2.1000 172.20.10.3    YES NVRAM  up                    up
```

### 2. Verificar OSPF en interfaces backup
```cisco
show ip ospf interface brief
```

**Esperado:**
- **BS.AS:** Gig0/2.1000 debe aparecer con costo 50
- **CÓRDOBA:** Fa0/24.1000 debe aparecer con costo 50
- **MENDOZA:** Gig0/2.1000 debe aparecer con costo 50

### 3. Verificar vecindades OSPF (¡CRÍTICO!)
```cisco
show ip ospf neighbor
```

**✅ CADA ROUTER DEBE MOSTRAR EXACTAMENTE 4 VECINOS:**

**BS.AS debe ver:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2          0    FULL/  -        00:00:3x    10.10.1.10      GigabitEthernet0/0/0
3.3.3.3          0    FULL/  -        00:00:3x    10.10.1.2       GigabitEthernet0/1/0
2.2.2.2         50    FULL/BDR        00:00:3x    172.20.10.2     GigabitEthernet0/2.1000
3.3.3.3         50    FULL/DROTHER    00:00:3x    172.20.10.3     GigabitEthernet0/2.1000
```

**CÓRDOBA debe ver:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          0    FULL/  -        00:00:3x    10.10.1.9       GigabitEthernet0/0
3.3.3.3          0    FULL/  -        00:00:3x    10.10.1.18      GigabitEthernet0/1/0
1.1.1.1        100    FULL/DR         00:00:3x    172.20.10.1     FastEthernet0/24.1000
3.3.3.3         50    FULL/DROTHER    00:00:3x    172.20.10.3     FastEthernet0/24.1000
```

**MENDOZA debe ver:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          0    FULL/  -        00:00:3x    10.10.1.1       GigabitEthernet0/0/0
2.2.2.2          0    FULL/  -        00:00:3x    10.10.1.17      GigabitEthernet0/1/0
1.1.1.1        100    FULL/DR         00:00:3x    172.20.10.1     GigabitEthernet0/2.1000
2.2.2.2         50    FULL/BDR        00:00:3x    172.20.10.2     GigabitEthernet0/2.1000
```

---

## 📝 RESUMEN DE INTERFACES POR ROUTER

### Router BS.AS (2911)
- **Gig0/1** → Trunk a SW-BS-AS (VLANs 30/100/200)
  - Gig0/1.30 → LAN (192.168.30.0/24)
  - Gig0/1.100 → WAN1 (42.25.25.1/29)
  - Gig0/1.200 → WAN2 (43.26.26.1/29)
- **Gig0/0/0** → P2P a CÓRDOBA (10.10.1.9/30)
- **Gig0/1/0** → P2P a MENDOZA (10.10.1.1/30)
- **Gig0/2** → Trunk a SW_OSPF_BACKUP (VLAN 1000)
  - Gig0/2.1000 → Backup OSPF (172.20.10.1/29)

### Router CÓRDOBA (2911)
- **Gig0/1** → Trunk a switch local (VLANs 10/20)
  - Gig0/1.10 → LAN (192.168.10.0/24)
  - Gig0/1.20 → LAN (192.168.20.0/24)
- **Gig0/0** → P2P a BS.AS (10.10.1.10/30)
- **Gig0/1/0** → P2P a MENDOZA (10.10.1.17/30)
- **Fa0/24** → Trunk a SW_OSPF_BACKUP (VLAN 1000)
  - Fa0/24.1000 → Backup OSPF (172.20.10.2/29)

### Router MENDOZA (2911)
- **Gig0/1** → Trunk a switch local (VLANs 44/55/70)
  - Gig0/1.44 → WiFi Internos (192.168.44.0/24)
  - Gig0/1.55 → WiFi Invitados (192.168.55.0/24)
  - Gig0/1.70 → Management (192.168.70.0/24)
- **Gig0/0/0** → P2P a BS.AS (10.10.1.2/30)
- **Gig0/1/0** → P2P a CÓRDOBA (10.10.1.18/30)
- **Gig0/2** → Trunk a SW_OSPF_BACKUP (VLAN 1000)
  - Gig0/2.1000 → Backup OSPF (172.20.10.3/29)

---

## ✅ CONFIRMACIÓN FINAL

Las correcciones aplicadas en la **Guía 03** ahora coinciden **exactamente** con los diagramas de topología física:

- ✅ **BS.AS:** Usa Gig0/2 y Gig0/2.1000
- ✅ **CÓRDOBA:** Usa Fa0/24 y Fa0/24.1000
- ✅ **MENDOZA:** Usa Gig0/2 y Gig0/2.1000

**Próximo paso:** Aplicar la configuración en Packet Tracer y verificar las 4 vecindades OSPF por router.
