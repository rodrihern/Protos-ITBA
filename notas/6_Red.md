---
materia: protos
tipo: apuntes
---

# Capa de Red y Protocolos de Routing

---

## Servicios de la Capa de Red

El objetivo de la capa de red es **trasladar paquetes de un host a otro**, que pueden estar en distintas redes.

### Servicios Básicos
- **Encapsulamiento:** Colocar el dato o segmento dentro de un paquete/datagrama IP.
- **Forwarding (Reenvío):** Paso de paquetes de una interfaz de entrada a una de salida usando reglas locales (tablas de ruteo).
- **Routing (Enrutamiento):** Proceso de determinar la ruta que deben tomar los paquetes (algoritmos y protocolos de ruteo).

> [!NOTE]
> **Servicios Adicionales**
> Un protocolo de capa de red puede ofrecer: entrega garantizada, demora mínima garantizada, orientación a conexión, entrega en orden, ancho de banda mínimo, integridad de datos y encriptación.

### Redes de Datagramas vs. Circuitos Virtuales

| Característica | Datagrama (Internet) | Circuito Virtual |
| :--- | :--- | :--- |
| Establecer conexión | No requerido | Requerido |
| Direccionamiento | ID de host origen y destino | Número de CV |
| Info de estado | No | Tabla de subred con números de CV en routers |
| Enrutamiento | Cada paquete una ruta independiente | Todos los paquetes una misma ruta |
| Falla de nodo | Se pierden algunos paquetes | Todos los CV que pasan por el nodo finalizan |
| Control de congestión | Complejo | Simple |
| Complejidad | En la capa de transporte | En la capa de red |

---

## Direccionamiento IPv4 y Subredes

IPv4 utiliza direcciones de 32 bits ($2^{32} \approx 4.294$ millones de direcciones).

> [!TIP]
> **Direcciones IP Reservadas**
> - **Loopback:** 127.x.x.x (RFC 990).
> - **Redes Privadas (RFC 1918):** `10.x.x.x`, `172.16.x.x` a `172.31.x.x`, `192.168.x.x`.
> - **Link Local (RFC 3927):** `169.254.x.x`.

![Pasted image 20260416200416.png](attachments/Pasted%20image%2020260416200416.png)

### VLSM (Máscara de Subred de Longitud Variable)
Permite dividir una red en subredes de distinto tamaño. Busca resolver el agotamiento de direcciones y simplificar la administración.
- **Motivos:** Hosts distantes, interconexión de capas físicas, filtrado de tráfico, mapeo de estructura organizacional.

### CIDR (Enrutamiento entre Dominios sin Clases) / Superredes
Permite agregar varias subredes continuas bajo una única máscara más corta para optimizar las tablas de ruteo.
- El router decide por dónde enviar el paquete basándose en la **coincidencia del prefijo más largo** (Longest Prefix Match).

![Pasted image 20260416210610.png](attachments/Pasted%20image%2020260416210610.png)

![Pasted image 20260416210549.png](attachments/Pasted%20image%2020260416210549.png)

### Fragmentación y Ensamblado
Si un paquete supera el **MTU** (Maximum Transmission Unit) de un enlace, se fragmenta. Los fragmentos se identifican por el mismo ID y se ordenan mediante el **Offset**. El flag **MF** (More Fragments) indica si hay más fragmentos en camino.

---

## DHCP (Dynamic Host Configuration Protocol)

Asigna direcciones IP y otros parámetros de red de forma dinámica.

> [!NOTE]
> **Proceso DORA**
> 1. **D**iscover: El cliente busca servidores DHCP (Broadcast).
> 2. **O**ffer: El servidor ofrece una IP (Broadcast o Unicast).
> 3. **R**equest: El cliente solicita la IP ofrecida (Broadcast).
> 4. **A**CK: El servidor confirma la asignación (Broadcast o Unicast).

- **DHCP Relay Agent:** Permite que un servidor DHCP atienda a clientes en subredes distintas a la suya, reenviando los mensajes Discover como Unicast al servidor.

---

## NAT (Network Address Translation)

### Tipos de NAT
- **Source NAT (SNAT):** Traduce la dirección de origen. Permite que múltiples hosts privados compartan una IP pública.
- **Destination NAT (DNAT / Port Forwarding):** Traduce la dirección de destino para permitir el acceso a servidores internos desde el exterior.
- **NAT64:** Permite la comunicación entre redes IPv6 y servidores IPv4 mediante traducción.

> [!NOTE]
> **Mecanismos de Relación**
> Para permitir comunicación entre dispositivos detrás de NAT se pueden usar protocolos como **UPnP** (IGD) o servidores **Relay**.

---

## ICMP (Internet Control Message Protocol)

Protocolo de control utilizado para notificaciones de error y diagnóstico.

### Mensajes Comunes
- **Echo Request / Reply:** Utilizado por el comando `ping`.
- **Destination Unreachable:** Notifica que un destino no es alcanzable.
- **Redirect Message:** El router informa al host sobre un gateway mejor para un destino específico.
- **Time Exceeded:** Utilizado por `traceroute` enviando paquetes con TTL incremental.

---

## IPv6

Diseñado para resolver el agotamiento de direcciones IPv4.
- **Direcciones de 128 bits.**
- **Encabezado fijo** de 40 bytes para ruteo más rápido.
- **No permite fragmentación** en routers (solo en origen).
- **Seguridad (IPSec)** incorporada por diseño.

### Tipos de Direcciones
- **Global Unicast:** Ruteable en Internet (prefijo `2000::/3`).
- **Unique Local Address (ULA):** Uso interno organizacional (prefijo `fc00::/7`).
- **Link Local Address:** Configuración automática obligatoria (prefijo `fe80::/10`).

### Transición IPv4-IPv6
- **Dual Stack:** Ambas pilas funcionando en paralelo.
- **Tunneling:** Encapsular paquetes IPv6 dentro de IPv4 para atravesar redes que no soportan IPv6.
- **NAT64 / DNS64:** Traducción entre protocolos.

---

## IPSec y Seguridad de Red

La seguridad se rige por la tríada **CIA**: **Confidencialidad**, **Integridad** y **Disponibilidad** (Availability).

### Protocolos de IPSec
- **AH (Authentication Header):** Provee integridad y autenticidad. No cifra datos.
- **ESP (Encapsulating Security Payload):** Provee confidencialidad (cifrado), integridad y autenticidad.

### Modos de Operación
- **Modo Transporte:** Protege el payload pero mantiene la cabecera IP original.
- **Modo Túnel:** Encapsula el paquete IP original completo dentro de uno nuevo. Ideal para VPNs.

---

## Firewalls e IPTables

**IPTables** es el sistema de filtrado de paquetes en Linux, organizado en tablas y cadenas.

### Tablas y Cadenas Principales

| Tabla      | Función                                                      |
| :--------- | :----------------------------------------------------------- |
| **filter** | Filtrado (ACCEPT, DROP, REJECT).                             |
| **nat**    | Traducción (DNAT, SNAT, Masquerade).                         |
| **mangle** | Modificaciones avanzadas (TTL, ToS).                         |
| **raw**    | Manipulacion de paquetes antes del seguimiento de conexiones |

| Cadena | Momento de Acción |
| :--- | :--- |
| **PREROUTING** | Antes del enrutamiento. |
| **INPUT** | Paquetes dirigidos al sistema local. |
| **FORWARD** | Paquetes que atraviesan el sistema. |
| **OUTPUT** | Paquetes generados localmente. |
| **POSTROUTING** | Después del enrutamiento. |

#### Ejemplo de Comandos

```bash
# Habilitar Masquerade (SNAT dinámico)
iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE

# Publicar un servidor interno (DNAT)
iptables -t nat -A PREROUTING -d 24.232.138.143 -p tcp --dport 80 -j DNAT --to-destination 192.168.2.3
```