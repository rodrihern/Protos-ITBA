
# Parcial 2025 2C parctico

## Ejercicio 1

![](attachments/Pasted%20image%2020260523182925.png)

La red es 192.168.108.0/24

1. Tenemos que brindar una ip fija 192.168.108.8 a la mac 0B:08:27:AE:C8:6A
2. Un dns que se de vuelta y le pegue a 8.8.8.8 y 8.8.4.4
3. El rango de direcciones para asignar van de la 10-200

Instalamos isc-dhcp-server y desactivamos el network manager:

```sh
sudo systemctl stop NetworkManager.service
sudo apt install isc-dhcp-server
```

luego en `/etc/dhcp/dhcpd.conf`

```
subnet 192.168.108.0 netmask 255.255.255.0 {
	range 192.168.108.10 192.168.108.200;
	option domain-name-servers 8.8.8.8, 8.8.4.4;
	option subnet-mask 255.255.255.0;
	option broadcast-address 192.168.108.255;
	default-lease-time 20;
	max-lease-time 600;
	
	host impresora {
		hardware ethernet 0B:08:27:AE:C8:6A;
		fixed-address 192.168.108.8;
		option host-name "impresora";
	}
}
```

tambien hay que agregarnos una ip en la red esta

```sh
sudo ip addr add 192.168.108.1/24 dev enp0s9
```

y finalmente restartear el servicio con 

```sh
systemctl restart isc-dhcp-server
```

Lo podemos ver usando el comando que nos dan:

```sh
sudo nmap --script broadcast-dhcp-discover \
    --script-args "broadcast-dhcp-discover.mac=AA:BB:CC:DD:EE:FF"
```

o con el de la impresora

```sh
sudo nmap --script broadcast-dhcp-discover \
    --script-args "broadcast-dhcp-discover.mac=0B:0B:27:AE:C8:6A"
```


---

## Ejercicio 2

![](attachments/Pasted%20image%2020260523191234.png)

No lo podemos probar pero

Para escanear una red se puede usar `nmap`

```sh
nmap -sn 10.0.0.0/24
```

luego ahi podemos ver que hosts hay, si tenemos que escanearle los puertos

```sh
nmap -sV -p- 10.0.0.0/24
```

luego para ver los routers que intervienen puedo usar `traceroute`

## Ejercicio 3

![](attachments/Pasted%20image%2020260523193824.png)

![](attachments/Pasted%20image%2020260523193857.png)

Errores en el archivo de zona:

### Error 0

Falta `$ORIGIN`

El archivo no declara `$ORIGIN`, por lo que no queda claro a qué dominio corresponde el `@`. BIND usa el nombre de zona del `named.conf` como origen por defecto, así que técnicamente funciona, pero es mala práctica. Correcto:

```
$ORIGIN it.itba.edu.ar.
```


### Error 1

Email en SOA usa `@` en vez de `.`

```
@ IN SOA ns1.it.itba.edu.ar. admin@it.itba.edu.ar. (
```

En un archivo de zona, el `@` del email debe reemplazarse con `.`. Correcto:

```
@ IN SOA ns1.it.itba.edu.ar. admin.it.itba.edu.ar. (
```

### Error 2

`www` usa registro `A` con un nombre, no una IP

```
www    IN A pampero
```

El registro `A` debe tener una dirección IP como valor. Para apuntar a otro nombre se usa `CNAME`. Correcto:

```
www    IN CNAME pampero
```

### Error 3  

Falta el registro `A` para `mail`

El registro MX apunta a `mail`, pero no hay ningún registro `A` que resuelva `mail` a una IP. Hay que agregar:

```
mail   IN A <ip-del-servidor-de-mail>
```

### Error 4

`ns1.it.itba.edu.ar` en la sección Hosts sin punto final

```
ns1.it.itba.edu.ar    IN A 192.168.70.1
```

Sin el `.` final, BIND lo interpreta como nombre relativo y le agrega el `$ORIGIN` al final, resultando en algo como `ns1.it.itba.edu.ar.it.itba.edu.ar.`. Debe ser `ns1.it.itba.edu.ar.` (con punto final) o simplemente `ns1`.

### Despues

Para que los cambios "se propaguen de manera correcta y rapido" hay 2 cosas mas que hacer

1. Incrementar el serial -> podemos poner la fecha 20260723
2. reducir el refresh a algo corto y tambien quizas el ttl

## Ejercicio 4

![](attachments/Pasted%20image%2020260523195731.png)


Hay que usar nginx como **reverse proxy HTTPS**. La URL no debe cambiar en el browser, por lo que no se puede hacer un redirect (301/302) — hay que usar `proxy_pass` para que nginx sirva el contenido de `parcial.protos.foo` de forma transparente bajo `seguro.protos.foo`.

### 1. Generar certificado TLS (self-signed)

```sh
openssl req -x509 -newkey rsa:4096 \
  -keyout /etc/ssl/private/seguro.key \
  -out /etc/ssl/certs/seguro.crt \
  -days 365 -nodes \
  -subj "/CN=seguro.protos.foo"
```

### 2. Config nginx

En `/etc/nginx/sites-available/seguro`:

```nginx
server {
    listen 443 ssl;
    server_name seguro.protos.foo;

    ssl_certificate     /etc/ssl/certs/seguro.crt;
    ssl_certificate_key /etc/ssl/private/seguro.key;

    location / {
        proxy_pass http://parcial.protos.foo;
    }
}
```

```sh
ln -s /etc/nginx/sites-available/seguro /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

### 3. Hacer el certificado confiable

La consigna dice que `curl` no puede llevar opciones adicionales (como `-k`), así que el certificado tiene que estar en el trust store del sistema:

```sh
cp /etc/ssl/certs/seguro.crt /usr/local/share/ca-certificates/seguro.crt
update-ca-certificates
```

### 4. DNS

Agregar un registro A para `seguro.protos.foo` apuntando a la IP del servidor nginx.

### Verificación

```sh
curl https://seguro.protos.foo
```
