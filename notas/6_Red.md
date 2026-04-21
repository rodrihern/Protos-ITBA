---
materia: protos
tipo: apuntes
---

# Capa de Red y Protocolos de Routing

---

## Servicios de la Capa de Red

La capa de red tiene el objetivo principal de **trasladar paquetes de un host a otro**, que pueden estar en distintas redes. Sus servicios básicos son:
- **Encapsulamiento:** Colocar el dato o segmento dentro de un paquete/datagrama IP.
- **Forwarding (Reenvío):** Pasar los paquetes de una interfaz de entrada a una de salida usando reglas locales.
- **Routing (Enrutamiento):** Decidir la ruta que debe tomar el paquete a través de la red (a esto se dedican los protocolos de ruteo como OSPF y BGP).

> [!NOTE] Redes de Datagramas vs Circuitos Virtuales
> Existen dos formas históricas de manejar la capa de red:
> - **Datagramas (Internet/IP):** Cada paquete encuentra su ruta independientemente. No hay información de estado en la red y es tolerante a fallas. La complejidad se maneja en las capas superiores.
> - **Circuitos Virtuales:** Se establece un camino (conexión) antes de transmitir. Todos los paquetes siguen esa ruta. Los routers mantienen el estado de los circuitos. Si un nodo falla, el circuito se corta.

---

## Direccionamiento IPv4 y Subredes

IPv4 utiliza direcciones de 32 bits ($2^{32} \approx 4.294$ millones de direcciones).

> [!TIP] Direcciones IPv4 Reservadas (RFCs)
> - **Loopback (127.x.x.x):** Usada para probar la propia pila TCP/IP del host local.
> - **Privadas (RFC 1918):** `10.x.x.x`, `172.16.x.x` - `172.31.x.x`, `192.168.x.x`. No son enrutables en internet.
> - **Link Local:** `169.254.x.x` (asignación automática sin DHCP).


![](attachments/Pasted%20image%2020260416200416.png)
### VLSM (Máscara de Subred de Longitud Variable)
Permite dividir una red en múltiples subredes de distinto tamaño. En lugar de una máscara fija (clase A, B o C), la máscara indica qué porción es red y qué porción es host de forma arbitraria (ej: `/26`, `/27`).
- Permite mapear la estructura de la organización, optimizar hosts y aislar el tráfico.

### CIDR (Enrutamiento entre Dominios sin Clases) / Superredes
Permite agregar varias subredes continuas bajo una única máscara más corta (superred), achicando la tabla de ruteo de los routers de la internet pública (optimiza y hace escalable el enrutamiento).
- El router decide por dónde enviar el paquete basándose en la regla de **coincidencia del prefijo más largo** (Longest Prefix Match).


![](attachments/Pasted%20image%2020260416210610.png)

![](attachments/Pasted%20image%2020260416210549.png)

---

## IPv6

Diseñado como sucesor de IPv4 debido al agotamiento de direcciones. 
- **Direcciones de 128 bits** ($2^{128}$), escritas en bloques hexadecimales.
- **Encabezado fijo**, diseñado para ser ruteado más rápido.
- **No permite fragmentación** en routers intermedios, solo en origen/destino.
- **Seguridad (IPSec)** incorporada por diseño.

### Tipos de Direcciones IPv6
- **Global Unicast Address:** Direcciones públicas y ruteables mundialmente (prefijo típico `2000::/3`).
- **Unique Local Address (ULA):** Análogo a las IPs privadas de IPv4 (prefijo `fc00::/7`).
- **Link Local Address:** Configuración automática obligatoria de una interfaz en un enlace (prefijo `fe80::/10`).

> [!NOTE] Formatos e Interoperabilidad
> - El identificador de interfaz de IPv6 puede usar **EUI-64** (basado en la MAC) o el RFC 7217 (números aleatorios estables para privacidad).
> - Para convivir con IPv4 se usa **NAT64/DNS64** o túneles (encapsular paquete IPv6 dentro de IPv4).

---

## IPSec y Seguridad de Red

La seguridad se evalúa bajo los principios de la tríada **CIA** (Confidencialidad, Integridad y Disponibilidad/Availability).

**IPSec** provee servicios de seguridad en el nivel de red. Tiene dos modos de funcionamiento principales:
- **Modo Transporte:** Protege el payload de extremo a extremo, pero **mantiene la cabecera IP en texto claro**. Útil para asegurar el tráfico entre dos hosts directamente.
- **Modo Túnel:** El paquete IP original completo es **encapsulado** y protegido en un nuevo paquete IP. Usado típicamente en VPNs (Virtual Private Networks) para conectar dos redes distintas de forma segura sobre Internet.

Usa dos protocolos principales:
- **AH (Authentication Header):** Provee integridad y autenticidad a los datos. No encripta.
- **ESP (Encapsulating Security Payload):** Encripta todo el paquete y provee confidencialidad, autenticidad e integridad.

---

## Firewalls y NAT

### IPTables
Sistema nativo de Linux para filtrado de paquetes y NAT. Organiza el tráfico en **Tablas** y **Cadenas**.

**Tablas principales:**
- **filter:** Filtra (permitir/denegar).
- **nat:** Modifica IPs o puertos (DNAT, SNAT, Masquerade).
- **mangle:** Modifica cabeceras avanzadas (TTL, ToS).

**Cadenas principales:** `PREROUTING`, `INPUT`, `FORWARD`, `OUTPUT`, `POSTROUTING`.

Ejemplos de comandos:
```bash
# Habilitar SNAT (Masquerade) para dar internet a la red interna eth1
iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE

# DNAT: Publicar un Web Server interno (192.168.2.3) al exterior
iptables -t nat -A PREROUTING -d 24.232.138.143 -p tcp --dport 80 -j DNAT --to-destination 192.168.2.3
```

### Network Address Translation (NAT)
Medida creada para aliviar la escasez de direcciones IP públicas en IPv4. Viola el principio de comunicación "end-to-end" directo, lo que afecta a protocolos P2P y obliga a usar técnicas adicionales (como relaying o UPnP).

Variaciones:
- **SNAT (Source NAT):** Traduce la dirección IP de *origen*. Usado cuando una red privada sale a internet compartiendo una IP pública.
- **NAPT (Network Address Port Translation):** Traduce no solo la IP, sino también el **Puerto**. El firewall/router mantiene una tabla de traducciones.
- **DNAT (Destination NAT / Port Forwarding):** Traduce la dirección IP de *destino*.

---

## Protocolos de Control y Asignación de IPs

### DHCP (Dynamic Host Configuration Protocol)
Asigna direcciones IP de forma dinámica y permite reservar IPs para direcciones MAC específicas. Reemplazó a BOOTP.
A través del proceso en 4 pasos (DORA: **D**iscover, **O**ffer, **R**equest, **A**CK), provee al host con la IP, máscara de red, gateways (ruteadores por defecto) y servidores DNS.

>[!note]
>Es el unico protocolo que tiene puerto de origen fijo (el 68)
>el destino es 67


### ICMP (Internet Control Message Protocol)
Utilizado para notificar errores y realizar diagnósticos, ya que el protocolo IP no maneja retransmisiones.
- **Mensajes Comunes:** Echo Reply/Request (Ping), Destino inalcanzable (Net, Host, Protocol, Port), Redirect (actualiza tablas de ruteo localmente), TTL Exceeded (usado por Traceroute).

---

## Conceptos de Routing

> [!NOTE] Routing vs Protocolos de Routing
> - **Routing o Forwarding:** Es la acción de reenviar paquetes IP hacia el siguiente host basándose en la información de las tablas de ruteo.
> - **Protocolos de Routing:** Son programas e implementaciones algorítmicas que intercambian información entre ruteadores para **construir y mantener** las tablas de ruteo.

Las configuraciones de ruteo más comunes son **Routing mínimo** (redes aisladas), **Routing estático** (configuración manual) y **Routing dinámico** (creación a través de protocolos algorítmicos).

### Categorías de Routing Dinámico
- **IGP (Routing Interno):** Diseñados para operar dentro de un dominio o red controlada por una sola organización. Ejemplos: **RIP, OSPF**, IS-IS.
- **EGP (Routing Externo):** Diseñados para el intercambio entre redes bajo el control de distintas organizaciones (Sistemas Autónomos). Protocolo predominante: **BGP**.

---

## Algoritmos de Routing Dinámico (IGP)

### Vector de Distancia (Distance Vector)
Cada router recibe la tabla de enrutamiento completa de sus vecinos directos y suma la distancia hacia ellos.

> [!IMPORTANT] Problema: Cuenta a Infinito
> Las "buenas noticias viajan rápido", pero si un enlace cae, los vecinos pueden retroalimentarse asumiendo que el otro aún tiene un camino válido, sumando costos lentamente hacia $\infty$. Soluciones:
> - **Split Horizon:** No devolver una ruta hacia el vecino por el cual se aprendió.
> - **Ruta Envenenada (Poison Reverse):** Anunciar rutas caídas con costo infinito.
> - **Triggered Updates:** Enviar rápidamente las malas noticias.

### Estado de Enlace (Link State)
Cada router sondea el costo de su enlace con sus vecinos directos y lo distribuye por inundación (flooding) a **toda la red**.
Cada nodo cuenta con un grafo idéntico de la topología global y utiliza algoritmos como **Dijkstra** para calcular localmente el camino óptimo.

---

## Protocolos Específicos

### RIP (Routing Information Protocol)
Protocolo IGP basado en **Vector de Distancia**. Utilizado históricamente para redes pequeñas.
- Su métrica es estricta: selecciona rutas por **cantidad de saltos**. Diámetro máximo de 15 saltos (16 = infinito).
- Demonio en Linux: `routed`.
- **RIP v1:** Broadcast y carece de soporte para subredes (classful).
- **RIP v2:** Usa multicast, soporta CIDR e incluye mecanismos de autenticación.

### OSPF (Open Shortest Path First)
Protocolo IGP basado en **Estado de Enlace** (Link State).
- Se actualiza solo ante cambios detectados en la topología (event-driven).
- Permite asignar un "peso" o costo a los enlaces.
- Demonio en Linux: `gated`.

> [!NOTE] Modelo Jerárquico en OSPF
> Para evitar inundar demasiada información, OSPF separa la red en **Áreas**.
> - Todas las áreas secundarias se conectan al **Area 0** (Backbone).
> - Los **ABR (Area Border Routers)** sumarizan las rutas internas de un área para publicarlas afuera.
> - Las **Stub areas** son redes terminales que asumen una ruta por defecto estática.

---

## Comparativa Resumen: Distance Vector vs Link State

| Vector de Distancia | Estado de Enlace (Link State) |
| :--- | :--- |
| Visualiza la topología basándose solo en lo que ven sus vecinos inmediatos | Construye la topología completa y global de toda la red |
| Suma las distancias de router a router paso a paso | Ejecuta localmente Dijkstra para conocer todas las rutas más cortas |
| Envíos periódicos y frecuentes de toda su *tabla de ruteo* | Envía actualizaciones (estado de los enlaces directos) **solo cuando hay cambios** |
| Límite restrictivo del tamaño de red (ej. 15 saltos RIP) | No tiene límite de diámetro impuesto a priori |
