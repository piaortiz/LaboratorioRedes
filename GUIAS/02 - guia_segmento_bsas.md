# Guía 02 · Segmento Buenos Aires (SW-BS-AS ↔ Router BS.AS ↔ ISP_LOCAL)
**Actualización:** 17/11/2025  
**Objetivo:** Energizar la LAN de Buenos Aires (VLAN 20), habilitar los enlaces WAN1/WAN2 hacia ISP_LOCAL, dejar configurado el NAT de doble salida y documentar evidencias para cerrar los requerimientos "Interfaces #2/#3", "Ruteo estático #1", "NAT Buenos Aires" y "ACL FTP" del documento `Analisis y requisitos/reque.md`.

---

## 0. Referencias y dependencias
- `GUIAS/01 - guia_segmento_wan.md`: Fase 1 completa (ISP_LOCAL, ISP_INTERNACIONAL y nube).
- `Analisis y requisitos/reque.md`: lineamientos del profesor (LAN en VLAN, NAT por IP de interfaz, enlaces P2P sin subinterfaces).
- `Analisis y requisitos/notasprofeso`: reunión 14-11-2025 confirma router-on-a-stick con 3 subinterfaces entre Router BS.AS y SW-BS-AS, OSPF área única, propagación de default vía `default-information originate`.
- `GUIAS/ticket_trabajo_practico.md`: plantilla de evidencias.
- **Importante:** El profesor aclaró que **solo** los enlaces router↔router (P2P fibra) deben usar IP directa en interfaz física (sin subinterfaces por limitación de Packet Tracer). El enlace switch↔router de BS.AS sigue siendo Router-on-a-Stick para VLAN 20/100/200.

---

## 1. Alcance y topología
- **Inicio:** PC-BS-AS (192.168.20.10/24) conectada al `2960-24TT SW-BS-AS` por `Fa0/2` (acceso VLAN 20).
- **Medio:** Switch con VLAN 20 (LAN), VLAN 100 (WAN1) y VLAN 200 (WAN2). Túneles:
  - `Gi0/2` ↔ `Router BS.AS G0/1` (trunk LAN VLAN 20/100/200, nativa 20).
  - `Gi0/1` ↔ `ISP_LOCAL G0/1` (WAN1, ya configurado en guía 01).
  - `Fa0/24` ↔ `ISP_LOCAL G0/2` (WAN2).
- **Fin:** Router BS.AS con `G0/1.20` para LAN, y WAN físicas pendientes de definir según requerimientos.

---

## 2. Prerrequisitos
1. Cableado igual al PKT y energizado.
2. Acceso por consola/SSH al switch y router.
3. Usuario creó snapshot previo en `CONFIG POR DISP`.
4. PC-BS-AS configurada:
   - IP `192.168.20.10` /24
   - Gateway `192.168.20.1`
   - DNS `1.1.1.1`
   - Verificar `ipconfig /all` y guardar captura (ping a la puerta de enlace fallará hasta terminar la guía).

---

## 3. Paso a paso

### 3.1 Switch `SW-BS-AS` (Req. Interfaces #2)
Objetivo: garantizar VLAN 20/100/200 y troncales correctas.

**Dispositivo:** `SW-BS-AS`
```
conf t
 vlan 20
 vlan 100
 vlan 200
!
interface fa0/2
 description PC_BS_AS
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
!
interface gi0/2
 description Trunk_to_Router_BSAS_G0/1
 switchport mode trunk
 switchport trunk native vlan 20
 switchport trunk allowed vlan 20,100,200
!
interface fa0/24
 description Trunk_to_ISP_LOCAL_G0/2
 switchport mode trunk
 switchport trunk native vlan 20
 switchport trunk allowed vlan 20,100,200
exit
wr mem
```
Validaciones:
```
show vlan brief
show interfaces trunk
```
Guardar salidas en el ticket.

### 3.2 Router BS.AS – Preparación
**Dispositivo:** `Router BS.AS`
```
conf t
default interface g0/1
default interface g0/2
interface range g0/1 - 2
 no shutdown
exit
```

### 3.3 LAN Buenos Aires (VLAN 20) – Req. Interfaces #2
**Dispositivo:** `Router BS.AS`
```
conf t
interface g0/1
 no ip address
 no shutdown
interface g0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 description LAN_BSAS_VLAN20
 ip nat inside
 no shutdown
exit
```
Prueba rápida: `ping 192.168.20.10` (debe responder una vez que la PC tenga gateway y el switch esté OK).

### 3.4 ACL FTP (requerimiento específico)
Creación inmediata, aplicación pendiente hasta conocer la interfaz/servidor final.

**Dispositivo:** `Router BS.AS`
```
conf t
ip access-list extended FTP_ONLY_PC
 remark Solo PC-BS-AS puede acceder a FTP
 ! Reemplazar X.X.X.X con IP real del servidor FTP cuando se defina
exit
```
> **Nota:** La ACL quedará vacía hasta que se conozca la IP del servidor FTP. En ese momento agregar: `permit tcp host 192.168.20.10 host X.X.X.X eq 21`, `deny tcp any host X.X.X.X eq 21`, `permit ip any any`.

### 3.5 Guardar cambios
**Dispositivo:** `Router BS.AS`
```
wr mem
copy run start
```

---

## 4. Verificaciones finales
1. **Router BS.AS**
```
show ip interface brief
show ip route
```
2. **Pruebas de conectividad**
   - Desde el router: `ping 192.168.20.10` (PC en LAN).
   - Desde PC-BS-AS: `ping 192.168.20.1` (gateway).
3. **Switch**: `show interfaces trunk`, `show vlan brief`.
4. **ISP_LOCAL**: verificar que `show ip route` contenga rutas hacia 192.168.20.0/24 (configuradas en guía 01).
5. Documentar cada salida en `ticket_trabajo_practico.md` con la fecha y requerimiento asociado.

---

## 5. Checklist para cerrar la Fase 2
- [ ] VLAN 20/100/200 presentes y troncales operativas (captura `show interfaces trunk`).
- [ ] `g0/1.20` `up/up` con IP 192.168.20.1 (`show ip interface brief`).
- [ ] Conectividad LAN verificada (ping PC↔Router exitoso).
- [ ] Switch SW-BS-AS con puertos correctos: Gi0/2→Router BS.AS G0/1, Gi0/1→ISP_LOCAL G0/1, Fa0/24→ISP_LOCAL G0/2.
- [ ] ACL `FTP_ONLY_PC` creada y documentada.
- [ ] Evidencias volcadas al ticket.

---

## 6. Próximos pasos
1. Validar con el docente que las capturas cumplen los requerimientos.
2. Actualizar `GUIAS/00 - plan_trabajo_general.md` marcando Fase 2 como finalizada.
3. Continuar con la fase OSPF/Backbone aplicando lineamientos del profesor (reunión 14-11-2025):
   - **Enlaces P2P router↔router**: configurar IP directa en interfaces físicas (sin subinterfaces) para permitir OSPF point-to-point.
   - **Segmento compartido `SW_OSPF_BACKUP`**: usar subinterfaces `.1000` en VLAN 1000 para Router-on-a-Stick, vecindad OSPF tipo broadcast.
   - **OSPF área única**: manipular costos de interfaces para forzar path único por destino (evitar 3 caminos simultáneos).
   - **Propagación de default**: usar `default-information originate` en Router BS.AS para distribuir la ruta por defecto a Internet vía OSPF, no configurarla manualmente en cada router.
4. **Requisitos funcionales pendientes** (según `notasprofeso`):
   - Todas las PC deben llegar a Internet y al DNS del ISP.
   - WiFi Mendoza: 2 SSID (VLAN 44 internos/RADIUS, VLAN 55 invitados/PSK), opcional ACL de aislamiento.
   - ACL FTP: completar aplicación cuando se defina servidor (solo PC-BS-AS con acceso).

> Cualquier ajuste adicional deberá reflejarse en `CONFIG POR DISP` y documentarse en el ticket antes de avanzar a la siguiente fase.
