---
title: Friendly 2
layout: default
nav_order: 1
parent: HackmyVM
---

# Friendly 2
{:.no_toc}

---

1. List
{:toc}

---

## Reconocimiento

&nbsp;

```bash
nmap -sn 192.168.1.0/24

nmap -p- --open -sS --min-rate 5000 192.168.1.68 -vvv -oG allPorts -n -Pn
```

Vemos que tiene los puertos 22 y 80 abiertos.

&nbsp;

```bash
nmap -sCV -p22,80 192.168.1.7 -oN targeted
```

```bash
gobsuter dir -u 192.168.1.7 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,php,txt
```
&nbsp;

Se descubren dos directorios en la carpeta raiz del server: **tools y assets**. En assets solo hay archivos, en tools otra pagina diferente.

![](/assets/images/Imagenes/Pasted image 20240524100916.png)
&nbsp;

Al comprobar el código fuente de la sección tools nos encontramos:

![](/assets/images/Imagenes/Pasted image 20240524101140.png)
&nbsp;

Al comprobarlo para ver si ejecuta el codigo php desde la barra de busqueda podemos ver que se ejecuta el código:

![](/assets/images/Imagenes/Pasted image 20240524101322.png)
&nbsp;

Mediante esta vulnerabilidad se nos permite la lectura de archivos del servidor web vulnerable. Por ejemplo, comprobar que existe un archivo en el sistema. Podemos hacerlo con archivo /etc/passwd. 

Si intentamos hacerlo desde el propio directorio del servidor destinada a mostrar la web (/var/html/www/tools) solo nos confirmara que existe, pero no mostrara nada. Con esto vemos que está lo suficientemente saneado para no hacerlo.

![](/assets/images/Imagenes/Pasted image 20240524102458.png)
&nbsp;

Por lo que para poder enumerar el archivo debemos ascender ala máximo directorio posible para poder enumerar el resto de ellos que estén por debajo de él. Para ello habría que retroceder hasta la raiz del servidor con el "comando" **../**, varias veces hasta llegar a la raiz, pudiendo verlo.

![](/assets/images/Imagenes/Pasted image 20240524102430.png)

![](/assets/images/Imagenes/Pasted image 20240524102544.png)

## Explotación

&nbsp;

Con esto ya podemos ver que usuarios tienen una shell asignada (/bin/bash). Uno de los usuarios es **gh0st**, con un directorio personal (/home/gh0st). Con esto al poder enumerar cualquier archivo del servidor podemos comprobar si el usuario gh0st tiene una clave privada RSA y con ello poder ganar potencial acceso. El directorio donde se almacena es **/home/usuario/.ssh/id_rsa**.

![](/assets/images/Imagenes/Pasted image 20240524103106.png)

&nbsp;

Con esto crearemos un archivo llamado id_rsa en nuestra máquina con la información de la rsa del usuario. Además de eso para funcionar correctamente la clave rsa debe contar con permisos de **600**.

![](/assets/images/Imagenes/Pasted image 20240524104004.png)

&nbsp;

Con esto ya podremos intentar acceder como este usuario mediante ssh incluyendo la clave privada del usuario al que se la hemos copiado, de modo que no accederemos con contraseña y el sistema creerá que somos el propio usuario.

![](/assets/images/Imagenes/Pasted image 20240524103932.png)

&nbsp;

Sin embargo esta clave está encriptada, por lo que nos pedirá una contraseña para desencriptar la propia clave, por lo que deberemos extraer los hashes para desencriptarlos con **ssh2john**

![](/assets/images/Imagenes/Pasted image 20240524104221.png)

&nbsp;

Con estos hashes ya podremos intentar desencriptarlo con **john the ripper** mediante un ataque de diccionario con **rockyou.txt**

![](/assets/images/Imagenes/Pasted image 20240524104801.png)

&nbsp;

Con esto ya podriamos acceder con la clave rsa del usuario y la contraseña de esta (**celtic**)

![](/assets/images/Imagenes/Pasted image 20240524104856.png)

&nbsp;
