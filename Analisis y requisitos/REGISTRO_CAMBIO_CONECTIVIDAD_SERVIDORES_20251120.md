# Registro de Cambios y Solución de Conectividad - Servidores WEB/DNS y Sedes Internas

## Fecha: 20/11/2025

### Problema detectado
Las PCs de Córdoba (y otras sedes) no podían acceder a los servidores WEB (192.168.100.9) y DNS (192.168.100.2) ni a Internet, a pesar de que Buenos Aires sí tenía acceso. El problema se debía a rutas de retorno y rutas hacia los servidores incompletas en los routers ISP_LOCAL e ISP_INTERNACIONAL.

---

### Solución aplicada

#### 1. En BS.AS
- Se verificó y mantuvo:
  - Rutas estáticas para 192.168.100.0/29 y 192.168.100.8/29 apuntando a ISP_LOCAL.
  - NAT dinámico (PAT) para las redes internas.
  - NAT estático para los servidores WEB y DNS.

#### 2. En ISP_LOCAL
- Se agregaron rutas para los servidores apuntando a ISP_INTERNACIONAL:
  - `ip route 192.168.100.0 255.255.255.248 164.25.0.1`
  - `ip route 192.168.100.8 255.255.255.248 164.25.0.1`
- Se mantuvieron rutas de retorno para todas las redes internas apuntando a BS.AS.

#### 3. En ISP_INTERNACIONAL
- Se verificó la existencia de subinterfaces para VLAN 100 (DNS) y VLAN 101 (WEB).
- Se agregaron rutas de retorno para todas las redes internas apuntando a ISP_LOCAL:
  - `ip route 192.168.10.0 255.255.255.0 164.25.0.2`
  - `ip route 192.168.20.0 255.255.255.0 164.25.0.2`
  - `ip route 192.168.30.0 255.255.255.0 164.25.0.2`
  - `ip route 172.20.10.0 255.255.255.248 164.25.0.2`

#### 4. En SW_MS_CORE
- Se verificó que los puertos hacia los servidores estén en access en la VLAN correcta (100 para DNS, 101 para WEB).
- El trunk hacia ISP_INTERNACIONAL permite VLANs 100 y 101.

#### 5. En los servidores
- El DNS Server (192.168.100.2) tiene como gateway 192.168.100.1 (ISP_INTERNACIONAL).
- El WEB Server (192.168.100.9) tiene como gateway 192.168.100.14 (ISP_INTERNACIONAL).

---

### Resultado
- Las PCs de Córdoba y otras sedes pueden acceder a los servidores WEB y DNS y a Internet.
- El tráfico de respuesta de los servidores vuelve correctamente a cualquier sede interna.
- La red cumple con los requerimientos de ruteo, NAT y segmentación definidos en las guías y requerimientos del trabajo práctico.

---

> Documentar estos cambios permite recordar la importancia de las rutas de retorno y la correcta segmentación de VLANs y gateways en escenarios multi-sede y multi-ISP.
