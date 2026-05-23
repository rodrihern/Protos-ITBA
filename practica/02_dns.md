---
tags:
  - protos
  - practica
  - dns
  - guia
---

# Guía Práctica: DNS

---

## dig — el comando más importante

```sh
dig <dominio>                          # consulta A record (IPv4)
dig <dominio> MX                       # registro MX (servidores de mail)
dig <dominio> NS                       # servidores autoritativos
dig <dominio> SOA                      # inicio de autoridad (serial, TTL, etc.)
dig <dominio> TXT                      # registros de texto (SPF, DKIM)
dig <dominio> AAAA                     # IPv6
dig <dominio> +trace                   # traza completa desde root servers
dig <dominio> @<ip-servidor>           # usar un DNS específico
dig <dominio> @1.1.1.1                 # usar Cloudflare como resolver
dig <dominio> @8.8.8.8                 # usar Google como resolver
dig -x <ip>                            # resolución inversa (PTR)
```

### Patrones de uso en parcial

```sh
# Ver qué serial tiene www.google.com según un DNS específico
dig SOA google.com @192.168.124.53

# Consultar MX del ITBA
dig MX itba.edu.ar

# Traza de resolución
dig pampero.it.itba.edu.ar +trace

# SPF (registro TXT)
dig TXT itba.edu.ar
```

→ Ver [[Respuestas_guia#E42|E42]], [[Respuestas_guia#E44|E44]], [[Respuestas_guia#E45|E45]], [[Respuestas_guia#E62|E62]]  
→ Ver [[Ejercicio_integrador#ketylanuchi]]

---

## Tipos de registros DNS

| Tipo | Para qué sirve | Ejemplo de valor |
|---|---|---|
| **A** | Hostname → IPv4 | `6.7.8.9` |
| **AAAA** | Hostname → IPv6 | `2001:db8::1` |
| **NS** | Servidor autoritativo del dominio | `crystal.it.itba.edu.ar.` |
| **MX** | Servidor de mail (+ prioridad) | `10 aspmx.l.google.com.` |
| **CNAME** | Alias → nombre canónico | `www → servidor-real.com.` |
| **PTR** | IP → nombre (inverso) | `137.121.5.200.in-addr.arpa.` |
| **TXT** | Texto libre (SPF, verificación) | `"v=spf1 include:..."` |
| **SOA** | Info técnica de la zona | serial, refresh, retry, expire |

> En registros MX: menor número = mayor prioridad.

---

## /etc/hosts y /etc/resolv.conf

```sh
# Agregar alias local (se consulta ANTES que DNS)
echo "127.0.0.1 localhost foo bar" >> /etc/hosts
echo "192.168.1.10 vault.local" >> /etc/hosts

# Ver/cambiar el servidor DNS que usa el sistema
cat /etc/resolv.conf
# nameserver 1.1.1.1
# nameserver 8.8.8.8
```

---

## whois

```sh
whois itba.edu.ar    # dueño, registrador, fechas, NS autoritativos
```

→ Ver [[Respuestas_guia#E46|E46]]

---

## Configurar servidor BIND9

### Instalación

```sh
sudo apt install bind9
sudo -i   # escalar a root
```

### `/etc/bind/named.conf.local` — definir zona

```bind
zone "foo.pdc.lab" {
    type master;
    file "/etc/bind/foo.pdc.lab.local";
};
```

### `/etc/bind/foo.pdc.lab.local` — archivo de zona

```bind
$ORIGIN foo.pdc.lab.
$TTL 1m

@ IN SOA ns.foo.pdc.lab. rohernandez.itba.edu.ar. (
            20260331   ; Serial (fecha o número, incrementar al modificar)
            7d         ; Refresh (cada cuánto el slave consulta al master)
            1d         ; Retry (si falla el refresh, reintentar cada tanto)
            10d        ; Expire (el slave deja de responder si no contacta al master)
            1m         ; Negative TTL (caché de "no existe")
)

; Servidores DNS de la zona
@    1m    IN NS ns1
@    2m    IN NS ns2
ns1        IN A 1.2.3.5
ns2        IN A 1.2.3.6

; Servidores de mail
@   2h IN MX 1 nsmail
@   2h IN MX 2 ns2mail
nsmail  IN A 2.2.2.2
ns2mail IN A 2.2.2.3

; Hosts
www     IN A 6.7.8.9
@       IN A 6.7.8.10
w3      CNAME www

; Registro TXT
@  TXT "hola manola"
```

> **Regla crítica**: `Expire > Refresh + Retry`

### Comandos de mantenimiento

```sh
systemctl restart bind9
systemctl status bind9
tail -f /var/log/syslog    # ver errores en tiempo real

# Verificar que funcionó
dig SOA foo.pdc.lab @localhost
dig A ns1.foo.pdc.lab @localhost
```

### Configurar forwarder (para que el servidor resuelva internet)

En `/etc/bind/named.conf.options`, agregar:

```bind
forwarders {
    1.1.1.1;
    8.8.8.8;
};
```

→ Ver [[Respuestas_guia#E49|E49]], [[Respuestas_guia#E53|E53]]

---

## DNS + HTTP: resolver sin internet

Si no hay internet, para que `curl <nombre>` funcione sin DNS real:

1. Agregar entrada en `/etc/hosts`
2. O levantar un servidor BIND local
3. O especificar dirección IP directamente

```sh
# Ejemplo: acceder a "foo.pdc.lab" sin DNS externo
echo "192.168.1.10 foo.pdc.lab" >> /etc/hosts
curl foo.pdc.lab
```

→ Ver [[Respuestas_guia#E47|E47]]

---

## Split-Horizon

Un mismo nombre resuelve a diferentes IPs según el origen de la consulta:
- Desde adentro de la red → IP privada
- Desde afuera → IP pública

Ocurre en pampero.it.itba.edu.ar: si usás un DNS externo (8.8.8.8) desde dentro del ITBA, obtenés la IP pública, pero la conexión se rechaza por firewall.

→ Ver [[notas/3_dns#Pregunta 1|Pregunta DNS 1]]
