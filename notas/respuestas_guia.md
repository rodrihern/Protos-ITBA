# Respuestas

## Entorno UNIX

### E1

![](attachments/Pasted%20image%2020260314145813.png)

```sh
grep -rl 'boot' /var/log
```

- `-r` lo hace recursivo
- `-l` hace que use solo el nombre y no el coso completo

## Sistemas Operativos

### E2

![](attachments/Pasted%20image%2020260314145828.png)

Es mas rapido con sendfile

La ventaja principal es que son menos syscalls.

## Sobre codificaciones generales

### E3

![](attachments/Pasted%20image%2020260314145837.png)

ascii codifica en 1 byte, bah en 7 bits
unicode puede usar mas bytes pero es backwards compatible. Permite codificar de 1 hasta 4 bytes en utf-8

### E4

![](attachments/Pasted%20image%2020260314145846.png)

No es lo mismo, en unicode tambien hay utf-16, utf-32 etc. unicode es tipo: voy a asignarle un codigo unico a cada caracter, utf-8 es una implementacion de unicode

### E5

![](attachments/Pasted%20image%2020260314145853.png)

El criterio que usaria es ver en promedio cual hace que la cadena sea mas corta, por ejemplo si voy a usar muchos caracteres que en utf-8 son de largo 4B o 3B capaz que en utf-16 son de 2B

### E6

![](attachments/Pasted%20image%2020260314145904.png)
![](attachments/Pasted%20image%2020260314145913.png)

```java
❯ javac ej6.java
❯ java ej6
ἵ5
2
```

el \u es 4 digitos hexadecimales que dieron el simbolo raro ese, y despues el 5. osea \u1f35 y despues el 5 comun. En js me dio lo mismo pero no se si esperaba que sea 🍕



### E7

![](attachments/Pasted%20image%2020260314145923.png)

Si de la nada lees un byte en el medio de una secuencia podes saber si corresponde al primero de una secuencia o si estas en el medio. Si te encontras en un byte que no es de inicio puedes saltar al siguiente sin tener que empezar desde 0.

Si bien no nos permite acceder por indice a cosas (al no tener todos una longitud fija), nos permite saltar al medio de una secuencia (por ejemplo al byte 5000 de una secuencia de 1GB) y recuperarnos por mas que caigamos en el medio de un caracter.

### E8

![](attachments/Pasted%20image%2020260314145929.png)

En la diferencia entre little y big endian es el orden de los bytes cuando guardamos un dato de mas de un byte

el numero 0x12345678 se ve de la siguiente manera

- little endian: 78 56 34 12
- big endian: 12 34 56 78

### E9

![](attachments/Pasted%20image%2020260314150035.png)

Un archivo de texto plano es uno en el cual la secuencia de bytes representa caracteres. En uno ascii cada byte es un caracter, mientras que en uno utf-8 cada los caracteres pueden ser de 1 hasta 4 bytes. (Un ascii es utf-8 porque utf-8 es backwards compatible).

Un archivo binario los bytes que no pueden representar cualquier cosa, desde un jpeg o pdf hasta machine code para ser ejecutado.

### E10

![](attachments/Pasted%20image%2020260314150245.png)
![](attachments/Pasted%20image%2020260314150255.png)

Para serializar cosas tenemos esencialmente 2 maneras:
- indicamos el tamaño al principio (*chunking*)
- ponemos una marca de fin (*null-terminated*)

luego ponemos los datos como bytes, igual hay un tema con los punteros porque lo que tienen guardado es una direccion virtual, ahi no se bien que pasa.

#### Ventaja null-terminated

- gastamos menos espacio pues no tenemos que indicar el tamaño 
- podemos empezar a enviarlo sin conocer el tamaño

#### Ventaja chunked

- sabemos el tamaño sin necesidad de tener que recorrerlo, lo cual permite poder leer todo el buffer de una si quisieramos o reservar memoria
- permite enviar el caracter \0

### E11


![](attachments/Pasted%20image%2020260314152551.png)
![](attachments/Pasted%20image%2020260314152602.png)

no lo pienso hacer pero la idea seria hacerlo en big-endian que es el estandar para transmitir datos binarios

pero podemos estructurarlo de la siguiente manera
- 8 bytes: timestamp (ms desde 1970)
- 1 byte: 0x00 indica warning y 0x01 indica info
- 2 bytes: tamaño en bytes del string
- hasta $2^{16}$ bytes: el texto, que lo haria en solo ascii por simplicidad

tlt escribir los programas

el 2 habla de un estandar para indicar formato de archivos


## Direccionamiento y HTTP

### E12
![](attachments/Pasted%20image%2020260314153029.png)

se llama *fragmento* y no se envia al servidor, es para el cliente nomas. Sirve para que el cliente sepa que parte mostrar

se usa `#` para el fragmento y `?` para lo que son query params, que estos ultimos si se envian al servidor en la request

### E13

![](attachments/Pasted%20image%2020260314153423.png)

NASH bajo y dijo:


Una URI (Uniform Resource Identifier) identifica un recurso de forma completa y única. Tiene la siguiente forma `scheme:[//authority]path[?query][#fragment]` y algunos ejemplos válidos son:

- https://www.ejemplo.com/index.html
- mailto:alguien@dominio.com
- ftp://[ftp.ejemplo.com/archivo.txt](http://ftp.ejemplo.com/archivo.txt)

Una URI no puede estar vacía ni contener sólo el fragmento.

Una URI reference es una referencia a una URI y puede ser parcial, incluso solo un fragmento. Algunos ejemplos válidos de URI references son:

- https://www.ejemplo.com/index.html → URI y URI reference
- /images/logo.png → URI reference relativa
- #seccion3 → URI reference fragmentaria

Una URI reference puede incluir una URI completa o parcial y puede tener solo el fragmento (#id), lo que no es válido como URI sola.

![](attachments/Pasted%20image%2020260314153723.png)

dice esto 

"g:h"           =  "g:h"
"g"             =  "[http://a/b/c/g](http://a/b/c/g)"
"./g"           =  "[http://a/b/c/g](http://a/b/c/g)"
"g/"            =  "[http://a/b/c/g/](http://a/b/c/g/)"
"/g"            =  "[http://a/g](http://a/g)"
"//g"           =  "http://g"
"?y"            =  "http://a/b/c/d;p?y"
"g?y"           =  "[http://a/b/c/g?y](http://a/b/c/g?y)"
"#s"            =  "http://a/b/c/d;p?q#s"
"g#s"           =  "[http://a/b/c/g#s](http://a/b/c/g#s)"
"g?y#s"         =  "[http://a/b/c/g?y#s](http://a/b/c/g?y#s)"
";x"            =  "http://a/b/c/;x"
"g;x"           =  "http://a/b/c/g;x"
"g;x?y#s"       =  "http://a/b/c/g;x?y#s"
""              =  "http://a/b/c/d;p?q"
"."             =  "[http://a/b/c/](http://a/b/c/)"
"./"            =  "[http://a/b/c/](http://a/b/c/)"
".."            =  "[http://a/b/](http://a/b/)"
"../"           =  "[http://a/b/](http://a/b/)"
"../g"          =  "[http://a/b/g](http://a/b/g)"
"../.."         =  "[http://a/](http://a/)"
"../../"        =  "[http://a/](http://a/)"
"../../g"       =  "[http://a/g](http://a/g)"

### E15

![](attachments/Pasted%20image%2020260314154045.png)

el nombre del host es case insensitive asi que abc.com = ABC.com

el puerto 80 es el default para http asi que es lo mismo ponerlo, no ponerlo o poner solo :

`~` se puede representar mediante *percent-encoding* %7E que es el mismo numero hexadecimal que %7e

Lo unico que cambia es el final, .html != .htm, luego **NO** son equivalentes

### E16

![](attachments/Pasted%20image%2020260314154813.png)

>[!note]
>Recordemos:
>URL: location
>URN: name

las que estan como `xmlns` funcionan principalmente como URN

mientras que las que las que tienen `schemaLocation` como URL


## netcat

### E17

para ponerse de listener 

```sh
nc -l <port>
```

para ponerse a escuchar en un puerto

```sh
nc <ip> <port>
```

para hablarle. Con `ifconfig` es una de las maneras para saber la ip local


### E18

![](attachments/Pasted%20image%2020260321125553.png)

con ctrl+v en la terminal decimos "el proximo caracter tomalo literal" entonces hacemos `ctrl+v` luego `enter` y eso nos da `^M` que es `\r` y luego simplemente `enter` que es `\n`

### E19

![](attachments/Pasted%20image%2020260321131412.png)

esto es lo que llego:

```sh
GET / HTTP/1.1
Host: localhost:9090
Connection: keep-alive
sec-ch-ua: "Chromium";v="146", "Not-A.Brand";v="24", "Brave";v="146"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-GPC: 1
Accept-Language: es-419,es;q=0.5
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br, zstd
Cookie: JSESSIONID=node017e5jikbylbpt1mtg8aho21zk30.node0
```

el navegador cuando le respondo cualquier cosa me muestra

![](attachments/Pasted%20image%2020260321131603.png)

si le respondo con

```sh
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 20

<h1>Hola Mundo</h1>
```

me muestra bien el hola mundo

![](attachments/Pasted%20image%2020260321132020.png)

### E20

![](attachments/Pasted%20image%2020260321132322.png)

en el primero me da un codigo 200 porque google es google y aunque no pongas el host te manda su buscador, pero el header host es requerido

para arreglarlo, agregaria en el header del request un `Host: www.google.com.ar`

con google.com.ar me dio un 301 que lo redirigio a google.com.

### E21

![](attachments/Pasted%20image%2020260321133935.png)

- **wget:** Ideal para **descargar archivos completos** o espejar sitios web de forma recursiva.
- **curl:** Pensado para **transferir datos** de forma granular, soportando múltiples protocolos y permitiendo inspeccionar encabezados o enviar datos complejos (POST, PUT)
	- `-0`, `--http1.1`, `--http2` indican la version
	- `--compressed` indica que lo mande comprimido con gzip
	- `-d` para enviar datos en un post
	- `-H` para mandar cosas en el header como `curl xxx -H "host:tumama"`
	- `-i` para que en la respuesta incluya los headers ademas del html
	- `-s` silent, muestra menos cosas tipo por si lo usas en un script esta bueno
	- `--socks5` usa un proxy SOCKS5, para navegar de forma anonima o para saltar bloqueos de red

### E22

![](attachments/Pasted%20image%2020260321134506.png)

El header que participa para el continue es range.

para eso el servidor manda el header `Accept-Ranges` y con el header `Content-Range` indica en que rango esta mandando, luego uno en el continue indica que rango quiere y con eso puede resumir la descarga. 

No funciona en todos, sobretodo no funciona en los que generan los pdf dinamicamente

## HTTP: Funcionalidades que provee el protocolo

### E23

![](attachments/Pasted%20image%2020260322111917.png)

Sip, es lo mismo pues son case insensitive los headers en http

### E24

![](attachments/Pasted%20image%2020260322112033.png)

Significa que es un fragmento, no todo el contenido, usualmente porque no se conoce todo el largo del contenido. Empieza con el tamaño del chunk y termina con un chunk de tamaño 0.

| Tipo       | Descripcion                                                                                                                 |
| ---------- | --------------------------------------------------------------------------------------------------------------------------- |
| `chunked`  | El estándar actual. Envía el cuerpo en fragmentos con su tamaño en hexa.                                                    |
| `compress` | Usa el formato de compresion LZW, muy raro de ver hoy                                                                       |
| `deflate`  | Usa la estructura zlib con el algoritmo de compresión deflate.                                                              |
| `gzip`     | Usa el formato de compresión GNU zip. (Ojo: hoy se prefiere usar `Content-Encoding: gzip` en lugar de `Transfer-Encoding`). |
| `identity` | Significa que no hay transformación. Es el valor por defecto (equivale a no poner el header).                               |

### E25

![](attachments/Pasted%20image%2020260322113619.png)

El media type es la etiqueta universal para identificar el formato de un archivo en la web. Sirve para indicar al cliente que software o plugin tiene que usar para abrir el contenido. La estructura es `tipo/subtipo`


| Recurso     | Media Type                               |
| ----------- | ---------------------------------------- |
| html        | text/html                                |
| txt         | text/plain                               |
| css         | text/css                                 |
| js          | text/javascript o application/javascript |
| imagenes    | image/jpeg, image/gif, image/png         |
| pdf         | application/pdf                          |
| Ejecutables | application/octet-stream                 |
### E26

![](attachments/Pasted%20image%2020260322114240.png)

Para que se puedan usar conexiones persistentes de http, ambos las tienen que soportar, La ventaja es poder realizar varios pedidos sin necesidad de tener que cerrar y abrir la conexion cada vez.

en http/1.1 son persistentes de por si, en cambio en 1.0 tienen que añadir el header `connection: keep-alive`

*Pipelining:* permite poder mandar varias peticiones sin necesidad de esperar a la respuesta de la peticion anterior

### E27

![](attachments/Pasted%20image%2020260322114524.png)

los metodos que provee son:
- GET
- POST
- PUT
- DELETE
- TRACE
- HEAD
- OPTIONS
- PATCH

Un metodo es *idempotente* si al realizar la misma peticion varias veces, produce **el mismo efecto en el servidor** que hacerla una vez.

En el caso de ABM, es un problema que se borren cosas con GET a mi criterio.

Lo que va a pasar es que va a borrar todo y la culpa es del desarrollador. Posta como vas a hacer que se borren cosas con GET? GET debe ser un metodo *seguro*, osea no debe cambiar el estado del servidor. Se soluciona usando DELETE para borrar como cualquier persona normal.

DELETE es idempotente pues si lo ejecuto 1 vez o 10000 es lo mismo, borra el recurso. En el caso de las 10000 veces, en la primera lo borra y en las siguientes 9999 no hace nada.

en [w3]([w3](https://www.w3.org/2001/tag/doc/whenToUseGet.html)) dicen
 
1.3 Quick Checklist for Choosing HTTP GET or POST
- Use GET if:
    - The interaction is more like a question (i.e., it is a safe operation such as a query, read operation, or lookup).
- Use POST if:
    - The interaction is more like an order, or
    - The interaction changes the state of the resource in a way that the user would perceive (e.g., a subscription to a service), or
    - The user be held accountable for the results of the interaction.

### E28

![](attachments/Pasted%20image%2020260322115653.png)

dice:
Error 405 (Method Not Allowed)!!1

(el 1 no es un typo mio sino de la response jajaj)

ahora no se usa trace por motivos de seguridad (XST: Cross-side tracing)

### E29

![](attachments/Pasted%20image%2020260322120538.png)
![](attachments/Pasted%20image%2020260322122913.png)

a. 301: moved permanently
b. 403: forbidden
c. 401: unauthorized
d. 409: conflict
e. La diferencia es que el 410 es probablemente permanente
f. 400 bad request
g. 400 bad request
h 415 unsoported media type
i. 503 service unavailable
j. 405 method not allowed

### E30

![](attachments/Pasted%20image%2020260322123327.png)

Son las credenciales y como dice basic, estan en base64, osea no esta encriptado.

decodificado en internet queda:
algunusuario:algunapassword

tambien se pude usar 

### E31

![](attachments/Pasted%20image%2020260322123643.png)

Ofrece estrategias como: 
- caching
- conexiones persistentes
- compresion de datos
- range requests
## HTPP: Entity tags y caching

### E32

![](attachments/Pasted%20image%2020260322124739.png)

```sh
❯ curl -i http://protos.foo
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 22 Mar 2026 15:56:54 GMT
Content-Type: text/html
Content-Length: 12
Last-Modified: Thu, 13 Mar 2025 19:51:16 GMT
Connection: keep-alive
ETag: "67d33734-c"
Accept-Ranges: bytes

Hola mundo!
❯ curl -i -H "if-modified-since: Thu, 13 Mar 2025 19:51:16 GMT" http://protos.foo
HTTP/1.1 304 Not Modified
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 22 Mar 2026 15:57:53 GMT
Last-Modified: Thu, 13 Mar 2025 19:51:16 GMT
Connection: keep-alive
ETag: "67d33734-c"

❯ curl -i -H 'If-None-Match: "67d33734-c"' http://protos.foo/
HTTP/1.1 304 Not Modified
Server: nginx/1.18.0 (Ubuntu)
Date: Sun, 22 Mar 2026 15:58:23 GMT
Last-Modified: Thu, 13 Mar 2025 19:51:16 GMT
Connection: keep-alive
ETag: "67d33734-c"
```

### E33

![](attachments/Pasted%20image%2020260322130055.png)

tipo eso significa que ese recurso esta bien por 3600 segundos = 1h. Osea por ese tiempo lo puede cachear. Si ya paso una hora, el browser debe volver a hacer la peticion. Si no paso una hora puede ahorrarse la peticion y devolver lo del cache.

### E34

![](attachments/Pasted%20image%2020260322130326.png)

con *shallow etag*, primero genera el recurso, luego el hash md5 y eso lo valida contra el etag. Por lo que solo ahorramos band-width, pero igualmente tenemos que generar el recurso.

En cambio con *deep-tag* el servidor se puede dar cuenta si cambio o no sin necesidad de generarlo, ahorrandonos tanto band-width como procesamiento

### E35

![](attachments/Pasted%20image%2020260322130718.png)

a. envia los datos codificados en la url

b. usa cookies



## HTTP: Configuracion de servidor http, virtual hosts
### E36
![](attachments/Pasted%20image%2020260318115515.png)

primero instalamos nginx con `sudo apt install ngnix-core`

como muchas de estas cosas vamos a necesitar permisos de root, podemos escalar a root con `sudo -i`

luego en `/etc/nginx/sites-available` creamos los archivos `foo`, `bar` y `default`
una vez ahi los configuramos, ejemplo foo:
```sh
server {
    listen 80;
    # listen 80 default server; seria para el default
    
    server_name foo; # Para el default podemos poner de nombre _

    root /var/www/foo;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

luego en `/etc/nginx/sites-enabled` creamos links a los archivos que creamos recien, ejemplo con foo:

```sh
ln -s  /etc/nginx/sites-available/foo  foo
```

luego creamos los html en `/var/www/`
ejemplo con foo:

```sh
mkdir foo
echo "Bienvenido a foo!" > ./foo/index.html
```

Finalmente en `/etc/hosts` agregarmos foo y bar como "aliases" de localhost

```sh
127.0.0.1 localhost foo bar
```

Para chequear que todo este bien

```sh
nginx -t
```

y si esta todo bien le hacemos reload o restart al servidor 

```sh
systemctl reload nginx
```

ahora ya podemos hacerle `curl`

![](attachments/Pasted%20image%2020260318121637.png)


### E39

![](attachments/Pasted%20image%2020260324190929.png)

para lograrlo cambiamos el archivo `/etc/nginx/sites-available` y le agregamos la funcion `proxy_pass` para que se de vuelta y llame al servidor que esta corriendo en el 8080

```sh
server {
    listen 80;
    
    server_name foo; 

    root /var/www/foo;
    index index.html;

    location / {
        proxy_pass http://localhost:8080 # agregamos esta linea en lugar del try_files
    }
}

```


## Cliente DNS

### E42

![](attachments/Pasted%20image%2020260331162924.png)

el comando a usar es `dig`

buscamos el soa

podes tambien agregarle que servidor de dns queres con `dig google.com @1.1.1.1`

### E44
![](attachments/Pasted%20image%2020260331170932.png)

- el root -> .ar
- .ar -> itba.edu.ar (.edu.ar son del mismo)
- .itba.edu.ar -> me da un nombre de dominio que no sabe la ip, y se hace recursivo el problema hasta que lo encuentro
- ahora si llegamos a pampero: 200.5.121.137

usamos `dig ... @<ip-ns>`

### E45

![](attachments/Pasted%20image%2020260407112733.png)

Sirve para que diga a quienes le tienen que preguntar

```sh
❯ dig pampero.it.itba.edu.ar +trace

; <<>> DiG 9.10.6 <<>> pampero.it.itba.edu.ar +trace
;; global options: +cmd
.			510697	IN	NS	a.root-servers.net.
.			510697	IN	NS	b.root-servers.net.
.			510697	IN	NS	c.root-servers.net.
.			510697	IN	NS	d.root-servers.net.
.			510697	IN	NS	e.root-servers.net.
.			510697	IN	NS	f.root-servers.net.
.			510697	IN	NS	g.root-servers.net.
.			510697	IN	NS	h.root-servers.net.
.			510697	IN	NS	i.root-servers.net.
.			510697	IN	NS	j.root-servers.net.
.			510697	IN	NS	k.root-servers.net.
.			510697	IN	NS	l.root-servers.net.
.			510697	IN	NS	m.root-servers.net.
.			510697	IN	RRSIG	NS 8 0 518400 20260420050000 20260407040000 54393 . A80cnbfc9EOFlZh52POcqLy9oJU2u9Ccs7rJ8ATe6/DMrLZXb96+W4zn g9BnVx8kRsFfUqD6N3pPmMw6SUTaU22J+D2wZLw5TsmiAaKnGvFMm0/7 w8P+k5NWzrKJoYkeTCe8uGZvPbROVrHHscTqZTQ3fxhETnnu0YzEo66j dNQO6Ri8UgIe81BcZXdt+LWYz+cczc53WxBAqXCQ5G+2ASeFRHfKrUyb Z3MB8inEwhr1pM9AiYk7MFYCyk2tjZ098G+shpd+1NDI4jRt4LZJqfcY WOT161ClKRjULezbebHfxvT8wInW38BsniftBqzd+pgPCun+Jlmf95CJ /JBwBw==
;; Received 525 bytes from 1.1.1.1#53(1.1.1.1) in 18 ms

ar.			172800	IN	NS	a.lactld.org.
ar.			172800	IN	NS	c.dns.ar.
ar.			172800	IN	NS	d.dns.ar.
ar.			172800	IN	NS	e.dns.ar.
ar.			172800	IN	NS	f.dns.ar.
ar.			86400	IN	DS	19606 8 2 4415CF1A2CF10DE94B92BC020F21D1BF4163B2E90F2A6F6A5D2A1740 339D566C
ar.			86400	IN	RRSIG	DS 8 1 86400 20260420050000 20260407040000 54393 . A+BPpXZBAWudSV3m6h/cybhilKpYwj8UwLDRNBiYGwnTXDwt2YczFNmZ UtC2ec29cfUotcWLEQCMlL7gByMj22RO1/7R0nHzLuhALefHSaMfc2SN UfMW5PTwNqqm0scSJYDhG4Q/rNGOywN2L6ECi1AHuMnH7tdd937siGjR Qo0thdxcxCc8eKRyH9WEzMu4bs2HQFADwnn+75I19MvwqvaWfKR3fFaC JhS3xWg18/NfQZOuNfAF4xguKjE4SaovJNJI26Kps+U8xQzkxq8Zxq0S I+OB9+PAxnniP8Rbld2XaEW0OsbwUeL39+JTSflGOJwZR43mbMhTFclh ByRwvw==
;; Received 700 bytes from 199.7.83.42#53(l.root-servers.net) in 16 ms

itba.edu.ar.		3600	IN	NS	salvador.ns.cloudflare.com.
itba.edu.ar.		3600	IN	NS	marlowe.ns.cloudflare.com.
76mqjgvprdik2e0chhpmkbpu3fk61mj3.edu.ar. 3600 IN NSEC3 1 0 5 3A380D45674C4532 76RI418V56LO9VDH77EKBR0MN2BQS3L5  NS
76mqjgvprdik2e0chhpmkbpu3fk61mj3.edu.ar. 3600 IN RRSIG NSEC3 8 3 3600 20260508163332 20260407042947 62154 edu.ar. YSSaACRzxb+6WXY00F6SqUqh0nk7tGmyXkk7rs2rDDVkrQ4MXmgv4IrK JUyjRCXGmAo69U76nAmmLA6RdvT012kGNZiedjhSakgxylHCymqs2ZLj rzsLjqm224QoppUPyFlzjCYMJhXzGLbeIm42zz5UAG8icMgiHlSlPlSN FpE=
;; Received 361 bytes from 130.59.31.20#53(f.dns.ar) in 236 ms

it.itba.edu.ar.		60	IN	NS	crystal.it.itba.edu.ar.
;; Received 89 bytes from 162.159.38.212#53(marlowe.ns.cloudflare.com) in 33 ms

pampero.it.itba.edu.ar.	3600	IN	A	200.5.121.137
it.itba.edu.ar.		3600	IN	NS	crystal.it.itba.edu.ar.
;; Received 105 bytes from 200.5.121.139#53(crystal.it.itba.edu.ar) in 8 ms

```


### E46

![](attachments/Pasted%20image%2020260407113309.png)

Con whois podés ver, por ejemplo:
- dueño o contacto administrativo del dominio
- registrador
- fechas de creación y vencimiento
- servidores DNS autoritativos
- datos del bloque IP y del organismo que lo asignó

## DNS y HTTP
### E47

![](attachments/Pasted%20image%2020260407113503.png)

Se puede hacer con un reverse proxy con nginx como en [E39](#E39) y agregamos a `etc/hosts` que mapee foo.pdc.lab a 127.0.0.1

### E48

![](attachments/Pasted%20image%2020260407114404.png)

ahora podemos devolver un redirect, poniendo en `/etc/nginx/sites-available` 

```sh
server {

    listen 80;

    server_name foo.pdc.lab.alt;

  

    location / {

        return 302 http://protos.foo$request_uri;

    }

}
```

## Configuracion de DNS Server

### E49 - E52

![](attachments/Pasted%20image%2020260406105130.png)

Instalamos bind con `sudo apt install bind9`

nos cambiamos a root para hacer todo mas facil `sudo -i`

configuramos los archivos `/etc/bind/named.conf.local` con

```sh
zone "foo.pdc.lab" {
	type master;
	file "/etc/bind/foo.pdc.lab.local";
};
```

y luego creamos uno `/etc/bind/foo.pdc.lab.local`

la sintaxis es `nombre [ttl] IN tipo [prioridad] valor`

para la pirmera linea la sintaxis es `@ IN SOA <servidor-maestro> <email-contacto> (...)` donde el @ del mail de contacto es un punto.

el origin debe coincidir con el nombre definido en la zona en el archivo `named.config.local`

```sh
$ORIGIN foo.pdc.lab.
$TTL 1m

@ IN SOA ns.foo.pdc.lab. rohernandez.itba.edu.ar. (
					20260331   ; Serial
					7d  ; Refresh
					1d  ; Retry
					10d ; Expire
					1m  ; Negative TTL
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

- serial: numero de cuando lo modificaste, puede ser 1, 2, 3 o como en este caso, formato fecha. versiones mas nuevas, serial mas arto
- por 7 dias es bueno, despues de 7 dias no lo tiro, volve a intentar cada 1d si no funca y expira en total en 10d
- siempe $expire > refresh + retry$ 
- negative ttl: si te dio como que no existe, por 1 minuto te cacheas que no existe


para reiniciar el servidor `systemctl restart bind9`

para debuguear `tail -f /var/log/syslog`

con eso ya deberia estar andando y podemos probarlo con
```sh
dig soa foo.pdc.lab @localhost
dig A ns1.foo.pdc.lab @localhost
```

### E53-54

![](attachments/Pasted%20image%2020260407115016.png)

para saber que servidor DNS se esta usando y modificarlo, esto se hace en el archivo `/etc/resolv.conf`

Un **fowarder** es un servidor dns al que le hacemos las consultas (como 8.8.8.8 de google o 1.1.1.1 de cloudflare), sin un *fowarder*, tendriamos que hacer consultas recursivas preguntandole a los root servers y toda la bola 

