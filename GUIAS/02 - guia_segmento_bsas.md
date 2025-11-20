# Guía 02 · Segmento Buenos Aires (SW-BS-AS ↔ Router BS.AS ↔ ISP_LOCAL)
**Actualización:** 19/11/2025 ⚠️ CAMBIO CRÍTICO: VLAN 30  
**Objetivo:** Energizar la LAN de Buenos Aires (**VLAN 30** según NUEVOSREQUERIMIENTOS), habilitar los enlaces WAN1/WAN2 hacia ISP_LOCAL, dejar configurado el NAT de doble salida y documentar evidencias para cerrar los requerimientos "Interfaces #2/#3", "Ruteo estático #1", "NAT Buenos Aires" y "ACL FTP" del documento `Analisis y requisitos/reque.md`.

---

## 0. Referencias y dependencias
- `GUIAS/01 - guia_segmento_wan.md`: Fase 1 completa (ISP_LOCAL, ISP_INTERNACIONAL y nube).
- `Analisis y requisitos/reque.md`: lineamientos del profesor (LAN en VLAN, NAT por IP de interfaz, enlaces P2P sin subinterfaces).
- `Analisis y requisitos/notasprofeso`: reunión 14-11-2025 confirma router-on-a-stick con 3 subinterfaces entre Router BS.AS y SW-BS-AS, OSPF área única, propagación de default vía `default-information originate`.
- `GUIAS/ticket_trabajo_practico.md`: plantilla de evidencias.

**Notas importantes:**
- El profesor aclaró que **solo** los enlaces router↔router (P2P fibra) deben usar IP directa en interfaz física (sin subinterfaces por limitación de Packet Tracer). El enlace switch↔router de BS.AS sigue siendo Router-on-a-Stick para VLAN 30/100/200.
- **⚠️ Configuración crítica:** Esta guía incluye la configuración de **rutas de retorno en ISP_INTERNACIONAL** (sección 3.6) hacia las redes WAN 42.25.25.0/29 y 43.26.26.0/29. Sin estas rutas, el tráfico NAT desde Buenos Aires llegará a los servidores e Internet, pero las respuesas se perderán. Aunque el profesor no lo especificó explícitamente, es la configuración correcta para una topología funcional completa.

---

## 1. Alcance y topología
- **Inicio:** PC-BS-AS (**192.168.30.10/24** ⚠️ CAMBIO) conectada al `2960-24TT SW-BS-AS` por `Fa0/2` (acceso **VLAN 30**).
- **Medio:** Switch con **VLAN 30** (LAN), VLAN 100 (WAN1) y VLAN 200 (WAN2). Enlaces:
  - `Gi0/2` ↔ `Router BS.AS G0/1` (trunk VLAN 30/100/200 para Router-on-a-Stick).
  - `Gi0/1` ↔ `ISP_LOCAL G0/1` (acceso VLAN 100, WAN1).
  - `Fa0/24` ↔ `ISP_LOCAL G0/2` (acceso VLAN 200, WAN2).
- **Fin:** Router BS.AS con `G0/1.30` para LAN, `G0/1.100` y `G0/1.200` para las dos salidas WAN.

**Redes a alcanzar:**
- LAN Buenos Aires: 192.168.30.0/24 (VLAN 30) ⚠️ CAMBIO
- Servidores: 192.168.100.0/29 (VLAN 100) - DNS-SERVER (.2) y WEB-SERVER (.9)
- WAN1: 42.25.25.0/29, WAN2: 43.26.26.0/29 (hacia ISP_LOCAL)
- Internet: 164.25.0.0/29 (ISP_INTERNACIONAL ↔ ISP_LOCAL)

**⚠️ Razón del cambio:** Evitar conflicto de red 192.168.20.0/24 con Córdoba VLAN 20.

---

## 2. Prerrequisitos
1. Cableado igual al PKT y energizado.
2. Acceso por consola/SSH al switch, Router BS.AS y Router ISP_INTERNACIONAL.
3. Usuario creó snapshot previo en `CONFIG POR DISP`.
4. PC-BS-AS configurada:
   - IP `192.168.30.10` /24 ⚠️ CAMBIO: antes era 192.168.20.10
   - Gateway `192.168.30.1` ⚠️ CAMBIO: antes era 192.168.20.1
   - DNS `1.1.1.1`
   - Verificar `ipconfig /all` y guardar captura (ping a la puerta de enlace fallará hasta terminar la guía).

---

## 3. Paso a paso

### 3.0 PC-BS-AS (Configuración de host)
Objetivo: configurar la PC de Buenos Aires con los parámetros de red correctos para la VLAN 30.

**Dispositivo:** `PC-BS-AS`
**Acceso:** Desktop → IP Configuration

**Configuración estática:**
- **IP Address:** `192.168.30.10`
- **Subnet Mask:** `255.255.255.0`
- **Default Gateway:** `192.168.30.1`
- **DNS Server:** `1.1.1.1`

**Validaciones iniciales:**
Desde la PC, ir a Desktop → Command Prompt y ejecutar:
```
ipconfig /all
```

**Resultado esperado:**
```
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0001.6451.F5E8
   Link-local IPv6 Address.........: FE80::201:64FF:FE51:F5E8
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.30.10
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: 192.168.30.1
   DNS Servers.....................: 1.1.1.1
```

**Nota:** En esta fase, el ping al gateway (192.168.30.1) fallará ya que el Router BS.AS aún no está configurado. Guardar captura de `ipconfig /all` para el ticket de trabajo.

---

### 3.1 Switch `SW-BS-AS` (Req. Interfaces #2)
Objetivo: garantizar VLAN 30/100/200 y troncales correctas.

**Dispositivo:** `SW-BS-AS`
**Acceso inicial:**
```
enable
configure terminal
```

**Configuración:**
```
configure terminal
vlan 30
 name LAN_BSAS
vlan 100
 name WAN1
vlan 200
 name WAN2
exit
!
interface fa0/2
 description PC_BS_AS
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit
!
interface gi0/2
 description Trunk_to_Router_BSAS_G0/1
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 30,100,200
exit
!
interface gi0/1
 description WAN1_to_ISP_LOCAL_G0/1
 switchport mode access
 switchport access vlan 100
exit
!
interface fa0/24
 description WAN2_to_ISP_LOCAL_G0/2
 switchport mode access
 switchport access vlan 200
exit
exit
```

**Guardar configuración:**
```
write memory
```

**Validaciones:**
```
show vlan brief
show interfaces trunk
```
Guardar salidas en el ticket.

### 3.2 Router BS.AS – Preparación
**Dispositivo:** `Router BS.AS`
**Acceso inicial:**
```
enable
configure terminal
```

**Limpieza de interfaces:**
```
default interface g0/1
default interface g0/2
interface range g0/1 - 2
 no shutdown
exit
exit
```

### 3.3 Router BS.AS – Subinterfaces LAN y WAN
**Dispositivo:** `Router BS.AS`

Configurar las 3 subinterfaces necesarias: LAN (VLAN 30) y las dos WAN (VLAN 100/200):

```
configure terminal
interface g0/1
 no ip address
 no shutdown
exit
!
! Subinterfaz LAN Buenos Aires
interface g0/1.30
 encapsulation dot1Q 30
 description LAN_BSAS_VLAN30
 ip address 192.168.30.1 255.255.255.0
 ip nat inside
 no shutdown
exit
!
! Subinterfaz WAN1 hacia ISP_LOCAL G0/1
interface g0/1.100
 encapsulation dot1Q 100
 description WAN1_VLAN100_to_ISP_LOCAL
 ip address 42.25.25.1 255.255.255.248
 ip nat outside
 no shutdown
exit
!
! Subinterfaz WAN2 hacia ISP_LOCAL G0/2
interface g0/1.200
 encapsulation dot1Q 200
 description WAN2_VLAN200_to_ISP_LOCAL
 ip address 43.26.26.1 255.255.255.248
 ip nat outside
 no shutdown
exit
exit
```

**Guardar configuración:**
```
write memory
```

**Verificación:**
```
show ip interface brief
```
> Esperado: G0/1 `up/up`, subinterfaces G0/1.30, G0/1.100, G0/1.200 todas `up/up`

**Pruebas de conectividad básica desde Router BS.AS:**
```
ping 192.168.30.10
ping 42.25.25.2
ping 43.26.26.2
```
> ✅ Los 3 pings deben responder exitosamente, confirmando conectividad LAN y WAN

### 3.4 Rutas estáticas hacia servidores
**Dispositivo:** `Router BS.AS`

Para alcanzar los servidores DNS y WEB a través del ISP_LOCAL:
```
configure terminal
!
! Ruta hacia red de servidores (VLAN 100: 192.168.100.0/29)
ip route 192.168.100.0 255.255.255.248 42.25.25.2 1
ip route 192.168.100.0 255.255.255.248 43.26.26.2 10
!
! Ruta por defecto hacia Internet (métricas diferentes para evitar ECMP)
ip route 0.0.0.0 0.0.0.0 42.25.25.2 1
ip route 0.0.0.0 0.0.0.0 43.26.26.2 10
exit
```

**Verificación de rutas:**
```
show ip route
```

### 3.5 NAT para salida a Internet (doble salida)
**Dispositivo:** `Router BS.AS`

Configurar NAT overload por ambas interfaces WAN para redundancia:
```
configure terminal
!
! ACL para identificar tráfico de LAN Buenos Aires  
access-list 1 permit 192.168.30.0 0.0.0.255
!
! NAT overload por ambas salidas WAN
ip nat inside source list 1 interface g0/1.100 overload
ip nat inside source list 1 interface g0/1.200 overload
exit
```

> **Nota:** Las interfaces `g0/1.30` (inside) y `g0/1.100`/`g0/1.200` (outside) ya fueron marcadas en la sección 3.3. Cisco usará ambas salidas según la tabla de routing (costo/métrica).

**Verificación:**
```
show ip nat translations
show ip nat statistics
```

### 3.6 Rutas de retorno en ISP_INTERNACIONAL (RECOMENDADO) ⚠️
**Dispositivo:** `ISP_INTERNACIONAL`

**¿Por qué es necesario?**
Para que el NAT configurado en Router BS.AS funcione completamente, ISP_INTERNACIONAL debe saber cómo devolver el tráfico hacia las redes WAN públicas de Buenos Aires (42.25.25.0/29 y 43.26.26.0/29).

Sin estas rutas:
- Los pings/HTTP desde PC-BS-AS **llegan** a 164.25.0.1 o a los servidores.
- Pero las **respuestas se pierden** porque ISP_INTERNACIONAL no sabe cómo volver a 42.25.25.1 / 43.26.26.1 (las IPs públicas de NAT).

**Configuración:**
```
configure terminal
!
! Rutas de retorno hacia las redes WAN de Buenos Aires
ip route 42.25.25.0 255.255.255.248 164.25.0.2
ip route 43.26.26.0 255.255.255.248 164.25.0.2
!
exit
write memory
```

**Verificación:**
```
show ip route
```
> Debe mostrar las rutas hacia 42.25.25.0/29 y 43.26.26.0/29 vía 164.25.0.2 (ISP_LOCAL)

**Justificación para el informe:**
"Se agregan rutas estáticas en ISP_INTERNACIONAL hacia las redes WAN 42.25.25.0/29 y 43.26.26.0/29 vía 164.25.0.2 para permitir el retorno de tráfico NAT desde los servidores e Internet hacia Buenos Aires. Esto garantiza la conectividad bidireccional completa para las pruebas de NAT y el acceso a servicios desde la LAN de Buenos Aires."

**Nota importante:**
- El profesor no lo pidió explícitamente, pero es la configuración correcta para una topología funcional.
- Sin esto, las pruebas de `ping 164.25.0.1` desde Router BS.AS y el acceso desde PC-BS-AS a servidores vía NAT **fallarán** en el retorno.
- Agregar esta configuración demuestra comprensión del ruteo de retorno en NAT.

### 3.7 ACL FTP (requerimiento específico)
Creación inmediata, aplicación pendiente hasta conocer la interfaz/servidor final.

**Dispositivo:** `Router BS.AS`
```
configure terminal
ip access-list extended FTP_ONLY_PC
 remark Solo PC-BS-AS puede acceder a FTP
 ! Reemplazar X.X.X.X con IP real del servidor FTP cuando se defina
exit
exit
```
> **Nota:** La ACL quedará vacía hasta que se conozca la IP del servidor FTP. En ese momento agregar: `permit tcp host 192.168.30.10 host X.X.X.X eq 21`, `deny tcp any host X.X.X.X eq 21`, `permit ip any any`.

### 3.8 Guardar cambios
**Dispositivo:** `Router BS.AS`
```
write memory
copy running-config startup-config
```

---

## 4. Verificaciones finales

### 4.1 Verificaciones de configuración
1. **Router BS.AS**
```
show ip interface brief
show ip route
```
2. **Switch**: `show interfaces trunk` (solo debe mostrar Gi0/2), `show vlan brief` (verificar Gi0/1 en VLAN 100, Fa0/24 en VLAN 200).
3. **ISP_LOCAL**: verificar que `show ip route` contenga rutas hacia 192.168.30.0/24 (configuradas en guía 01).

### 4.2 Pruebas de conectividad OBLIGATORIAS ✅
Con la configuración de esta guía, estas pruebas **deben funcionar exitosamente:**

**Desde Router BS.AS:**
- ✅ `ping 192.168.30.10` - PC en LAN (verifica subinterfaz .30 y conectividad LAN)
- ✅ `ping 42.25.25.2` - ISP_LOCAL WAN1 (verifica subinterfaz .100 y enlace WAN1)
- ✅ `ping 43.26.26.2` - ISP_LOCAL WAN2 (verifica subinterfaz .200 y enlace WAN2)

**Desde ISP_LOCAL:**
- ✅ `ping 164.25.0.1` - ISP_INTERNACIONAL (verifica conectividad hacia Internet)

**Desde PC-BS-AS:**
- ✅ `ping 192.168.30.1` - Gateway (verifica conectividad PC→Router BS.AS)
- ✅ `ping 192.168.100.2` - DNS-SERVER (verifica NAT, ruteo y conectividad hacia servidores)
- ✅ `ping 192.168.100.9` - WEB-SERVER (verifica NAT, ruteo y conectividad hacia servidores)

### 4.3 Pruebas adicionales (con rutas de retorno configuradas)
**Si completaste la sección 3.6** (rutas de retorno en ISP_INTERNACIONAL), estas pruebas también funcionarán:

**Desde Router BS.AS:**
- ✅ `ping 164.25.0.1` - ISP_INTERNACIONAL (verifica conectividad completa hacia Internet)
  - Con las rutas de retorno en ISP_INTERNACIONAL, este ping **debe funcionar**
  - Sin las rutas, el ping llega pero la respuesta se pierde

**Desde PC-BS-AS (usando NAT):**
- ✅ `ping 164.25.0.1` - ISP_INTERNACIONAL (verifica NAT completo hacia Internet)
  - Requiere que las rutas de retorno estén configuradas en ISP_INTERNACIONAL
  - Verificar con `show ip nat translations` en Router BS.AS para ver la traducción

**Nota:** Si no configuraste la sección 3.6, puedes usar `ping 164.25.0.1 source 192.168.30.1` desde Router BS.AS como alternativa, pero la PC-BS-AS no podrá alcanzar Internet sin las rutas de retorno.

### 4.4 Pruebas de servicios
**Desde PC-BS-AS:**
- `nslookup www.consultas.labo.com.ar 192.168.100.2` - Prueba DNS
- Abrir navegador web y acceder a `http://192.168.100.9` - Prueba WEB-SERVER
5. Documentar cada salida en `ticket_trabajo_practico.md` con la fecha y requerimiento asociado.

---

## 5. Checklist para cerrar la Fase 2
- [ ] VLAN 30/100/200 presentes y troncales operativas (captura `show interfaces trunk`).
- [ ] Subinterfaces operativas: `g0/1.30`, `g0/1.100`, `g0/1.200` todas `up/up` con IPs correctas (`show ip interface brief`).
- [ ] Conectividad LAN verificada (ping PC↔Router exitoso).
- [ ] Conectividad WAN verificada (ping Router BS.AS ↔ ISP_LOCAL por ambos enlaces).
- [ ] NAT funcionando (desde PC hacer ping a servidores y verificar `show ip nat translations`).
- [ ] **Rutas de retorno configuradas en ISP_INTERNACIONAL** (sección 3.6) para permitir tráfico NAT bidireccional.
  - `show ip route` en ISP_INTERNACIONAL debe mostrar rutas hacia 42.25.25.0/29 y 43.26.26.0/29
  - Ping desde PC-BS-AS a 164.25.0.1 debe funcionar
- [ ] Switch SW-BS-AS configurado correctamente:
  - Gi0/2 en trunk (VLAN 30,100,200) → Router BS.AS G0/1
  - Gi0/1 en access VLAN 100 → ISP_LOCAL G0/1
  - Fa0/24 en access VLAN 200 → ISP_LOCAL G0/2
- [ ] ACL `FTP_ONLY_PC` creada y documentada.
- [ ] Evidencias volcadas al ticket.

---

