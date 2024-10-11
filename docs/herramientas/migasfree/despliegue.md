---
title: Despliegue migasfree
layout: default
parent: Migasfree
---

# Despliegue Migasfree
{:.no_toc}

---

1. List
{:toc}

---

Para lanzar un paquete a los distintos equipos debemos crear una orden que después reciban los equipos que queramos que reciban el paquete o configuración.

## Panel de despliegue

Este está en nuestro panel principal, en la parte superior.

![](/assets/images/Imagenes/Pasted image 20241011174217.png)

Entraremos y clicaremos en el botón de **+**. *Nota*: dependiendo de si el paquete es local o externo elegiremos uno, en este caso al no estar en el equipo administrador indicaremos que es externo.

---
## Configuración de despliegue

A continuación deberemos especificar la información del despliegue: nombre del despliegue, sistemas que lo recibirán, comentarios, atributos de los sistemas, especificar el origen, etc. En este caso instalaremos **vlc**.


![](/assets/images/Imagenes/Pasted image 20241011174730.png)
![](/assets/images/Imagenes/Pasted image 20241011174739.png)

En el caso de los atributos debemos especificar al menos uno que compartan los sistemas que vamos a incluir, en este caso es CID-1, pero puede haber muchos más. También podemos especificar que sistemas con unos atributos queremos excluir.

![](/assets/images/Imagenes/Pasted image 20241011174945.png)

También le indicaremos el paquete que queramos instalar o quitar. 

Finalmente podemos programar una fecha de lanzamiento y si terminar el grabado o realizar otro.

![](/assets/images/Imagenes/Pasted image 20241011175040.png)

Con esto veremos el panel de despliegues, donde estará listo para que los clientes capten y realicen la orden.

![](/assets/images/Imagenes/Pasted image 20241011175207.png)

Una vez confirmado el despliegue debemos ir a los dispositivos y sincronizarse con el servidor maestro.

```bash
migasfree -u
```

Veremos que actualizará e instalará todos los paquetes instalados.

![](/assets/images/Imagenes/Pasted image 20241011175405.png)

![](/assets/images/Imagenes/Pasted image 20241011175429.png)