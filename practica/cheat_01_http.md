---
tags:
  - protos
  - practica
  - http
  - guia
---

# Guía Práctica: HTTP

---

## curl — flags esenciales

```sh
curl <url>                          # GET básico
curl -i <url>                       # incluye headers en la respuesta
curl -v <url>                       # verbose: muestra request + response completos
curl -X POST <url>                  # cambiar método
curl -H "Header: Valor" <url>       # agregar header
curl -d '{"key":"val"}' <url>       # body (implica POST)
curl --compressed <url>             # pedir compresión gzip
curl -0 / --http1.1 / --http2       # elegir versión HTTP
curl -s <url>                       # silent (sin progreso, útil en scripts)
curl -L <url>                       # seguir redirects
curl -x socks5h://host:port <url>   # proxy SOCKS5, DNS lo resuelve el proxy (usar en parcial)
curl -x socks5://host:port <url>   # proxy SOCKS5, DNS lo resuelve tu máquina
curl -o archivo <url>               # guardar respuesta en archivo
curl -r 0-999 <url>                 # Range request (reanudar descarga)
curl -u user:pass <url>             # autenticación básica
```

### Combinaciones frecuentes en parcial

```sh
# Proxy SOCKS5 con headers (típico de parcial)
# socks5h: el proxy resuelve el DNS → usar cuando el hostname solo existe en la red del lab
curl -x socks5h://proxy.host:1080 -H "Accept: text/plain" -H "Accept-Language: es" http://servidor:8080/ruta/

# GET con header personalizado
curl 192.168.1.1 -H "Tiki: Taka"

# POST con JSON
curl -X POST <url>/endpoint -H "Content-Type: application/json" -d '{"rta": "valor"}'

# Negociación de contenido por idioma
curl <url>/recurso -H "Accept-Language: es"

# ETag / caché condicional
curl -i -H 'If-None-Match: "abc123"' <url>
curl -i -H "If-Modified-Since: Thu, 13 Mar 2025 19:51:16 GMT" <url>

# Ver solo headers de respuesta (HEAD)
curl -I <url>
```

→ Ver aplicación en [[Ejercicio_integrador#Inicio]] y [[guia_respuestas#E19|E19]], [[guia_respuestas#E21|E21]]

---

## Netcat — HTTP manual

Para enviar \r\n (CRLF) en la terminal: `ctrl+v` luego `Enter` → imprime `^M`, después `Enter` normal.

> [!TIP]
> **Pruebas con Netcat**
> Al usar `netcat` para probar protocolos de texto, el flag `-C` es fundamental para enviar fines de línea tipo **CRLF** (\r\n), que es el estándar requerido por protocolos como SMTP y HTTP.
> Para pruebas reales, se puede utilizar el servidor **pampero**.

```sh
# Ponerse a escuchar en un puerto (simular servidor)
nc -l 9090

# Conectarse a un servidor (simular cliente)
nc <ip> 80
```

Ejemplo de request HTTP 1.1 manual:

```
GET / HTTP/1.1\r\n
Host: localhost\r\n
\r\n
```

Ejemplo de response HTTP:

```
HTTP/1.1 200 OK\r\n
Content-Type: text/html\r\n
Content-Length: 20\r\n
\r\n
<h1>Hola Mundo</h1>
```

→ Ver [[guia_respuestas#E17|E17]], [[guia_respuestas#E18|E18]], [[guia_respuestas#E19|E19]]

---

## Nginx — virtual hosts y reverse proxy

### Instalación

```sh
sudo apt install nginx-core
sudo -i   # escalar a root para configurar
```

### Configurar virtual host

Crear `/etc/nginx/sites-available/foo`:

```nginx
server {
    listen 80;
    server_name foo;

    root /var/www/foo;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Para el server **default**: usar `server_name _;` y `listen 80 default_server;`

```sh
# Habilitar: crear symlink
ln -s /etc/nginx/sites-available/foo /etc/nginx/sites-enabled/foo

# Crear contenido
mkdir /var/www/foo
echo "Bienvenido a foo!" > /var/www/foo/index.html

# Agregar alias en /etc/hosts
echo "127.0.0.1 localhost foo bar" >> /etc/hosts

# Verificar y recargar
nginx -t
systemctl reload nginx

# Probar
curl -v http://localhost -H "host:foo"
```

### Reverse proxy

Reemplazar `try_files` por `proxy_pass` en `/etc/nginx/sites-available/xxx`. Nginx **no reenvía headers personalizados por defecto** — declararlos con `proxy_set_header`:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Custom-Header $http_x_custom_header;
    }
}
```

>[!important]
>Las variables `$http_*` mapean los headers del cliente: todo a minúsculas y guiones → guiones bajos.

| Header del cliente | Variable nginx |
|---|---|
| `X-Parcial-Protos` | `$http_x_parcial_protos` |
| `Accept-Language` | `$http_accept_language` |

→ Ver [[guia_respuestas#E36|E36]], [[guia_respuestas#E39|E39]], [[2024_2c_practico#Ejercicio 1]]

---

## Caché HTTP

| Mecanismo | Header de respuesta | Header en revalidación | Respuesta si no cambió |
|---|---|---|---|
| max-age | `Cache-Control: max-age=3600` | — (no consulta al server) | — |
| Last-Modified | `Last-Modified: <fecha>` | `If-Modified-Since: <fecha>` | `304 Not Modified` |
| ETag | `ETag: "abc123"` | `If-None-Match: "abc123"` | `304 Not Modified` |

→ Ver [[guia_respuestas#E32|E32]], [[guia_respuestas#E33|E33]], [[guia_respuestas#E34|E34]]

---

## Códigos de estado importantes

| Código | Significado |
|---|---|
| 200 | OK |
| 301 | Moved Permanently |
| 302 | Found (redirect temporal) |
| 304 | Not Modified (caché válida) |
| 400 | Bad Request |
| 401 | Unauthorized (falta autenticación) |
| 403 | Forbidden (autenticado pero sin permiso) |
| 404 | Not Found |
| 405 | Method Not Allowed |
| 409 | Conflict |
| 410 | Gone (eliminado permanentemente) |
| 415 | Unsupported Media Type |
| 503 | Service Unavailable |

→ Ver [[guia_respuestas#E29|E29]]

---

## Headers clave

```
# Request
Host: www.ejemplo.com          # obligatorio en HTTP/1.1
Accept-Language: es            # negociación de idioma
Accept: application/json       # negociación de tipo
Range: bytes=0-999             # descarga parcial
If-None-Match: "etag"          # caché condicional
Cookie: session=abc123         # sesión

# Response
Set-Cookie: session=abc123; Expires=...
Content-Type: text/html; charset=utf-8
Content-Length: 1234
Transfer-Encoding: chunked
Cache-Control: max-age=3600, public
ETag: "67d33734-c"
Location: /nueva-url           # en redirects
WWW-Authenticate: Basic realm="zona"
```

---

## Idempotencia y seguridad de métodos

| Método | Seguro (no modifica) | Idempotente |
|---|---|---|
| GET | ✅ | ✅ |
| HEAD | ✅ | ✅ |
| PUT | ❌ | ✅ |
| DELETE | ❌ | ✅ |
| POST | ❌ | ❌ |
| PATCH | ❌ | ❌ |

→ Ver [[guia_respuestas#E27|E27]]

---

## Redirect de dominio con nginx

```nginx
server {
    listen 80;
    server_name foo.pdc.lab.alt;

    location / {
        return 302 http://protos.foo$request_uri;
    }
}
```

→ Ver [[guia_respuestas#E48|E48]]
