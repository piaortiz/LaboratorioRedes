# Guía Operativa Segmento Buenos Aires ↔ ISP_LOCAL
**Fecha:** 17/11/2025  
**Objetivo:** Dejar operativa la conexión LAN de Buenos Aires (PC-BS-AS) hacia ISP_LOCAL y la nube de Internet, con doble enlace WAN y NAT (Requerimientos "Ruteo estático #1" y "NAT Buenos Aires #1" en `reque.md`).

---

## 1. Visión General
- **Topología:** PC-BS-AS → Switch 2960 SW-BS-AS → Router BS.AS → (WAN1/WAN2) → Router ISP_LOCAL → Internet (164.25.0.0/29) (Interfaces #2 y Nota del profesor sobre enlaces directos).
- **LAN objetivo:** VLAN 20 (192.168.20.0/24) para usuarios de Buenos Aires (Interfaces #2).
- **WANs:**
  - WAN1 VLAN 100 → 42.25.25.0/29 (BS.AS = .1, ISP_LOCAL = .2).
  - WAN2 VLAN 200 → 43.26.26.0/29 (BS.AS = .1, ISP_LOCAL = .2).
- **Servicios requeridos:** NAT/PAT en Router BS.AS, rutas de retorno en ISP_LOCAL, conectividad hasta Internet cloud.

---

## 2. Preparación de la LAN BS.AS
### 2.1 Configuración PC-BS-AS
1. Asignar IP estática `192.168.20.10`, máscara `255.255.255.0` (Interfaces #2).
2. Puerta de enlace: `192.168.20.1` (subinterfaz 20 del Router BS.AS) (Interfaces #2).
3. DNS temporal: `1.1.1.1` (permite cumplir NAT/Internet de NAT Buenos Aires #1).

### 2.2 Switch 2960 SW-BS-AS
> Puerto Fa0/1 hacia PC, Fa0/24 hacia router (Interfaces #2, STP #1).
```
conf t
vlan 20
 name LAN_BSAS
interface fa0/1
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
interface fa0/24
 description Link-to-Router
 switchport mode trunk
 switchport trunk allowed vlan 20,100,200
 switchport trunk native vlan 20
end
wr mem
show vlan brief
show interfaces trunk
```

---

## 3. Router BS.AS (Dual-WAN + NAT)
Asume `g0/0` hacia el switch (Router-on-a-Stick), `g0/1` WAN1, `g0/2` WAN2 (Interfaces #2 y Nota del profesor: usar interfaces físicas en enlaces directos para OSPF P2P).
```
conf t
hostname R-BSAS
!
interface g0/0
 no shutdown
interface g0/0.20
 encapsulation dot1q 20
 ip address 192.168.20.1 255.255.255.0
 description LAN_BSAS_VLAN20
 ip nat inside
!
interface g0/1
 description WAN1_to_ISP_LOCAL
 ip address 42.25.25.1 255.255.255.248
 no shutdown
 ip nat outside
!
interface g0/2
 description WAN2_to_ISP_LOCAL
 ip address 43.26.26.1 255.255.255.248
 no shutdown
 ip nat outside
!
ip access-list standard LAN_BSAS
 permit 192.168.20.0 0.0.0.255
ip nat inside source list LAN_BSAS interface g0/1 overload
ip nat inside source list LAN_BSAS interface g0/2 overload
!
! Rutas por defecto (puedes hacer floating si lo deseas)
ip route 0.0.0.0 0.0.0.0 42.25.25.2
ip route 0.0.0.0 0.0.0.0 43.26.26.2
end
wr mem
```

(Las dos rutas por defecto satisfacen "Ruteo estático #1" y los comandos de NAT cumplen "NAT Buenos Aires #1".)

**Comandos de verificación:**
```
show ip interface brief
show ip nat translations
show ip route
ping 42.25.25.2
ping 43.26.26.2
```

---

## 4. Router ISP_LOCAL
Interfaces sugeridas: `g0/0` a WAN1, `g0/1` a WAN2, `s0/0/0` a Internet (Interfaces #3).
```
conf t
hostname ISP_LOCAL
interface g0/0
 ip address 42.25.25.2 255.255.255.248
 no shutdown
interface g0/1
 ip address 43.26.26.2 255.255.255.248
 no shutdown
interface s0/0/0
 ip address 164.25.0.2 255.255.255.248
 clock rate 64000
 no shutdown
!
ip route 192.168.20.0 255.255.255.0 42.25.25.1
ip route 192.168.20.0 255.255.255.0 43.26.26.1
ip route 0.0.0.0 0.0.0.0 164.25.0.1
end
wr mem
```

(Rutas específicas cumplen "Ruteo estático #2" para comunicación entre ISP_LOCAL e ISP_INTERNACIONAL.)

**Validación:**
```
show ip route
ping 42.25.25.1
ping 43.26.26.1
ping 192.168.20.10
ping 164.25.0.1
```

---

## 5. Internet Cloud (simulada)
- Configurar la interfaz lado ISP con `IP 164.25.0.1 255.255.255.248` (Interfaces #3).
- Añadir ruta de retorno `192.168.20.0/24` hacia `164.25.0.2` si el dispositivo lo permite (Ruteo estático #2).

---

## 6. Plan de Pruebas
1. **LAN:** `ping 192.168.20.1` desde PC-BS-AS (Interfaces #2).
2. **WAN:** Desde R-BSAS `ping 42.25.25.2` y `ping 43.26.26.2` (Interfaces #3 y Nota del profesor en enlaces directos).
3. **Internet:** Desde PC ejecutar `ping 8.8.8.8` y `tracert 8.8.8.8` (Ruteo estático #1 + NAT Buenos Aires #1).
4. **NAT:** `show ip nat translations` debe mostrar entradas dinámicas (NAT Buenos Aires #1).
5. **Redundancia:** Apagar temporalmente interfaz g0/1 para verificar salida por g0/2 (Ruteo estático #1).

---

## 7. Bitácora y Evidencias
- Guardar capturas de `show running-config`, `show ip route`, `show interfaces trunk` (STP #1 y Ruteo estático #1).
- Registrar cualquier desviación en `ticket_trabajo_practico.md` bajo Fase 1 / segmento BA (trazabilidad requerida por el profesor).

> **Nota:** Si la topología evoluciona a 192.168.0.0/24 (como en otras capturas), mantener consistencia en todos los dispositivos antes de activar NAT.
