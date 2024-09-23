---
title: Configuración FoxyProxy con Burp Suite
parent: Hacking Web
layout: default
nav_order: 2
---

# Configuración FoxyProxy con Burp Suite

---

Para empezar deberemos tener instalado tanto Burp Suite, utilidad muy variada que sirve entre otras cosas como proxy, y foxy proxy, extensión de navegador que nos permitirá elegir enviar las respuestas y peticiones a burp suite.

![](/assets/images/Imagenes/Pasted image 20240912210533.png)

&nbsp;

Después deberemos añadir la opción de redirigir el trafico a burpsuite, por lo que entraremos en opciones de la extensión.

![](/assets/images/Imagenes/Pasted image 20240912210705.png)

&nbsp;

Y en la pestana proxies añadiremos la IP de nuestro equipo local y el puerto al que queremos redirigir el trafico, en este caso 127.0.0.1:8080.

![](/assets/images/Imagenes/Pasted image 20240912210912.png)

&nbsp;

Ahora podremos elegir la opción de burpsuite al pulsar en la extensión, sin embargo deberemos realizar un paso mas.

![](/assets/images/Imagenes/Pasted image 20240912211106.png)

&nbsp;

Al no tener un certificado valido de confianza del proxy al que se es redirigido el navegador no nos permitirá acceder a la respuesta de el servidor, por lo que deberemos descargarlo y añadirlo. Para ello deberemos, con burpsuite y foxyproxy abiertos, además de este ultimo redirigiendo el trafico a burp, acceder a la direccion 127.0.0.1:8080, para poder descargar el certificado clickando en **CA Certificate**. Se nos descargara un archivo tipo **.der**.

![](/assets/images/Imagenes/Pasted image 20240912211506.png)

&nbsp;

Después accederemos a los ajustes de nuestro navegador, en mi caso Brave, y buscaremos el apartado **certificados**, donde podremos administrar los certificados y añadir el de burpsuite. En el caso de Brave hay que acceder a la pestana autoridades y añadirlo.

Con esto ya podríamos interceptar peticiones y respuestas de un servidor y demás.