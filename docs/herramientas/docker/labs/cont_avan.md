---
title: Contenedores avanzados
layout: default
nav_order:
parent: Laboratorios
---

# Contenedores avanzados
{:.no_toc}

---

1. List
{:toc}

---


## Exportación del sistema de archivos de un contenedor

Con esta operación exportaremos todo el sistema de archivos de un contenedor, en este caso con el contenedor tomcat. Si comprobamos vemos que están funcionando actualmente

```bash
docker run tomcat:latest -d

docker ps -a
```

![](/assets/images/Imagenes/Pasted image 20241010090637.png)

Para exportar sus datos podemos usar el comando:

```bash
docker export tomcat > tomcat_export.tar
```

![](/assets/images/Imagenes/Pasted image 20241010093825.png)

Y comprobamos que se ha exportado al directorio actual.

---

## Importración del sistema de archivos de un contenedor (docker import)

Al contrario que el exportar los datos, también podemos importar datos a un contenedor para crear una nueva imagen, los cuáles contienen un sistema de archivos. Podemos usar tanto una URL como un archivo local, en nuestro caso siendo el archivo exportado anteriormente:

```bash
docker import tomcat_export.tar
```

![](/assets/images/Imagenes/Pasted image 20241010094024.png)

Veremos que ahora tenemos una imagen creada con el sistema de archivos del otro contenedor. Al no tener nombre podemos arrancarlo indicando su id con docker run.

---

## Ejecución de comandos contra un contenedor (docker exec)

En ocasiones puede interesarnos el ejecutar comandos contra un contenedor para que este nos de un resultado, o de conectarnos al interior del contenedor para buscar anomalías. En este caso nos conectaremos al contenedor de **tomcat** con el objetivo de poner al día los repositorios e instalarle la herramienta **htop**, que no está incluida en él.

- Primero, si no está ya en marcha iniciaremos el contenedor en segundo plano.

```bash
docker run --name=tomcat -d tomcat
```

- Después indicaremos que queremos una terminal del propio contenedor

```bash
docker exec -it tomcat bash
```

Con el parámetro **-it** indicamos que queremos una terminal interactiva de entrada-salida. El último parámetro será el nombre del binario que se encuentre dentro del contenedor que queremos ejecutar, en este caso bash para abrir un proceso de consola para conectarnos dentro. Podremos ver que el prompt del sistema cambia, y la shell conmutará a usuario root, dentro del contenedor.

![](/assets/images/Imagenes/Pasted image 20241010095512.png)

Todo lo que ejecutemos lo estaremos realizando a nivel del sandbox del contenedor docker, no en la máquina anfitrión. Para actualizar repositorios:

```bash
apt-get udpate
```

Y para instalar la herramienta htop:

```bash
apt-get install htop
```

![](/assets/images/Imagenes/Pasted image 20241010095721.png)

Si la ejecutamos con **htop** podremos ver los procesos del contenedor:

![](/assets/images/Imagenes/Pasted image 20241010095756.png)

Una vez hecho esto podemos salir de htop con *F10* o *q* y del contenedor con **exit**.

![](/assets/images/Imagenes/Pasted image 20241010095915.png)

---

## Cambios en el sistema de un contenedor que está en funcionamiento (docker commit)

Otro comando útil es *docker commit*, que nos sirve para crear una imagen de un contenedor existente, tomando el estado en el que se encuentre y guardándolo como una imagen, permitiendo que ese estado se reutilice o se distribuya. 

Esto nos permite no tener que usar un dockerfile para construir una imagen con *docker build*. Sin embargo este método no es aconsejable ya que no deja constancia de las operaciones que se hayan realizado.

Podemos probarlo con el contenedor anterior de tomcat:

```bash
docker commit <id_contenedor> <nueva_imagen_a_generar>

docker commit -m="Mi imagen tomcat modificada" --author="Adri" tomcat nueva_imagen_tomcat:1.0.6
```

Veremos que se ha creado una nueva imagen:

![](/assets/images/Imagenes/Pasted image 20241010101003.png)

- Docker commit solo guarda las diferencias entre la última imagen de contenedor y el estado actual. 
- Las actualizaciones son muy ligeras, ya que solo almacenan los cambios entre la imagen de ejecución del contenedor y la nueva imagen generada. 
- También está permitido agregar más información a la imagen 
- Como el autor, una descripción y un tag a la imagen

---

## Inspección de contenedores (docker inspect)

Si queremos comprobar la estructura de armado del contenedor tenemos disponible el comando *docker inspect*. Puede aplicarse a cualquier entidad de docker (redes, volúmenes, imágenes). Podemos ver la información de la imagen que creamos anteriormente.

```bash
docker inspect nueva_imagen_tomcat
```

![](/assets/images/Imagenes/Pasted image 20241010101752.png)

Podemos ver toda su información, desde el autor, comentarios, y su configuración. Podemos filtrar la información para ver exactamente que queremos consultar.

---

## Aislamiento de procesos del contenedor

Un contenedor es realmente un proceso aislado del propio kernel, de forma que internamente si ejecutamos un comando tipo **ps** únicamente deberíamos observar el propio lanzaminto del comando, así como el proceso que mantenga al contenedor activo como hilo principal, ya sea un servidor web, consola shellm, etc.

Primero debemos conectarnos iniciando una ejecución bash:

```bash
docker exec -it tomcat bash
```

Después ya dentro podremos ver los procesos del contenedor con el comando:

```bash
ps -aux
```

![](/assets/images/Imagenes/Pasted image 20241010102743.png)


> **NOTA**: Un contenedor posee como proceso principal (PID 1) el proceso de ejecución del contenedor, si este cae, el contenedor muere. Se trata del mismo comportamiento que cualquier sistema linux, si el proceso 1 (initd/systemd) muere, el sistema operativo muere.


En caso de que el contenedor tuviera algún problema y se cayera a nivel interno, el propio contenedor se pararía de forma directa. Finalmente para salir del contenedor ejecutamos *exit*.

---

## Adjuntándonos al proceso (docker attach)

Con el comando anteriormente visto **docker exec** podemos llevar a cabo la ejecución de un comando concreto sobre un contenedor, de manera adicional en otro proceso. Pero en otras ocasiones quizás nos interese conectarnos a ese mismo contenedor con el mismo proceso.

Con el comando **attach** podemos adquirir en primer plano el proceso. Podemos hacerlo con el comando:

```bash
docker attach tomcat
```

Con esto estamos directamente manejando el proceso del que se encarga el proceso, por lo que si abrimos en otra consola el mimo proceso con **docker attach tomcat**, al cerrarlo en una consola veremos que se cierra también en la otra, ya que estamos administrando el proceso en primer plano.

![](/assets/images/Imagenes/Pasted image 20241010111014.png)

---

## Visualización de procesos (docker top)

Otro comando para ver los procesos internos del docker sin tener que entrar en él desde fuera es **docker top**. Para ello:

- Eliminamos el contenedor anterior de tomcat 

```bash
docker rm tomcat
```

- Después volvemos a poner en marcha el contenedor en segundo plano

```bash
docker run --name=tomcat -d tomcat
```

- Por último podremos ver los procesos del contendor

```bash
docker top tomcat

madrid@PruebaMax12:~$ docker top tomcat
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                9026                9005                11                  11:16               ?                   00:00:02            /opt/java/openjdk/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED --enable-native-access=ALL-UNNAMED -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start

```

![](/assets/images/Imagenes/Pasted image 20241010111709.png)

Podemos ver toda la información del proceso, su usuario, el tiempo activo y el contenido del proceso.

---

## Obtención de traza del contenedor (docker logs)

Otro comando para obtener información de un contenedor es **docker logs**, con el podemos vemos los logs emitidos por un contenedor, tanto para su salida estándar como de errores o logs históricos.

![](/assets/images/Imagenes/Pasted image 20241010112808.png)

Podemos filtrar con el apéndice **grep** para encontrar una línea concreta en el log.

```bash
docker logs tomcat | grep error
```

![](/assets/images/Imagenes/Pasted image 20241010112944.png)

Podemos añadir el apéndice **-f** para el seguimiento real de los logs del contenedor.

---

## Borrar todos los contenedores

Podemos borrar todos los contenedores disponibles con **docker rm $(docker ps -a -q) -f**. Con ello primero se muestran todos los contenedores de docker y se fuerza su borrado.

```bash
docker rm $(docker ps -a -q) -f
```

![](/assets/images/Imagenes/Pasted image 20241010113420.png)

---

## Errores
### Error al importar el contenido

*write /dev/stdout: no space left on device*

Este error ocurre al no tener espacio suficiente en el sistema, si estamos en una máquina virtual debemos expandir el almacenamiento en las propiedades de discos virtuales, y en la máquina virtual extender la partición actual con gparted o similar.

*Error response from daemon: unexpected EOF*

Error al no haber contenido en una exportación de un contenedor.

