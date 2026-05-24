---
tags:
  - protos
  - practica
  - enlace
  - arp
  - ethernet
  - guia
---

# Guía Práctica: Capa de Enlace (ARP, Ethernet)

---

## ARP — comandos esenciales

```sh
# Ver tabla ARP (IP ↔ MAC)
arp -n
arp -a

# Agregar entrada estática a la tabla ARP
arp -s <ip> <mac>                  # sin interfaz
arp -s <ip> <mac> -i <interfaz>    # con interfaz específica
arp -s <ip> <mac> temp             # temporal (BSD/macOS)

# Borrar una entrada
sudo arp -d <ip>
```

→ Ver [[guia_respuestas#E96|E96]]

---

## arping — enviar ARP requests manuales

```sh
sudo apt install arping

# Argumentos disponibles
arping [-t <target-mac>] [-T <target-ip>] [-s <source-mac>] [-S <source-ip>] <destino>

# Ejemplo: verificar que una IP está en uso
sudo arping -I <interfaz> <ip>
```

> arping no modifica tu tabla ARP ni requiere tener la IP.

→ Ver [[Laboratorio#arping]]

---

## Cómo funciona ARP

1. Host A quiere enviar a B (misma red), no tiene su MAC.
2. A manda **ARP Request** en broadcast (`ff:ff:ff:ff:ff:ff`): "¿Quién tiene IP X?"
3. B responde con **ARP Reply** unicast: "Yo tengo IP X, mi MAC es YY:YY:YY:YY:YY:YY"
4. A guarda la entrada en su tabla ARP y envía la trama.

**Si el destino está en otra red**: A hace ARP al **Gateway** (router), no al host final.

---

## Forwarding IP — qué cambia y qué no en cada salto

| Campo | ¿Cambia en forwarding? |
|---|---|
| **Ethernet src MAC** | ✅ sí — pasa a ser la MAC del router |
| **Ethernet dst MAC** | ✅ sí — pasa a ser la MAC del próximo salto |
| **IP src** | ❌ no — sigue siendo el origen original |
| **IP dst** | ❌ no — sigue siendo el destino final |
| **IP TTL** | ✅ sí — se decrementa en 1 |

> Regla de oro: **Ethernet es local** (MACs cambian en cada salto). **IP es end-to-end** (IPs no cambian).

→ Ver [[guia_respuestas#E97|E97]]

---

## ARP Spoofing (MITM)

Para interceptar el tráfico entre dos hosts (ej. H y el router R):

```sh
# Decirle a H que vos sos R (poner tu MAC con la IP del router)
# Esto se hace con arping (o arpspoof de dsniff)
sudo arping -S <ip_router> <ip_H>

# Habilitar forwarding para que el tráfico pase igual
sysctl net.ipv4.ip_forward=1

# Agregar SNAT para que el router no descarte los paquetes
iptables -t nat -A POSTROUTING -j MASQUERADE
```

→ Ver [[guia_respuestas#E98|E98]], [[Laboratorio#Enlace]]

---

## Estructura de trama Ethernet

```
| Preámbulo (8B) | MAC destino (6B) | MAC origen (6B) | Tipo (2B) | Datos (46-1500B) | CRC (4B) |
```

- **MAC broadcast**: `ff:ff:ff:ff:ff:ff`
- **CRC**: verifica que la trama llegó sin errores
- **MTU**: máximo 1500 bytes de datos (sin headers Ethernet)

---

## CSMA/CD — detección de colisiones (Ethernet cableado)

Para que una colisión sea detectable, la trama debe tardar más en transmitirse de lo que tarda la señal en viajar de un extremo al otro y volver:

```
Condición: L/R > 2 × d_prop

L = tamaño de trama en bits
R = tasa de transmisión
d_prop = tiempo de propagación end-to-end
```

Por eso Ethernet impone:
- **Tamaño mínimo de trama: 64 bytes** (para que L/R sea suficientemente grande)
- **Longitud máxima de cable** (para que d_prop sea pequeño)

→ Ver [[notas/8_enlace#Pregunta 12|Pregunta Enlace 12]]

---

## Escenarios comunes con ARP

### Caso: entrada ARP estática apuntando a MAC equivocada

Si se agrega `arp -s <ip_buena> <mac_mala>`:
- Los paquetes a `<ip_buena>` se envían a la MAC equivocada
- El host real con `<ip_buena>` nunca los recibe
- Si hay forwarding habilitado en el host con `<mac_mala>`, podría redirigir los paquetes

### Caso: dos entradas ARP para la misma IP

Solo queda una: la tabla ARP asocia una IP con una única MAC. Si intentás agregar dos, la segunda pisa la primera.

→ Ver [[guia_respuestas#E96|E96]] puntos e, f, g, h, j
