# Guía 03 · OSPF y Enlaces P2P - PASO A PASO
**Fecha:** 19/11/2025  
**Prerrequisito:** Guía 02 completada (Buenos Aires operativo con LAN VLAN 30)

---

## 🎯 OBJETIVO

Conectar los tres sitios (Buenos Aires ↔ Córdoba ↔ Mendoza) mediante:
- 3 enlaces P2P de fibra óptica (redes /30)
- 1 red de backup VLAN 1000 (red /29 compartida)
- OSPF área única para propagar todas las redes locales
- Ruta por defecto desde BS.AS hacia los otros sitios

---

## 📋 REDES QUE VAMOS A CONFIGURAR

### Enlaces P2P (Fibra Óptica)
| Enlace | Router A | IP A | Router B | IP B | Red /30 |
|--------|----------|------|----------|------|---------|
| P2P-1 | BS.AS Gig0/0/0 | 10.10.1.9/30 | CORDOBA Gig0/0 | 10.10.1.10/30 | 10.10.1.8/30 |
| P2P-2 | BS.AS Gig0/1/0 | 10.10.1.1/30 | MENDOZA Gig0/0/0 | 10.10.1.2/30 | 10.10.1.0/30 |
| P2P-3 | CORDOBA Gig0/1/0 | 10.10.1.17/30 | MENDOZA Gig0/1/0 | 10.10.1.18/30 | 10.10.1.16/30 |

### Red de Backup (VLAN 1000)
| Router | Puerto Switch | Puerto Router | Subinterfaz | IP |
|--------|---------------|---------------|-------------|-----|
| BS.AS | Fa0/22 | Gig0/2 | Gig0/2.1000 | 172.20.10.1/29 |
| CORDOBA | Fa0/24 | Gig0/0 | Gig0/0.1000 | 172.20.10.2/29 |
| MENDOZA | Fa0/23 | Gig0/1 | Gig0/1.1000 | 172.20.10.3/29 |

> ⚠️ **IMPORTANTE:** El puerto del switch NO tiene que coincidir con el puerto del router. Verificar el cableado físico en Packet Tracer.

---

## 📝 PASO 1: SWITCH SW_OSPF_BACKUP

### Configuración
```cisco
enable
configure terminal

hostname SW_OSPF_BACKUP

! Crear VLAN 1000
vlan 1000
 name BACKUP_OSPF
exit

! Puerto hacia BS.AS
interface fa0/22
 description To_Router_BSAS_Gig0/2
 switchport mode trunk
 switchport trunk allowed vlan 1000
exit

! Puerto hacia CORDOBA
interface fa0/24
 description To_Router_CORDOBA_Gig0/0
 switchport mode trunk
 switchport trunk allowed vlan 1000
exit

! Puerto hacia MENDOZA
interface fa0/23
 description To_Router_MENDOZA_Gig0/1
 switchport mode trunk
 switchport trunk allowed vlan 1000
exit

spanning-tree vlan 1000 priority 4096
exit
write memory
```

> ⚠️ **CRÍTICO:** NO configurar `switchport trunk native vlan 1000` en los puertos. Dejar el native VLAN en 1 (default). Esto es necesario para que el etiquetado 802.1Q funcione correctamente con las subinterfaces.

### Verificación
```cisco
show vlan brief
show interfaces trunk
show interfaces status
```

**✅ Esperado:**
- VLAN 1000 creada con nombre BACKUP_OSPF
- Fa0/22, Fa0/23, Fa0/24 en modo **trunk**
- VLAN 1000 permitida en los tres puertos
- Native VLAN debe ser 1 (no 1000)

---

## 📝 PASO 2: ROUTER BS.AS

### 2A. Configurar Interfaces P2P y Backup
```cisco
enable
configure terminal

! P2P a CORDOBA
interface gig0/0/0
 description P2P_to_CORDOBA
 ip address 10.10.1.9 255.255.255.252
 no shutdown
exit

! P2P a MENDOZA
interface gig0/1/0
 description P2P_to_MENDOZA
 ip address 10.10.1.1 255.255.255.252
 no shutdown
exit

! Interfaz física para backup
interface gig0/2
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 no shutdown
exit

! Subinterfaz backup VLAN 1000
interface gig0/2.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.1 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit
exit
```

### 2B. Configurar OSPF
```cisco
configure terminal

! Proceso OSPF
router ospf 1
 router-id 1.1.1.1
 log-adjacency-changes
 passive-interface gig0/1.30
 network 192.168.30.0 0.0.0.255 area 0
 network 10.10.1.8 0.0.0.3 area 0
 network 10.10.1.0 0.0.0.3 area 0
 network 172.20.10.0 0.0.0.7 area 0
 default-information originate
exit

! Parámetros OSPF en interfaces P2P
interface gig0/0/0
 ip ospf network point-to-point
 ip ospf cost 10
exit

interface gig0/1/0
 ip ospf network point-to-point
 ip ospf cost 10
exit

! Parámetros OSPF en backup
interface gig0/2.1000
 ip ospf priority 100
 ip ospf cost 50
exit
exit
write memory
```

### Verificación
```cisco
show ip interface brief
show ip ospf interface brief
show running-config | section ospf
```

**✅ Esperado:**
- Gig0/0/0, Gig0/1/0, Gig0/2.1000 en estado **up/up**
- Las 3 interfaces aparecen en OSPF
- Gig0/1.30 configurada como passive

---

## 📝 PASO 3: ROUTER CORDOBA

### 3A. Configurar Interfaces P2P, LANs y Backup
```cisco
enable
configure terminal

! P2P a BSAS
interface gig0/0
 description P2P_to_BSAS
 ip address 10.10.1.10 255.255.255.252
 no shutdown
exit

! P2P a MENDOZA
interface gig0/1/0
 description P2P_to_MENDOZA
 ip address 10.10.1.17 255.255.255.252
 no shutdown
exit

! Interfaz física para LANs locales
interface gig0/1
 description Trunk_to_DIS-CORD
 no ip address
 no shutdown
exit

! Subinterfaces LANs
interface gig0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 description LAN_CORDOBA_VLAN10
exit

interface gig0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 description LAN_CORDOBA_VLAN20
exit

! Interfaz física para backup
interface gig0/0
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 no shutdown
exit

! Subinterfaz backup VLAN 1000
interface gig0/0.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.2 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit
exit
```

### 3B. Configurar OSPF
```cisco
configure terminal

! Proceso OSPF
router ospf 1
 router-id 2.2.2.2
 log-adjacency-changes
 passive-interface gig0/1.10
 passive-interface gig0/1.20
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 10.10.1.8 0.0.0.3 area 0
 network 10.10.1.16 0.0.0.3 area 0
 network 172.20.10.0 0.0.0.7 area 0
exit

! Parámetros OSPF en interfaces P2P
interface gig0/0
 ip ospf network point-to-point
 ip ospf cost 10
exit

interface gig0/1/0
 ip ospf network point-to-point
 ip ospf cost 10
exit

! Parámetros OSPF en backup
interface gig0/0.1000
 ip ospf priority 50
 ip ospf cost 50
exit
exit
write memory
```

### Verificación
```cisco
show ip interface brief
show ip ospf interface brief
show running-config | section ospf
```

**✅ Esperado:**
- Gig0/0, Gig0/1/0, Gig0/0.1000 en OSPF
- Gig0/1.10 y Gig0/1.20 como passive

---

## 📝 PASO 4: ROUTER MENDOZA

### 4A. Configurar Interfaces P2P, LANs y Backup
```cisco
enable
configure terminal

! P2P a BSAS
interface gig0/0/0
 description P2P_to_BSAS
 ip address 10.10.1.2 255.255.255.252
 no shutdown
exit

! P2P a CORDOBA
interface gig0/1/0
 description P2P_to_CORDOBA
 ip address 10.10.1.18 255.255.255.252
 no shutdown
exit

! Interfaz física para backup Y LANs locales (compartida)
interface gig0/1
 description Trunk_to_SW_OSPF_BACKUP_and_LANs
 no ip address
 no shutdown
exit

! Subinterfaces LANs
interface gig0/1.44
 encapsulation dot1Q 44
 ip address 192.168.44.1 255.255.255.0
 description WiFi_INTERNOS_VLAN44
exit

interface gig0/1.55
 encapsulation dot1Q 55
 ip address 192.168.55.1 255.255.255.0
 description WiFi_INVITADOS_VLAN55
exit

interface gig0/1.70
 encapsulation dot1Q 70
 ip address 192.168.70.1 255.255.255.0
 description MANAGEMENT_VLAN70
exit

! Subinterfaz backup VLAN 1000 (misma interfaz física)
interface gig0/1.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.3 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit
exit
```

### 4B. Configurar OSPF
```cisco
configure terminal

! Proceso OSPF
router ospf 1
 router-id 3.3.3.3
 log-adjacency-changes
 passive-interface gig0/1.44
 passive-interface gig0/1.55
 passive-interface gig0/1.70
 network 192.168.44.0 0.0.0.255 area 0
 network 192.168.55.0 0.0.0.255 area 0
 network 192.168.70.0 0.0.0.255 area 0
 network 10.10.1.0 0.0.0.3 area 0
 network 10.10.1.16 0.0.0.3 area 0
 network 172.20.10.0 0.0.0.7 area 0
exit

! Parámetros OSPF en interfaces P2P
interface gig0/0/0
 ip ospf network point-to-point
 ip ospf cost 10
exit

interface gig0/1/0
 ip ospf network point-to-point
 ip ospf cost 10
exit

! Parámetros OSPF en backup
interface gig0/1.1000
 ip ospf priority 50
 ip ospf cost 50
exit
exit
write memory
```

### Verificación
```cisco
show ip interface brief
show ip ospf interface brief
show running-config | section ospf
```

**✅ Esperado:**
- Gig0/0/0, Gig0/1/0, Gig0/1.1000 en OSPF
- Gig0/1.44, Gig0/1.55, Gig0/1.70 como passive

---

## 🔍 PASO 5: VERIFICACIONES CRÍTICAS

### 5A. Verificar Vecindades OSPF (EN CADA ROUTER)
```cisco
show ip ospf neighbor
```

**✅ CADA ROUTER DEBE MOSTRAR EXACTAMENTE 4 VECINOS:**

#### En BS.AS:
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2          0    FULL/  -        00:00:3x    10.10.1.10      GigabitEthernet0/0/0
3.3.3.3          0    FULL/  -        00:00:3x    10.10.1.2       GigabitEthernet0/1/0
2.2.2.2         50    FULL/BDR        00:00:3x    172.20.10.2     GigabitEthernet0/2.1000
3.3.3.3         50    FULL/DROTHER    00:00:3x    172.20.10.3     GigabitEthernet0/2.1000
```

#### En CORDOBA:
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          0    FULL/  -        00:00:3x    10.10.1.9       GigabitEthernet0/0
3.3.3.3          0    FULL/  -        00:00:3x    10.10.1.18      GigabitEthernet0/1/0
1.1.1.1        100    FULL/DR         00:00:3x    172.20.10.1     GigabitEthernet0/0.1000
3.3.3.3         50    FULL/DROTHER    00:00:3x    172.20.10.3     GigabitEthernet0/0.1000
```

#### En MENDOZA:
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          0    FULL/  -        00:00:3x    10.10.1.1       GigabitEthernet0/0/0
2.2.2.2          0    FULL/  -        00:00:3x    10.10.1.17      GigabitEthernet0/1/0
1.1.1.1        100    FULL/DR         00:00:3x    172.20.10.1     GigabitEthernet0/1.1000
2.2.2.2         50    FULL/BDR        00:00:3x    172.20.10.2     GigabitEthernet0/1.1000
```

⚠️ **Si no ves 4 vecinos:**
- Verifica que todas las interfaces estén **up/up**: `show ip interface brief`
- Verifica que los comandos `network` en OSPF estén correctos
- Espera 40 segundos (Dead Timer) y vuelve a verificar

---

### 5B. Verificar Tabla de Ruteo
```cisco
show ip route ospf
```

**✅ En CORDOBA y MENDOZA debe aparecer:**
- Ruta **O*E2** 0.0.0.0/0 (default route desde BS.AS)
- Rutas **O** a todas las redes LAN remotas

**Ejemplo en CORDOBA:**
```
O*E2  0.0.0.0/0 [110/1] via 10.10.1.9, 00:05:32, GigabitEthernet0/0
O     192.168.30.0/24 [110/11] via 10.10.1.9, 00:05:32, GigabitEthernet0/0
O     192.168.44.0/24 [110/21] via 10.10.1.18, 00:05:20, GigabitEthernet0/1/0
O     192.168.55.0/24 [110/21] via 10.10.1.18, 00:05:20, GigabitEthernet0/1/0
O     192.168.70.0/24 [110/21] via 10.10.1.18, 00:05:20, GigabitEthernet0/1/0
```

---

### 5C. Pruebas de Conectividad
**Desde Router BS.AS:**
```cisco
ping 192.168.10.1
ping 192.168.20.1
ping 192.168.44.1
ping 192.168.55.1
ping 192.168.70.1
```

**✅ Todos los pings deben responder exitosamente**

---

## 🚨 TROUBLESHOOTING: "No tengo 4 vecinos"

### Problema: Solo veo 2 vecinos en CORDOBA
**Causa más probable:** OSPF no está configurado en `Gig0/1/0` (enlace P2P a MENDOZA)

**Solución:**
```cisco
conf t
interface gig0/1/0
 ip ospf network point-to-point
 ip ospf cost 10
!
router ospf 1
 network 10.10.1.16 0.0.0.3 area 0
end
```

### Problema: Solo veo 3 vecinos en MENDOZA
**Causa:** Interfaz `Gig0/1/0` apagada o sin OSPF

**Solución:**
```cisco
conf t
interface gig0/1/0
 no shutdown
 ip ospf network point-to-point
 ip ospf cost 10
!
router ospf 1
 network 10.10.1.16 0.0.0.3 area 0
end
```

### Problema: El ping funciona pero no veo vecino OSPF
**Explicación:** El tráfico puede estar enrutándose por un camino alternativo (por ejemplo, a través de BS.AS como intermediario).

**Validación:**
```cisco
show ip route 10.10.1.16
traceroute 10.10.1.18
```

---

## ✅ CHECKLIST FINAL

- [ ] Switch SW_OSPF_BACKUP: VLAN 1000 creada, Fa0/22/23/24 en trunk
- [ ] Router BS.AS: 4 vecinos OSPF visibles (2 P2P + 2 backup en Gig0/2.1000)
- [ ] Router CORDOBA: 3-4 vecinos OSPF visibles (1-2 P2P + 2 backup en Gig0/0.1000)
- [ ] Router MENDOZA: 3-4 vecinos OSPF visibles (1-2 P2P + 2 backup en Gig0/1.1000)
- [ ] CORDOBA y MENDOZA muestran ruta O*E2 0.0.0.0/0
- [ ] Ping desde BS.AS a todos los gateways LAN remotos exitoso
- [ ] `show ip ospf interface brief` muestra costos correctos (P2P=10, Backup=50)
- [ ] Configuraciones guardadas con `write memory`

---

## 📄 DOCUMENTACIÓN

Guardar las siguientes salidas para el informe:
```cisco
show ip ospf neighbor
show ip route
show ip ospf interface brief
show running-config | section ospf
```

---

**¡Guía completada! 🎉**  
Con estos pasos, los tres sitios estarán conectados y comunicándose vía OSPF.
