---
title: Gestión de imágenes
layout: default
nav_order:
parent: Laboratorios
---

# Gestión de imágenes
{:.no_toc}

---

1. List
{:toc}

## Gestión de imágenes

---
## Descarga de imágenes (docker pull)

Para descargar una imagen debemos indicar el nombre de la imagen, siendo en este caso ubuntu, y sin indicar la versión para descargar la última. En este caso probaremos esto en el sistema Linux MAX con docker ya instalado. El comando resultante sería:

```bash
docker pull ubuntu
```

![](/assets/images/Imagenes/Pasted image 20241007000738.png)

Si volvemos a ejecutar el comando veremos que al estar ya descargada la última imagen en local no la descargará ya que está corriendo la actual. También podemos ver que el ID es el mismo, ya que mientras la imagen no se regenere seguirá el ID sha256 será el mismo.

```bash
docker pull ubuntu
```

![](/assets/images/Imagenes/Pasted image 20241007001343.png)

---
## Guardado de imágenes (docker save)

Otro aspecto interesante de docker es poder guardar las imágenes docker de un equipo que estén en el propio docker engine, en un archivo comprimido, para después exportarlas a otro sistema o para realizar una copia de seguridad. Probaremos a guardar la última versión de ubuntu previa, **ubuntu:latest**.

```bash
docker save ubuntu:latest | gzip > ubuntu_latest.tar.gz
```

![](/assets/images/Imagenes/Pasted image 20241007002116.png)

Podemos ver que ahora tendremos un archivo comprimido con la imagen llamado **ubuntu_latest.tar.gz**.

---
## Listado de imágenes (docker images)

Para visualizar las imágenes que tenemos en nuestro docker engine podemos usar el comando **docker images**.

```bash
docker images
```

![](/assets/images/Imagenes/Pasted image 20241007002637.png)

Vemos que nos muestra una serie de datos sobre cada imagen.

- **REPOSITORY**

Nombre de la imagen de docker y de el repositorio

- **TAG**

Versión de la imagen mostrada, en este caso ambas imágenes son latest

- **IMAGE ID**

Identificador único de las imágenes, se genera automáticamente por el docker engine cada vez que una imagen se regenera.

- **CREATED**

Fecha de la última vez que se realizó una operación de construcción (build) con esa imagen. Puede ser confusa ya que no es la fecha de cuando se creó el docker engine.

- **SIZE**

Tamaño de la imagen

---

## Carga de imágenes (docker load)

En el caso contrario de que queramos cargar una imagen ya guardada y queramos cargarla en nuestro docker engine debemos utilizar el comando **docker load**.

```bash
docker load < ubuntu_latest.tar.gz
```

![](/assets/images/Imagenes/Pasted image 20241007102736.png)

---
## Eliminación de imágenes (docker rmi)

En el momento que haya imágenes docker en nuestro sistema que no vayamos a utilizar, podemos eliminarlas. Podemos indicar de forma explícita el TAG para ser más específicos, ya que si no lo hacemos eliminará la última versión. Sería con el siguiente comando:

```bash
docker rmi ubuntu:latest
```

![](/assets/images/Imagenes/Pasted image 20241007003350 1.png)

El sistema nos informa de las capas que han sido eliminadas. Si volvemos a listar las imágenes veremos que ya no existe la imagen de ubuntu:latest.

```bash
docker images
```

![](/assets/images/Imagenes/Pasted image 20241007003504.png)

---

## Etiquetado de imágenes (docker tag)

En el caso de queramos poder acceder a una imagen con una etiqueta distinta podemos hacerlo con **docker tag**. Esto crea un alias para acceder a la imagen. No se crea una copia, si no que solamente en el caso de que queramos acceder a esa imagen podemos hacerlo con otro nombre. Esto sirve para:

- Subir una imagen con un nombre más concreto
- Control de versiones (v1,v2,v3)
- Renombrar la imagen

```bash
docker tag ubuntu:latest prueba:1.0.4
```

![](/assets/images/Imagenes/Pasted image 20241007103402.png)

Si nos fijamos ambas imágenes comparten el mismo ID.

Después de esto, si borramos la imagen ubuntu con **docker rmi ubuntu**, el acceso a la imagen con prueba:1.0.4 seguirá vigente.



