# Guía 06 · NAT para todas las sedes
**Fecha:** 20/11/2025

## Objetivo
Configurar NAT (PAT) en el Router BS.AS para que todas las redes locales (Buenos Aires, Córdoba y Mendoza) puedan salir a Internet, cumpliendo los requerimientos del profesor.

---

## 1. Ampliar la ACL de NAT


```cisco
enable
configure terminal
access-list 1 permit 192.168.30.0 0.0.0.255
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.44.0 0.0.0.255
access-list 1 permit 192.168.55.0 0.0.0.255
access-list 1 permit 192.168.70.0 0.0.0.255
exit
```

---


## 2. Configurar NAT Overload en ambas salidas WAN

> **Importante:** Estos comandos deben ejecutarse en modo de configuración global (el prompt debe mostrar `(config)#`).

```cisco
ip nat inside source list 1 interface g0/1.100 overload
ip nat inside source list 1 interface g0/1.200 overload
```

---

## 3. Verificar NAT

```cisco
show ip nat translations
show ip nat statistics
```

---

## 4. Notas
- Si ya existe una ACL 1, edítala para incluir todas las redes locales.
- Esta configuración permite que PC2 y FILE SERVER de Córdoba, así como dispositivos de Mendoza, tengan salida a Internet.
- Cumple con los requerimientos del profesor: "natear ambos tráficos con la IP de la interfaz que corresponda" para todas las LAN.
