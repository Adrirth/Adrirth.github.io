---
title: Instalación
layout: default
nav_order:
parent: Herramientas
nav_order: 2
---

# Instalación
{:.no_toc}

---

1. List
{:toc}

---
## Prerrequisitos 

- Sistema de 64 bits (Ubuntu Noble 24.04, Jammy 22.04, Focal 20.04).
- Kernel con versión 3.10+
- Permite instalar Docker EE o Docker CE.
- Para tener la última versión debemos usar los repositorios oficiales de docker, después podremos descargar el software.

Existen dos versiones:

- Docker EE (Enterprise Edition)
- Docker CE (Community Edition), [documentación](https://docs.docker.com/engine/install/ubuntu/)

Soporta la instalación en Desktop, Cloud (Amazon AWS, Microsoft Azure).

En este caso instalaremos Docker CE en un sistema Linux (MAX) en una máquina virtual, con la configuración de red en **Adaptador puente** para estar en la misma red local que la máquina host. 

---

**NOTA: En caso de recibir errores probar a lanzar los comandos como usuario root**

---
## Comprobaciones

Para su instalación es necesario que el sistema sea x64 y con instrucciones de virtualización activadas.

Para comprobarlo, estando en linux, podemos usar el comando **uname -m**, para saber la arquitectura de nuestro sistema.

```bash
uname -m
```

![](/assets/images/Imagenes/Pasted image 20241003153829.png)

Vemos que es válido. Para saber si tenemos las instrucciones de virtualización activadas podemos descargar la utilidad **kvm-ok**, que en caso de no estar instalada podemos descargarla con **sudo apt-get install cpu-checker -y**.

![](/assets/images/Imagenes/Pasted image 20241003154156.png)


Debemos comprobar la versión de nuestro kernel para saber si es compatible (3.10+), por lo que podemos usar el comando **uname -r**.

```bash
uname -r
```

![](/assets/images/Imagenes/Pasted image 20241003131852.png)

Antes de instalar es recomendable actualizar todos los paquetes del sistema, en nuestro caso al ser ubuntu podemos usar el gestor **apt**.

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

![](/assets/images/Imagenes/Pasted image 20241003132630.png)

---
## Instalación de docker con APT

Primero debemos añadir el repositorio **apt** de docker con la clave GPG oficial (insertar comandos en el orden descrito):

```bash
# Actualizar paquetes

sudo apt-get update

# Instalar los paquetes de ca-certificates y la utilidad curl

sudo apt-get install ca-certificates curl

# Crea un directorio con permisos específicos (lectura, escritura y ejecución solo para el propietario, ejecución y lectura para el resto) 

sudo install -m 0755 -d /etc/apt/keyrings

# Descargar el recurso de clave oficial de docker con curl de la página oficial

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Dar permisos de lectura a todos los usuarios a la clave 

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

![](/assets/images/Imagenes/Pasted image 20241003133432.png)

![](/assets/images/Imagenes/Pasted image 20241003133635 1.png)

Después añadiremos el repository a los recursos **apt** del sistema:

**NOTA: Es posible que en caso de utilizar otras distribuciones de Ubuntu como Linux Mint deba usarse UBUNTU_CODENAME en lugar de VERSION_CODENAME**.

```bash
# 

sudo echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Actualizar paquetes
  
sudo apt-get update
```

![](/assets/images/Imagenes/Pasted image 20241003134535.png)

Con esto ya podremos descargar los paquetes oficiales de docker:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

![](/assets/images/Imagenes/Pasted image 20241003134913.png)

Podemos comprobar si se ha instalado con el comando **docker -v**, que comprueba la versión del programa.

```bash
docker -v
```

![](/assets/images/Imagenes/Pasted image 20241003135157.png)

En nuestro caso vemos que está instalada la versión 27.3.1, en la versión CE (Community Edition).

Por último para verificar la instalación podemos lanzar nuestro primer contenedor con **docker run**.

```bash
sudo docker run hello-world
```

Con este comando le pedimos a docker lanzar la imagen hello-world, a lo que nos dirá que no estará instalada localmente, y la descargará de la librería /hello-world. 

![](/assets/images/Imagenes/Pasted image 20241003135933.png)

Veremos que descargará la última versión (hello-world:latest). Al lanzar el contenedor vemos que nos indica la serie de pasos que ha realizado:

- El cliente docker ha llamado al daemon docker
- El daemon ha descargado la imagen "hello-world" de Docker Hub.
- El daemon docker ha creado un nuevo contenedor a partir de la imagen que corre el ejecutable que produce este mismo texto.
- El daemon ha transmitido el resultado al cliente docker, que a su vez lo ha mandado a la terminal.

Además nos dice que podemos probar a lanzar otra imagen más ambiciosa como es la de un contenedor con ubuntu. Podemos hacerlo con **docker run -it ubuntu bash**. Es posible que debamos lanzar el comando con permisos de administrador, por lo que añadiremos **sudo** al comando.

![](/assets/images/Imagenes/Pasted image 20241003140603.png)

Para ver todas las imágenes en el sistema podemos usar el comando **docker ps -a**.

```bash
sudo docker ps -a
```

![](/assets/images/Imagenes/Pasted image 20241003140844.png)

---

## Instalación de docker con paquetes

En caso de que no podamos acceder al repositorio de docker u otras razones de no poder conectarnos a él, podemos descargar los paquetes necesarios e instalarlos de manera manual. Para descargarlos podemos visitar [este enlace](https://download.docker.com/linux/ubuntu/dists/). 

Dependiendo de la versión de ubuntu descargaremos un paquete u otro. En este caso para Ubuntu 22.04 la versión es Jammy, en el caso de que no sepamos cuál es podemos usar el comando **lsb_release -a**.

![](/assets/images/Imagenes/Pasted image 20241003160707.png)

Las versiones más estables estarán en **/pool/stable** de cada versión de sistema. Podemos descargarnos versiones anteriores, pero lo más recomendable es descargar la más actual posible. Al momento de esta instalación la más reciente en la **1.7.22-1**.

Debemos descargar todos los archivos indicados, que cambiaran dependiendo de la versión del sistema en el que vayamos a instalar docker:

- `containerd.io_<version>_<arch>.deb`
- `docker-ce_<version>_<arch>.deb`
- `docker-ce-cli_<version>_<arch>.deb`
- `docker-buildx-plugin_<version>_<arch>.deb`
- `docker-compose-plugin_<version>_<arch>.deb`

![](/assets/images/Imagenes/Pasted image 20241003161516.png)

Una vez descargado todo lo necesario accederemos al directorio donde estén los paquetes y lanzaremos el comando sudo **sudo dpkg -i** seguido de todos los paquetes. 

```bash
sudo dpkg -i ./containerd.io_<version>_<arch>.deb \
  ./docker-ce_<version>_<arch>.deb \
  ./docker-ce-cli_<version>_<arch>.deb \
  ./docker-buildx-plugin_<version>_<arch>.deb \
  ./docker-compose-plugin_<version>_<arch>.deb
```

![](/assets/images/Imagenes/Pasted image 20241003164253.png)

Para comprobar que se ha instalado correctamente podemos usar el comando **docker -v** como anteriormente. 

![](/assets/images/Imagenes/Pasted image 20241003164855.png)

Por último para comprobar que funciona correctamente podemos probar a lanzar la imagen de hello-world con **sudo docker run hello-world**.

![](/assets/images/Imagenes/Pasted image 20241003164929.png)

---

## Instalación en grupo docker

Por defecto docker se **ejecuta** como usuario root, lo cuál no es conveniente a nivel de seguridad y comodidad a la hora de tener que ejecutar comandos con permisos de superusuario.

Por lo que podemos configurar un usuario que ejecute los procesos de docker sin que sea necesario que sea root. Según el tipo de instalación el grupo docker debería estar creado, sino se debe crear y reiniciar el servicio.

Primero, comprobamos que el grupo docker existe, podemos hacerlo con el comando **cat /etc/group | grep docker**, para que nos muestre los grupos del sistema y nos filtre por aquella línea que empiece con docker.

```bash
cat /etc/group | grep docker
```

![](/assets/images/Imagenes/Pasted image 20241004090701.png)

En el caso de que el grupo no exista deberemos crearlo con el comando **sudo groupadd docker**, y volver a comprobar si se ha creado correctamente.

```bash
sudo groupadd docker
```

Con el grupo ya creado necesitamos el usuario docker que sea el encargado en adelante de ejecutar docker. Al igual que con el grupo, para comprobar si existe podemos usar el comando **cat /etc/passwd | grep docker**.

Parecido a la creación del grupo podemos usar **sudo usermod -aG docker $USER** para crear el usuario docker. 

Con el comando usermod modificamos propiedades de un usuario, con el apéndice -a lo agregamos, con -G especificamos los grupos a los que queremos que se añada, en este caso docker, y con el parámetro $USER (variable de entorno) dependiendo del usuario con el que ejecutemos el comando será el que tendrá privilegios para la ejecución de los comandos de docker. Por ejemplo, si accedemos como usuario adri la variable $USER tendrá el valor adri, siendo ese el usuario con privilegios.

```bash
sudo usermod -aG docker $USER
```

Si volvemos a ver el grupo de docker veremos que ahora se ha añadido el usuario "cmadrid".

![](/assets/images/Imagenes/Pasted image 20241004092409.png)

Para finalizar y que sucedan los cambios debemos reiniciar la shell del perfil añadido. Para saber que gestor de sesiones podemos usar el comando **systemctl status display-manager**.

```bash
sudo systemctl status display-manager
```

Dependiendo de nuestro gestor deberemos reiniciarlo. En nuestro caso es lightdm, por lo que lo reiniciaremos.

![](/assets/images/Imagenes/Pasted image 20241004104445.png)


```bash
sudo systemctl restart lightdm
```

```bash
sudo systemctl restart gdm
```

Después de reiniciarse deberemos volver a iniciar sesión, y podremos lanzar comandos de docker sin necesidad de ser root.

![](/assets/images/Imagenes/Pasted image 20241004104551.png)

Para finalizar podemos ver la versión del cliente de interacción de docker y del servidor con el comando docker version, además de otro comando para mostrar información del motor Docker Engine (imágenes en el sistema, cantidad de contenedores, versión del kernel del nodo...), docker info.

```bash
docker version
```

![](/assets/images/Imagenes/Pasted image 20241004105236.png)


```bash
docker info
```

![](/assets/images/Imagenes/Pasted image 20241004105156.png)



