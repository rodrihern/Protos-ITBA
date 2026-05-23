---
tags:
  - protos
  - practica
  - transporte
  - tcp
  - udp
  - guia
---

# Guía Práctica: Capa de Transporte (TCP/UDP)

---

## Netcat — herramienta universal

```sh
# Escuchar en un puerto (TCP)
nc -l <puerto>
nc -l <puerto> > archivo_recibido.txt   # guardar lo que llega

# Conectarse a un host:puerto
nc <ip> <puerto>

# Enviar contenido de un archivo
cat archivo.txt | nc <ip> <puerto>

# UDP: agregar -u
nc -u -l <puerto>
cat archivo.txt | nc -u <ip> <puerto>

# CRLF (para protocolos como SMTP/HTTP)
nc -C localhost 25

# Mantener el servidor abierto después de la primera conexión
nc -kl <puerto>
```

→ Ver [[Respuestas_guia#E74|E74]]

---

## nmap — escaneo de red y puertos

```sh
# Escaneo básico de puertos
nmap <ip>
nmap -p 22,80,443 <ip>          # puertos específicos
nmap -p 1-1024 <ip>             # rango
nmap -p- <ip>                    # todos los puertos (1-65535)
nmap -p 49000-50000 <ip>         # rango acotado

# Tipos de escaneo
sudo nmap -sS <ip>               # SYN scan (stealth, default, no completa handshake)
nmap -sT <ip>                    # TCP connect (sin sudo, completa handshake)
sudo nmap -sU <ip>               # UDP scan (lento)
sudo nmap -sn 192.168.0.0/24     # solo ver qué hosts están up (sin escanear puertos)

# Detección de servicios y OS
sudo nmap -sV <ip>               # detectar versiones de servicios
sudo nmap -O <ip>                # detectar sistema operativo
sudo nmap -A <ip>                # agresivo: versiones + OS + scripts + traceroute

# Velocidad
sudo nmap -T4 <ip>               # más rápido (T0=paranoid, T5=insane, default=T3)
```

### Estados de un puerto en nmap

| Estado | Significado |
|---|---|
| `open` | Hay un servicio escuchando |
| `closed` | Respondió con RST (no hay servicio) |
| `filtered` | Sin respuesta (probable firewall) |
| `open\|filtered` | No se puede distinguir (típico de UDP) |

→ Ver [[Respuestas_guia#E77|E77]], [[Respuestas_guia#E78|E78]], [[Ejercicio_integrador#The Vaullt]]

---

## Puertos de servicios comunes

| Servicio | Puerto | Protocolo |
|---|---|---|
| FTP | 21 | TCP |
| SSH | 22 | TCP |
| Telnet | 23 | TCP |
| SMTP | 25 | TCP |
| DNS | 53 | UDP/TCP |
| HTTP | 80 | TCP |
| POP3 | 110 | TCP |
| IMAP | 143 | TCP |
| HTTPS | 443 | TCP |
| daytime | 13 | TCP/UDP |
| time | 37 | TCP/UDP |

---

## TCP: 3-way handshake y cierre

### Apertura (3-way handshake)

```
Cliente  →  Servidor: SYN (Seq=x)
Servidor →  Cliente:  SYN+ACK (Seq=y, Ack=x+1)
Cliente  →  Servidor: ACK (Ack=y+1)
```

### Cierre (4-way)

```
A → B: FIN
B → A: ACK
B → A: FIN
A → B: ACK
```

### Flags TCP

| Flag | Uso |
|---|---|
| `SYN` | Iniciar conexión |
| `ACK` | Confirmación de recepción |
| `FIN` | Cerrar conexión de forma ordenada |
| `RST` | Cortar conexión abruptamente |
| `PSH` | Push datos directamente a la aplicación |
| `URG` | Datos urgentes |

---

## TCP vs UDP: cuándo usar cada uno

| | TCP | UDP |
|---|---|---|
| Orientado a conexión | ✅ | ❌ |
| Confiable (garantiza entrega) | ✅ | ❌ |
| Control de flujo y congestión | ✅ | ❌ |
| Overhead | Mayor (20 bytes header) | Menor (8 bytes header) |
| Demultiplexación | (IP src, puerto src, IP dst, puerto dst) | (IP dst, puerto dst) |
| Uso típico | HTTP, SSH, SMTP, FTP | DNS, streaming, VoIP |

---

## traceroute / tracepath

```sh
traceroute <ip o dominio>       # muestra cada salto (usa UDP por default)
traceroute -I <destino>         # usa ICMP en lugar de UDP
traceroute -T <destino>         # usa TCP
tracepath <destino>             # alternativa más simple
```

Funciona enviando paquetes con TTL=1, 2, 3... Los routers intermedios devuelven ICMP Time Exceeded cuando el TTL llega a 0.

→ Ver [[Respuestas_guia#E79|E79]], [[Respuestas_guia#E80|E80]]

---

## Fragmentación IP

Si un paquete supera el **MTU** (Maximum Transmission Unit, típicamente 1500 bytes en Ethernet):
- IP fragmenta el paquete
- Los fragmentos tienen el mismo **ID**
- El campo **Offset** indica la posición de cada fragmento (en bloques de 8 bytes)
- El flag **MF** (More Fragments) = 1 en todos excepto el último

Overhead de headers: UDP=8B, IPv4=20B, Ethernet=24B → payload disponible = MTU − 52

→ Ver [[Respuestas_guia#E81|E81]], [[notas/6_Red#Pregunta 3|Pregunta Red 3]]

---

## Wireshark: qué buscar

| Situación | Qué ver en Wireshark |
|---|---|
| TCP conectado | SYN → SYN+ACK → ACK al inicio |
| TCP rechazado | SYN → RST+ACK |
| UDP enviado | Solo los datagramas, sin handshake |
| Forwarding IP | Las MACs cambian en cada salto, las IPs no |
| DHCP | DISCOVER → OFFER → REQUEST → ACK |
| ARP | Broadcast request → unicast reply |

→ Ver [[Respuestas_guia#E73|E73]], [[Respuestas_guia#E75|E75]], [[Respuestas_guia#E76|E76]]

---

## Tabla de estados TCP (lo que muestra `netstat`)

```sh
netstat -tulpn        # ver puertos escuchando
netstat -an           # ver todas las conexiones con estados
```

| Estado | Significado |
|---|---|
| `LISTEN` | Esperando conexiones entrantes |
| `SYN_SENT` | SYN enviado, esperando SYN+ACK |
| `ESTABLISHED` | Conexión activa |
| `TIME_WAIT` | Esperando que expiren paquetes en tránsito |
| `CLOSE_WAIT` | Recibió FIN del otro extremo |

---

## ICMP

```sh
ping <ip>                  # envía ICMP Echo Request, espera Echo Reply
ping -c 4 <ip>             # solo 4 pings
ping -s 4000 <ip>          # paquetes grandes (para probar fragmentación)
```

Tipos de mensajes ICMP relevantes:
- **Echo Request/Reply** (ping)
- **Destination Unreachable** (el destino no es alcanzable)
- **Time Exceeded** (TTL=0 → usado por traceroute)
- **Redirect** (hay un gateway mejor)
