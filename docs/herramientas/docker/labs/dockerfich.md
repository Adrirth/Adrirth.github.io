---
title: Dockerfile
layout: default
nav_order:
parent: Laboratorios
---

# Dockerfile
{:.no_toc}

---

1. List
{:toc}

---

## Construcción de archivo dockerfile

Crearemos el archivo con el siguiente contenido para crear nuestra propia imagen. Las instrucciones deben ir siempre en mayúscula seguidos de los argumentos, además de ir de manera secuencial, donde cada instrucción genera una capa y luego genera la imagen.

```dockerfile
# Version: 0.0.1 
FROM ubuntu:14.04 
RUN apt-get update 
RUN apt-get install -y nginx 
RUN echo 'Mira, un nuevo contenedor!' \   >/usr/share/nginx/html/index.html 
EXPOSE 80
```

- Podemos poner comentarios con el caracter *#*, en este caso indicamos la versión del dockerfile
- Con **FROM** indicamos la imagen base, que en este caso será ubuntu:14.04. Las órdenes siguientes se ejecutarán en una imagen que parte de esta versión del SO.
- Con **RUN** indicamos comandos a realizar dentro de la imagen, de ubuntu:14.04, siendo en este caso actualizar el sistema e instalar después nginx
- Con **EXPOSE** indicamos que el contenedor exponga un puerto que le indiquemos, en este caso el 80 (HTTP), para acceder a una posible página web del contenedor.

Con estas intrucciones ya podemos crear un nuevo archivo, que no tenga extensión, llamado prueba, con el contenido anterior. Es importante que el nombre del archivo sea **Dockerfile**, o en el caso de tener varios indicar el nombre con el parámetro **-f**.

```bash
# Archivo con nombre por defecto

nano Dockerfile

# Archivo personal

nano prueba
```

![](/assets/images/Imagenes/Pasted image 20241007111336.png)

---
## Construcción de imagen

Una vez creado nuestro archivo dockerfile con las instrucciones necesitamos "construirlo", para más tarde poder lanzarlo como contenedor. Debemos asegurarnos primero de estar en el mismo directorio que el dockerfile. Lo siguiente será construirlo con el comando **docker build**. En nuestro caso al tener el nombre prueba podemos usar el apéndice **-f** para indicar el nombre del archivo:

```bash
docker build -f prueba -t prueba:1.0 .
```

Con el parámetro **-t** indicamos que el nombre de la imagen será prueba:1.0, siendo el tag opcional, y con **.** que Docker debe buscar el dockerfile en el directorio actual.

![](/assets/images/Imagenes/Pasted image 20241007113401.png)

![](/assets/images/Imagenes/Pasted image 20241007114515.png)

---

## Historial de la creación de la imagen

Podemos mostrar el historial de la creación de la imagen con el comando **docker history** seguido del nombre de la imagen que hemos construido.

```bash
docker history prueba:1.0
```

![](/assets/images/Imagenes/Pasted image 20241007114813 1.png)

---

## Instrucciones Dockerfile

### CMD

Permite ejecutar comandos tras la ejecución del contenedor, similar a RUN pero no ejecutándose en la construcción. Su equivalencia sería **docker run -it nombre_imagen /bin/sh**.

```dockerfile
CMD ["/bin/sh"]
```

Podemos concatenar argumentos:

```dockerfile
CMD ["/bin/bash","-l"]
```

De esta manera ya no es necesario indicar el comando al lanzar el contenedor:

```bash
docker run -it prueba:1.0
```

Si después de indicar el comando en la construcción lanzamos la imagen con el apéndice de consola el comando CMD se sobreescribirá y no se ejecutará:

```bash
docker run -it prueba:1.0 /bin/ps
```

![](/assets/images/Imagenes/Pasted image 20241007121824.png)

Lo sabemos ya que no tenemos una consola interactiva del contenedor

---
### ENTRYPOINT

Especifica el comando que se usará cuando se inicie el contenedor a partir de una imagen, a diferencia de CMD, que puede ser sobrescrito al iniciarlo, ENTRYPOINT define el comportamiento inicial del contenedor. Se diferencia de RUN porque este ejecuta comandos al construirse la imagen.

- RUN, comandos en la construcción de la imagen
- CMD, comandos al iniciar el contenedor, pueden sobrescribirse
- ENTRYPOINT, comandos al iniciarse el contenedor, son fijos

```dockerfile
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"] CMD ["-h"]
```

Podemos modificar el ENTRYPOINT especificándolo con --entrypoint.

```bash
docker run -it --entrypoint /bin/sh prueba:1.0
```

---
### WORKDIR

Establece un directorio de trabajo en el que se realizarán las intrucciones subyacentes, como RUN, CMD, ENTRYPOINT, COPY y ADD. En caso de no existir el directorio WORKDIR lo creará. Podemos apilar varias órdenes:

```dockerfile
WORKDIR /app
WORKDIR src

# El directorio será /app/src
```

Podemos cambiar el directorio con la opción **-w** en docker run:

```bash
docker run -w /root prueba:1.0 
```

---
### ENV

Se utiliza para la creación de variables de entorno en el contenedor

```dockerfile
ENV USR_HOME /root
```

Con esto indicamos que la variables USR_HOME sea el directorio root.

Estas variables serán persistentes en el contenedor. Podemos crearlas fuera de este con el el apéndice **-e**.

```bash
docker run -it -e USR_HOME=/Otro kane_project/testing bash

root@ec2fb2d5b5bb:/# echo $USR_HOME

/Otro
```

---
### USER

Especifica el usuario que debe ejecutar los comandos.

```dockerfile
RUN useradd nginx
USER nginx
```

Todas las órdenes siguientes se ejecutarán con el usuario definido. Si definimos el dockerfile de la siguiente forma:

```dockerfile
# Version: 0.0.1 
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y nginx
RUN echo 'Mira, un nuevo contenedor!' \   >/usr/share/nginx/html/index.html
EXPOSE 80
RUN useradd nginx
USER nginx
CMD ["/bin/sh"]
```

Al construirlo y lanzarlo con consola veremos que el contenedor utiliza el usuario nginx. Veremos que al entrar al contenedor seremos ese usuario.

```bash
docker build -f prueba -t prueba:1.0 .
```

```bash
madrid@PruebaMax12:~$ docker run -it prueba:1.0 bash
nginx@e65a26df8b3e:/$ whoami
nginx
nginx@e65a26df8b3e:/$  
```

![](/assets/images/Imagenes/Pasted image 20241007125836.png)

---
### VOLUME

Con esta instrucción podemos crear volúmenes en un contenedor creado a partir de una imagen. Permite definir un directorio dentro de uno o más contenedores para datos compartidos persistentes. Es especialmente útil para los siguientes casos:

- Los datos que se escriban en los volúmenes serán persistentes, incluso si el contenedor se detiene o elimina, como bases de datos y otras aplicaciones
- Compartición de datos, con lo que varios contenedores podrán acceder al volumen para que por ejemplo un contenedor escriba datos en un volumen y otro contenedor los lea.
- Aislamiento de datos, ya que al estar separados del sistema de archivos del contenedor podemos modificarlo sin que afecte al volumen.
- Mejora de rendimiento, ya que docker los gestiona de manera más eficiente.

```dockerfile
VOLUME ["/data"]
```

El equivalente en línea de comandos sería:

```bash
docker run -v path_host:path_contenedor
```

Un caso concreto con los volúmenes es que en cierta medida son directorios virtuales que solo usan los contenedores, de manera que aunque borremos todos los contenedores que usan ese volumen, el volumen y sus datos no se borran, si no que quedan a la espera de ser usados por otro contenedor que le indiquemos.

---
### ADD

Permite copiar archivos y directorios desde el sistema de archivos local al sistema de archivos de la imagen que se está construyendo. La sintaxis:

```bash
ADD origen destino
```

El origen puede ser un archivo, directorio o URL. La ruta será donde deseamos copiar los archivos dentro de la imagen. Podemos además indicar un archivo comprimido como **.tar, .zip**  que al copiarse se descomprimirá de forma automática. Ejemplo:

```dockerfile
FROM ubuntu:20.04 

# Copiar un archivo local a la imagen 
ADD mi_aplicacion /usr/src/app/ 

# Copiar un archivo comprimido y descomprimirlo automáticamente 
ADD configuracion.tar.gz /etc/config/ 

# Descargar un archivo de la web 
ADD https://ejemplo.com/archivo.txt /usr/src/app/
```

Debemos especificar que la ruta sea abierta (**/usr/src/app/**) para que se descompriman los archivos que así queramos.

---
### COPY 

Al igual que ADD copia archivos y directorios desde el sistema local a la imagen, sin embargo no tiene capacidad para copiar URLs ni descompresión de archivos. Por lo que es recomendable que se use este para archivos locales y ADD para copiar URLs y descomprimir archivos


---
### ONBUILD

Agrega acciones que se ejecutarán en el momento en que la imagen base se utilice como base para otra imagen, como al nosotros usar anteriormente *FROM ubuntu*. Son similares a los **triggers** en base de datos.

Ejemplo:

```dockerfile
FROM ubuntu

# Acción que se ejecutará cuando esta imagen se utilice como base para otra
ONBUILD COPY . /app 
ONBUILD RUN npm install
```

Cuando se construya una imagen basada en la nuestra se copiará el directorio actual al directorio **/app** y después de lanzará el comando **npm install**.

---
## Creación imagen multi-stage

Para practicar la creación de una imagen con multi-stage para generar un archivo index.html en una primera fase con el servidor Nginx, y luego copiar ese archivo a una segunda fase con el servidor Apache, esto con el objetivo de optimizar la imagen final y evitar herramientas secundarias.

Primera utilizaremos la imagen de Nginx como base (**nginx:latest**), después ejecutaremos un comando que cree el archivo **index.html** y colocarlo en la ubicación que nginx utiliza para servir archivos estáticos. Este paso o stage se denomina **builder**.

Crearemos el dockerfile con las instrucciones:

```dockerfile
FROM nginx:latest as builder

RUN echo 'Hello from first multi-stage' > /usr/share/nginx/html/index.html
```

![](/assets/images/Imagenes/Pasted image 20241007152347 1.png)

Después en el mismo fichero añadiremos la segunda fase o stage para crear la imagen final con apache, que sustituirá la imagen de nginx por la de apache. Después usaremos la instrucción **COPY** para copiar el archivo index.html creado anterior para añadirlo al directorio donde apache busca los archivos para server(**/usr/local/apache2/htdocs**).

![](/assets/images/Imagenes/Pasted image 20241007152353.png)

Con la opción **--from=builder** le indicamos que ese fichero lo saque de una stage anterior, en este caso de nginx y que lo copie en el directorio de apache para servir el fichero index.hmtl.

Con el dockerfile ya hecho podemos guardarlo y construirlo. En este caso como anteriormente podemos usar el comando **docker build**:

```bash
docker build -f nginx -t httpd-multistage .
```

![](/assets/images/Imagenes/Pasted image 20241007152855 1.png)

Con esto ya tendremos nuestra imagen creada, solo falta lanzarla en un contenedor para poder acceder al archivo en el servidor. Para ello indicamos que queremos ejecutar el contenedor con el siguiente comando:

```bash
docker run -d -p 8081:80 httpd-multistage
```

Con este comando hacemos las siguientes instrucciones al lanzar el contenedor:

- **docker run**, lanzamos un contenedor con una imagen
- **-d**, con esto le decimos que se lance en segundo plano, sin log ni bloqueo de terminal
- **-p**, con el que indicamos el mapeo de puertos, en este caso indicamos que queremos acceder por el puerto 8081 de nuestra máquina host al 80 del contenedor, que es donde el sevidor HTTP está escuchando peticiones.
- **httpd-multistaged**, nombre de la imagen que hemos creado anteriormente

![](/assets/images/Imagenes/Pasted image 20241008083446.png)

Veremos que se ejecutará en segundo plano y para comprobar que efectivamente funciona podemos o bien hacer una petición al puerto 8081 de nuestra máquina o con el navegador buscar esa dirección, **localhost:8081**.

![](/assets/images/Imagenes/Pasted image 20241008083629.png)

![](/assets/images/Imagenes/Pasted image 20241008083655.png)

Y vemos que podemos hacer la petición al servidor http del contenedor docker. Podemos comprobar también que está en funcionamiento con el comando **docker ps**, para mostrarnos los contenedores que están corriendo actualmente.

![](/assets/images/Imagenes/Pasted image 20241008083939.png)

---

## Errores comunes

#### Nombre de archivo Dockerfile incorrecto

Es importante asegurarse que el archivo con las instrucciones de construcción de imagen se llame **Dockerfile**, sin extensión y con la primera mayúscula, ya que si no docker no encontrará el archivo.

```bash
ls

Dockerfile

docker build -t prueba:1.0 . 
```

En caso de que hayamos guardado el archivo con otro nombre podemos construirlo junto al parámetro **-f**. 

```bash
docker build -f prueba -t prueba:1.0 .
```


#### Directorio del archivo docker no indicado

En el caso de que no indiquemos donde se encuentra el archivo dockerfile no podremos construir la imagen, por lo que es importante especificarlo siempre, ya sea en local o de un repositorio. En el caso de estar en el directorio actual podemos indicarlo con *.* ,

```bash
# Fichero dockerfile en el fichero actual

docker build -t prueba:1.0 .
```

```bash
# Fichero dockerfile en un repositorio github

docker build -t prueba:1.0 github.com/ruta-repo
```

#### No poder borrar imagen por estar usándose



#### Otros errores

- Cada vez que la imagen está construida, trata las imágenes de los pasos anteriores como caché - **docker sytem prune -a**
- Las imágenes construidas en los pasos anteriores implica que ya no se procesan 

- Podemos forzar a que se realice de nuevo la construcción por medio de la opción --no-cache en caso de que no detecte 

- Un truco para refrescar la caché cuando queramos es agregando variables de entorno 

- Al agregar una variable de entorno, si esta es modificada, la caché es ignorada automáticamente desde la posición del cambio