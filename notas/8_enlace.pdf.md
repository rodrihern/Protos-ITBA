---
materia: protos
tipo: apuntes
---
# Enlace

La **capa de enlace** (Data Link Layer) es el nivel 2 del modelo OSI. Su función principal es el traslado de paquetes desde un nodo al nodo adyacente a través de un enlace de comunicación físico.

---

## Servicios del Nivel de Enlace

El nivel de enlace ofrece una serie de servicios fundamentales para la comunicación entre nodos vecinos:

- **Encapsulamiento**: Empaqueta los datagramas de la capa de red en tramas (frames), agregando un encabezado y un trailer.
- **Acceso al medio**: Determina cómo se comparte el canal de comunicación.
- **Comunicación confiable**: Garantiza que las tramas lleguen sin errores (opcional en algunos protocolos).
- **Control de Flujo**: Evita que un emisor rápido sature a un receptor lento.
- **Detección y Corrección de Errores**: Identifica si hubo bits alterados durante la transmisión y, en algunos casos, permite reconstruirlos.
- **Modos de transmisión**:
	- **Half-Duplex**: Los nodos pueden transmitir y recibir, pero no simultáneamente.
	- **Full-Duplex**: Los nodos pueden transmitir y recibir al mismo tiempo.

---

## Tipos de Enlaces y Protocolos

Existen dos categorías principales de comunicación en el nivel de enlace:

### 1. Comunicaciones Punto a Punto (WAN)
Utilizadas típicamente en enlaces directos entre dos nodos.
- **SLIP (Serial Line IP)**: Protocolo muy simple para encapsular IP sobre líneas seriales.
- **PPP (Point-to-Point Protocol)**: Sucesor de SLIP. Soporta múltiples protocolos, autenticación, detección de errores y negociación de opciones.

### 2. Redes de Difusión o Broadcast (LAN)
Un canal compartido por múltiples nodos donde pueden ocurrir **colisiones** (cuando un nodo recibe dos o más señales simultáneamente). Requieren protocolos de **Acceso al Medio (MAC)**:

- **Acceso Aleatorio**:
	- **ALOHA**: Los nodos transmiten apenas tienen datos. Si hay colisión, retransmiten tras un tiempo aleatorio.
	- **CSMA (Carrier Sense Multiple Access)**: El nodo "escucha" el canal antes de transmitir.
		- *1-Persistent*: Si está ocupado, espera y transmite apenas se libera.
		- *Non-Persistent*: Si está ocupado, espera un tiempo aleatorio.
	- **CSMA/CD (Collision Detection)**: Usado en Ethernet cableado. El nodo detecta la colisión mientras transmite y aborta la operación.
	- **CSMA/CA (Collision Avoidance)**: Usado en redes inalámbricas.
- **Acceso Controlado**:
	- **Polling**: Un nodo maestro coordina quién transmite.
	- **Token Passing**: Se rota un "token" (permiso) entre los nodos (ej. Token Ring).
- **Acceso Canalizado**:
	- **FDMA** (Frecuencia), **TDMA** (Tiempo), **CDMA** (Código).

---

## Ethernet

Es el estándar más utilizado en LANs cableadas debido a su bajo costo y simplicidad.

![](attachments/Pasted%20image%2020260430204237.png)

> [!IMPORTANT]
> **Dirección MAC (Media Access Control)**: Identificador único de 48 bits (6 bytes) grabado en la placa de red (NIC).
> - Primeros 3 bytes: **OUI** (Organizationally Unique Identifier).
> - Últimos 3 bytes: Número de serie de la placa.

### Estructura de la Trama Ethernet
| Preámbulo (8B) | Destino (6B) | Origen (6B) | Tipo (2B) | Datos (46-1500B) | CRC (4B) |
| :--- | :--- | :--- | :--- | :--- | :--- |

- **Preámbulo**: Sincronización (patrón 10101010).
- **CRC**: Verificación de redundancia cíclica para descartar tramas con errores.

---

## Protocolo ARP (Address Resolution Protocol)

ARP permite obtener la **Dirección MAC** de un destino conociendo su **Dirección IP**.

> [!NOTE]
> Los hosts mantienen una **Tabla ARP** dinámica que relaciona IPs con MACs. Se puede visualizar con el comando `arp -a`.

### Funcionamiento de ARP
1. Si un host A quiere enviar a un host B en la misma red y no tiene su MAC, envía un **ARP Request** (Broadcast: `ff:ff:ff:ff:ff:ff`).
2. El host B responde con un **ARP Reply** (Unicast) informando su MAC.
3. El host A guarda la entrada en su tabla y envía la trama de datos.

### ARP en diferentes subredes
Si el destino está en otra red:
1. El host A determina que debe enviar el paquete al **Gateway** (Router).
2. Host A usa ARP para obtener la MAC del Router.
3. El Router recibe la trama, desencapsula el paquete IP, y usa ARP en la red destino para encontrar la MAC del host final.

### Variantes de ARP
- **RARP**: De MAC a IP (obsoleto, reemplazado por DHCP).
- **Proxy ARP**: El router responde con su propia MAC a un request destinado a otra red.
- **Gratuitous ARP**: Un host anuncia su propia MAC para detectar conflictos de IP.

---

## Seguridad: ARP Spoofing

Consiste en enviar respuestas ARP falsas para asociar la MAC del atacante con la IP de otra víctima (o el Gateway).

- **Ataques posibles**: Man-in-the-Middle (MITM), Denial of Service (DoS), Session Hijacking.
- **Defensas**: Entradas ARP estáticas, DHCP Snooping, herramientas de monitoreo como `arpwatch`.

---
Ver [[1_Introduccion#Capas de Red|Capas de Red]] | Ver [[6_Red#Direccionamiento IP|Direccionamiento IP]]
