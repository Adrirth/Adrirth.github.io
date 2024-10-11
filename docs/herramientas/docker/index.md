---
title: Docker
layout: default
parent: Herramientas
nav_order: 1
---

# Docker
{:.no_toc}

---

1. List
{:toc}


## 1. Contenedor

### Historia de aplicaciones

- 1960 - 1970: Main Frame
- 1980 - 1990: Grandes servidores, arquitecturas de 2 capas
- 2000 - 2010: Boom de la web
- 2010: Boom de aplicaciones móviles, computación en la nube
- 2020: Automatización empresarial, IA aplicada a todo tipo de sectores. 

---
### Problemas planteados

- Stack de **aplicaciones** muy amplio, tanto en infraestructura como en funcionamiento.
- Puesta en marcha continua de **versiones** cada vez más acelerada
- Necesidad de **réplica** de entornos de infraestructura y aplicaciones.

---
### Nacimiento de docker

- Empaquetamiento de stack de software de una forma **estándar** que necesitamos para que funcione posteriormente.
- Posteriormente, independientemente del **hardware** podríamos poner en marcha el sistema en cuestión de una forma estandarizada

---
### Beneficios

- Entornos de funcionamiento **limpios, seguros y portátiles** entre distintos tipos de infraestructura.
- Despliegues **reproducibles** 
- Aislamiento al despliegue de distintas aplicaciones
- Casi cero problemas de compatibilidad
- Velocidad de **despliegue** aumentada, ahorro de costes en infraestructura (En una misma máquina poder usar muchos más sistemas que antes en igualdad de recursos)
- Ventajas de ser una virtualización de espacio aislado sin penalización tan alta de consumo recursos

---
### Separación de responsabilidades 

1. **Equipo Desarrollo**
	- Información dentro del container
		- Código fuente 
		- Librerías
		- Gestor de dependencias
		- Datos
	- Para el equipo todos los equipos Linux son iguales

2. **Equipo DevOps**
	- Exterior del container
		- Login de acceso
		- Accesos remotos
		- Configuración de la red
		- Monitorización
	- Todos los contenedores se arrancan, paran, copian, pegan, migran de la misma forma

---

## 2. Docker

- Funcionamiento en sistemas Linux
- Kernel 2.7+
- La distribución no importa
- Funcionamiento en hardware físico (local, nube)
- Orientado a el lado del servidor
- Configuración de red propia
- Puede funcionar como **root**
- Puede disponer de su propio /sbin/init (proceso de arranque, herramientas del usuario)

- **Portable**: No depende del sistema operativo
- **Inmutable**: Empaqueta las dependencias necesarias
- **Ligero**: Usa cgroups y namespaces de linux

---

### Contenedores vs VMs

- Los contenedores están aislados pero comparten SO y opcionalmente binarios y librerías
- Despliegues más rápidos
- Menor coste de IT
- Facilidad de migración
- Facilidad de reinicio
- No se requiere de SO adicional ni recursos a ser reiniciados

![](/assets/images/Imagenes/Pasted image 20241003093325.png)

---

### Archivo dockerfile

- Receta o molde para preparar un sistema con todo lo necesario para su puesta en marcha
- Prepara una imagen específica donde posteriormente se sacarán instancias (containers) para poner en marcha el sistema en sí.
- Archivo de configuración que dicta como se monta un container docker

Ejemplo:

```dockerfile
FROM ubuntu:1610
MAINTAINER Adrián Román-Toledo

RUN apt-get update && apt-get install -y apache2

ENV APACHE_LOG_DIR /var/log/apache2

EXPOSE 80
CMD ["/usr/sbin/apache2", "-D","FOREGROUND"]
```

---

### Container

- Resultado de instanciar una imagen definida por el dockerfile
- Aplicación paquetizada, aislada

---

### Host (Anfitrión)

- Portacontenedores
- Puede ser local, en la nube privada o pública, servidor virtual o físico
- Expone los recursos disponibles
- Dependiendo de los recursos del sistema cabrán más o menos contenedores

---

### Imagen

- Fichero binario que contiene todo el sistema de archivos de un contenedor
- Sistema ficheros Union **(UnionFS)**.
- Estructurado en capas (layers) por delta

---

### Volumen

- Discos o directorios externos que podemos montar en el contenedor
- Recursos externos (alojados en el host) que sobreviven al contenedor
- Configuración /Datos /Recursos

---

### Registro

- Biblioteca de imágenes para poner en marcha en containers, siempre operativas para poner en marcha
- Registro público, compartidas por la comunidad, de libre acceso (Dockerhub)
- Registro privado, containers privados o corporativos (Repositorio de empresa)

---

### Ciclos de vida

#### Contenedor

Para iniciar un contenedor primero se debe crear siempre a partir de una imagen, además de que para pararlo o reiniciarlo o terminarlo siempre debe "matarse". 

![](/assets/images/Imagenes/Pasted image 20241003102839.png)

#### Imagen

![](/assets/images/Imagenes/Pasted image 20241003103128.png)



---
## Diccionario

- **Kernel**: Núcleo de los sistemas operativos encargado de gestionar los recursos de hardware, con permisos especiales, que están disponibles para el sistema operativo

- **cgroups**: Permiten la asignación de recursos para los procesos de forma precisa, de forma que no estos no se canibalicen.

- **SoC**: Separation of Concerns, principio fundamental en desarrollo software que busca romper sistemas complejos en otros más pequeños y manejables.

- **chroot**: Comando que permite cambiar de directorio raíz aparente para un proceso en ejecución seleccionado y sus subsidiarios. Este no puede acceder al resto de directorios y archivos fuera de su entorno de directorios.

- **DevOps**:

- **/sbin/init**: Primer programa ejecutado por el kernel de linux durante el proceso de inicio, responsable de inicializar el sistema y configura el entorno de usuario.

- **Sistema Ficheros Union (UnionFS)**: Sistema de archivos liviano con capas y de alto rendimiento.