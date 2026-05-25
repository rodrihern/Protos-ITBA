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

→ Ver [[guia_respuestas#E42|E42]], [[guia_respuestas#E44|E44]], [[guia_respuestas#E45|E45]], [[guia_respuestas#E62|E62]]  
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

→ Ver [[guia_respuestas#E46|E46]]

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

@ IN SOA ns1.foo.pdc.lab. rohernandez.itba.edu.ar. (
            20260331   ; Serial (fecha o número, incrementar al modificar)
            7d         ; Refresh (cada cuánto el slave consulta al master)
            1d         ; Retry (si falla el refresh, reintentar cada tanto)
            10d        ; Expire (el slave deja de responder si no contacta al master)
            1m         ; Negative TTL (caché de "no existe")
)

; Servidores DNS de la zona (NS es obligatorio — sin esto bind9 rechaza cargar la zona)
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
# Validar ANTES de recargar (hacerlo siempre)
named-checkconf                                              # verifica named.conf
named-checkzone foo.pdc.lab /etc/bind/foo.pdc.lab.local     # verifica zone file

systemctl reload bind9     # recargar sin bajar el servicio
systemctl restart bind9    # reinicio completo
systemctl status bind9
tail -f /var/log/syslog | grep named    # ver errores en tiempo real

# Verificar que funcionó
dig SOA foo.pdc.lab @localhost
dig A ns1.foo.pdc.lab @localhost
```

### Diagnóstico de errores comunes

| Síntoma | Causa probable |
|---|---|
| `SERVFAIL` + sin flag `aa` + `AUTHORITY: 0` | La zona no se cargó — sintaxis incorrecta en el zone file o path mal en named.conf.local |
| `NXDOMAIN` con flag `aa` | La zona cargó bien pero el registro no existe |
| `SERVFAIL` con flag `aa` | La zona cargó pero no puede resolver (ej: falta forwarder para internet) |
| Cambios que no se reflejan | Se olvidó hacer `systemctl reload bind9` |

**Errores frecuentes en zone files:**
- **Faltan registros NS** — zona obligatoriamente necesita al menos un NS. Sin NS, bind9 rechaza cargar la zona → SERVFAIL + sin flag `aa` + AUTHORITY: 0
- **NS apunta a un nombre sin A record** — el nombre del NS debe tener su propio A record en la zona
- **Targets de MX sin A record** — si `mail20` aparece en un MX, necesita un `mail20 IN A ...` en la zona
- **Falta el `.` al final de un FQDN** — `example.org` sin punto final se expande a `example.org.carlitos.com.ar.`; siempre poner punto al referenciar nombres externos
- **SOA MNAME distinto a los NS records** — el primer campo del SOA debería coincidir con uno de los NS records
- **Serial sin incrementar** después de un cambio (los slaves no propagan)

> Siempre correr `named-checkzone` antes de recargar — si da error, bind9 ignora silenciosamente la zona y sigue con la anterior (o sin ella).

### Configurar forwarder (para que el servidor resuelva internet)

En `/etc/bind/named.conf.options`, agregar:

```bind
forwarders {
    1.1.1.1;
    8.8.8.8;
};
```

→ Ver [[guia_respuestas#E49|E49]], [[guia_respuestas#E53|E53]]

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

→ Ver [[guia_respuestas#E47|E47]]

---

## Split-Horizon

Un mismo nombre resuelve a diferentes IPs según el origen de la consulta:
- Desde adentro de la red → IP privada
- Desde afuera → IP pública

Ocurre en pampero.it.itba.edu.ar: si usás un DNS externo (8.8.8.8) desde dentro del ITBA, obtenés la IP pública, pero la conexión se rechaza por firewall.

→ Ver [[notas/3_dns#Pregunta 1|Pregunta DNS 1]]
