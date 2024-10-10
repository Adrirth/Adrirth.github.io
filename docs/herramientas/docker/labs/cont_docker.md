---
title: Contenedores docker
layout: default
nav_order:
parent: Laboratorios
---

# Contenedores docker
{:.no_toc}

---

1. List
{:toc}

---

## Arrancar un contenedor (docker run)

Para arrancar un contenedor podemos usar docker run, indicando el nombre. En este caso lanzaremos la imagen de tomcat.

```bash
docker run --name=tomcat -d tomcat
```

![](/assets/images/Imagenes/Pasted image 20241008091643.png)

Al no estar la imagen disponible localmente la descarga del hub. Con el comando anterior le indicamos que queremos lanzar la imagen con nombre tomcat y que el contenedor se lance en segundo plano. Si volviéramos a lanzar un contenedor con la imagen base de tomcat gracias a la caché lo lanzaría más rápido, al no tener que descargar nada del hub.

---
## Listado de contenedores (docker ps)

Para listar los contenedores activos podemos usar el comando **docker container ls | docker ps**.

```bash
docker container ls

docker ps
```

![](/assets/images/Imagenes/Pasted image 20241008093042.png)

Vemos que ambos muestran el mismo contenido, en este caso dos contenedores, uno con tomcat y otro con un servidor httpd.

- **IMAGE**, indica el nombre de la imagen original de la que parte el contenedor.
- **COMMAND**, indica el comando del proceso que ha ejecutado el contenedor (ID 1).
- **CREATED**, indica la fecha de creación del contenedor
- **STATUS**, nos indica el estado del contenedor. En caso de salida, indicará el id del proceso (0 OK).
- **PORTS**, los puertos publicados al exterior.
- **NAMES**, por defecto, los contenedores poseen un nombre único que no se puede repetir para un contenedor.

---
## Parar un contenedor (docker stop)

Cuando paremos un contenedor, este dejará de ofrecer el servicio, podemos hacerlo con el siguiente comando:

```bash
docker stop tomcat
```

![](/assets/images/Imagenes/Pasted image 20241008094447.png)

Vemos que al listar contenedores activos el de tomcat ya no está ya que lo hemos pausado.

---
## Listado de contenedores detenidos (docker ps -a)

Al igual que anteriormente, para listar los contenedores debemos usar o bien docker container ls o docker ps, pero para listar todos los contenedores que se hayan lanzado o detenido debemos añadir el apéndice **-a** (all), para que nos muestre todos los contenedores.

```bash
docker container ls -a

docker ps -a
```

![](/assets/images/Imagenes/Pasted image 20241008094823.png)

Vemos que el contenedor de tomcat y otros más tiene de STATUS **Exited**, mostrando que se han detenido.

---

## Iniciar un contenedor detenido (docker start)

Podemos volver un contenedor que hayamos detenido con el siguiente comando:

```bash
docker start tomcat
```

![](/assets/images/Imagenes/Pasted image 20241008095555.png)

---

## Pausar un contenedor (docker pause)

Otra opción es la de pausar un contenedor, con lo que **congelamos** el programa así, interrumpiéndose de manera temporal. Lo hacemos con el siguiente comando:

```bash
docker pause tomcat
```

![](/assets/images/Imagenes/Pasted image 20241008095814.png)

Vemos que ahora en su estado está pausado.

---

## Deshaciendo el pausado (docker unpause)

De igual manera podemos reanudar un contenedor que haya sido pausado con el siguiente comando:

```bash
docker unpause tomcat
```

![](/assets/images/Imagenes/Pasted image 20241008100145.png)

Ahora vemos que en su estado ya no está pausado.

---

## Reinicio de contenedor (docker restart)

