---
tags:
  - protos
  - practica
  - red
  - ip
  - dhcp
  - nat
  - routing
  - guia
---

# Guía Práctica: Capa de Red (IP, DHCP, NAT, Routing)

---

## Comandos de red esenciales

```sh
# Ver interfaces y sus IPs
ifconfig
ip addr

# Asignar IP a una interfaz
ifconfig <interfaz> <ip>/<bits>
ip addr add <ip>/<bits> dev <interfaz>
ip link set <interfaz> up

# Ver tabla de ruteo
route -n         # formato clásico (usar en parcial)
ip route         # formato moderno

# Agregar ruta
route add -net <ip>/<bits> [gw <gateway>] [dev <interfaz>]
ip route add <ip>/<bits> via <gateway>
ip route add <ip>/<bits> via <gateway> dev <interfaz>

# Habilitar forwarding (hacer que un host funcione como router)
sysctl net.ipv4.ip_forward=1

# Ver puertos abiertos
netstat -tulpn
```

→ Ver [[Laboratorio#IP]] y [[Ejercicio_integrador#Inicio]]

---

## Subnetting — fórmulas clave

| Lo que quiero saber | Fórmula |
|---|---|
| Bits para hosts | `n bits de host` |
| Cantidad de hosts usables | `2^n - 2` (sin dirección de red ni broadcast) |
| Dirección de red | IP AND máscara |
| Broadcast | Dirección de red OR (NOT máscara) |
| Bits necesarios para k subredes | `⌈log₂(k)⌉` bits |
| Hosts necesarios para N hosts | `⌈log₂(N+2)⌉` bits |

### Máscaras por prefijo

| /prefijo | Máscara | Hosts usables |
|---|---|---|
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /28 | 255.255.255.240 | 14 |
| /29 | 255.255.255.248 | 6 |
| /30 | 255.255.255.252 | 2 |

### Truco para verificar si una IP pertenece a una red

Hacer AND entre la IP y la máscara. Si da la dirección de red, pertenece.

```
IP: 192.168.1.93   = 1100 0000.1010 1000.0000 0001.0101 1101
Máscara /27 = 255.255.255.224 = ...1110 0000
AND:                                              0101 1100 = 92 = 0101 1100 
¿Es .64? → red es 64 (0100 0000), broadcast es 95 (0101 1111)
93 está entre 65-94 → SI pertenece
```

→ Ver [[notas/6_Red#Preguntas|Preguntas IP]]

---

## DHCP — proceso DORA

```
D → Discover: cliente hace broadcast buscando servidor DHCP
O → Offer:    servidor ofrece una IP
R → Request:  cliente pide formalmente la IP ofrecida (broadcast)
A → ACK:      servidor confirma la asignación
```

### Instalar y configurar servidor DHCP (en R)

```sh
sudo apt install isc-dhcp-server

# Matar NetworkManager primero
sudo systemctl stop NetworkManager.service

# Darle IP fija a la interfaz que mira al cliente
sudo ip addr add 192.168.100.1/24 dev enp0s8
```

Editar `/etc/dhcp/dhcpd.conf`:

```
subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.10 192.168.100.50;
    option domain-name-servers 1.1.1.1;
    option subnet-mask 255.255.255.0;
    option routers 192.168.100.1;
    option broadcast-address 192.168.100.255;
    default-lease-time 20;
    max-lease-time 600;

    # IP fija para un host específico
    host maquina_h {
        hardware ethernet aa:bb:cc:dd:ee:ff;
        fixed-address 192.168.100.5;
        option host-name "maquina-H";
    }
}
```

Editar `/etc/default/isc-dhcp-server` → configurar la interfaz que mira al cliente (ej: `enp0s8`).

```sh
systemctl restart isc-dhcp-server
systemctl status isc-dhcp-server
journalctl -xeu isc-dhcp-server   # ver logs de error
```

### Denegar DHCP a un host

```
host maquina_h {
    hardware ethernet <mac>;
    deny booting;
}
```

→ Ver [[Respuestas_guia#E88|E88]], [[Respuestas_guia#E91|E91]], [[Respuestas_guia#E92|E92]], [[Laboratorio#DHCP]]

---

## NAT con iptables

```sh
# Ver tabla NAT
iptables -t nat -L
iptables -t nat -L --line-numbers   # con números de línea

# Borrar una regla
iptables -t nat -D <número>

# SNAT (enmascarar tráfico saliente — para que H salga a internet a través de R)
iptables -t nat -A POSTROUTING -o <interfaz_salida> -j MASQUERADE

# DNAT (redirigir tráfico entrante — exponer un servicio interno)
iptables -t nat -A PREROUTING -p tcp --dport <puertoR> -i <interfaz_interna> -j DNAT --to <ipH>:<puertoH>
```

> SNAT se aplica después del ruteo (POSTROUTING). DNAT se aplica antes (PREROUTING).

→ Ver [[Laboratorio#Salir a internet]]

---

## Routing — agregar rutas manualmente

```sh
# Ruta a una red específica via un gateway
route add -net 192.168.124.0/24 gw 192.168.123.200
ip route add 192.168.124.0/24 via 192.168.123.200

# Ruta default (para todo lo que no matchee otras rutas)
route add default gw 192.168.1.1
ip route add default via 192.168.1.1
```

→ Ver [[Ejercicio_integrador#Inicio]]

---

## Routing dinámico — resumen conceptual

| Protocolo | Tipo | Métrica | Límite | Uso |
|---|---|---|---|---|
| **RIP v1** | Distance Vector | Saltos | 15 hops | Redes pequeñas, sin VLSM |
| **RIP v2** | Distance Vector | Saltos | 15 hops | Redes medianas con VLSM |
| **OSPF** | Link State | Costo | Sin límite | Redes grandes, convergencia rápida |
| **BGP** | Path Vector | Políticas | — | Entre sistemas autónomos (ISPs) |

**Distance Vector (RIP)**: cada router solo conoce el camino a través de sus vecinos. Problema: **cuenta al infinito**.

**Link State (OSPF)**: cada router tiene mapa completo de la red. Usa Dijkstra. Converge más rápido.

### Instalar RIP con FRR

```sh
sudo apt install frr

# Habilitar RIP
# En /etc/frr/daemons: ripd=no → ripd=yes

sudo systemctl restart frr
sudo systemctl status frr
```

→ Ver [[Respuestas_guia#E93|E93]], [[notas/7_Routing|Routing]]

---

## ICMP y diagnóstico

```sh
ping <ip>                    # verificar conectividad básica
ping -c 4 <ip>               # 4 pings
traceroute <ip>              # ruta de saltos hasta el destino
traceroute -I <ip>           # con ICMP en lugar de UDP
```

---

## Opciones de red en VirtualBox

| Modo | Descripción |
|---|---|
| **NAT** | La VM usa la IP del host para salir. No es accesible desde afuera |
| **Bridge** | La VM tiene su propia IP en la red física |
| **Internal** | Solo VMs entre sí, sin acceso al host ni internet |
| **Host-Only** | Red privada entre host y VMs, sin internet |
| **Red NAT** | Como NAT pero las VMs se ven entre sí |

Para configurar 2 VMs conectadas entre sí: usar **Internal Network** con el mismo nombre en ambas.

→ Ver [[Laboratorio#Opciones de red vm]]

---

## IPs reservadas

| Rango | Uso |
|---|---|
| `127.x.x.x` | Loopback |
| `10.x.x.x` | Red privada clase A |
| `172.16.x.x` a `172.31.x.x` | Red privada clase B |
| `192.168.x.x` | Red privada clase C |
| `169.254.x.x` | Link-local (APIPA, cuando no hay DHCP) |
| `224.x.x.x` a `239.x.x.x` | Multicast |
