# 🎓 GUÍA DE PRESENTACIÓN AL PROFESOR

Esta guía te ayudará a explicar tu red de manera clara, estructurada y profesional.

---

## 📖 Índice
1. [Estructura de la Presentación](#estructura)
2. [Introducción y Contexto](#introduccion)
3. [Explicación de Tecnologías](#tecnologias)
4. [Demostración de Funcionalidades](#demostracion)
5. [Preguntas Frecuentes](#preguntas)
6. [Consejos para la Presentación](#consejos)

---

## 🔷 1. Estructura de la Presentación {#estructura}

### **Duración Sugerida: 15-20 minutos**

**Fase 1: Introducción (3 min)**
- Presentación del proyecto
- Objetivos de la red
- Alcance y requerimientos

**Fase 2: Arquitectura (5 min)**
- Topología general
- Sitios y conexiones
- Tecnologías implementadas

**Fase 3: Tecnologías Clave (7 min)**
- OSPF y enrutamiento
- VLANs y segmentación
- NAT y seguridad
- Redundancia

**Fase 4: Demostración (3 min)**
- Flujos de tráfico
- Failover y redundancia
- ACLs en acción

**Fase 5: Conclusiones (2 min)**
- Logros alcanzados
- Desafíos superados
- Preguntas

---

## 🔷 2. Introducción y Contexto {#introduccion}

### **Apertura (30 segundos)**

> "Buenos días/tardes. Hoy voy a presentar una infraestructura de red empresarial multi-sitio que implementa tecnologías avanzadas de enrutamiento, seguridad y alta disponibilidad."

### **Contexto del Proyecto (1 min)**

**Qué decir:**
> "La red conecta tres ubicaciones geográficas: Buenos Aires (sede central), Córdoba y Mendoza (sucursales). El objetivo principal es proporcionar conectividad segura, redundante y escalable entre todos los sitios, con salida a Internet centralizada."

**Mostrar:** Diagrama de topología física

### **Requerimientos Principales (1.5 min)**

**Qué decir:**
> "Los requerimientos clave que debía cumplir la red son:
> 
> 1. **Enrutamiento dinámico** entre sucursales usando OSPF
> 2. **Segmentación de red** mediante VLANs para diferentes tipos de usuarios
> 3. **Traducción de direcciones (NAT)** para salida a Internet
> 4. **Control de acceso** mediante ACLs para proteger recursos críticos
> 5. **Alta disponibilidad** con múltiples niveles de redundancia
> 6. **Servicios centralizados** como DHCP para WiFi
> 7. **Servidores públicos** accesibles desde Internet (DNS y Web)"

---

## 🔷 3. Explicación de Tecnologías {#tecnologias}

### **3.1 OSPF - Enrutamiento Dinámico (2 min)**

**Qué decir:**
> "Para el enrutamiento entre sucursales, implementé OSPF (Open Shortest Path First), un protocolo de estado de enlace que permite convergencia automática ante fallas.
>
> **Características clave:**
> - Todos los routers están en Área 0 (backbone)
> - Enlaces P2P directos con costo 10 (rutas preferidas)
> - Enlace de backup por VLAN 1000 con costo 50
> - Router Buenos Aires inyecta la ruta por defecto hacia Internet
> - Interfaces LAN configuradas como 'passive' para seguridad"

**Mostrar:** Diagrama de topología OSPF

**Ejemplo práctico:**
> "Por ejemplo, si un PC en Córdoba quiere comunicarse con uno en Mendoza, OSPF calcula automáticamente la mejor ruta. Normalmente usa el enlace directo (costo 10), pero si este falla, OSPF reconverge en ~40 segundos usando la ruta alternativa vía Buenos Aires."

**Comando a mostrar:**
```
show ip route ospf
show ip ospf neighbor
```

---

### **3.2 VLANs - Segmentación de Red (1.5 min)**

**Qué decir:**
> "La red está segmentada en múltiples VLANs para separar diferentes tipos de tráfico:
>
> **Buenos Aires:**
> - VLAN 30: LAN principal
> - VLANs 100 y 200: Enlaces WAN redundantes
>
> **Córdoba:**
> - VLAN 10: Usuarios
> - VLAN 20: Servidores (con ACLs de protección)
>
> **Mendoza:**
> - VLAN 44: WiFi para empleados
> - VLAN 55: WiFi para invitados (segregado)
> - VLAN 70: Management (VLAN nativa)
>
> Esta segmentación mejora la seguridad, reduce el dominio de broadcast y facilita la gestión."

**Mostrar:** Diagrama de VLANs

**Técnica implementada:**
> "Utilicé Router-on-a-Stick con subinterfaces 802.1Q para enrutar entre VLANs, lo que permite usar un solo enlace físico para múltiples VLANs."

---

### **3.3 NAT - Traducción de Direcciones (1.5 min)**

**Qué decir:**
> "Para la salida a Internet, implementé dos tipos de NAT:
>
> **1. NAT Overload (PAT) en Buenos Aires:**
> - Todas las redes internas (192.168.x.x) se traducen a una IP pública
> - Configuré dos interfaces de salida redundantes (WAN1 y WAN2)
> - Si WAN1 falla, el tráfico automáticamente usa WAN2
>
> **2. NAT Estático en ISP Internacional:**
> - DNS Server: 192.168.100.2 → 1.1.1.1 (accesible desde Internet)
> - Web Server: 192.168.100.9 → 45.162.20.10 (accesible desde Internet)
>
> Esto permite conservar direcciones IP públicas y proporcionar servicios públicos."

**Mostrar:** Flujo de NAT

**Comando a mostrar:**
```
show ip nat translations
show ip nat statistics
```

---

### **3.4 ACLs - Control de Acceso (1.5 min)**

**Qué decir:**
> "Para proteger el servidor de archivos en Córdoba (192.168.20.10), implementé ACLs extendidas en múltiples puntos:
>
> **Requerimiento:** Solo el PC de Buenos Aires (192.168.30.10) puede acceder al servidor.
>
> **Implementación:**
> 1. **ACL en Router Córdoba (interfaz VLAN 20):**
>    - Permite tráfico desde 192.168.30.10
>    - Deniega todo lo demás hacia el servidor
>
> 2. **ACL en Router Córdoba (interfaz P2P):**
>    - Bloquea FTP desde Mendoza (VLANs 44, 55, 70)
>
> 3. **ACL en Switch ACC-CORD (puerto del servidor):**
>    - Defensa en profundidad a nivel de capa 2
>
> Esta estrategia de múltiples capas asegura que el servidor esté protegido incluso si una ACL falla."

**Mostrar:** Flujo de tráfico bloqueado vs permitido

**Comando a mostrar:**
```
show access-lists
show ip interface GigabitEthernet0/2.20
```

---

### **3.5 Redundancia y Alta Disponibilidad (1 min)**

**Qué decir:**
> "La red implementa redundancia en tres niveles:
>
> **Nivel 1: WAN Redundante**
> - Dos enlaces independientes a ISP Local
> - Failover automático usando distancias administrativas (AD)
> - WAN1: AD=1 (primario), WAN2: AD=5 (backup)
>
> **Nivel 2: OSPF Redundante**
> - Enlaces P2P directos (costo 10)
> - Enlace de backup por VLAN 1000 (costo 50)
> - Convergencia automática en ~40 segundos
>
> **Nivel 3: Spanning Tree (STP)**
> - Prevención de loops en capa 2
> - Root Bridges configurados por VLAN
> - PortFast en puertos de acceso para convergencia rápida
>
> Esto garantiza que la red permanezca operativa incluso ante múltiples fallas."

**Mostrar:** Diagrama de redundancia

---

## 🔷 4. Demostración de Funcionalidades {#demostracion}

### **Demo 1: Conectividad entre Sucursales (1 min)**

**Qué hacer:**
1. Hacer ping desde PC Córdoba a PC Mendoza
2. Mostrar `traceroute` para ver la ruta tomada
3. Explicar que OSPF eligió la ruta directa (menor costo)

**Qué decir:**
> "Aquí vemos un ping exitoso desde Córdoba a Mendoza. El traceroute muestra que el tráfico va directamente entre los routers, gracias a OSPF que calculó la mejor ruta."

**Comandos:**
```
ping 192.168.44.20 (desde PC Córdoba)
tracert 192.168.44.20
```

---

### **Demo 2: Failover de WAN (1 min)**

**Qué hacer:**
1. Mostrar tabla de enrutamiento con ruta por defecto vía WAN1
2. Apagar interfaz WAN1
3. Mostrar que la ruta cambia automáticamente a WAN2

**Qué decir:**
> "Inicialmente, la ruta por defecto apunta a WAN1. Si simulo una falla apagando la interfaz, en menos de 2 segundos la ruta cambia automáticamente a WAN2, sin intervención manual."

**Comandos:**
```
show ip route | include 0.0.0.0
interface GigabitEthernet0/1.100
shutdown
show ip route | include 0.0.0.0
```

---

### **Demo 3: ACL Bloqueando FTP (1 min)**

**Qué hacer:**
1. Intentar FTP desde PC Córdoba al servidor → Bloqueado
2. Intentar FTP desde PC Buenos Aires al servidor → Permitido

**Qué decir:**
> "Aquí vemos la ACL en acción. Cuando un usuario de Córdoba intenta acceder al servidor FTP, la conexión es bloqueada. Sin embargo, el PC autorizado de Buenos Aires puede acceder sin problemas."

**Comandos:**
```
telnet 192.168.20.10 21 (desde PC Córdoba) → Connection refused
telnet 192.168.20.10 21 (desde PC BS.AS) → Connected
```

---

## 🔷 5. Preguntas Frecuentes {#preguntas}

### **Pregunta 1: ¿Por qué usar OSPF en lugar de rutas estáticas?**

**Respuesta:**
> "OSPF ofrece convergencia automática ante fallas. Con rutas estáticas, si un enlace cae, tendría que reconfigurar manualmente todos los routers. OSPF detecta la falla y recalcula rutas automáticamente en ~40 segundos, lo que mejora la disponibilidad de la red."

---

### **Pregunta 2: ¿Qué pasa si falla el router de Buenos Aires?**

**Respuesta:**
> "Buenos Aires es el gateway principal a Internet, por lo que su falla afectaría la salida a Internet. Sin embargo, la comunicación entre Córdoba y Mendoza seguiría funcionando gracias a OSPF, que enrutaría el tráfico directamente entre ellos. Para mejorar esto, podría implementar un segundo router en Buenos Aires con HSRP/VRRP para redundancia de gateway."

---

### **Pregunta 3: ¿Por qué usar VLAN 70 como nativa en Mendoza?**

**Respuesta:**
> "La VLAN nativa (70) es la VLAN de management. Al configurarla como nativa, el tráfico de gestión (CDP, VTP, acceso a switches) viaja sin etiqueta, lo que simplifica la configuración y es una práctica común para VLANs de administración."

---

### **Pregunta 4: ¿Cómo se asegura que solo Buenos Aires acceda al servidor?**

**Respuesta:**
> "Implementé ACLs extendidas en tres puntos:
> 1. Router Córdoba (interfaz VLAN 20): Permite solo 192.168.30.10
> 2. Router Córdoba (interfaz P2P): Bloquea FTP desde Mendoza
> 3. Switch ACC-CORD (puerto del servidor): Defensa adicional
>
> Esta estrategia de 'defensa en profundidad' asegura que incluso si una ACL falla, las otras siguen protegiendo el servidor."

---

### **Pregunta 5: ¿Qué es el costo OSPF y cómo se calcula?**

**Respuesta:**
> "El costo OSPF es la métrica que OSPF usa para elegir la mejor ruta. Se calcula como:
> 
> Costo = Ancho de Banda de Referencia / Ancho de Banda de la Interfaz
>
> Por defecto, el ancho de banda de referencia es 100 Mbps. En mi red:
> - Enlaces P2P (1 Gbps): Costo = 100/1000 = 0.1 → Configuré manualmente a 10
> - Enlace de backup (VLAN 1000): Configuré manualmente a 50 para que sea secundario
>
> OSPF siempre elige la ruta con menor costo total."

---

### **Pregunta 6: ¿Qué es NAT Overload (PAT)?**

**Respuesta:**
> "NAT Overload, también llamado PAT (Port Address Translation), permite que múltiples dispositivos internos compartan una sola IP pública usando diferentes puertos.
>
> Por ejemplo:
> - PC1 (192.168.30.5:50000) → 42.25.25.1:1024
> - PC2 (192.168.10.8:50001) → 42.25.25.1:1025
>
> El router mantiene una tabla de traducciones para saber a qué dispositivo interno enviar las respuestas. Esto permite miles de conexiones simultáneas con una sola IP pública."

---

### **Pregunta 7: ¿Qué es un Designated Router (DR) en OSPF?**

**Respuesta:**
> "En redes broadcast (como VLAN 1000), OSPF elige un DR (Designated Router) y un BDR (Backup DR) para centralizar la distribución de LSAs (Link State Advertisements).
>
> En mi red:
> - Buenos Aires: Prioridad 100 → DR
> - Córdoba/Mendoza: Prioridad 50 → Uno será BDR
>
> Esto reduce el tráfico OSPF, ya que los routers solo intercambian LSAs con el DR, no entre todos."

---

### **Pregunta 8: ¿Qué es DHCP Relay y por qué se usa?**

**Respuesta:**
> "DHCP Relay (IP Helper) permite que un router reenvíe solicitudes DHCP broadcast a un servidor DHCP en otra red.
>
> En Mendoza:
> - Clientes WiFi (VLAN 44/55) envían DHCP Discover (broadcast)
> - Router Mendoza convierte el broadcast a unicast hacia 192.168.70.10
> - Servidor DHCP (VLAN 70) responde
>
> Esto permite centralizar el servidor DHCP en lugar de tener uno por VLAN."

---

### **Pregunta 9: ¿Qué es Spanning Tree y por qué es necesario?**

**Respuesta:**
> "Spanning Tree Protocol (STP) previene loops en capa 2 bloqueando puertos redundantes.
>
> Sin STP, si hay dos caminos entre switches, los frames de broadcast circularían infinitamente (broadcast storm), colapsando la red.
>
> STP crea un árbol lógico sin loops eligiendo un Root Bridge y bloqueando puertos estratégicamente. En mi red, configuré prioridades para controlar qué switch es Root Bridge por VLAN."

---

### **Pregunta 10: ¿Qué es una Passive Interface en OSPF?**

**Respuesta:**
> "Una passive interface es una interfaz donde OSPF NO envía paquetes Hello, pero SÍ anuncia la red conectada.
>
> La configuré en todas las interfaces LAN (VLANs de usuarios) por dos razones:
> 1. **Seguridad:** No exponer OSPF a usuarios finales
> 2. **Eficiencia:** No desperdiciar ancho de banda enviando Hellos a PCs
>
> Las redes siguen siendo anunciadas en OSPF, pero no se forman adyacencias en esas interfaces."

---

## 🔷 6. Consejos para la Presentación {#consejos}

### **Antes de la Presentación**

✅ **Practica varias veces** con el material  
✅ **Verifica que todos los dispositivos estén operativos**  
✅ **Prepara comandos de demostración** en un documento  
✅ **Ten diagramas impresos** como respaldo  
✅ **Conoce bien los conceptos teóricos**  
✅ **Prepara respuestas a preguntas comunes**  

### **Durante la Presentación**

✅ **Habla con confianza** y claridad  
✅ **Usa terminología técnica correcta**  
✅ **Explica el "por qué", no solo el "qué"**  
✅ **Relaciona conceptos teóricos con la implementación**  
✅ **Mantén contacto visual** con el profesor  
✅ **No leas las diapositivas**, úsalas como apoyo  

### **Al Responder Preguntas**

✅ **Escucha atentamente** la pregunta completa  
✅ **Tómate un momento** para pensar antes de responder  
✅ **Si no sabes algo, admítelo** honestamente  
✅ **Relaciona la respuesta** con lo implementado  
✅ **Usa ejemplos concretos** de tu red  

### **Lenguaje Corporal**

✅ **Postura erguida** y profesional  
✅ **Gestos naturales** para enfatizar puntos  
✅ **Evita cruzar los brazos**  
✅ **Sonríe** cuando sea apropiado  
✅ **Muestra entusiasmo** por tu trabajo  

---

## 🎯 Estructura de Diapositivas Sugerida

### **Diapositiva 1: Título**
- Nombre del proyecto
- Tu nombre
- Fecha

### **Diapositiva 2: Objetivos**
- Conectividad multi-sitio
- Alta disponibilidad
- Seguridad
- Escalabilidad

### **Diapositiva 3: Topología General**
- Diagrama de topología física
- Ubicaciones (BS.AS, Córdoba, Mendoza)
- ISPs

### **Diapositiva 4: Tecnologías Implementadas**
- OSPF
- VLANs
- NAT/PAT
- ACLs
- STP
- DHCP Relay

### **Diapositiva 5: OSPF**
- Diagrama de área 0
- Router IDs
- Costos de enlaces
- Redundancia

### **Diapositiva 6: VLANs**
- Tabla de VLANs por sitio
- Router-on-a-Stick
- Segmentación

### **Diapositiva 7: NAT**
- NAT Overload (PAT)
- NAT Estático
- Redundancia WAN

### **Diapositiva 8: ACLs**
- Requerimiento de seguridad
- Implementación multicapa
- Flujo bloqueado vs permitido

### **Diapositiva 9: Redundancia**
- Niveles de redundancia
- Tiempos de convergencia
- Escenarios de falla

### **Diapositiva 10: Demostración**
- Capturas de pantalla
- Comandos clave
- Resultados

### **Diapositiva 11: Conclusiones**
- Logros alcanzados
- Desafíos superados
- Aprendizajes

### **Diapositiva 12: Preguntas**
- "¿Preguntas?"
- Información de contacto

---

## 📝 Script de Ejemplo (Apertura)

> "Buenos días/tardes, profesor. Mi nombre es [TU NOMBRE] y hoy voy a presentar una infraestructura de red empresarial multi-sitio que diseñé e implementé.
>
> El proyecto consiste en conectar tres ubicaciones geográficas: Buenos Aires, que actúa como sede central, y dos sucursales en Córdoba y Mendoza. La red implementa tecnologías avanzadas como OSPF para enrutamiento dinámico, VLANs para segmentación, NAT para salida a Internet, ACLs para seguridad, y múltiples niveles de redundancia para alta disponibilidad.
>
> El objetivo principal es proporcionar conectividad segura, eficiente y resiliente entre todos los sitios, con capacidad de recuperación automática ante fallas.
>
> Voy a comenzar explicando la arquitectura general de la red..."

---

## 🎓 Frases Clave para Impresionar

### **Al Hablar de OSPF:**
> "OSPF es un protocolo de estado de enlace que utiliza el algoritmo de Dijkstra para calcular el árbol de rutas más cortas, lo que garantiza convergencia óptima ante cambios en la topología."

### **Al Hablar de VLANs:**
> "La segmentación mediante VLANs no solo mejora la seguridad al aislar dominios de broadcast, sino que también optimiza el rendimiento de la red al reducir el tráfico innecesario."

### **Al Hablar de NAT:**
> "Implementé NAT Overload (PAT) para conservar direcciones IPv4 públicas, permitiendo que cientos de dispositivos internos compartan una sola IP pública mediante multiplexación de puertos."

### **Al Hablar de ACLs:**
> "Apliqué el principio de 'defensa en profundidad' implementando ACLs en múltiples capas: router, switch y puerto, asegurando que el servidor esté protegido incluso ante fallas de configuración."

### **Al Hablar de Redundancia:**
> "La red implementa redundancia activa-pasiva en tres niveles: WAN con failover automático mediante distancias administrativas, OSPF con múltiples rutas y costos diferenciados, y STP para prevención de loops en capa 2."

---

## ✅ Checklist Final

**Antes de Presentar:**
- [ ] Todos los dispositivos están encendidos y operativos
- [ ] Las configuraciones están guardadas (write memory)
- [ ] Los comandos de demostración están preparados
- [ ] Los diagramas están listos
- [ ] Has practicado al menos 3 veces
- [ ] Conoces las respuestas a las 10 preguntas frecuentes
- [ ] Tienes agua cerca (para no quedarte sin voz)
- [ ] Llegas 10 minutos antes

**Durante la Presentación:**
- [ ] Presentación clara y estructurada
- [ ] Uso correcto de terminología técnica
- [ ] Demostraciones exitosas
- [ ] Respuestas seguras a preguntas
- [ ] Tiempo controlado (15-20 min)

**Después de la Presentación:**
- [ ] Agradece al profesor por su tiempo
- [ ] Ofrece documentación adicional si la solicita
- [ ] Toma nota de feedback para mejorar

---

## 🌟 Mensaje Final

**Recuerda:**
- Has hecho un excelente trabajo implementando esta red
- Conoces tu proyecto mejor que nadie
- La confianza viene de la preparación
- Es normal estar nervioso, pero confía en tu conocimiento
- El profesor quiere verte tener éxito

**¡Mucha suerte en tu presentación!** 🚀

---

**Esta guía te prepara para explicar tu red de manera profesional y responder cualquier pregunta con confianza.**
