---
title: Greenhorn
layout: default
parent: HackTheBox
---


# Greenhorn
{:.no_toc}

---

1. List
{:toc}

---

Realizada gracias a la resolución de [César Guerrero Dominguez](https://medium.com/@cesarguerrerodominguez/resolviendo-la-maquina-greenhorn-htb-writeup-en-espa%C3%B1ol-9565fe2d20c7).

## Reconocimiento

Primero comprobaremos que tenemos conexión con la máquina con un simple ping. Después veremos los puertos que tiene abiertos.

![](/assets/images/Imagenes/Pasted image 20240915133519.png)

Vemos que tiene un ttl de 63, por lo que podemos deducir que es una sistema linux.

```shell
nmap -p- -sS --open --min-rate 5000 10.10.11.25 -n -Pn -oG allPorts
```

![](/assets/images/Imagenes/Pasted image 20240915132938.png)

Vemos que tiene los puertos 22, 80, 3000 y 8000 abiertos, por lo que ahora veremos sus versiones.

```shell
nmap -sCV -p22,80,3000,6000 10.10.11.25 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-15 13:31 CEST
Nmap scan report for 10.10.11.25
Host is up (0.027s latency).

PORT     STATE  SERVICE VERSION
22/tcp   open   ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 57:d6:92:8a:72:44:84:17:29:eb:5c:c9:63:6a:fe:fd (ECDSA)
|_  256 40:ea:17:b1:b6:c5:3f:42:56:67:4a:3c:ee:75:23:2f (ED25519)
80/tcp   closed http
3000/tcp open   ppp?
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=ee02b3363f37c5c5; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=wTtBuISJ7bBuBymj6tanWqNcudY6MTcyNjM5OTg3NzYwNzgxMjcwMg; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Sun, 15 Sep 2024 11:31:17 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>GreenHorn</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLCJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYX
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: HEAD
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=4b6c86e3378c9713; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=pnbyz8zeoH-YWvAVeskLE2JgjoA6MTcyNjM5OTg4Mjc5MTcxNzE1MA; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Sun, 15 Sep 2024 11:31:22 GMT
|_    Content-Length: 0
6000/tcp closed X11
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
```

Como habíamos deducido el sistema es Linux, en este caso con Ubuntu Jammy.

Si intentemos acceder a la ip de la máquina desde un navegador vemos que intenta resolver con la dirección greenhorn.htb, por lo que tendremos que añadirla al archivo de resolución de hosts **/etc/hosts**.

![](/assets/images/Imagenes/Pasted image 20240915134614.png)

Podemos ver una página de bienvenida con dos pestañas, en las cuáles abajo existe un link para entrar como administrador.

![](/assets/images/Imagenes/Pasted image 20240915134742.png)

![](/assets/images/Imagenes/Pasted image 20240915134828.png)

Vemos que utiliza el software Pluck, que parece ser un administrador de contenidos, en su versión 4.7.18. Si buscamos con searchsploit vemos que existen varias vulnerabilidades ante esta versión, como RCE y XSS. 

![](/assets/images/Imagenes/Pasted image 20240915135248.png)

Buscando en internet vemos que existe un script para subir archivos de forma no autorizada.

![](/assets/images/Imagenes/Pasted image 20240915135205.png)

Con esta necesitamos editar el archivo **poc.py** con la dirección de la máquina vícitima, además de crear un payload con un archivo zip que contenga una reverse shell con php. En mi caso y siguiendo el consejo del repositorio he usado [esta](https://github.com/pentestmonkey/php-reverse-shell).

Sin embargo parece ser que para subir el archivo necesitamos unas credenciales válidas, las cuáles no tenemos. 

Buscando por los otros puertos veo que el 6000 no es accesible pero el 3000 pertenece a un software llamado gitea, que sirve para el control de versiones de software. Investigando por la página, encuentro que existe un repositorio abierto en el que se encuentra un archivo login.php.

![](/assets/images/Imagenes/Pasted image 20240915150154.png)

En el caso de que pluck este instalado se utiliza un archivo para loggear llamado pass.php.


![](/assets/images/Imagenes/Pasted image 20240915150319.png)

Este se encuentra dentro de el directorio data/settings

![](/assets/images/Imagenes/Pasted image 20240915150434.png)

Dentro de él parece haber una cadena de caracteres, que podría ser una hash.

```php
<?php
$ww = 'd5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe9612e3a4d770486743c5580556c0d838b51749de15530f87fb793afdcc689b6b39024d7790163'?>
```

Si utilizamos la web hashes.com vemos que efectivamente es un hash y su valor

```shell
d5443aef1b64544f3685bf112f6c405218c573c7279a831b1fe9612e3a4d770486743c5580556c0d838b51749de15530f87fb793afdcc689b6b39024d7790163:iloveyou1

[SEARCH AGAIN](https://hashes.com/en/decrypt/hash)
```

![](/assets/images/Imagenes/Pasted image 20240915151709.png)

## Explotación

Si intentamos acceder con la contraseña que hemos sacado en la página de login vemos que estamos en la pestaña de administrador. Por lo que ahora podemos probar a subir la reverse shell con estas credenciales (admin/iloveyou1). Si intentamos enviar el payload pero esta vez con las credenciales, además de antes estar en escucha por el puerto 443, vemos que hemos conseguido entrar al sistema.

![](/assets/images/Imagenes/Pasted image 20240915161340.png)


Para estabilizar la shell utilizaré la siguiente serie de comandos:

```shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
CTRL + Z
stty raw -echo; fg
ENTER
```


## Escalada de privilegios

En la máquina existe el usuario junior y git, sin embargo este último no tenemos permisos suficientes para verlo. Podemos intentar probar la contraseña que hemos encontrado anteriormente para ver si es válida, y efectivamente estamos logueados como junior, con la flag de usuario en su directorio.

![](/assets/images/Imagenes/Pasted image 20240915162040.png)

Vemos que hay además un archivo pdf, el cuál no podemos abrir desde la máquina víctima, por lo que tendremos que enviarlo a nuestra máquina con netcat.

![](/assets/images/Imagenes/Pasted image 20240915163409.png)

El pdf estará donde hayamos realizado el netcat, con el siguiente contenido:

![](/assets/images/Imagenes/Pasted image 20240915163944.png)

Si buscamos como quitar el blur de la contraseña veremos que existen programas para realizarlo, en este caso usaré [spipm/Depix](https://github.com/spipm/Depix?source=post_page-----9565fe2d20c7--------------------------------).

Primero clonaremos el repositorio, después deberemos pasar el pdf a imagen con la utilidad pdfimages (poppler-utils), y después pasar la imagen por el script de de-blur.

![](/assets/images/Imagenes/Pasted image 20240915165213.png)

![](/assets/images/Imagenes/Pasted image 20240915170032.png)

Solo quedaría introducir una imagen de referencia de blur, pudiendo usar alguna de las que inlcuye el repositorio, el nombre de salida y el archivo de la imagen pixelada (.ppm).
  
```
python3 depix.py -p  ./openvas-000.ppm -s images/searchimages/debruinseq_notepad_Windows10_close.png -o ./solved.png
```

![](/assets/images/Imagenes/Pasted image 20240915171349.png)

![](/assets/images/Imagenes/Pasted image 20240915171559.png)

Al finalizar la imagen resultante estará en el directorio donde hayamos indicado, en la cuál si la abrimos vemos que ha reconstruido parte de la imagen y llegando a ser legible. En el caso de no tener un resultado más funcional, podemos probar con otras imágenes. Ésta no es legible al 100%, pero podemos deducir que puede decir _sidefromside_ o _hidefromhide_. Después de probarla, finalmente es _sidefromsidetheothersidesidefromsidetheotherside_. Si accedemos al directorio root encontraremos la flag de este (root.txt).

![](/assets/images/Imagenes/Pasted image 20240915165620.png)

