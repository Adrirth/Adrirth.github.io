---
title: Imágenes
layout: default
nav_order:
parent: Herramientas
---

# Imágenes
{:.no_toc}

---

1. List
{:toc}

---

La imagen en docker es el fichero binario que contiene todo el sistema de ficheros, que posteriormente podrá utilizar el contenedor. 

## Sistemas de ficheros Union

Las imágenes se estructuran por capas (layers) delta, de manera que se pueda tener capacidad de re-construir el binario, compilando únicamente aquellas partes que hayan cambiado.

![](/assets/images/Imagenes/Pasted image 20241004110010.png)

Cuando indicamos una imagen el formato estándar es este:

	<nombre-imagen-docker>:<version>


Al crear nuestra propia imagen de docke es vital establecer una nomenclatura clara (nomenclatura/versionado) para poder tener siempre una gestión de los sistemas, su actualización y despliegue.

Se establece el siguiente criterio:

	<nombre-empresa/sistema>

Ejemplo:

- Nombre de la empresa

		mycompany1

- Nombre del producto

		bill-system

- Versión del sistema compilado

		2.0.5

Como nombre podría ser:

	mycompany1/bill-syste,:2.0.5


## Tags en docker

Docker tiene un tag especial llamado **latest**. Al descargar una imagen del hub se requiere de una versión concreta especficiada. En el caso de que no la indiquemos como en aquí:

	docker run ubuntu

Lo que se indica es esto:

	docker run ubuntu:latest

También debemos considerar que docker primero busca de manera local si está disponible la imagen, en caso de no ser así se descarga del hub. Esto puede resultar un problema al no descargar la última versión disponible. Esto puede resolverse con dos comandos encadenados, donde se descarga la última versión disponible en el hub y actualizar los cambios locales:

	docker pull ubuntu:latest && docker run ubuntu:latest

Sin embargo debemos tener cuidado y saber la versión que utilice el proyecto, para que no entre en conflicto entre el desarrollo.


#### Recomendaciones de gestión de imágenes

- Indicar como versión de la imagen docker, la misma con la que desarrollo haya etiquetado la release de la aplicación, esto facilita que cuando haya una incidencia que se indique la misma versión de la imagen de docker que desarrollo tiene con el TAG del código fuente.

- Nunca llevar a cabo despliegues en producción de versiones latest, ya que en el caso de salir una incidencia lo único que va a poder decirse a desarrollo es que es en la versión latest, siendo una versión que es muy probable que no tenga las correcciones necesarias implementadas en versiones anteriores.