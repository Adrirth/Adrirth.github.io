---
title: Migasfree
layout: default
parent: Herramientas
---

# Migasfree
{:.no_toc}

---

1. List
{:toc}

---

## Que és Migasfree

Software gratuito y de código abierto que permite el despliegue de paquetes entre equipos. Sus objetivos son:

**Integridad - Automatización - Centralización**

#### Requisitos

- Versión de Ubuntu 22.04
- FQDN (Fully Qualified Domain Name), en caso de no tener uno podemos añadirlo en nuestro documento **/etc/hosts** para emularlo en un entorno de test, o usando la IP del servidor.
- Tener *[docker engine](https://docs.docker.com/engine/installation/)*, *[docker-compose](https://docs.docker.com/compose/install/)*, *haveged* instalados.

---
## Instalación software necesario (Ubuntu)

##### Instalación manual docker engine:

```bash
# Añadir clave oficial GPG de Docker:

sudo apt-get update

sudo apt-get install ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

# Añadir el repositorio:

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get update

# Instalar la última versión

 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verificar funcionamiento

 sudo docker run hello-world
```


Deberá aparecer el siguiente mensaje:

![](/assets/images/Imagenes/Pasted image 20241001190120 1.png)

##### Instalar docker-desktop:

- Tener ya añadido el repositorio oficial de docker (ya instalado en el paso anterior).
- Descargar la última versión de [paquete DEB](https://desktop.docker.com/linux/main/amd64/docker-desktop-amd64.deb).
- Instalar el paquete con apt:

```bash
sudo apt-get update
sudo apt-get install ./docker-desktop-<arch>.deb
```

Es posible que al intentar iniciarla nos de un error de no tener soporte de virtualización, por lo que es posible que haya que activar la opción de *Nested Virtualization* en las opciones de la máquina, o activarlo con VBoxManage desde terminal.

##### Instalar haveged

```shell
sudo apt-get install haveged
```

##### Instalar docker-compose

```bash
sudo apt-get install docker-compose
```

##### Instalación script automática

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

---
## Instalación de migasfree

- Descargar archivo *docker-compose.yml* y el archivo de variables de github.

```shell
mkdir mf
cd mf
wget https://github.com/migasfree/migasfree-docker/raw/master/mf/docker-compose.yml
wget https://github.com/migasfree/migasfree-docker/raw/master/mf/variables
```

- Configurar archivo de variables

```shell
nano variables
```

> En este archivo debemos indicarle donde estará el servidor de base de datos, en este caso al ser otro contenedor será **db**, nombre por defecto en el docker-compose file. Además es posible que no podamos acceder desde un determinado sistema, por lo que debemos indicar la ip de este.

```sh
export FQDN=db
...
export POSTGRES_ALLOW_HOSTS="0.0.0.0/0"
```

#### Lanzar

Actualizamos el archivo variables y lanzaremos los contenedores con docker-compose.

```bash
. variables
docker-compose up -d
```

#### Probar

```
xdg-open http://127.0.0.1
...
http://localhost
```

Debería abrirse nuestro navegador con la siguiente página:

![](/assets/images/Imagenes/Pasted image 20241002105129.png)

---
## Inicio sesión

Una vez en la página de inicio de sesión podemos con las credenciales admin/admin

![](/assets/images/Imagenes/Pasted image 20241011170757.png)

---
##########################################################################################################################################################################################

---

## Pruebas sin documentar

### Ficheros de configuración

/var/lib/migasfree/127.0.0.1/conf/settings.py

	psql -h db-db -U migasfree -d migasfree

Dentro del contenedor:

vi /etc/postgresql/9.6/main/pg_hba.conf

```bash
# DO NOT DISABLE!
# If you change this first entry you will need to make sure that the
# database superuser can access the database using some other method.
# Noninteractive access to all databases is required during automatic
# maintenance (custom daily cronjobs, replication, and similar tasks).
#
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
host    all             all             0.0.0.0/0            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            md5
#host    replication     postgres        ::1/128                 md5

```


## Entrar en postgres

```bash
psql -U postgres -h localhost
```

