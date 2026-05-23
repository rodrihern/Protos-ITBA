---
tags:
  - protos
  - practica
  - guia
  - indice
---

# Índice — Apuntes Prácticos 72.07 Protocolos de Comunicación

Referencia de comandos y ejercicios para el parcial práctico.

---

## Temas

| # | Tema | Archivo | Temas clave |
|---|---|---|---|
| 01 | HTTP | [[01_http]] | curl, netcat, nginx, caché, códigos, headers |
| 02 | DNS | [[02_dns]] | dig, bind9, registros, /etc/hosts |
| 03 | Mail | [[03_mail]] | SMTP, POP3, MIME, base64, SPF/DKIM |
| 04 | Transporte | [[04_transporte]] | TCP, UDP, nmap, traceroute, netcat |
| 05 | Red / IP | [[05_red]] | subnetting, DHCP, NAT, iptables, routing |
| 06 | Enlace / ARP | [[06_enlace]] | arp, arping, Ethernet, forwarding |
| 07 | SSH | [[07_ssh]] | claves, SCP, tunnels -L/-R/-D |
| 08 | Sockets (C) | [[08_sockets]] | Berkeley API, TCP/UDP server/client, select |

---

## Ejercicios de referencia

- [[Respuestas_guia]] — ejercicios resueltos de la guía (E1 → E104)
- [[Laboratorio]] — comandos y configuraciones del laboratorio
- [[Ejercicio_integrador]] — ejercicio integrador completo (HTTP + DNS + SSH + nmap + base64)

---

## Cheatsheet rápido por herramienta

### curl
```sh
curl <url>                                              # GET
curl -X POST <url> -H "Content-Type: application/json" -d '{"k":"v"}'
curl -H "Accept-Language: es" <url>                    # negociación de contenido
curl -H "If-None-Match: \"etag\"" <url>                # caché condicional
curl -i <url>                                          # ver headers de respuesta
curl -I <url>                                          # solo headers (HEAD)
```

### nmap
```sh
sudo nmap -sS <ip>                    # stealth scan
sudo nmap -sU <ip>                    # UDP scan
sudo nmap -sV <ip>                    # detección de versiones
sudo nmap -p 49000-50000 <ip>         # rango de puertos
sudo nmap -O <ip>                     # detección de OS
```

### dig
```sh
dig <dominio>                         # A record
dig <dominio> MX                      # servidores de mail
dig <dominio> SOA                     # info de zona (serial)
dig <dominio> +trace                  # traza completa
dig <dominio> @<dns-server>           # usar un DNS específico
```

### SSH tunnels
```sh
ssh -L <local>:<host>:<remoto> user@server   # acceder al remoto desde local
ssh -R <remoto>:<host>:<local> user@server   # exponer local en el servidor
ssh -D <local> user@server                   # proxy SOCKS
```

### netcat
```sh
nc -l <puerto>                        # escuchar (TCP)
nc -u -l <puerto>                     # escuchar (UDP)
nc -C localhost 25                    # SMTP (con CRLF)
cat archivo | nc <ip> <puerto>        # enviar archivo
```

### iptables NAT
```sh
iptables -t nat -A POSTROUTING -o <iface> -j MASQUERADE          # SNAT
iptables -t nat -A PREROUTING -p tcp --dport <p> -j DNAT --to <ip>:<p>  # DNAT
iptables -t nat -L --line-numbers                                 # ver reglas
iptables -t nat -D <nro>                                          # borrar regla
```

### arp
```sh
arp -n                                # ver tabla
arp -s <ip> <mac>                     # entrada estática
sudo arp -d <ip>                      # borrar entrada
```

### Tabla de ruteo
```sh
route -n                              # ver tabla
ip route                              # ver tabla (moderno)
ip route add <red>/<mask> via <gw>    # agregar ruta
sysctl net.ipv4.ip_forward=1          # habilitar forwarding
```
