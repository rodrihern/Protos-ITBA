---
materia: practica
tipo: apuntes
tags:
  - redes
  - dns
  - smtp
  - http
---

# Práctica de Laboratorio: Protocolos y Servicios

---

## Serialización y Protocolos de Aplicación

### Conceptos Fundamentales de Serialización
La **serialización** consiste en poner los datos de las estructuras en serie para su transmisión o almacenamiento. Este proceso es fundamental en el [[../notas/1_Introduccion#Encapsulamiento|encapsulamiento]] de capas en la red, donde cada capa añade su propia información de control.

#### Codificación Unicode (UTF-8)
Unicode tiene la propiedad de que los bits de inicio de cada octeto indican su posición y función en la secuencia, lo que permite detectar el inicio de un carácter y recuperarse ante pérdidas de sincronía.

- **Un byte (`0xxxxxxx`):** Caracteres ASCII estándar (0-127). Ejemplo: `0x65` (0110 0101) es la 'e'.
- **Dos bytes (`110xxxxx 10xxxxxx`):** Los bits `110` indican el inicio de una secuencia de 2 bytes; los bits `10` indican un byte de continuación.
- **Tres bytes (`1110xxxx 10xxxxxx 10xxxxxx`):** El prefijo `1110` indica 3 bytes.
- **Cuatro bytes (`11110xxx 10xxxxxx 10xxxxxx 10xxxxxx`):** El prefijo `11110` indica 4 bytes.

> [!TIP]
> Esta estructura permite que si empezamos a leer en el medio de una transmisión, sepamos inmediatamente si estamos ante el inicio de un carácter o un byte de continuación (que siempre empieza con `10`).

---

### Idempotencia en HTTP
Un método es **idempotente** cuando realizar la operación una vez o $n$ veces tiene el mismo efecto en el estado del servidor.

Ejemplos de [[2_HTTP#Métodos Principales|métodos HTTP]] idempotentes:
- **GET:** Solo recupera información, no cambia el estado.
- **PUT:** Reemplaza un recurso; si se envía lo mismo varias veces, el resultado final es el mismo.
- **DELETE:** Elimina un recurso; una vez eliminado, sucesivas llamadas no cambian que el recurso ya no esté.

---

## Administración de Servidores Web (Nginx)

Para gestionar configuraciones sin afectar la disponibilidad:

```sh
# Recarga la configuración sin downtime
systemctl reload nginx

# Verifica la sintaxis del archivo de configuración
nginx -t

# Prueba de conectividad con header de Host específico
curl -v http://localhost -H "host:foo"
```

---

## Sistema de Nombres de Dominio (DNS)

### Arquitectura y Funcionamiento
Los **root-servers** (13 en total) vienen hardcodeados en los sistemas operativos y resolutores.

> [!NOTE]
> En registros MX, el número antes del dominio indica la **prioridad** (menor número = mayor prioridad).
> Ejemplo: `10 aspmx.l.google.com` se intentará antes que uno con prioridad 20.


![[attachments/Drawings/Dominios]]

> [!QUESTION] Investigación
> Investigar el ataque de **envenenamiento de DNS de Kaminsky**, que explota la predictibilidad de los IDs de consulta para inyectar entradas falsas en el caché.

---

### Configuración Práctica (BIND9)

#### Instalación y Acceso
```sh
sudo apt install bind9
sudo -i # Acceso como root para configuración
```

#### Definición de Zona (`/etc/bind/named.conf.local`)
```bind
zone "foo.pdc.lab" {
	type master;
	file "/etc/bind/foo.pdc.lab.local";
};
```

#### Archivo de Zona (`/etc/bind/foo.pdc.lab.local`)
```bind
$ORIGIN foo.pdc.lab.
$TTL 1m

@ IN SOA ns.foo.pdc.lab. rohernandez.itba.edu.ar. (
					20260331   ; Serial (YYYYMMDD o incremental)
					7d         ; Refresh (tiempo para que el slave pregunte al master)
					1d         ; Retry (si falla el refresh, reintentar cada tanto)
					10d        ; Expire (tiempo tras el cual el slave deja de responder si no contacta al master)
					1m         ; Negative TTL (tiempo de caché para respuestas NXDOMAIN)
)

@    1m    IN NS ns1
@    2m    IN NS ns2

ns  30 IN A 1.2.3.4
ns1    IN A 1.2.3.5
ns2    IN A 1.2.3.6

@   2h IN MX 1 nsmail
@   2h IN MX 2 ns2mail
@   2h IN MX 3 ns3mail

nsmail   IN A 2.2.2.2
ns2mail  IN A 2.2.2.3
ns3mail  IN A 2.2.2.4

www      IN A 6.7.8.9
@        IN A 6.7.8.10
w3       CNAME www

@        TXT "hola manola"
```

> [!IMPORTANT]
> **Regla de Tiempos**
> Se debe cumplir siempre que $Expire > Refresh + Retry$ para asegurar la consistencia antes de dar por muerta la zona en un secundario.

#### Mantenimiento y Debugging
```sh
# Reiniciar el servicio
systemctl restart bind9

# Ver logs en tiempo real para debugging
tail -f /var/log/syslog

# Pruebas con dig
dig soa foo.pdc.lab @localhost
dig A ns1.foo.pdc.lab @localhost
```

---

## Protocolo SMTP y Formatos de Correo

### Estructura MIME
**MIME** (Multipurpose Internet Mail Extensions) permite identificar el tipo de contenido en el cuerpo del mensaje.

- **Headers:** Se pueden ver descargando el "original" en clientes como Gmail.
- **Content-Type:** Define la naturaleza del dato (ej: `text/plain`, `image/jpeg`).
- **Boundary:** Cadena de texto utilizada para separar las distintas partes en mensajes multipart.

#### Tipos de Multipart
- `multipart/mixed`: Utilizado cuando el mensaje contiene partes de distinta naturaleza (ej: texto y archivos adjuntos).
- `multipart/related`: Utilizado para partes que dependen entre sí (ej: un HTML que referencia una imagen adjunta mediante un `Content-ID`).
- `multipart/alternative`: Utilizado para dar alternativas de lo mismo (ej: una en texto plano y otra en html).

#### Codificación Quoted-Printable
Utiliza la sintaxis `=<HEX_ASCII>` para representar caracteres no ASCII o caracteres especiales.
Ejemplo: `=3D` representa el signo `=`.

![[attachments/Pasted image 20260407164322.png]]

> [!TIP]
> **Pruebas con Netcat**
> Al usar `netcat` para probar protocolos de texto, el flag `-C` es fundamental para enviar fines de línea tipo **CRLF** (\r\n), que es el estándar requerido por protocolos como SMTP y HTTP.
> Para pruebas reales, se puede utilizar el servidor **pampero**.

---

### Servidor smtp

```sh
sudo apt install postfix
```

despues ponemos `internet site` y luego el nombre del dominio, por ejemplo `pdc.lab`

ahora vemos con `netstat -tulpn` podemos ver que esta corriendo en que puerto

luego con nos conectamos con

```sh
nc -C localhost 25
```

![[attachments/Pasted image 20260409193740.png]]

Luego podemos ver ese mail en el directorio `/var/spool/mail`

![[attachments/Pasted image 20260409194029.png]]

despues podemos tambien hacer uno tipo `email.txt`

y luego 

```sh
cat email.txt | nc -C localhost 25
```

si mandamos varios, los mensajes se van a ir concatenando en el archivo


Para cambiar no se que cosa, agregamos al principio de `/etc/postfix/main.cf`

```sh
home_mailbox=Maildir/
```

ahora los mails llegan al directorio `~/Maildir`

![[attachments/Pasted image 20260409202428.png]]

---

### Servidor para leer mails pop3

```sh
sudo apt install dovecot-pop3d
```

nos conectamos con 

```sh
nc -C localhost pop3
```

luego ahi podes decirle cosas como 
```sh
USER rodri
PASS <contrasenia en texto plano>
LIST    ; te lista los mails
RETR 1  ; es un retrieve para leer el mail
DELE 1  ; marca un mail para borrar
RSET    ; deshace los cambios
UIDL    ; te lista los id que son unicos 
QUIT    ; se impactan los cambios

```

Podes estar en 3 estados

AUTH -> TX -> UPDATE

si estas en auth solamente podes usar comandos USER y PASS

luego en transaction (TX) tenes los demas para leer mails y demas.
Podes hacer cambios en el servidor pero todavia no se ven reflejados
tipo si haces un dele y despues ctrl+c, no se borra

En update ahi si se ven los cambios, como borrar el mensaje

---

### Ej tipo parcial

Si quiero mandar un mail con ñ agarramos y lo hacemos con gmail y luego nos descargamos el original y le agregamos las cosas que le tenemos que agregar

```sh
EHLO localhost
MAIL FROM: <hola@example.com>
RCPT TO: <rodri@pdc.lab>
DATA
MIME-Version: 1.0
Date: Thu, 9 Apr 2026 21:44:07 -0300
Message-ID: <CA+rx+vrsorZtTTPp1uxbUHXHC00BhKrv60iNduGY0u6s6F94hQ@mail.gmail.com>
Subject: ñññññ
From: Rodrigo Alejandro Hernandez <rohernandez@itba.edu.ar>
To: Rodrigo Alejandro Hernandez <rohernandez@itba.edu.ar>
Content-Type: multipart/alternative; boundary="00000000000039027e064f106e3f"

--00000000000039027e064f106e3f
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: base64

w7HDscOxw6HDoQ0K
--00000000000039027e064f106e3f
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: base64

PGRpdiBkaXI9Imx0ciI+w7HDscOxw6HDoTwvZGl2Pg0K
--00000000000039027e064f106e3f--
.
QUIT

```

---

## Capa de transporte

netcat labura sobre tcp, cuando le pongo crtl+d hago un shutdown (es una syscall). La comunicacion es full duplex pero con un ctrl+d cierro una parte, es decir yo ya no voy a mandar nada mas, es un EOF.

### header format

![[attachments/Pasted image 20260417103406.png]]


Se van intercambiando el tamaño de ventana de cada uno, para que el que manda sepa no mandar mas que lo que el otro puede recibir. 

Si la ventana se queda en 0 no manda mas nada, salvo un paquetito para preguntarle "che seguis en 0?" cada cierto tiempo. 

---

## Opciones de red vm

- **NAT**: La VM usa la IP del host para salir a Internet, como si estuviera detrás de un router. No es accesible desde el exterior.
- **Bridge**: La VM se conecta directamente a la red física del host, obteniendo su propia IP (como un dispositivo más en tu red).
- **Internal**: Red privada y aislada solo para que las VMs se comuniquen entre sí. Sin acceso al host ni a Internet.
- **Host only**: Crea una red privada entre la máquina host y las VMs. Las VMs no tienen acceso a Internet.
- **Controlador genérico**: Permite usar controladores de red especiales definidos por el usuario (como túneles UDP o VDE).
- **Red NAT**: Similar a NAT, pero permite que múltiples VMs compartan la misma red interna y se comuniquen entre sí, además de tener salida a Internet.
- **Red en la nube [EXPERIMENTAL]**: Permite conectar la VM a una red virtual alojada en servicios en la nube.
- **No conectado**: La tarjeta de red está configurada en la VM, pero simula que el cable de red está desconectado.

---

## Xinetd

```sh
sudo apt install xinetd
```

Para activar algun servicio como por ejemplo echo hay que ir a `/etc/xinetd.d/echo` y cambiarle `disabled = yes` por `disabled = no`

los servicios los podemos ver haciendo `ls` en `/etc/xinetd.d`

Sirven para boludear o debuggear

Para crear un sevicio nosotros podemos hacer uno de telnet con 

```sh
sudo apt install telnetd
```

esto nos crea un binario en `usr/sbin/telnetd`

ahora en `/etc/xinetd.d` podemos crear un archivo `telnet` y ponerle la siguiente configuracion

```sh
service telnet {
	disable = no
	id = telnet
	socket_type = stream
	protocol = tcp
	wait = no
	user = root
	server = /usr/sbin/telnetd
}
```
