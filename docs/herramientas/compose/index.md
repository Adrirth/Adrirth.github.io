---
title: Docker compose
layout: default
parent: Herramientas
---

# Docker compose
{:.no_toc}

---

1. List
{:toc}

---

Es una forma de crear contenedores docker de forma reproducible usando un archivo de configuración en vez de comandos Docker más largos. Esto es especialmente útil para aplicaciones multiservicio que necesitan de múltiples contenedores.

Esto se realiza mediante un archivo YAML para configurar y mantener múltiples servicios de aplicación. Con un solo comando se crea y lanza todos los servicios de la configuración de este archivo.

En él especificamos la versión, los servicios y toda la información de estos: nombre del contenedor, imagen, puertos
## Ejercicio

Configurar un docker-compose para dar servicios de mongodb y servicio web.

```yaml
version: '3.1'

services:
  mongodb:                 # Nombre del contenedor
    image: mongo:4.4       # Nombre de la imagen
    ports:                 # Mapeo de puertos del servicio
      - 27017:27017        # Puerto host:contenedor
    environment:           # variables de sistema del contenedor
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password

  mongo-express:
    image: mongo-express
    ports:
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb  # Nombre correcto del servicio

```

Por defecto compose crea una sola red para la aplicación, comunicándose vía nombre de contenedor.

---

## Lanzar docker-compose 

Para lanzar el docker-compose debemos ejecutar el siguiente comando:

```bash
docker-compose -f docker-compose.yml up
```

- Con **-f** indicamos el archivo y con up que se lance el docker compose.

![](/assets/images/Imagenes/Pasted image 20241010172803.png)

Después de lanzarlo podremos acceder a través de localhost:8081 a la web mongo-express:

![](/assets/images/Imagenes/Pasted image 20241010172619.png)






---
## Errores

### Versión de mongodb no compatible









