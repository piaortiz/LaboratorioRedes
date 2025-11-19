# Guía 03 · Enlaces P2P y OSPF Multi-Sitio (BS.AS ↔ CORDOBA ↔ MENDOZA)
**Actualización:** 17/11/2025  
**Objetivo:** Configurar la conectividad entre los tres sitios mediante enlaces P2P de fibra y red de backup VLAN 1000, implementar OSPF área única con propagación de redes locales, manipular costos para path único y habilitar `default-information originate` desde Router BS.AS según requerimientos del profesor.

---

## 0. Referencias y dependencias
- `GUIAS/01 - guia_segmento_wan.md`: ISP_LOCAL e ISP_INTERNACIONAL operativos.
- `GUIAS/02 - guia_segmento_bsas.md`: LAN Buenos Aires (VLAN 20) funcional.
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
| P2P-1 | BS.AS | Gig0/0/0 | 10.10.1.9/30 | CORDOBA | Gig0/0/0 | 10.10.1.10/30 | 10.10.1.8/30 |
| P2P-2 | BS.AS | Gig0/1/0 | 10.10.1.1/30 | MENDOZA | Gig0/0/0 | 10.10.1.2/30 | 10.10.1.0/30 |
| P2P-3 | CORDOBA | Gig0/2 | 10.10.1.17/30 | MENDOZA | Gig0/1/0 | 10.10.1.18/30 | 10.10.1.16/30 |

> **⚠️ CRÍTICO:** Configurar IPs **directamente en las interfaces físicas** (NO usar subinterfaces .500). Esta es una corrección del profesor para que OSPF point-to-point funcione correctamente en Packet Tracer.

### 1.2 Red de Backup (Broadcast - VLAN 1000)
Switch `SW_OSPF_BACKUP` conecta los tres routers mediante VLAN 1000:

| Router | Interfaz física | Subinterfaz | IP | VLAN | Red |
|--------|----------------|-------------|-----|------|-----|
| BS.AS | Fa0/22 (hacia switch) | Fa0/22.1000 | 172.20.10.1/29 | 1000 | 172.20.10.0/29 |
| CORDOBA | Fa0/24 | Fa0/24.1000 | 172.20.10.2/29 | 1000 | 172.20.10.0/29 |
| MENDOZA | Fa0/23 | Fa0/23.1000 | 172.20.10.3/29 | 1000 | 172.20.10.0/29 |

> **✅ IMPORTANTE:** Esta configuración SÍ usa subinterfaz .1000 (Router-on-a-Stick) porque es un enlace compartido en switch. NO confundir con los enlaces P2P de fibra que van directo en interfaces físicas. Los requerimientos especifican: "NO CONFIGURAR IP EN LAS INTERFACES FÍSICAS DE DICHO ENLACE" para SW_OSPF_BACKUP.

**Direccionamiento sugerido VLAN 1000:**
- BS.AS: 172.20.10.1/29
- CORDOBA: 172.20.10.2/29
- MENDOZA: 172.20.10.3/29

### 1.3 Redes LAN a propagar por OSPF
| Sitio | VLAN | Red | Gateway | Estado |
|-------|------|-----|---------|--------|
| BS.AS | **30** ⚠️ | **192.168.30.0/24** | 192.168.30.1 (G0/1.30) | ⚠️ Requiere reconfig (Fase 2) |
| CORDOBA | 10 | 192.168.10.0/24 | 192.168.10.1 | ⏳ Pendiente config |
| CORDOBA | 20 | 192.168.20.0/24 | 192.168.20.1 | ✅ Sin conflicto ahora |
| MENDOZA | 44 | 192.168.44.0/24 | 192.168.44.1 | ⏳ Pendiente config |
| MENDOZA | 55 | 192.168.55.0/24 | 192.168.55.1 | ⏳ Pendiente config |
| MENDOZA | 70 | 192.168.70.0/24 | 192.168.70.1 | ⏳ Pendiente config |

> **✅ Conflicto resuelto:** Según `NUEVOSREQUERIMIENTOS`, BS.AS ahora usa **VLAN 30** con red **192.168.30.0/24**, eliminando el conflicto con Córdoba VLAN 20. Ver `CAMBIOS_CRITICOS_PROFESOR.md` para detalles completos.

---

## 2. Prerrequisitos
1. **⚠️ CRÍTICO:** Reconfigurar Fase 2 con VLAN 30 antes de continuar (ver guía 02 actualizada).
2. Fases 1 y 2 actualizadas: ISP_LOCAL operativo con rutas a 192.168.30.0/24, Router BS.AS con LAN en VLAN 30.
3. Cableado físico verificado según imagen:
   - Enlaces P2P fibra conectados (Gig0/0/0, Gig0/1/0, Gig0/2).
   - Cables al `SW_OSPF_BACKUP` desde Fa0/22 (BS.AS), Fa0/24 (CORDOBA), Fa0/23 (MENDOZA).
4. Acceso por consola/SSH a los tres routers y al switch de backup.
5. Snapshot previo de configuraciones en `CONFIG POR DISP`.

---

## 3. Paso a paso

### 3.1 Switch `SW_OSPF_BACKUP` – Red Backup VLAN 1000
**Dispositivo:** `SW_OSPF_BACKUP`

```
enable
conf t
hostname SW_OSPF_BACKUP
!
vlan 1000
 name BACKUP_OSPF
!
interface fa0/22
 description To_Router_BSAS_Fa0/22
 switchport mode access
 switchport access vlan 1000
!
interface fa0/24
 description To_Router_CORDOBA_Fa0/24
 switchport mode access
 switchport access vlan 1000
!
interface fa0/23
 description To_Router_MENDOZA_Fa0/23
 switchport mode access
 switchport access vlan 1000
!
spanning-tree vlan 1000 priority 4096
end
wr mem
```

Validaciones:
```
show vlan brief
show interfaces status
```

---

### 3.2 Router BS.AS – Interfaces P2P y Backup

**Dispositivo:** `Router BS.AS`

#### 3.2.1 Configurar enlaces P2P (IP directa)
```
conf t
!
interface gig0/0/0
 description P2P_to_CORDOBA
 ip address 10.10.1.9 255.255.255.252
 no shutdown
!
interface gig0/1/0
 description P2P_to_MENDOZA
 ip address 10.10.1.1 255.255.255.252
 no shutdown
!
exit
```

#### 3.2.2 Configurar subinterfaz backup VLAN 1000
```
conf t
interface fa0/22
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 no shutdown
!
interface fa0/22.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.1 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit
```

#### 3.2.3 Configurar OSPF
```
conf t
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
interface gig0/0/0
 ip ospf network point-to-point
 ip ospf cost 10
!
interface gig0/1/0
 ip ospf network point-to-point
 ip ospf cost 10
!
interface fa0/22.1000
 ip ospf network broadcast
 ip ospf priority 100
 ip ospf cost 50
end
wr mem
```

> **⚠️ CAMBIO:** Red BS.AS es 192.168.30.0/24 (VLAN 30), interfaz pasiva es G0/1.30

> **Costos:** P2P con costo 10 (preferidos), backup con costo 50 (secundario). Ajustar según necesidad de balanceo.

---

### 3.3 Router CORDOBA – Interfaces P2P y Backup

**Dispositivo:** `Router CORDOBA`

#### 3.3.1 Configurar enlaces P2P
```
conf t
!
interface gig0/0/0
 description P2P_to_BSAS
 ip address 10.10.1.10 255.255.255.252
 no shutdown
!
interface gig0/2
 description P2P_to_MENDOZA
 ip address 10.10.1.17 255.255.255.252
 no shutdown
!
exit
```

#### 3.3.2 Configurar subinterfaz backup VLAN 1000
```
conf t
interface fa0/24
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 no shutdown
!
interface fa0/24.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.2 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit
```

#### 3.3.3 Configurar LAN Córdoba (ajuste previo conflicto VLAN)
> **Acción previa:** Cambiar VLAN 20 a VLAN 30 (192.168.30.0/24) para evitar overlap con BS.AS.

```
conf t
interface gig0/1
 no ip address
 no shutdown
!
interface gig0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 description LAN_CORDOBA_VLAN10
!
interface gig0/1.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 description LAN_CORDOBA_VLAN30
exit
```

#### 3.3.4 Configurar OSPF
```
conf t
router ospf 1
 router-id 2.2.2.2
 log-adjacency-changes
 passive-interface gig0/1.10
 passive-interface gig0/1.30
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 10.10.1.8 0.0.0.3 area 0
 network 10.10.1.16 0.0.0.3 area 0
 network 172.20.10.0 0.0.0.7 area 0
exit
!
interface gig0/0/0
 ip ospf network point-to-point
 ip ospf cost 10
!
interface gig0/2
 ip ospf network point-to-point
 ip ospf cost 10
!
interface fa0/24.1000
 ip ospf network broadcast
 ip ospf priority 50
 ip ospf cost 50
end
wr mem
```

---

### 3.4 Router MENDOZA – Interfaces P2P y Backup

**Dispositivo:** `Router MENDOZA`

#### 3.4.1 Configurar enlaces P2P
```
conf t
!
interface gig0/0/0
 description P2P_to_BSAS
 ip address 10.10.1.2 255.255.255.252
 no shutdown
!
interface gig0/1/0
 description P2P_to_CORDOBA
 ip address 10.10.1.18 255.255.255.252
 no shutdown
!
exit
```

#### 3.4.2 Configurar subinterfaz backup VLAN 1000
```
conf t
interface fa0/23
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 no shutdown
!
interface fa0/23.1000
 encapsulation dot1Q 1000
 ip address 172.20.10.3 255.255.255.248
 description BACKUP_OSPF_VLAN1000
 no shutdown
exit
```

#### 3.4.3 Configurar LAN Mendoza (VLANs 44/55/70)
```
conf t
interface gig0/2
 no ip address
 no shutdown
!
interface gig0/2.44
 encapsulation dot1Q 44
 ip address 192.168.44.1 255.255.255.0
 description WiFi_INTERNOS_VLAN44
!
interface gig0/2.55
 encapsulation dot1Q 55
 ip address 192.168.55.1 255.255.255.0
 description WiFi_INVITADOS_VLAN55
!
interface gig0/2.70
 encapsulation dot1Q 70
 ip address 192.168.70.1 255.255.255.0
 description MANAGEMENT_VLAN70
exit
```

#### 3.4.4 Configurar OSPF
```
conf t
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
interface gig0/0/0
 ip ospf network point-to-point
 ip ospf cost 10
!
interface gig0/1/0
 ip ospf network point-to-point
 ip ospf cost 10
!
interface fa0/23.1000
 ip ospf network broadcast
 ip ospf priority 50
 ip ospf cost 50
end
wr mem
```

---

## 4. Verificaciones finales

### 4.1 Verificar vecindades OSPF
**En cada router:**
```
show ip ospf neighbor
```

Resultado esperado:
- BS.AS: 4 vecinos (CORDOBA y MENDOZA por P2P x2, ambos por VLAN 1000)
- CORDOBA: 4 vecinos (BS.AS y MENDOZA por P2P x2, ambos por VLAN 1000)
- MENDOZA: 4 vecinos (BS.AS y CORDOBA por P2P x2, ambos por VLAN 1000)

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
ping 192.168.10.1
ping 192.168.30.1
ping 192.168.44.1
ping 192.168.55.1
ping 192.168.70.1
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
- [ ] Switch SW_OSPF_BACKUP configurado con VLAN 1000 y puertos asignados.
- [ ] Router BS.AS: enlaces P2P (Gig0/0/0, Gig0/1/0) operativos, subinterfaz Fa0/22.1000 up/up, OSPF activo, `default-information originate` configurado.
- [ ] Router CORDOBA: enlaces P2P operativos, LAN VLAN 10/30 configuradas, OSPF propagando redes.
- [ ] Router MENDOZA: enlaces P2P operativos, LAN VLAN 44/55/70 configuradas, OSPF propagando redes.
- [ ] Vecindades OSPF: 4 vecinos por router (2 P2P + 2 broadcast VLAN 1000).
- [ ] Rutas OSPF presentes en todos los routers hacia LAN remotas.
- [ ] Default route propagada a CORDOBA y MENDOZA.
- [ ] Pings cruzados exitosos entre routers.
- [ ] Costos configurados para forzar path único por destino.
- [ ] Evidencias documentadas en ticket.

---

## 6. Próximos pasos
1. Validar con el docente que las vecindades y rutas cumplen los requerimientos.
2. Actualizar `GUIAS/00 - plan_trabajo_general.md` marcando Fase 3 como finalizada.
3. Continuar con Fase 4: configurar switches de acceso/core en cada sitio (STP, VLANs), WiFi en Mendoza.
4. Fase 5: NAT estático para servidores, pruebas end-to-end, ACL FTP completar.

> **Nota crítica:** Antes de propagar rutas OSPF, **resolver conflicto VLAN 20 Córdoba**. Cambiar a VLAN 30 (192.168.30.0/24) o coordinar NAT con el profesor según `notasprofeso`.

> Cualquier ajuste adicional deberá reflejarse en `CONFIG POR DISP` y documentarse en el ticket antes de avanzar a la siguiente fase.
