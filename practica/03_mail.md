---
tags:
  - protos
  - practica
  - mail
  - smtp
  - pop3
  - mime
  - guia
---

# Guía Práctica: Mail (SMTP, POP3, MIME, TLS)

---

## SMTP — comandos y sesión manual

SMTP usa **TCP puerto 25**. Con netcat se puede simular un cliente:

```sh
nc -C localhost 25   # -C envía CRLF (\r\n) — OBLIGATORIO en SMTP
```

### Secuencia de comandos

```
EHLO localhost
MAIL FROM: <remitente@dominio>
RCPT TO: <destinatario@dominio>
DATA
<headers MIME>
<línea en blanco>
<cuerpo del mensaje>
.                    ← línea con solo un punto termina el mensaje
QUIT
```

### Ejemplo completo con MIME

```
EHLO localhost
MAIL FROM: <hola@example.com>
RCPT TO: <rodri@pdc.lab>
DATA
MIME-Version: 1.0
Date: Thu, 9 Apr 2026 21:44:07 -0300
Subject: Asunto
From: Rodrigo <rohernandez@itba.edu.ar>
To: Rodrigo <rohernandez@itba.edu.ar>
Content-Type: multipart/alternative; boundary="limite123"

--limite123
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: base64

SGVsbG8gV29ybGQ=
--limite123--
.
QUIT
```

→ Ver [[Laboratorio#Servidor smtp]] y [[Respuestas_guia#E57|E57]]

---

## Instalar y configurar Postfix (servidor SMTP)

```sh
sudo apt install postfix
# Elegir: "internet site" y dominio: "pdc.lab"

# Ver en qué puerto está escuchando
netstat -tulpn

# Conectarse al servidor local
nc -C localhost 25

# Ver mails almacenados (si no está configurado Maildir)
ls /var/spool/mail/

# Para usar Maildir (un archivo por mail)
# Agregar al inicio de /etc/postfix/main.cf:
home_mailbox=Maildir/
systemctl restart postfix
```

---

## POP3 — leer mails remotamente

```sh
sudo apt install dovecot-pop3d

nc -C localhost pop3   # o nc -C localhost 110
```

### Comandos POP3 (en orden)

```
USER rodri          # identificar usuario
PASS contraseña     # autenticar (en texto plano)
LIST                # listar mails (número + tamaño)
RETR 1              # leer el mail nro 1
DELE 1              # marcar mail 1 para borrar
RSET                # deshacer los DELE de esta sesión
UIDL                # listar IDs únicos de los mails
QUIT                # aplicar cambios y cerrar
```

### Estados POP3

```
AUTH → (USER + PASS) → TRANSACTION → (QUIT) → UPDATE
```

Los DELE solo se aplican al hacer QUIT. Si hacés Ctrl+C, no se borra nada.

---

## MIME — codificación y tipos

### Headers MIME esenciales

```
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: base64
```

### Content-Type multipart

```
Content-Type: multipart/mixed;      boundary="limite"   # partes de distinta naturaleza (texto + adjunto)
Content-Type: multipart/alternative; boundary="limite"  # misma info en formatos distintos (text/html + text/plain)
Content-Type: multipart/related;    boundary="limite"   # partes que se referencian (HTML + imagen embebida)
```

Estructura de un cuerpo multipart:

```
--limite
Content-Type: text/plain
...texto...
--limite
Content-Type: image/jpeg
Content-Transfer-Encoding: base64
...datos en base64...
--limite--     ← cierre del multipart (doble guión al final)
```

### Content-Transfer-Encoding

| Valor | Cuándo usarlo |
|---|---|
| `7bit` | Solo ASCII 7 bits, líneas cortas |
| `base64` | Datos binarios (imágenes, adjuntos) |
| `quoted-printable` | Texto mayormente ASCII con algunos caracteres especiales |

### Quoted-Printable

Representa caracteres no-ASCII como `=HH` (hex). Ej: `=C3=B1` es `ñ`, `=3D` es `=`.

---

## Base64

```sh
# Codificar
echo "hola mundo" | base64

# Decodificar
echo "aG9sYSBtdW5kbwo=" | base64 -d

# Decodificar contenido de mail
echo "UGVybyBxIGJ1ZW5v..." | base64 -d
```

**Tamaño**: base64 convierte 3 bytes → 4 caracteres, por lo tanto el tamaño es `⌈original × 4/3⌉`.  
Si tengo 9 MB y el límite es 10 MB: `9 × 4/3 = 12 MB` → **no entra**.

→ Ver [[Respuestas_guia#E63|E63]], [[Respuestas_guia#E65|E65]], [[Ejercicio_integrador#papopepoparapapapapiparapopepo]]

---

## Autenticación de mails (anti-spam)

| Mecanismo | Cómo funciona | Limitación |
|---|---|---|
| **SPF** | Registro TXT en DNS con IPs autorizadas para enviar | Se rompe con forwarding |
| **DKIM** | Firma criptográfica en cada mail, clave pública en DNS | Más complejo de configurar |
| **Greylisting** | Rechazo temporal 4xx al primer intento (los spambots no reintentan) | Demora el primer mail |
| **Filtros bayesianos** | Estadística sobre palabras del contenido | Requiere entrenamiento |

```sh
# Ver registro SPF del ITBA
dig TXT itba.edu.ar

# Ver registros MX
dig MX itba.edu.ar
```

→ Ver [[Respuestas_guia#E61|E61]], [[Respuestas_guia#E62|E62]]

---

## TLS / SMTP seguro

| Puerto | Protocolo |
|---|---|
| 25 | SMTP (texto plano, entre servidores) |
| 587 | SMTP con STARTTLS (envío desde cliente) |
| 465 | SMTPS (SSL/TLS desde el inicio) |
| 993 | IMAPS |
| 995 | POP3S |
| 443 | HTTPS |

**STARTTLS**: permite "subir" una conexión de texto plano a TLS sin cambiar de puerto. Se inicia con el comando `STARTTLS` en la sesión SMTP.

---

## Ver a qué servidor SMTP hay que conectarse

Para enviar un mail a `usuario@dominio.com`, el MTA mira el registro **MX** de `dominio.com`:

```sh
dig MX dominio.com
# Resultado: 10 mail.dominio.com.  ← conectarse al puerto 25 de esta IP
```

→ Ver [[notas/4_mail-tls-ssl#Pregunta 4|Pregunta Mail 4]]
