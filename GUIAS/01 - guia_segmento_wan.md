# Guía 01 · Segmento WAN (ISP_LOCAL + Internet Cloud)
**Fecha:** 19/11/2025 - ACTUALIZADA CON NUEVOS REQUERIMIENTOS  
**Fase:** 1 de 6  
**Objetivo:** Configurar la infraestructura de Internet (ISP_INTERNACIONAL, ISP_LOCAL, servidores) lista para recibir tráfico desde BS.AS.

**⚠️ CAMBIOS IMPORTANTES:**
- Buenos Aires usa red **192.168.30.0/24** (VLAN 30), no 192.168.20.0/24
- **Servidores en VLANs separadas:**
  - **VLAN 100 (DNS):** 192.168.100.0/29 - Gateway .1, DNS-SERVER .2
  - **VLAN 101 (WEB):** 192.168.100.8/29 - Gateway .14, WEB-SERVER .9

**Referencias:**
- `NUEVOSREQUERIMIENTOS`: Documento oficial del profesor
- `Analisis y requisitos/CAMBIOS_CRITICOS_PROFESOR.md`: Análisis detallado de cambios
- `00 - plan_trabajo_general.md`: Plan maestro del proyecto

---

## 1. Alcance de la Fase
Esta guía configura la infraestructura de salida a Internet que será utilizada por todos los sitios (Buenos Aires, Córdoba, Mendoza).

**Dispositivos a configurar:**
1. **ISP_INTERNACIONAL** (Router 2911): Gateway hacia Internet
2. **ISP_LOCAL** (Router 2911): Proveedor local que conecta con BS.AS
3. **SW_MS_CORE** (Switch 2960): Switch de servicios en la nube de Internet
4. **Servidores**: WEB Server y DNS Server con servicios activos

**Redes involucradas:**
- Internet: 164.25.0.0/29 (ISP_INTERNACIONAL ↔ ISP_LOCAL)
- **VLAN 100 (DNS):** 192.168.100.0/29 - DNS-SERVER (.2), Gateway (.1)
- **VLAN 101 (WEB):** 192.168.100.8/29 - WEB-SERVER (.9), Gateway (.14)
- Red BS.AS: 192.168.30.0/24 (rutas estáticas hacia esta red)

---

## 2. Prerrequisitos
1. **VLAN 30** lista para la LAN ⚠️ CAMBIO: antes era VLAN 20, y disponibilidad de los IDs 100/200 para WAN (Interfaces #2/#3).  
2. PC-BS-AS con IP `192.168.30.10/24`, gateway `192.168.30.1`, DNS `1.1.1.1`. ⚠️ CAMBIO: antes era 192.168.20.x  
3. Identificar puertos físicos usados: Fa0/2 (PC→switch), **Gi0/1 del switch** ↔ **G0/1 del ISP_LOCAL** (WAN1), **Fa0/24 del switch** ↔ **G0/2 del ISP_LOCAL** (WAN2), y G0/0 del ISP_LOCAL hacia Internet.  
4. Credenciales de acceso al switch y al ISP_LOCAL.

---

## 3. Switch `SW-BS-AS` (VLANs + Trunk hacia router)
> Cumple Interfaces #2/#3 (segmentación) y STP #1 (puertos consistentes).

**Acceso inicial al switch:**
```
enable
configure terminal
```

1. **Crear/nombrar VLANs necesarias**
```
vlan 30
 name LAN_BSAS
vlan 100
 name WAN1
vlan 200
 name WAN2
exit
exit
```

**Guardar configuración:**
```
write memory
```

**Verificación:**
```
show vlan brief
show interfaces status
```

> **Qué esperar:**
> - `show vlan brief`: Debe mostrar VLAN 30, 100, 200 creadas con sus nombres
> - `show interfaces status`: Debe mostrar Fa0/2 como "connected" si la PC está conectada

2. **Configurar puerto de acceso hacia la PC (si aplica Fa0/2)**
```
configure terminal
interface fa0/2
 description PC_BSAS
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit
exit
```

3. **Configurar los uplinks en modo ACCESS hacia el ISP_LOCAL**
```
configure terminal
interface gi0/1
 description Uplink_to_ISP_LOCAL_G0_1_WAN1
 switchport mode access
 switchport access vlan 100
exit
!
interface fa0/24
 description Uplink_to_ISP_LOCAL_G0_2_WAN2
 switchport mode access
 switchport access vlan 200
exit
exit
```

**Guardar configuración:**
```
write memory
```

**Verificación:**
```
show vlan brief
show interfaces gi0/1 switchport
show interfaces fa0/24 switchport
show running-config interface gi0/1
show running-config interface fa0/24
```

> **Qué esperar:**
> - `show vlan brief`: Gi0/1 debe aparecer en VLAN 100, Fa0/24 en VLAN 200
> - `show interfaces gi0/1 switchport`: Debe mostrar "Switchport: Enabled", "Administrative Mode: access", "Access Mode VLAN: 100"
> - `show running-config interface gi0/1`: Debe mostrar la configuración que acabas de aplicar

**Verificación adicional del estado de los puertos:**
```
show interfaces gi0/1 status
show interfaces fa0/24 status
```

> **📝 Nota técnica:** No se usa trunk en estos enlaces porque cada WAN (/29) viaja por un cable físico separado. El router ISP_LOCAL tiene IPs directamente en G0/1 y G0/2 (sin subinterfaces), por lo que espera tramas sin etiquetar. El modo ACCESS es la configuración correcta.
> 
> Mantén los dos cables exactamente así: `ISP_LOCAL G0/1 ↔ SW-BS-AS Gi0/1` (VLAN 100/WAN1) y `ISP_LOCAL G0/2 ↔ SW-BS-AS Fa0/24` (VLAN 200/WAN2).

---

## 4. Router ISP_LOCAL
> Aplica "Interfaces #3" y "Ruteo estático #2" para retorno hacia la LAN. Estas rutas estáticas duales se mantendrán hasta implementar OSPF, momento en que Router BS.AS propagará la default con `default-information originate` (según notas del profesor, reunión 14-11-2025).

**Acceso inicial al router:**
```
enable
configure terminal
```

**Configuración de interfaces y rutas:**
```
configure terminal
interface g0/0
 description Internet_uplink
 ip address 164.25.0.2 255.255.255.248
 no shutdown
exit
!
interface g0/1
 description WAN1_from_BSAS
 ip address 42.25.25.2 255.255.255.248
 no shutdown
exit
!
interface g0/2
 description WAN2_from_BSAS
 ip address 43.26.26.2 255.255.255.248
 no shutdown
exit
!
ip route 192.168.30.0 255.255.255.0 42.25.25.1
ip route 192.168.30.0 255.255.255.0 43.26.26.1
ip route 0.0.0.0 0.0.0.0 164.25.0.1
exit
```

**Guardar configuración:**
```
write memory
```

### 4.1 Procedimiento paso a paso
1. **Limpiar interfaces (si ya tenían IP):** `conf t` ➜ `interface g0/x` ➜ `no ip address` ➜ `exit`. Repite para G0/0–G0/2.
2. **Asignar Internet a G0/0:**
	- `interface g0/0`
	- `description Internet_uplink`
	- `ip address 164.25.0.2 255.255.255.248`
	- `no shutdown`
3. **Configurar enlaces hacia BS.AS:**
	- `interface g0/1` ➜ `description WAN1_from_BSAS` ➜ `ip address 42.25.25.2 255.255.255.248` ➜ `no shutdown`.
	- `interface g0/2` ➜ `description WAN2_from_BSAS` ➜ `ip address 43.26.26.2 255.255.255.248` ➜ `no shutdown`.
4. **Definir rutas estáticas (⚠️ CAMBIO: red 192.168.30.0/24):**
	- `ip route 192.168.30.0 255.255.255.0 42.25.25.1`
	- `ip route 192.168.30.0 255.255.255.0 43.26.26.1`
	- `ip route 0.0.0.0 0.0.0.0 164.25.0.1`
5. **Verificar básico:** `show ip int brief`, `show ip route`.
6. **Verificar conectividad:** Solo después de completar sección 5 (Internet Cloud): `ping 164.25.0.1`.
7. **Guardar cambios:** `write memory`.

**Verificación:**
```
show ip interface brief
show ip route
```

**📋 Estado esperado después de la sección 4:**
```
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     164.25.0.2      YES manual up                    down 
GigabitEthernet0/1     42.25.25.2      YES manual up                    up 
GigabitEthernet0/2     43.26.26.2      YES manual up                    up 
```

> **✅ Explicación del estado:**
> - **G0/1 y G0/2**: `up/up` → Correcto, conexiones WAN hacia Router BS.AS listas
> - **G0/0**: `up/down` → **NORMAL**, porque ISP_INTERNACIONAL no está configurado aún
> - **Rutas estáticas**: Deben aparecer hacia 192.168.30.0/24 con next-hop .1

**Verificación avanzada (solo después de completar sección 5 - Internet Cloud):**
```
ping 164.25.0.1
```

> **⚠️ Nota importante:** 
> - El `up/down` en G0/0 es **esperado** hasta completar la sección 5
> - El ping a 164.25.0.1 **fallará** hasta configurar ISP_INTERNACIONAL
> - Los pings a 42.25.25.1 y 43.26.26.1 **fallarán** hasta configurar el Router BS.AS en la guía 02

**🎯 Si tu salida coincide con el estado esperado arriba, ¡perfecto! Continúa con la sección 5.**

---

## 5. Internet Cloud Simulada
### 5.1 Topología y roles
1. El ícono INTERNET abre un mini-escenario con:  
	- Router `2911 ISP_INTERNACIONAL`.  
	- Switch `2960-24TT SW_MS_CORE`.  
	- Servidores `WEB-SERVER` y `DNS-SERVER`.  
2. `ISP_INTERNACIONAL G0/0` se conecta a `ISP_LOCAL G0/0`. `G0/1` baja al switch `SW_MS_CORE` para transportar las VLANs 100 y 101.

### 5.2 Router `ISP_INTERNACIONAL` (lado Internet)
1. Abre el icono INTERNET, haz clic sobre el router azul (2911) y ve a la pestaña **CLI**.  
2. Entra al modo privilegiado y de configuración:
```
en
conf t
hostname ISP_INTERNACIONAL
```
3. Configura la interfaz hacia tu ISP_LOCAL:
```
configure terminal
interface g0/0
 description Hacia_ISP_LOCAL_G0/0
 ip address 164.25.0.1 255.255.255.248
 no shutdown
exit
exit
```
4. Configura la conexión al switch usando subinterfaces para VLAN 100 y 101:
```
configure terminal
interface g0/1
 no ip address
 no shutdown
exit
!
interface g0/1.100
 description Hacia_SW_MS_CORE_VLAN100_DNS
 encapsulation dot1Q 100
 ip address 192.168.100.1 255.255.255.248
 no shutdown
exit
!
interface g0/1.101
 description Hacia_SW_MS_CORE_VLAN101_WEB
 encapsulation dot1Q 101
 ip address 192.168.100.14 255.255.255.248
 no shutdown
exit
exit
```
5. Carga la ruta de retorno hacia la LAN de Buenos Aires por medio del ISP_LOCAL:
```
configure terminal
ip route 192.168.30.0 255.255.255.0 164.25.0.2
exit
write memory
```
6. Comprueba el estado:
```
show ip interface brief
ping 164.25.0.2
```

**📋 Estado esperado después de la sección 5.2:**

**`show ip interface brief` debe mostrar:**
```
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     164.25.0.1      YES manual up                    up 
GigabitEthernet0/1     unassigned      YES unset  up                    up 
GigabitEthernet0/1.100 192.168.100.1   YES manual up                    up 
GigabitEthernet0/1.101 192.168.100.14  YES manual up                    up 
```

**`ping 164.25.0.2` debe mostrar:**
```
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms
```

> **✅ Qué significa cada resultado:**
> - **G0/0 up/up** → Conexión hacia ISP_LOCAL establecida
> - **G0/1.100 up/up** → Subinterfaz para servidores operativa  
> - **Ping 80-100% éxito** → Conectividad con ISP_LOCAL funcionando
> - **Primer ping puede fallar** → Normal por proceso ARP inicial
> 
> 	Si el ping falla, revisa que el puerto del ISP_LOCAL esté `up/up` y conectado.
 	Si `GigabitEthernet0/1.100` aparece `administratively down`, vuelve a `interface g0/1` y ejecuta `no shutdown`; la subinterfaz hereda el estado físico.

### 5.3 Switch `SW_MS_CORE`
1. Entra al switch (dentro del mismo icono), pestaña CLI, y crea las VLANs para los servidores:  
```
enable
configure terminal
vlan 100
 name LAN_DNS_SERVER
vlan 101
 name LAN_WEB_SERVER
exit
exit
```
2. Configura el enlace hacia el router como trunk y los puertos hacia los servidores en sus respectivas VLANs:
```
configure terminal
interface gi0/1
 description Uplink_to_ISP_INTERNACIONAL
 switchport mode trunk
 switchport trunk allowed vlan 100,101
 switchport trunk native vlan 1
exit
!
interface fa0/1
 description To_WEB_SERVER
 switchport mode access
 switchport access vlan 101
exit
!
interface fa0/2
 description To_DNS_SERVER
 switchport mode access
 switchport access vlan 100
exit
exit
write memory
```
3. Verifica lo anterior:
```
show vlan brief
show interfaces trunk
```

**📋 Estado esperado después de la sección 5.3:**

**`show vlan brief` debe mostrar:**
```
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
100  LAN_DNS_SERVER                   active    Fa0/2
101  LAN_WEB_SERVER                   active    Fa0/1
```

**`show interfaces trunk` debe mostrar:**
```
Port        Mode         Encapsulation  Status        Native vlan
Gig0/1      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Gig0/1      100,101
```

> **✅ Qué significa cada resultado:**
> - **VLANs 100 y 101 activas** → VLANs creadas correctamente
> - **Fa0/1 en VLAN 101** → Web Server en su VLAN correcta
> - **Fa0/2 en VLAN 100** → DNS Server en su VLAN correcta
> - **Gi0/1 trunking** → Enlace hacia ISP_INTERNACIONAL operativo

### 5.4 Servidores (dentro de la nube)
1. **WEB-SERVER**:
	- Pestaña Desktop → IP Configuration → coloca IP `192.168.100.9`, máscara `255.255.255.248`, gateway `192.168.100.14`, DNS `1.1.1.1`.  
	- **⚠️ IMPORTANTE:** Gateway es `192.168.100.14` (última IP usable de la subred .8/29)
	- **⚠️ IMPORTANTE:** Verifica que Subnet Mask sea `255.255.255.248`
	- Haz clic en **"Save"** después de configurar
	- Pestaña Services → HTTP → enciende el servicio (botón ON). 
	- **Contenido web (opcional):** En Services → HTTP puedes:
		- Mantener el `index.html` por defecto (página simple "Cisco Packet Tracer")
		- O editarlo con contenido personalizado como `<h1>Servidor Web Laboratorio Redes</h1>`
		- Para este laboratorio, **solo encender el servicio es suficiente**
	- Si tu Packet Tracer muestra el bloque **Static NAT/Public Address** (Config → Global → Settings), allí mapea `45.162.20.10`; en versiones legacy donde no aparece, solo documenta la IP pública prevista porque el NAT se implementará más adelante en el router BS.AS (según `notasprofeso`, reunión 14-11-2025: NAT estático WEB=45.162.20.10, DNS=1.1.1.1).
2. **DNS-SERVER**:
	- Desktop → IP Configuration → IP `192.168.100.2`, máscara `255.255.255.248`, gateway `192.168.100.1`.  
	- **⚠️ IMPORTANTE:** Gateway es `192.168.100.1`
	- **⚠️ IMPORTANTE:** Verifica que Subnet Mask sea `255.255.255.248`
	- Haz clic en **"Save"** después de configurar
	- Services → DNS → habilita el servicio (botón ON) y crea un registro `www.consultas.labo.com.ar` apuntando a `45.162.20.10`. 
	- **Configuración DNS:** En Services → DNS:
		- Asegúrate que el servicio está ON
		- En "Resource Records" agrega: Name: `www.consultas.labo.com.ar`, Type: `A Record`, Address: `45.162.20.10`
	- **Nota sobre IP pública:** En versiones legacy de Packet Tracer sin panel "Static NAT/Public Address", simplemente documenta que el NAT estático (WEB=45.162.20.10, DNS=1.1.1.1) se implementará en el router BS.AS según las notas del profesor.
3. Guarda la configuración de cada servidor con el botón Save en Packet Tracer.

**🔍 Verificación de servicios:**
- **WEB-SERVER**: Abre Desktop → Web Browser y navega a `http://192.168.100.9` - debe mostrar una página web
- **DNS-SERVER**: 
  - Verifica que el servicio está ON en Services → DNS
  - Verifica que el registro `www.consultas.labo.com.ar → 45.162.20.10` está creado
  - **Prueba DNS desde otro equipo:** La prueba de nslookup se hará desde ISP_INTERNACIONAL o desde la PC de Buenos Aires una vez configurada

> Nota: en Packet Tracer sin panel de “Public Address” en los servidores, el requisito de IP pública se cumple aplicando `ip nat inside source static` en el router BS.AS. No intentes modificar la IP interna de los servidores; mantén 192.168.100.9/29 y 192.168.100.2/29 y documenta el mapeo público en `ticket_trabajo_practico.md`.

### 5.5 Checklist final
1. `ISP_INTERNACIONAL`: `show ip int brief` debe mostrar `G0/0`, `G0/1.100` y `G0/1.101` `up/up`.  
2. `SW_MS_CORE`: VLANs 100 y 101 activas y `Gi0/1` en trunk.  
3. Servidores: IPs internas configuradas (192.168.100.9 y 192.168.100.2) y servicios activos.  
4. Desde `ISP_INTERNACIONAL`, ejecuta `ping 164.25.0.2` para validar el enlace.  
5. Desde cada servidor, ping al gateway correspondiente:
   - WEB-SERVER: `ping 192.168.100.14`
   - DNS-SERVER: `ping 192.168.100.1`
6. Si necesitas pruebas públicas, agrega destinos simulados (ej. `8.8.8.8`) y asegúrate de que el router tenga rutas adecuadas.

**📋 Verificación completa de conectividad:**

**✅ Lo que SÍ debe funcionar después de completar guía 01:**
```
# Desde ISP_INTERNACIONAL
ping 164.25.0.2          # ✅ Debe funcionar (hacia ISP_LOCAL)
ping 192.168.100.9       # ✅ Debe funcionar (hacia WEB-SERVER)  
ping 192.168.100.2       # ✅ Debe funcionar (hacia DNS-SERVER)

# Desde ISP_LOCAL  
ping 164.25.0.1          # ✅ Debe funcionar (hacia ISP_INTERNACIONAL)

# Verificar configuración de servidores (no pings)
# WEB-SERVER: Desktop → IP Configuration debe mostrar:
#   IP: 192.168.100.9, Mask: 255.255.255.248, Gateway: 192.168.100.14
# DNS-SERVER: Desktop → IP Configuration debe mostrar:  
#   IP: 192.168.100.2, Mask: 255.255.255.248, Gateway: 192.168.100.1
```

**⚠️ Lo que puede fallar (configuración de servidor):**
```
# Desde servidores hacia gateway
ping 192.168.100.14 desde WEB-SERVER    # ❌ Puede fallar si gateway mal configurado
ping 192.168.100.1 desde DNS-SERVER     # ❌ Puede fallar si gateway mal configurado
```

**🔍 Si los pings desde servidores fallan:**
1. Verifica en cada servidor: Desktop → IP Configuration
2. WEB-SERVER: Gateway = `192.168.100.14`
3. DNS-SERVER: Gateway = `192.168.100.1`
4. Asegúrate que Mask = `255.255.255.248`
5. Haz clic "Save" en cada servidor después de configurar

**✅ Lo que también DEBERÍA funcionar (gracias a la ruta default en ISP_LOCAL):**
```
# Desde ISP_LOCAL hacia servidores  
ping 192.168.100.9 desde ISP_LOCAL  # ✅ La default route envía tráfico a ISP_INTERNACIONAL
ping 192.168.100.2 desde ISP_LOCAL  # ✅ La default route envía tráfico a ISP_INTERNACIONAL
```

**⚠️ Lo que NO funcionará hasta completar otras guías:**
```
# Desde servidores hacia ISP_LOCAL
ping 164.25.0.2 desde WEB-SERVER    # ❌ Requiere configurar gateway en servidor
ping 164.25.0.2 desde DNS-SERVER    # ❌ Requiere configurar gateway en servidor

# Verificación DNS
nslookup www.consultas.labo.com.ar 192.168.100.2  # ❌ Requiere conectividad completa
```

> **🎯 Estado después de guía 01:** 
> - Conectividad completa en segmento Internet (ISP_INTERNACIONAL ↔ ISP_LOCAL ↔ Servidores)
> - La ruta default `0.0.0.0/0 → 164.25.0.1` permite que ISP_LOCAL alcance las subredes `/29` de servidores
> - Falta configurar Router BS.AS (Guía 02) para conectividad end-to-end

> **📝 Para verificar que todo está bien:** Ejecuta solo los pings marcados con ✅. Los marcados con ❌ son esperado que fallen por ahora.

---

## 6. Plan de Pruebas
| Paso | Comando | Estado después de guía 01 | Requiere para funcionar |
|------|---------|---------------------------|------------------------|
| ISP_LOCAL G0/1,G0/2 | `show ip interface brief` | ✅ `up/up` | Solo guía 01 |
| ISP_LOCAL G0/0 | `show ip interface brief` | ✅ `up/up` | Solo guía 01 |
| ISP conectividad | `ping 164.25.0.1` desde ISP_LOCAL | ✅ Debe funcionar | Solo guía 01 |
| Servidores locales | `ping 192.168.100.9` desde ISP_INTERNACIONAL | ✅ Debe funcionar | Solo guía 01 |
| Conectividad cruzada | `ping 164.25.0.1` desde servidores | ❌ Falla (sin rutas) | Rutas adicionales |
| WAN hacia BS.AS | `ping 42.25.25.1` desde ISP_LOCAL | ❌ Requiere Router BS.AS | Guía 02 completa |
| LAN BS.AS | `ping 192.168.30.1` desde PC | ❌ Requiere Router BS.AS | Guía 02 completa |
| End-to-end | `ping servidores` desde PC BS.AS | ❌ Requiere configuración completa | Guías 01+02 completas |

