# Guía 03 · Enlaces P2P y OSPF Multi-Sitio (BS.AS ↔ CORDOBA ↔ MENDOZA)
**Actualización:** 19/11/2025 - Corregida y alineada con Guías 01-02  
**Objetivo:** Configurar la conectividad entre los tres sitios mediante enlaces P2P de fibra y red de backup VLAN 1000, implementar OSPF área única con propagación de redes locales, manipular costos para path único y habilitar `default-information originate` desde Router BS.AS según requerimientos del profesor.

**✅ Correcciones aplicadas (19/11/2025):**
- SW_OSPF_BACKUP: puertos configurados como **trunk** (no access) para funcionar con subinterfaces .1000
- Córdoba: mantiene VLAN 10 y VLAN 20 (sin VLAN 30 duplicada)
- BS.AS: referencias actualizadas a VLAN 30 (192.168.30.0/24) e interfaz backup **Gig0/2** (no Fa0/22)
- OSPF BS.AS: eliminado network 192.168.100.0 (servidores accesibles vía default route)

**✅ Verificación de consistencia completa (19/11/2025):**
- Cableado lógico ↔ configuración coinciden 1:1 con topología
- Sin conflictos de redes ni interfaces duplicadas
- OSPF cumple requerimientos: P2P en interfaces físicas, backup en VLAN 1000, área única, default-information originate
- **Interfaz backup corregida:** CÓRDOBA usa Fa0/24 (no Gig0/0), MENDOZA usa Fa0/23 (no Gig0/1)

**⚠️ PROBLEMA FRECUENTE - Enlace CÓRDOBA ↔ MENDOZA:**
Si en diagnósticos anteriores el ping entre CÓRDOBA y MENDOZA funcionaba pero OSPF no formaba vecindad por el P2P:
- **Causa:** OSPF no configurado en Gig0/2 de CÓRDOBA, o Gig0/1/0 apagada en MENDOZA
- **Síntoma:** El ping funciona igual porque OSPF enruta por BS.AS o VLAN1000 (camino alternativo)
- **Solución:** Esta guía ya incluye `ip ospf network point-to-point` en CÓRDOBA Gig0/2 y `no shutdown` en MENDOZA Gig0/1/0
- **Validación correcta:** NO confiar solo en ping. Usar `show ip ospf neighbor` para confirmar las 4 vecindades por router

---

## 0. Referencias y dependencias
- `GUIAS/01 - guia_segmento_wan.md`: ISP_LOCAL e ISP_INTERNACIONAL operativos.
- `GUIAS/02 - guia_segmento_bsas.md`: LAN Buenos Aires (VLAN 30 – 192.168.30.0/24) funcional.
- `Analisis y requisitos/reque.md`: lineamientos OSPF área única, vecindades P2P y broadcast.
  - **⚠️ CAMBIO CRÍTICO DEL PROFESOR:** Las conexiones P2P entre routers (ENLACES DE FIBRA) deben configurarse **directamente en interfaces físicas** (NO usar subinterfaces .500).
  - **Razón:** Packet Tracer tiene limitación con OSPF en subinterfaces P2P.
- `Analisis y requisitos/CAMBIOS_CRITICOS_PROFESOR.md`: documento detallado con análisis completo del cambio.
- `Analisis y requisitos/notasprofeso`: reunión 14-11-2025 confirma:
  - **Enlaces P2P router↔router**: IP directa en interfaz física (no subinterfaces .500), por limitación Packet Tracer.
  - **Red backup `SW_OSPF_BACKUP`**: VLAN 1000 con subinterfaz .1000 (Router-on-a-Stick), vecindad OSPF tipo broadcast (según reque.md punto 4 OSPF: "NO CONFIGURAR IP EN LAS INTERFACES FISICAS DE DICHO ENLACE").
  - **OSPF área única**, manipular costos para un solo path por destino (no 3 caminos simultáneos).
  - **Propagación de default** con `default-information originate` desde BS.AS (no configurar manual en cada router).
  - **Rutas estáticas en BS.AS**: 2 rutas predeterminadas con métricas diferentes para evitar ECMP.
- `GUIAS/ticket_trabajo_practico.md`: documentar evidencias Fase 3.

---

## 1. Alcance y topología

### 1.1 Enlaces Point-to-Point (Fibra Óptica)
Según imagen adjunta y corrección del profesor:

| Enlace | Router A | Interfaz A | IP A | Router B | Interfaz B | IP B | Red /30 |
|--------|----------|------------|------|----------|------------|------|---------|
| P2P-1 | BS.AS | Gig0/0/0 | 10.10.1.9/30 | CORDOBA | Gig0/0 | 10.10.1.10/30 | 10.10.1.8/30 |
| P2P-2 | BS.AS | Gig0/1/0 | 10.10.1.1/30 | MENDOZA | Gig0/0/0 | 10.10.1.2/30 | 10.10.1.0/30 |
| P2P-3 | CORDOBA | Gig0/1/0 | 10.10.1.17/30 | MENDOZA | Gig0/1/0 | 10.10.1.18/30 | 10.10.1.16/30 |

> **⚠️ CRÍTICO:** Configurar IPs **directamente en las interfaces físicas** (NO usar subinterfaces .500). Esta es una corrección del profesor para que OSPF point-to-point funcione correctamente en Packet Tracer.

### 1.2 Red de Backup (Broadcast - VLAN 1000)
Switch `SW_OSPF_BACKUP` conecta los tres routers mediante VLAN 1000:

| Router | Puerto Switch | Interfaz Router | Subinterfaz | IP | VLAN | Red |
|--------|---------------|-----------------|-------------|-----|------|-----|
| BS.AS | Fa0/22 | Gig0/2 | Gig0/2.1000 | 172.20.10.1/29 | 1000 | 172.20.10.0/29 |
| CORDOBA | Fa0/24 | Gig0/2 | Gig0/2.1000 | 172.20.10.2/29 | 1000 | 172.20.10.0/29 |
| MENDOZA | Fa0/23 | Gig0/1 | Gig0/1.1000 | 172.20.10.3/29 | 1000 | 172.20.10.0/29 |

> **✅ IMPORTANTE:** Esta configuración SÍ usa subinterfaz .1000 (Router-on-a-Stick) porque es un enlace compartido en switch. NO confundir con los enlaces P2P de fibra que van directo en interfaces físicas. Los requerimientos especifican: "NO CONFIGURAR IP EN LAS INTERFACES FÍSICAS DE DICHO ENLACE" para SW_OSPF_BACKUP.

**Direccionamiento sugerido VLAN 1000:**
- BS.AS: 172.20.10.1/29
- CORDOBA: 172.20.10.2/29
- MENDOZA: 172.20.10.3/29

### 1.3 Redes LAN a propagar por OSPF
| Sitio | VLAN | Red | Gateway | Descripción |
|-------|------|-----|---------|-------------|
| BS.AS | 30 | 192.168.30.0/24 | 192.168.30.1 (G0/1.30) | ✅ LAN Buenos Aires |
| SERVIDORES DNS | 100 | 192.168.100.0/29 | 192.168.100.1 | ✅ DNS-SERVER (.2) |
| SERVIDORES WEB | 101 | 192.168.100.8/29 | 192.168.100.14 | ✅ WEB-SERVER (.9) |
| CORDOBA | 10 | 192.168.10.0/24 | 192.168.10.1 | ⏳ Pendiente config |
| CORDOBA | 20 | 192.168.20.0/24 | 192.168.20.1 | ✅ Sin conflicto con BS.AS |
| MENDOZA | 44 | 192.168.44.0/24 | 192.168.44.1 | ⏳ Pendiente config |
| MENDOZA | 55 | 192.168.55.0/24 | 192.168.55.1 | ⏳ Pendiente config |
| MENDOZA | 70 | 192.168.70.0/24 | 192.168.70.1 | ⏳ Pendiente config |

> **📝 Notas importantes:**
> - **Conflicto resuelto:** BS.AS ahora usa VLAN 30 (192.168.30.0/24), eliminando el conflicto con Córdoba VLAN 20. Ver `CAMBIOS_CRITICOS_PROFESOR.md` para detalles.
> - **Servidores DNS/WEB:** Estas redes (192.168.100.0/29 y 192.168.100.8/29) están en la nube de Internet detrás de ISP_LOCAL. **NO se propagan por OSPF**. Córdoba y Mendoza llegan a ellas usando la ruta por defecto que BS.AS origina con `default-information originate`.

---

## 2. Prerrequisitos
1. **✅ Fases 1 y 2 completadas:** ISP_LOCAL operativo con rutas a 192.168.30.0/24, Router BS.AS con LAN en VLAN 30 (ver guías 01 y 02).
2. Configuraciones actualizadas con nuevos requerimientos del profesor.
3. Cableado físico verificado según imagen:
   - Enlaces P2P fibra conectados (Gig0/0/0, Gig0/1/0, Gig0/2).
   - Cables al `SW_OSPF_BACKUP`:
     - Switch Fa0/22 ↔ Router BS.AS Gig0/2
     - Switch Fa0/24 ↔ Router CORDOBA Gig0/0
     - Switch Fa0/23 ↔ Router MENDOZA Gig0/1
4. Acceso por consola/SSH a los tres routers y al switch de backup.
5. Snapshot previo de configuraciones en `CONFIG POR DISP`.

---

## 3. Paso a paso

### 3.1 Switch `SW_OSPF_BACKUP` – Red Backup VLAN 1000
**Dispositivo:** `SW_OSPF_BACKUP`

**Acceso inicial:**
```
enable
configure terminal
```

**Configuración:**
```
hostname SW_OSPF_BACKUP
!
vlan 1000
 name BACKUP_OSPF
exit
!
interface fa0/22
 description To_Router_BSAS_Gig0/2
 switchport mode trunk
 switchport trunk allowed vlan 1000
 switchport trunk native vlan 1000
exit
!
interface fa0/24
 description To_Router_CORDOBA_Fa0/24
 switchport mode trunk
 switchport trunk allowed vlan 1000
 switchport trunk native vlan 1000
exit
!
interface fa0/23
 description To_Router_MENDOZA_Fa0/23
 switchport mode trunk
 switchport trunk allowed vlan 1000
 switchport trunk native vlan 1000
exit
!
spanning-tree vlan 1000 priority 4096
exit
```

> **✅ IMPORTANTE:** Los puertos deben ser **trunk** (no access) porque los routers usan subinterfaces con `encapsulation dot1Q 1000`. El modo trunk permite que el etiquetado 802.1Q funcione correctamente con las subinterfaces .1000.

**Guardar configuración:**
```
write memory
```

**Validaciones:**
```
show vlan brief
show interfaces trunk
show interfaces status
```
> Esperado: VLAN 1000 creada, Fa0/22/23/24 en modo **trunk** con VLAN 1000 permitida

---

### 3.2 Router BS.AS – Interfaces P2P y Backup

**Dispositivo:** `Router BS.AS`

**Acceso inicial:**
```
enable
configure terminal
```

#### 3.2.1 Configurar interfaces físicas y subinterfaces
```
configure terminal
!
! Enlaces P2P (IP directa en interfaz física)
interface gig0/0/0
 description P2P_to_CORDOBA
 ip address 10.10.1.9 255.255.255.252
 no shutdown
exit
!
interface gig0/1/0
 description P2P_to_MENDOZA
 ip address 10.10.1.1 255.255.255.252
 no shutdown
exit
!
! Interfaz física para backup (trunk)
interface gig0/2
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 no shutdown
exit
!
! Subinterfaz backup VLAN 1000
interface gig0/2.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.1 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit
exit
```

#### 3.2.2 Configurar OSPF y parámetros de interfaces
```
configure terminal
!
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
!
! Parámetros OSPF en interfaces P2P
interface gig0/0/0
 ip ospf network point-to-point
 ip ospf cost 10
exit
!
interface gig0/1/0
 ip ospf network point-to-point
 ip ospf cost 10
exit
!
! Parámetros OSPF en backup (debe configurarse DESPUÉS de tener IP)
interface gig0/2.1000
 ip ospf network broadcast
 ip ospf priority 100
 ip ospf cost 50
exit
exit
```

> **📝 Notas importantes:**
> - Red BS.AS es 192.168.30.0/24 (VLAN 30), interfaz pasiva es G0/1.30
> - **No se anuncia 192.168.100.0/29 en OSPF** porque esa red está detrás del ISP_LOCAL (fuera del dominio OSPF). Córdoba y Mendoza llegan a los servidores (192.168.100.2 DNS y 192.168.100.9 WEB) usando la ruta por defecto OSPF hacia BS.AS (`default-information originate`)

**Guardar configuración:**
```
write memory
```

> **Costos:** P2P con costo 10 (preferidos), backup con costo 50 (secundario). Ajustar según necesidad de balanceo.

---

### 3.3 Router CORDOBA – Interfaces P2P y Backup

**Dispositivo:** `Router CORDOBA`

**Acceso inicial:**
```
enable
configure terminal
```

#### 3.3.1 Configurar interfaces físicas y subinterfaces
```
configure terminal
!
! Enlaces P2P (IP directa en interfaz física)
interface gig0/0
 description P2P_to_BSAS
 ip address 10.10.1.10 255.255.255.252
 no shutdown
exit
!
interface gig0/1/0
 description P2P_to_MENDOZA
 ip address 10.10.1.17 255.255.255.252
 no shutdown
exit
!
! Interfaz física para LANs locales
interface gig0/1
 description Trunk_to_DIS-CORD
 no ip address
 no shutdown
exit
!
! Subinterfaces LANs
interface gig0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 description LAN_CORDOBA_VLAN10
exit
!
interface gig0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 description LAN_CORDOBA_VLAN20
exit
!
! Interfaz física para backup (trunk)
interface gig0/2
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 no shutdown
exit
!
! Subinterfaz backup VLAN 1000
interface gig0/2.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.2 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit
exit
```

> **✅ Conflicto resuelto:** El conflicto de red 192.168.20.0/24 se resolvió moviendo Buenos Aires a VLAN 30 (192.168.30.0/24). Córdoba mantiene sus VLANs originales: VLAN 10 (192.168.10.0/24) y VLAN 20 (192.168.20.0/24).

#### 3.3.2 Configurar OSPF y parámetros de interfaces
```
configure terminal
!
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
!
! Parámetros OSPF en interfaces P2P
interface gig0/0
 ip ospf network point-to-point
 ip ospf cost 10
exit
!
interface gig0/1/0
 ip ospf network point-to-point
 ip ospf cost 10
exit
!
! Parámetros OSPF en backup
interface gig0/2.1000
 ip ospf priority 50
 ip ospf cost 50
exit
exit
```

> **📝 Nota:** El comando `ip ospf network broadcast` no es necesario en Packet Tracer para subinterfaces conectadas a un switch. OSPF detecta automáticamente el tipo de red como broadcast.

**Guardar configuración:**
```
write memory
```

---

### 3.4 Router MENDOZA – Interfaces P2P y Backup

**Dispositivo:** `Router MENDOZA`

#### 3.4.1 Configurar interfaces físicas y subinterfaces
```
configure terminal
!
! Enlaces P2P (IP directa en interfaz física)
interface gig0/0/0
 description P2P_to_BSAS
 ip address 10.10.1.2 255.255.255.252
 no shutdown
exit
!
interface gig0/1/0
 description P2P_to_CORDOBA
 ip address 10.10.1.18 255.255.255.252
 no shutdown
exit
!
! Interfaz física para backup (trunk)
interface gig0/1
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 no shutdown
exit
!
! Subinterfaz backup VLAN 1000
interface gig0/1.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.3 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit
!
! Interfaz física para LANs locales
interface gig0/2
 description Trunk_to_SW-CORE-DIS-MEND
 no ip address
 no shutdown
exit
!
! Subinterfaces LANs
interface gig0/2.44
 encapsulation dot1Q 44
 ip address 192.168.44.1 255.255.255.0
 description WiFi_INTERNOS_VLAN44
exit
!
interface gig0/2.55
 encapsulation dot1Q 55
 ip address 192.168.55.1 255.255.255.0
 description WiFi_INVITADOS_VLAN55
exit
!
interface gig0/2.70
 encapsulation dot1Q 70
 ip address 192.168.70.1 255.255.255.0
 description MANAGEMENT_VLAN70
exit
exit
```

#### 3.4.2 Configurar OSPF y parámetros de interfaces
```
configure terminal
!
! Proceso OSPF
router ospf 1
 router-id 3.3.3.3
 log-adjacency-changes
 passive-interface gig0/2.44
 passive-interface gig0/2.55
 passive-interface gig0/2.70
 network 192.168.44.0 0.0.0.255 area 0
 network 192.168.55.0 0.0.0.255 area 0
 network 192.168.70.0 0.0.0.255 area 0
 network 10.10.1.0 0.0.0.3 area 0
 network 10.10.1.16 0.0.0.3 area 0
 network 172.20.10.0 0.0.0.7 area 0
exit
!
! Parámetros OSPF en interfaces P2P
interface gig0/0/0
 ip ospf network point-to-point
 ip ospf cost 10
exit
!
interface gig0/1/0
 ip ospf network point-to-point
 ip ospf cost 10
exit
!
! Parámetros OSPF en backup
interface gig0/1.1000
 ip ospf priority 50
 ip ospf cost 50
exit
end
write memory
```

> **📝 Nota:** El comando `ip ospf network broadcast` no es necesario en Packet Tracer para subinterfaces conectadas a un switch. OSPF detecta automáticamente el tipo de red como broadcast.

---

## 4. Verificaciones finales

### 4.1 Verificar vecindades OSPF ⚠️ CRÍTICO
**En cada router debe haber EXACTAMENTE 4 vecinos OSPF:**
```
show ip ospf neighbor
```

**✅ Resultado esperado en BS.AS:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2          0    FULL/  -        00:00:3x    10.10.1.10      GigabitEthernet0/0/0
3.3.3.3          0    FULL/  -        00:00:3x    10.10.1.2       GigabitEthernet0/1/0
2.2.2.2         50    FULL/BDR        00:00:3x    172.20.10.2     GigabitEthernet0/2.1000
3.3.3.3         50    FULL/DROTHER    00:00:3x    172.20.10.3     GigabitEthernet0/2.1000
```
→ **4 vecinos**: 2 por P2P (estado FULL/ -) + 2 por VLAN1000 (estado FULL/DR o FULL/BDR)

**✅ Resultado esperado en CÓRDOBA:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          0    FULL/  -        00:00:3x    10.10.1.9       GigabitEthernet0/0/0
3.3.3.3          0    FULL/  -        00:00:3x    10.10.1.18      GigabitEthernet0/2
1.1.1.1        100    FULL/DR         00:00:3x    172.20.10.1     FastEthernet0/24.1000
3.3.3.3         50    FULL/DROTHER    00:00:3x    172.20.10.3     FastEthernet0/24.1000
```
→ **4 vecinos**: 2 por P2P + 2 por VLAN1000

**✅ Resultado esperado en MENDOZA:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          0    FULL/  -        00:00:3x    10.10.1.1       GigabitEthernet0/0/0
2.2.2.2          0    FULL/  -        00:00:3x    10.10.1.17      GigabitEthernet0/1/0
1.1.1.1        100    FULL/DR         00:00:3x    172.20.10.1     FastEthernet0/23.1000
2.2.2.2         50    FULL/BDR        00:00:3x    172.20.10.2     FastEthernet0/23.1000
```
→ **4 vecinos**: 2 por P2P + 2 por VLAN1000

**⚠️ Si NO ves 4 vecinos:**
1. Verificar que todas las interfaces estén **up/up**: `show ip interface brief`
2. Verificar tipo de red OSPF: `show ip ospf interface` (debe ser point-to-point o broadcast según corresponda)
3. Verificar que todas las redes estén anunciadas en `router ospf 1`: `show running-config | section ospf`
4. Verificar conectividad L3: hacer ping entre las IPs de las interfaces antes de OSPF

### 4.2 Verificar tabla de ruteo
**En cada router:**
```
show ip route ospf
show ip route
```

Validar:
- Rutas OSPF (O) hacia todas las LAN remotas
- Default route (O*E2) propagada desde BS.AS a CORDOBA y MENDOZA

### 4.3 Pruebas de conectividad
**Desde Router BS.AS:**
```
ping 192.168.10.1   (Gateway VLAN 10 Córdoba)
ping 192.168.20.1   (Gateway VLAN 20 Córdoba)
ping 192.168.44.1   (Gateway VLAN 44 Mendoza)
ping 192.168.55.1   (Gateway VLAN 55 Mendoza)
ping 192.168.70.1   (Gateway VLAN 70 Mendoza)
```

**Desde PC-BS-AS (una vez que switches locales estén configurados):**
```
ping 192.168.10.10   (PC2-VLAN10 en Córdoba)
ping 192.168.44.x     (Laptop WiFi Mendoza)
```

### 4.4 Validar costos y path único
```
show ip ospf interface brief
show ip route <red_remota>
```

Confirmar que cada destino tiene un solo path activo (no múltiples ECMP).

### 4.5 Documentar evidencias
Capturar en cada router:
```
show running-config | section ospf
show ip ospf database
show ip protocols
```

Guardar en `GUIAS/ticket_trabajo_practico.md` con fecha y dispositivo.

---

## 5. Checklist para cerrar la Fase 3

### Configuración
- [ ] **Switch SW_OSPF_BACKUP:** VLAN 1000 creada, Fa0/22/23/24 en modo trunk, native VLAN 1000
- [ ] **Router BS.AS:** 
  - Gig0/0/0 (10.10.1.9/30) y Gig0/1/0 (10.10.1.1/30) configurados
  - Gig0/2.1000 (172.20.10.1/29) operativa
  - OSPF: network 192.168.30.0/24, 10.10.1.8/30, 10.10.1.0/30, 172.20.10.0/29
  - `default-information originate` presente
  - Passive-interface gig0/1.30
- [ ] **Router CÓRDOBA:**
  - Gig0/0/0 (10.10.1.10/30) y Gig0/2 (10.10.1.17/30) configurados
  - Gig0/0.1000 (172.20.10.2/29) operativa
  - Gig0/1.10 (192.168.10.1/24) y Gig0/1.20 (192.168.20.1/24) configuradas
  - OSPF: networks correctos, passive en .10 y .20
- [ ] **Router MENDOZA:**
  - Gig0/0/0 (10.10.1.2/30) y Gig0/1/0 (10.10.1.18/30) configurados
  - Gig0/1.1000 (172.20.10.3/29) operativa
  - Gig0/2.44/55/70 (192.168.44/55/70.1/24) configuradas
  - OSPF: networks correctos, passive en .44, .55 y .70

### Verificación
- [ ] **Vecindades OSPF:** 4 vecinos por router (2 P2P estado FULL/-, 2 Broadcast FULL/DR o FULL/BDR)
- [ ] **Rutas OSPF:** Todas las LANs remotas presentes como rutas O en cada router
- [ ] **Default route:** Córdoba y Mendoza muestran ruta O*E2 0.0.0.0/0 vía BS.AS
- [ ] **Costos:** P2P=10, Backup=50 verificados con `show ip ospf interface brief`
- [ ] **Pings exitosos:** Desde cada router hacia todos los gateways LAN remotos
- [ ] **Failover test:** Al apagar un P2P, el tráfico se redirige correctamente
- [ ] **Evidencias:** Capturas de `show ip ospf neighbor`, `show ip route`, `show ip ospf interface brief` guardadas en ticket

---

## 6. Validación Post-Implementación

### 6.1 Checklist de Validación en Packet Tracer
Una vez aplicada la configuración, verificar los siguientes puntos críticos:

#### Switch SW_OSPF_BACKUP
```
show interfaces trunk
```
✅ Esperado: Fa0/22, Fa0/23, Fa0/24 como trunk con VLAN 1000 allowed

#### OSPF - Vecindades (en cada router) ⚠️ CRÍTICO
```
show ip ospf neighbor
show ip ospf interface brief
```
✅ Esperado: **EXACTAMENTE 4 vecinos por router**

**BS.AS debe ver:**
- 2.2.2.2 en Gig0/0/0 (P2P a CÓRDOBA) → estado **FULL/ -**
- 3.3.3.3 en Gig0/1/0 (P2P a MENDOZA) → estado **FULL/ -**
- 2.2.2.2 en Gig0/2.1000 (VLAN1000) → estado **FULL/DR** o **FULL/BDR**
- 3.3.3.3 en Gig0/2.1000 (VLAN1000) → estado **FULL/DR** o **FULL/BDR** o **FULL/DROTHER**

**CÓRDOBA debe ver:**
- 1.1.1.1 en Gig0/0 (P2P a BS.AS) → estado **FULL/ -**
- 3.3.3.3 en Gig0/1/0 (P2P a MENDOZA) → estado **FULL/ -**
- 1.1.1.1 en Gig0/2.1000 (VLAN1000) → estado **FULL/DR** o **FULL/BDR**
- 3.3.3.3 en Gig0/2.1000 (VLAN1000) → estado **FULL/DR** o **FULL/BDR** o **FULL/DROTHER**

**MENDOZA debe ver:**
- 1.1.1.1 en Gig0/0/0 (P2P a BS.AS) → estado **FULL/ -**
- 2.2.2.2 en Gig0/1/0 (P2P a CÓRDOBA) → estado **FULL/ -**
- 1.1.1.1 en Gig0/1.1000 (VLAN1000) → estado **FULL/DR** o **FULL/BDR**
- 2.2.2.2 en Gig0/1.1000 (VLAN1000) → estado **FULL/DR** o **FULL/BDR** o **FULL/DROTHER**

**⚠️ Si algún vecino falta:**
- Verificar `show ip interface brief` → todas las interfaces deben estar **up/up**
- Verificar `show ip ospf interface <nombre>` → confirmar tipo de red y que esté en OSPF
- Verificar que la red de esa interfaz esté en `router ospf 1` con el comando `show running-config | section ospf`

#### OSPF - Tabla de Ruteo (en cada router)
```
show ip route ospf
```
✅ Esperado en **CÓRDOBA y MENDOZA:**
- Rutas **O** (intra-área) a todas las LAN remotas
- Ruta **O*E2** (default route externa) aprendida desde BS.AS

✅ Esperado en **BS.AS:**
- Rutas **O** a las LANs de Córdoba y Mendoza
- **NO** debe aprender default route (él la origina)

#### OSPF - Tipos de red y costos
```
show ip ospf interface brief
```
✅ Esperado:
- **P2P enlaces:** Gig0/0/0, Gig0/1/0, Gig0/2 → tipo **P2P**, cost **10**
- **Backup VLAN1000:** subinterfaces .1000 → tipo **BROADCAST**, cost **50**
- **LANs:** subinterfaces LAN → tipo **BROADCAST**, estado **PASSIVE** (no envían hellos)

#### Failover Test
```
! En Router CÓRDOBA
interface gig0/0/0
 shutdown
! Verificar en BS.AS
show ip route 192.168.10.0
```
✅ Esperado: El tráfico debe enrutarse por:
1. **Primer fallback:** P2P alternativo (vía Mendoza)
2. **Segundo fallback:** VLAN 1000 (cost 50)

```
! Restaurar
interface gig0/0/0
 no shutdown
```

### 6.2 Resumen de Consistencia Verificada

| Aspecto | Estado | Notas |
|---------|--------|-------|
| **Cableado Switch↔Routers** | ✅ | Fa0/22→Gig0/2 (BS.AS), Fa0/24→Gig0/2 (CBA), Fa0/23→Gig0/1 (MDZ) |
| **Direccionamiento P2P** | ✅ | Tres redes /30: 10.10.1.8, 10.10.1.0, 10.10.1.16 sin overlaps |
| **Direccionamiento LANs** | ✅ | Sin conflictos: BS.AS=30.x, CBA=10.x/20.x, MDZ=44.x/55.x/70.x |
| **VLAN 1000 Backup** | ✅ | 172.20.10.0/29 con .1/.2/.3, subinterfaces .1000 correctas |
| **OSPF Networks** | ✅ | Todos los networks con wildcard correctos (0.0.0.3 P2P, 0.0.0.255 LANs) |
| **OSPF Passive Interfaces** | ✅ | Solo LANs pasivas, P2P y backup activos |
| **Default Route** | ✅ | Solo BS.AS origina con default-information originate |
| **Costos OSPF** | ✅ | P2P=10 (preferido), Backup=50 (secundario) |
| **Prioridades OSPF** | ✅ | BS.AS priority 100 (DR preferido), otros priority 50 |

---

## 7. Troubleshooting: "No tengo 4 vecinos por router"

### 7.1 Diagnóstico sistemático

**Paso 1: Verificar conectividad L2/L3 básica**
```
show ip interface brief
```
✅ TODAS las interfaces P2P y backup deben estar **up/up**. Si alguna está down:
- Verificar cables en Packet Tracer (modo Realtime o Simulation)
- Verificar `no shutdown` en ambos extremos
- Verificar IPs configuradas correctamente

**Paso 2: Verificar que OSPF esté activo en las interfaces**
```
show ip ospf interface brief
```
✅ Debes ver 3 interfaces por router con OSPF activo:
- **BS.AS:** Gig0/0/0, Gig0/1/0, Gig0/2.1000
- **CÓRDOBA:** Gig0/0/0, Gig0/2, Fa0/24.1000
- **MENDOZA:** Gig0/0/0, Gig0/1/0, Fa0/23.1000

Si alguna interface NO aparece:
```
show running-config | section ospf
```
→ Verificar que la red de esa interfaz esté en algún `network` del `router ospf 1`

**Paso 3: Verificar tipo de red OSPF**
```
show ip ospf interface gig0/0/0
show ip ospf interface gig0/2.1000
```
✅ Esperado:
- **Interfaces P2P:** `Network Type POINT_TO_POINT` (debe aparecer sin elección DR/BDR)
- **Subinterfaces .1000:** `Network Type BROADCAST` (con elección DR/BDR)

Si el tipo es incorrecto, agregar:
```
interface <nombre>
 ip ospf network point-to-point    ! Solo para P2P
 ! o
 ip ospf network broadcast          ! Solo para VLAN1000
```

### 7.2 Problemas específicos comunes

#### Problema: "Solo veo 2 vecinos en CÓRDOBA"
**Síntoma:**
```
Router CORDOBA# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          0    FULL/  -        00:00:3x    10.10.1.9       GigabitEthernet0/0/0
1.1.1.1        100    FULL/DR         00:00:3x    172.20.10.1     FastEthernet0/24.1000
```
→ Faltan los 2 vecinos con MENDOZA (uno por P2P, otro por VLAN1000)

**Causa más probable:** OSPF no está configurado en `Gig0/1/0` (enlace P2P a MENDOZA)

**Solución:**
```
conf t
interface gig0/1/0
 ip ospf network point-to-point
 ip ospf cost 10
!
router ospf 1
 network 10.10.1.16 0.0.0.3 area 0
end
```

#### Problema: "Solo veo 3 vecinos en MENDOZA"
**Síntoma:**
```
Router MENDOZA# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1          0    FULL/  -        00:00:3x    10.10.1.1       GigabitEthernet0/0/0
1.1.1.1        100    FULL/DR         00:00:3x    172.20.10.1     FastEthernet0/23.1000
2.2.2.2         50    FULL/BDR        00:00:3x    172.20.10.2     FastEthernet0/23.1000
```
→ Falta vecino con CÓRDOBA por P2P

**Causa más probable:** Interfaz `Gig0/1/0` está apagada o sin OSPF

**Solución:**
```
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

#### Problema: "El ping funciona pero no veo el vecino OSPF"
**Explicación:** Esto pasa cuando hay rutas alternativas (por ejemplo, si CÓRDOBA↔MENDOZA no forman vecindad P2P, pero ambos tienen vecindad con BS.AS, el ping funciona usando BS.AS como intermediario).

**Validación correcta:**
```
show ip ospf neighbor    ! Debe mostrar 4 vecinos
show ip route 10.10.1.16 ! Ver por qué path se alcanza la red remota
traceroute 10.10.1.18    ! Ver los saltos intermedios
```

### 7.3 Checklist final de validación

| Verificación | Comando | Resultado Esperado |
|--------------|---------|-------------------|
| ✅ Interfaces up | `show ip int brief` | Todas las interfaces OSPF en **up/up** |
| ✅ OSPF activo | `show ip ospf int brief` | 3 interfaces por router (2 P2P + 1 backup) |
| ✅ Tipo de red P2P | `show ip ospf int gig0/0/0` | Network Type **POINT_TO_POINT**, Cost 10 |
| ✅ Tipo de red backup | `show ip ospf int fa0/24.1000` | Network Type **BROADCAST**, Cost 50 |
| ✅ 4 vecinos totales | `show ip ospf neighbor` | 4 líneas (2 estado FULL/-, 2 estado FULL/DR o BDR) |
| ✅ Rutas OSPF | `show ip route ospf` | Todas las LANs remotas visibles |
| ✅ Default route | `show ip route` (en CBA/MDZ) | Ruta **O*E2** 0.0.0.0/0 vía BS.AS |

---

## 8. Próximos pasos
1. **Aplicar configuración en Packet Tracer** siguiendo secciones 3.1 a 3.4
2. **Ejecutar validaciones** de la sección 4 y 6
3. **Troubleshooting si es necesario** usando la sección 7
4. Documentar salidas de `show ip ospf neighbor` y `show ip route` en el ticket
5. Actualizar `GUIAS/00 - plan_trabajo_general.md` marcando Fase 3 como finalizada
6. Continuar con Fase 4: switches de acceso/core en cada sitio (STP, VLANs), WiFi en Mendoza
7. Fase 5: NAT estático para servidores, pruebas end-to-end, ACL FTP completar

> **✅ Nota:** El conflicto de direccionamiento fue resuelto moviendo Buenos Aires a VLAN 30 (192.168.30.0/24). Córdoba mantiene sus VLANs originales 10 y 20. Ver `CAMBIOS_CRITICOS_PROFESOR.md` para detalles completos.

> Cualquier ajuste adicional deberá reflejarse en `CONFIG POR DISP` y documentarse en el ticket antes de avanzar a la siguiente fase.
