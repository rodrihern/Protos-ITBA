
# Parcial Practico

## Ejercicio 1






## Ejercicio 3


```sh
curl -v -X POST ejercicio3.protos.foo:3126/debugger -x socks5h://proxy.protos.foo:1080 -H "X-legajo: 65522" -H "Accept-Language: es" -H "Content-Type: application/json" -d '{"fix": "yes"}' -u "admin:admin"
```


## Ejercicio 4

Primero desde nuestra compu hacemos un tunel 

```sh
ssh -R 55522:localhost:8080 rohernandez@pampero.itba.edu.ar
```

y en nuestra compu vamos a redirigir la salida de el puerto 8080 a el `archivo.log`

```sh
nc -l localhost 8080 > archivo.log
```

Ahora en pampero

```sh
tail -f /tmp/parcial-20261 | nc localhost 55522
```

Y ahi ya lo esta redirigiendo, luego en nuestra compu podemos ver el contenido en tiempo real ejecutando

```sh
tail -f archivo.log
```


