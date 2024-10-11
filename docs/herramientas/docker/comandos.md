---
title: Comandos
layout: default
nav_order: 5
parent: Docker
---

# Comandos
{:.no_toc}

---

1. List
{:toc}

---


## Comandos de gestión

| Comando                             | Descripción                                                                                                                                                                        | Apéndices | Función                                                        |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------- | -------------------------------------------------------------- |
| **docker run**                      | Crea un contenedor y lo inicia. Si la imagen no está presente en el sistema local la descarga de **hub.docker.com**, que es el repositorio oficial para guardar imágenes de docker | -t        | Introduce una pseudo terminal                                  |
|                                     |                                                                                                                                                                                    | -i        |                                                                |
|                                     |                                                                                                                                                                                    | -w        | Añadir directorio de trabajo donde sucederán las instrucciones |
|                                     |                                                                                                                                                                                    |           | **docker run -w /root prueba:1.0 **                            |
|                                     |                                                                                                                                                                                    | -e        | Añadir variable de entorno persistente en el contenedor        |
|                                     |                                                                                                                                                                                    |           | **docker run -it -e USR_HOME=/Otro kane_project/testing bash** |
| **docker pull**                     | Con este comando indicamos que imagen queremos descargar, ejemplo: **docker pull ubuntu**. Con esto indicamos que queremos descargar la última versión de la imagen ubuntu         |           |                                                                |
| **docker save**                     | Guardar una imagen de docker para exportar a otro sistema o para realizar una copia de seguridad                                                                                   |           |                                                                |
| **docker rmi**                      | Elimina una imagen de docker engine, en caso de no especificar el TAG se borrará la latest. Ejem: **docker rmi ubuntu:latest**.                                                    |           |                                                                |
| **docker load**                     | Cargar una imagen guardada para incluirla en el docker engine                                                                                                                      |           |                                                                |
| **docker rm -f (ID)**               | Obliga forzar a borrar un contenedor indicado con ID                                                                                                                               |           |                                                                |
| **docker tag**                      | Crea un alias para acceder a una iamgen. Ejemplo: docker tag ubuntu:latest prueba:1.0.4                                                                                            |           |                                                                |
| **docker build**                    | Construir imagen a partir de un Dockerfile                                                                                                                                         | -f        | Indicar nombre del archivo. E                                  |
|                                     | docker build -f prueba -t prueba:1.0 .                                                                                                                                             |           |                                                                |
| **docker system prune -a**          | Borra todo el contenido de docker, como si estuviera nuevo                                                                                                                         |           |                                                                |
| **docker image prune -a**           | Borra todas las imágenes de el sistema docker                                                                                                                                      |           |                                                                |
| **docker export**                   | Exportar sistema de archivos de un contenedor                                                                                                                                      |           |                                                                |
| **docker import**                   | Importar sistema de archivos de un contenedor para crear una imagen                                                                                                                |           |                                                                |
| **docker exec**                     | Lanzar comandos contra un contenedor para obtener resultados                                                                                                                       | -it       | Nos genera una consola interactiva de entrada-salida           |
| **docker commit**                   | Crea una imagen con el estado actual de un contenedor                                                                                                                              |           |                                                                |
| **docker attach**                   | Nos pone el proceso del contenedor en primer plano para adminsitrarlo                                                                                                              |           |                                                                |
| **docker rm $(docker ps -a -q) -f** | Borrar todos los contenedores disponibles de docker                                                                                                                                |           |                                                                |


---

## Comandos de información y monitorización

| Comando            | Descripción                                                                                                           | Apéndices | Función                             |
| ------------------ | --------------------------------------------------------------------------------------------------------------------- | --------- | ----------------------------------- |
| **docker ps**      | Listado de contenedores activos                                                                                       | -a        | Mostrar todos los contenedores      |
| **docker**         |                                                                                                                       | -v        | Comprobar versión del programa      |
| **docker version** | Información sobre las versiones del clientede interacción de Docker y del servidor                                    |           |                                     |
| **docker info**    | Más información sobre Docker Engine (imágenes que tenemos, cantidad de contenedores, versión de kernel del nodo, etc) |           |                                     |
| **docker top**     | Nos muestra los procesos del contenedor                                                                               |           |                                     |
| **docker logs**    | Comprobar logs de un contenedor                                                                                       | -f        | Comprobación de logs en tiempo real |
| **docker stats**   | Información de consumo del contenedor                                                                                 |           |                                     |
| **docker events**  | Flujo de eventos de la vida del contenedor                                                                            |           |                                     |
| **docker images**  | Muestra las imágenes de docker engine                                                                                 |           |                                     |
|                    |                                                                                                                       | -q        | Listar id de imágenes               |
| **docker inspect** | Información del contenedor en formato JSON                                                                            |           |                                     |
