# Registro de cambio de configuración - SW-BS.AS y Router BS.AS

## Fecha: 20/11/2025

### Problema detectado
La PC de la red 192.168.30.0/24 (LAN de Buenos Aires) no tenía conectividad con el gateway (192.168.30.1) ni con el resto de la red. El router y la subinterfaz estaban correctamente configurados, pero el trunk entre el switch y el router tenía la native VLAN 30, lo que causaba que el tráfico de la VLAN 30 llegara sin tag al router, y la subinterfaz `G0/1.30` esperaba tráfico taggeado.

### Solución aplicada
- Se modificó la configuración del trunk en el switch SW-BS.AS (puerto GigabitEthernet0/2) para que la native VLAN vuelva a ser la 1 (por defecto), asegurando que todo el tráfico de la VLAN 30 viaje taggeado hacia el router.

#### Comando ejecutado en el switch:
```
interface GigabitEthernet0/2
 no switchport trunk native vlan 30
 switchport trunk native vlan 1
```

### Resultado
- La PC pudo hacer ping al gateway 192.168.30.1 y a todos los destinos de la red (WAN, Internet, OSPF, backup, servidores).
- La red de Buenos Aires quedó completamente funcional.

---

> Documentar este cambio permite recordar que, cuando se usan subinterfaces dot1Q en el router, la native VLAN del trunk debe ser la 1 (o la que corresponda a una subinterfaz sin encapsulation dot1Q, si existiera).
