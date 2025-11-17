# Guía de Configuración Segmento WAN BS.AS ↔ ISP_LOCAL ↔ Internet
**Fecha:** 17/11/2025  
**Objetivo:** Configurar el tramo mostrado (Switch BS.AS → Router ISP_LOCAL → Internet) respetando los requerimientos "Interfaces #3", "Ruteo estático #1/#2" y "NAT Buenos Aires #1" de `reque.md`.

**Referencias:**
- `Analisis y requisitos/reque.md`: lineamientos oficiales del TP.
- `Analisis y requisitos/notasprofeso`: reunión 14-11-2025 confirma que todas las PC deben llegar a Internet/DNS, NAT estático para servidores (WEB=45.162.20.10, DNS=1.1.1.1), y que la ruta por defecto a Internet se propagará vía OSPF en fases posteriores (no configurar manualmente en cada router interno).

---

## 1. Alcance del Segmento
- **Inicio:** PC-BS-AS conectada al switch `2960-24TT SW-BS-AS`.
- **Medio:** Tránsito por el switch hacia los enlaces WAN1 (VLAN 100, 42.25.25.0/29) y WAN2 (VLAN 200, 43.26.26.0/29) que llegan al router `2911 ISP_LOCAL`.
- **Fin:** Enlace serie 164.25.0.0/29 hacia la nube de Internet.
- **Servicios involucrados:** Acceso LAN, trunk en el switch, direccionamiento WAN en ISP_LOCAL, ruteo estático y conexión a Internet.

---

## 2. Prerrequisitos
1. VLAN 20 lista para la LAN y disponibilidad de los IDs 100/200 para WAN (Interfaces #2/#3).  
2. PC-BS-AS con IP `192.168.20.10/24`, gateway `192.168.20.1`, DNS `1.1.1.1`.  
3. Identificar puertos físicos usados: Fa0/2 (PC→switch), **G0/2 del switch** ↔ **G0/1 del ISP_LOCAL** (WAN1), **Fa0/24 del switch** ↔ **G0/2 del ISP_LOCAL** (WAN2), y G0/0 del ISP_LOCAL hacia Internet.  
4. Credenciales de acceso al switch y al ISP_LOCAL.

---

## 3. Switch `SW-BS-AS` (VLANs + Trunk hacia router)
> Cumple Interfaces #2/#3 (segmentación) y STP #1 (puertos consistentes).

1. **Crear/nombrar VLANs necesarias**
```
conf t
vlan 20
 name LAN_BSAS
vlan 100
 name WAN1
vlan 200
 name WAN2
end
wr mem
show vlan brief
```

2. **Configurar puerto de acceso hacia la PC (si aplica Fa0/2)**
```
conf t
interface fa0/2
 description PC_BSAS
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
end
```

3. **Configurar los uplinks en trunk hacia el ISP_LOCAL**
```
conf t
interface g0/2
 description Uplink_to_ISP_LOCAL_G0_1
 switchport mode trunk
 switchport trunk allowed vlan 20,100,200
 switchport trunk native vlan 20
!
interface fa0/24
 description Uplink_to_ISP_LOCAL_G0_2
 switchport mode trunk
 switchport trunk allowed vlan 20,100,200
 switchport trunk native vlan 20
end
wr mem
show interfaces trunk
```
> Nota: Mantén los dos cables exactamente así: `ISP_LOCAL G0/1 ↔ SW-BS-AS G0/2` y `ISP_LOCAL G0/2 ↔ SW-BS-AS Fa0/24`. Documenta este mapeo en Packet Tracer para evitar confusiones.

---

## 4. Router ISP_LOCAL
> Aplica "Interfaces #3" y "Ruteo estático #2" para retorno hacia la LAN. Estas rutas estáticas duales se mantendrán hasta implementar OSPF, momento en que Router BS.AS propagará la default con `default-information originate` (según notas del profesor, reunión 14-11-2025).
```
conf t
interface g0/0
 description Internet_uplink
 ip address 164.25.0.2 255.255.255.248
 no shutdown
interface g0/1
 description WAN1_from_BSAS
 ip address 42.25.25.2 255.255.255.248
 no shutdown
interface g0/2
 description WAN2_from_BSAS
 ip address 43.26.26.2 255.255.255.248
 no shutdown
!
ip route 192.168.20.0 255.255.255.0 42.25.25.1
ip route 192.168.20.0 255.255.255.0 43.26.26.1
ip route 0.0.0.0 0.0.0.0 164.25.0.1
end
wr mem
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
4. **Definir rutas estáticas:**
	- `ip route 192.168.20.0 255.255.255.0 42.25.25.1`
	- `ip route 192.168.20.0 255.255.255.0 43.26.26.1`
	- `ip route 0.0.0.0 0.0.0.0 164.25.0.1`
5. **Verificar:** `show ip int brief`, `show ip route`, `ping 42.25.25.1`, `ping 43.26.26.1`, `ping 164.25.0.1`.
6. **Guardar cambios:** `wr mem`.

**Verificación:**
```
show ip route
ping 42.25.25.1
ping 43.26.26.1
ping 164.25.0.1
```

---

## 5. Internet Cloud Simulada
### 5.1 Topología y roles
1. El ícono INTERNET abre un mini-escenario con:  
	- Router `2911 ISP_INTERNACIONAL`.  
	- Switch `2960-24TT SW_MS_CORE`.  
	- Servidores `WEB-SERVER` y `DNS-SERVER`.  
2. `ISP_INTERNACIONAL G0/0` se conecta a `ISP_LOCAL G0/0`. `G0/1` baja al switch `SW_MS_CORE` para transportar la VLAN 100 donde residen ambos servidores.  

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
interface g0/0
 description Hacia_ISP_LOCAL_G0/0
 ip address 164.25.0.1 255.255.255.248
 no shutdown
exit
```
4. Configura la conexión al switch usando subinterfaz para la VLAN 100 (ambos servidores están en esta red):
```
interface g0/1
 no ip address
 no shutdown
!
interface g0/1.100
 description Hacia_SW_MS_CORE_VLAN100
 encapsulation dot1Q 100
 ip address 192.168.100.1 255.255.255.248
 no shutdown
exit
```
5. Carga la ruta de retorno hacia la LAN de Buenos Aires por medio del ISP_LOCAL:
```
ip route 192.168.20.0 255.255.255.0 164.25.0.2
end
wr mem
```
6. Comprueba el estado:
```
show ip interface brief
ping 164.25.0.2
```
	Si el ping falla, revisa que el puerto del ISP_LOCAL esté `up/up` y conectado.
 	Si `GigabitEthernet0/1.100` aparece `administratively down`, vuelve a `interface g0/1` y ejecuta `no shutdown`; la subinterfaz hereda el estado físico.

### 5.3 Switch `SW_MS_CORE`
1. Entra al switch (dentro del mismo icono), pestaña CLI, y crea la VLAN utilizada por ambos servidores:  
```
en
conf t
vlan 100
 name LAN_SERVERS
```
2. Configura el enlace hacia el router como trunk y los puertos hacia los servidores como access (todos en VLAN 100):
```
interface gi0/1
 description Uplink_to_ISP_INTERNACIONAL
 switchport mode trunk
 switchport trunk allowed vlan 100
 switchport trunk native vlan 100
!
interface fa0/1
 description To_WEB_SERVER
 switchport mode access
 switchport access vlan 100
!
interface fa0/2
 description To_DNS_SERVER
 switchport mode access
 switchport access vlan 100
end
wr mem
```
3. Verifica lo anterior:
```
show vlan brief
show interfaces trunk
```

### 5.4 Servidores (dentro de la nube)
1. **WEB-SERVER**:
	- Pestaña Desktop → IP Configuration → coloca IP `192.168.100.9`, máscara `255.255.255.248`, gateway `192.168.100.1`, DNS `1.1.1.1`.  
	- Pestaña Services → HTTP → enciende el servicio y agrega contenido si lo necesitas. Si tu Packet Tracer muestra el bloque **Static NAT/Public Address** (Config → Global → Settings), allí mapea `45.162.20.10`; en versiones legacy donde no aparece, solo documenta la IP pública prevista porque el NAT se implementará más adelante en el router BS.AS (según `notasprofeso`, reunión 14-11-2025: NAT estático WEB=45.162.20.10, DNS=1.1.1.1).
2. **DNS-SERVER**:
	- Desktop → IP Configuration → IP `192.168.100.2`, máscara `255.255.255.248`, gateway `192.168.100.1`.  
	- Services → DNS → habilita el servicio y crea un registro `www.consultas.labo.com.ar` apuntando a `45.162.20.10`. La IP pública `1.1.1.1` se configura igual que en el punto anterior: si el panel de Static NAT no existe, basta con anotar que el router hará el NAT estático cuando llegues a la fase correspondiente.
3. Guarda la configuración de cada servidor con el botón Save en Packet Tracer.

> Nota: en Packet Tracer sin panel de “Public Address” en los servidores, el requisito de IP pública se cumple aplicando `ip nat inside source static` en el router BS.AS. No intentes modificar la IP interna de los servidores; mantén 192.168.100.9/29 y 192.168.100.2/29 y documenta el mapeo público en `ticket_trabajo_practico.md`.

### 5.5 Checklist final
1. `ISP_INTERNACIONAL`: `show ip int brief` debe mostrar `G0/0` y `G0/1.100` `up/up`.  
2. `SW_MS_CORE`: VLAN 100 activa y `Gi0/1` en trunk con VLAN 100 permitida/nativa.  
3. Servidores: IPs internas configuradas (192.168.100.9 y 192.168.100.2) y servicios activos.  
4. Desde `ISP_INTERNACIONAL`, ejecuta `ping 164.25.0.2` para validar el enlace.  
5. Desde cada servidor, `ping 164.25.0.2` y `ping 45.162.20.10`/`1.1.1.1` para comprobar resolución interna.  
6. Si necesitas pruebas públicas, agrega destinos simulados (ej. `8.8.8.8`) y asegúrate de que el router tenga rutas adecuadas.

---

## 6. Plan de Pruebas
| Paso | Comando | Requerimiento validado |
|------|---------|-------------------------|
| LAN up | `ping 192.168.20.1` desde PC | Interfaces #2 |
| WAN1/2 (ISP_LOCAL) | `ping 42.25.25.1`, `ping 43.26.26.1` | Interfaces #3 |
| Internet | `ping 8.8.8.8` desde PC (cuando el router BS.AS esté disponible) | Ruteo estático #2 |

### 6.1 Checklist de verificación en Cisco Packet Tracer
Sigue cada bloque en orden. Si alguno falla, corrige antes de avanzar.

1. **PC-BS-AS**
	- `Desktop → Command Prompt → ping 192.168.20.1`. Debe responder sin pérdida.  
	- `ipconfig /all` para confirmar IP `192.168.20.10`, máscara `/24`, gateway `192.168.20.1`, DNS `1.1.1.1`.
2. **Switch SW-BS-AS**
	- CLI → `show vlan brief`: verifica VLAN 20/100/200 creadas y puertos Fa0/1 (PC) y trunk (`G0/2`, `Fa0/24`) correctamente asignados.  
	- `show interfaces trunk`: debe listar `G0/2` y `Fa0/24` con VLANs permitidas `20,100,200` y nativa 20.  
	- `show interfaces status`: comprueba que los puertos involucrados estén `connected`.
3. **Router ISP_LOCAL**
	- `show ip interface brief`: `G0/0`, `G0/1`, `G0/2` deben estar `up/up` con IPs `164.25.0.2`, `42.25.25.2`, `43.26.26.2`.  
	- `show ip route`: debe incluir rutas estáticas a `192.168.20.0/24` vía `.1` y default `0.0.0.0/0` hacia `164.25.0.1`.  
	- `ping 42.25.25.1`, `ping 43.26.26.1`, `ping 164.25.0.1`. Si fallan, revisa cables o que los peers estén configurados.  
	- Guarda evidencia con `show running-config | section interface g0` si necesitas documentar.
4. **Router ISP_INTERNACIONAL (dentro del icono Internet)**
	- `show ip interface brief`: confirma `G0/0 164.25.0.1/29 up/up` y `G0/1 192.168.10.1/29 up/up`.  
	- `ping 164.25.0.2` para validar el enlace con ISP_LOCAL.  
	- `show ip route` debe contener `S 192.168.20.0/24 [1/0] via 164.25.0.2`.
5. **Switch SW_MS_CORE**
	- `show vlan brief`: VLAN 100 activa con Fa0/1 (WEB) y Fa0/2 (DNS).  
	- `show interfaces trunk`: `Gi0/1` en modo trunk con VLAN 100 como permitida/nativa.  
	- LEDs verdes en `Fa0/1` y `Fa0/2` indican conexión a servidores.  
6. **Servidores**
	- En cada equipo, `Desktop → IP Configuration`: validar IP interna, máscara, gateway.  
	- `Command Prompt → ping 164.25.0.1` y `ping 164.25.0.2`.  
	- En `WEB-SERVER`, abre un navegador dentro del mismo equipo y carga `http://192.168.10.9` para confirmar el servicio.  
	- En `DNS-SERVER`, en la pestaña DNS crea/valida el registro `www.consultas.labo.com.ar` → `45.162.20.10` y realiza `nslookup` desde la PC o el mismo servidor.
7. **Pruebas cruzadas**
	- Desde PC-BS-AS, una vez que Router BS.AS esté operativo, ejecuta:  
	  - `ping 42.25.25.2`, `ping 43.26.26.2` para comprobar tránsito hasta ISP_LOCAL.  
	  - `ping 45.162.20.10` y `nslookup www.consultas.labo.com.ar 1.1.1.1`.  
	- Documenta resultados con capturas o exportando la consola (`Copy` → pegar en `ticket_trabajo_practico.md`).

---

- Guardar capturas de `show running-config`, `show ip route`, `show vlan brief`, `show interfaces trunk`.  
- Registrar resultados en `ticket_trabajo_practico.md` (Fase 1, segmento WAN) con referencia a los requerimientos cumplidos.
