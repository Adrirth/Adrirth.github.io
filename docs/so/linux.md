---
title: Linux
layout: default
parent: Sistemas
nav_order: 1
---

# Linux
{:.no_toc}

---

1. List
{:toc}

---

Creado a partir del SO Unix (Ken Thompson, Dennis Ritchie, 1970), fue lanzado en 1977, siendo limitado por estar basado en Unix, propiedad de AT&T. Después de eso Richard Stallman empieza el proyecto GNU en 1983, con la finalidad de crear un sistema operativo parecido a Unix libre, dando como resultado la Licencia Pública General (GPL, General Public License). 

Primeramente Linux empezó como un proyecto personal empezado por un estudiante finlandés llamado Linus Torvalds, con el objetivo de crear un nuevo y libre kernel de sistema operativo. Lo que empezó como unas cuantas lineas de código en C con una licencia de prohibición de distribución comercial ha llegado a nuestros días con la última versión con más de 23 millones de líneas de código bajo la licencia de GNUv2.

Cuenta con más de 600 distribuciones, siendo las más usadas Debian, Ubuntu, Fedora, Arch, Manjaro, RedHat, Gentoo o Linux Mint.

Es considerado uno de los sistemas operativos más seguros, siendo cada vez menos frecuente vulnerabilidades de kernel. Es menos susceptible a malware que windows y siendo más estable y con mejor rendimiento. Sin embargo puede ser algo complicado para principiantes al no tener tantos drivers para hardware como Windows.

No solo se encuentra en ordenadores, sino que sistemas como Android están basados en Linux, además de servidores, consolas y mucho más. 

## Principios de Linux


| Principio                                                                                      | Descripción                                                                                                                                                                          |
| ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Todo es un archivo**                                                                         | Todos los archivos de configuración para varios de los servicios corriendo en el sistema de Linux están guardados en uno o más archivos de texto                                     |
| **Programas pequeños de un solo uso**                                                          | Linux ofrece diferentes herramientas con las que podremos trabajar, que pueden combinarse                                                                                            |
| **Posibilidad de encadenar diferentes programas conjuntamente para realizar tareas complejas** | La integración y combinación de las diferentes herramientas para permitir realizar varias tareas largas y complejas, como el procesado y filtrado de resultados de datos específicos |
| **Evitar interfaces de usuarios gráfica**                                                      | Diseñado para trabajar solamente con la shell (terminal), que le da al usuario mayor control del sistema operativo                                                                   |
| **Configuración de datos guardados en un archivo de texto**                                    | Un ejemplo de ésto es el archivo **/etc/passwd**, que almacena todos los usuarios registrados en el sistema                                                                          |

## Componentes

| Componente             | Descripción                                                                                                                                                                                                                                                                                                                     |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Bootloader**         | Parte del código que corre la guía para empezar el proceso de iniciar el sistema operativo. Parrot usa por ejemplo el bootloader GRUB                                                                                                                                                                                           |
| **Kernel SO**          | El kernel es el principal componente de un sistema operativo. Gestiona los recursos de los dispositivos I/O a nivel de hardware                                                                                                                                                                                                 |
| **Daemons**            | Servicios en segundo plano de Linux. Su propósito es asegurarse de funciones clave como imprimir, multimedia y planificación funcionar correctamente. Estos pequeños programas son cargados después de que hayamos encendido o iniciado sesión en nuestro ordenador.                                                            |
| **Shell SO**           | La shell del sistema operativo o interprete de lenguaje de comandos (cli), es la interfaz entre el sistema operativo y el usuario. Permite al usuario decir al sistema operativo que hacer. Las más usadas son Bash, Tcsh, Ksh, Zsh y Fish                                                                                      |
| **Servidor gráfico**   | Proporciona un sub-sistema gráfico (servidor) llamado "X" que permite programas gráficos para correr de forma local o remota en el sistema pantalla-X                                                                                                                                                                           |
| **Gestor de Ventanas** | También conocido como interfaz gráfica de usuario (GUI). Hay varias opciones, como GNOME, KDE, MATE, Unity o Cinnamon. Un escritorio tiene varias aplicaciones, como buscadores web o de archivos. Estos permiten al usuario acceso y gestión de las opciones y servicios esenciales y de uso frecuente de un sistema operativo |
| **Utilidades**         | Aplicaciones o utilidades son programas que ejecutan funciones particulares para el usuario u otros programas                                                                                                                                                                                                                   |

## Arquitectura

| Capa                     | Descripción                                                                                                                                                                                                                                                                               |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Hardware**             | Dispositivos periféricos como la RAM del sistema, discos duros, CPU y otros                                                                                                                                                                                                               |
| **Kernel**               | El núcleo del sistema operativo de Linux cuya función es la de virtualizar y controlar recursos comunes de hardware como la CPU, memoria asignada, datos accesibles y otros. El kernel proporciona a cada proceso sus propios recursos virtuales y previene/mitiga conflictos entre estos |
| **Shell**                | Interfaz de línea de comandos (CLI), también conocido como shell con la que un usuario puede introducir comandos para ejecutar funciones del kernel                                                                                                                                       |
| **Utilidad del sistema** | Hace accesible al usuario toda la funcionalidad del sistema operativo                                                                                                                                                                                                                     |

## Jerarquía de sistema de archivos

![](/assets/images/Imagenes/Pasted image 20240528105140.png)

https://www.pathname.com/fhs/

| **Ruta** | **Descripción**                                                                                                                                                                                                                                                                                                                      |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `/`      | El directorio de nivel superior es el sistema de archivos raíz y contiene todos los archivos necesarios para arrancar el sistema operativo antes de que se monten otros sistemas de archivos, así como los archivos necesarios para arrancar los otros sistemas de archivos. Después del arranque, todos los demás sistemas de archivos se montan en puntos de montaje estándar como subdirectorios del raíz. |
| `/bin`   | Contiene binarios de comandos esenciales                                                                                                                                                                                                                                                                                             |
| `/boot`  | Consiste en el cargador de arranque estático, el ejecutable del kernel y los archivos necesarios para arrancar el sistema operativo Linux.                                                                                                                                                                                             |
| `/dev`   | Contiene archivos de dispositivos para facilitar el acceso a cada dispositivo de hardware conectado al sistema.                                                                                                                                                                                                                      |
| `/etc`   | Archivos de configuración del sistema local. Los archivos de configuración de las aplicaciones instaladas también pueden guardarse aquí.                                                                                                                                                                                              |
| `/home`  | Cada usuario en el sistema tiene un subdirectorio aquí para almacenamiento.                                                                                                                                                                                                                                                           |
| `/lib`   | Archivos de bibliotecas compartidas que se requieren para el arranque del sistema.                                                                                                                                                                                                                                                   |
| `/media` | Los dispositivos de medios extraíbles externos, como unidades USB, se montan aquí.                                                                                                                                                                                                                                                   |
| `/mnt`   | Punto de montaje temporal para sistemas de archivos regulares.                                                                                                                                                                                                                                                                        |
| `/opt`   | Los archivos opcionales, como herramientas de terceros, pueden guardarse aquí.                                                                                                                                                                                                                                                        |
| `/root`  | El directorio de inicio del usuario root.                                                                                                                                                                                                                                                                                            |
| `/sbin`  | Este directorio contiene ejecutables utilizados para la administración del sistema (archivos binarios del sistema).                                                                                                                                                                                                                  |
| `/tmp`   | El sistema operativo y muchos programas utilizan este directorio para almacenar archivos temporales. Este directorio generalmente se limpia al arrancar el sistema y puede ser eliminado en otros momentos sin previo aviso.                                                                                                          |
| `/usr`   | Contiene ejecutables, bibliotecas, archivos de manual, etc.                                                                                                                                                                                                                                                                           |
| `/var`   | Este directorio contiene archivos de datos variables, como archivos de registro, buzones de correo, archivos relacionados con aplicaciones web, archivos de cron, y más.                                                                                                                                                              |

https://explainshell.com/


**which** -> Muestra la ruta o link que debería ejecutarse

```shell
Adrirth@htb[/htb]$ which python

/usr/bin/python
```

**find** -> Buscar archivos y carpetas, filrtar resultados

```shell

Adrirth@htb[/htb]$ find / -type f -name *.conf -user root -size +20k -newermt 2020-03-03 -exec ls -al {} \; 2>/dev/null

```shell-session
-rw-r--r-- 1 root root 136392 Apr 25 20:29 /usr/src/linux-headers-5.5.0-1parrot1-amd64/include/config/auto.conf
```


| **Opción**            | **Descripción**                                                                                                                                                                                                                                                                         |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-type f`             | Aquí definimos el tipo de objeto buscado. En este caso, '`f`' significa '`file`' (archivo).                                                                                                                                                                                               |
| `-name *.conf`        | Con '`-name`', indicamos el nombre del archivo que estamos buscando. El asterisco (`*`) representa 'todos' los archivos con la extensión '`.conf`'.                                                                                                                                      |
| `-user root`          | Esta opción filtra todos los archivos cuyo propietario es el usuario root.                                                                                                                                                                                                               |
| `-size +20k`          | Podemos filtrar todos los archivos encontrados y especificar que solo queremos ver los archivos que son más grandes que 20 KiB.                                                                                                                                                          |
| `-newermt 2020-03-03` | Con esta opción, establecemos la fecha. Solo se presentarán los archivos más recientes que la fecha especificada.                                                                                                                                                                        |
| `-exec ls -al {} \;`  | Esta opción ejecuta el comando especificado, usando las llaves como marcadores de posición para cada resultado. La barra invertida escapa el siguiente carácter para evitar que el shell lo interprete, ya que de otro modo, el punto y coma terminaría el comando sin alcanzar la redirección. |
| `2>/dev/null`         | Esta es una redirección de `STDERR` al 'dispositivo null', que veremos en la próxima sección. Esta redirección asegura que no se muestren errores en la terminal. Esta redirección no debe ser una opción del comando 'find'.                                                              |


**locate** -> A diferencia de find, locate utiiza una base de datos local que contiene toda la informacion sobre archivos y carpetas, que podemos actualizar con **sudo updatedb**

```shell
Adrirth@htb[/htb]$ locate *.conf

/etc/GeoIP.conf
/etc/NetworkManager/NetworkManager.conf
/etc/UPower/UPower.conf
/etc/adduser.conf
<SNIP>
```

Número total de paquetes en Debian: 

		dpkg -l | grep '^ii' | wc -l

#### Bibliografía

[**How-to**](https://linux.die.net/HOWTO/HOWTO-INDEX/categories.html)
