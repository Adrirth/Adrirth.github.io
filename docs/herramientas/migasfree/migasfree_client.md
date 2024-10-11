---
title: Migasfree client
layout: default
parent: Migasfree
---

# Migasfree client
{:.no_toc}

---

1. List
{:toc}

---

## Instalación

```bash
wget -O - http://migasfree.org/pub/install-client | bash
```

Después registraremos el sistema con el comando:

```bash
wget -O - http://migasfree.org/pub/install-client | bash
```

Al terminar debería aparecer algo parecido a esto:

```bash
-python-setuptools_44.0.0-2ubuntu0.1_all.deb
-python-soupsieve_1.9.5+dfsg-1_all.deb
-python-suds_1:0.8.4-1max1_all.deb
-python-urllib3_1.22-1ubuntu0.18.04.2max1_all.deb
-python-utidylib_0.5-3max2_all.deb
-python-webassets_3:0.12.1-2max1_all.deb
-python-webencodings_0.5.1-1ubuntu1_all.deb
-python-zope.interface_4.7.1-1_amd64.deb
-python3-update-manager_1:20.04.10.11_all.deb
-tilda_1.5.2-0ubuntu1_amd64.deb
-ubuntu-advantage-tools_27.14.4~20.04_amd64.deb
-update-manager-core_1:20.04.10.11_all.deb
-update-manager_1:20.04.10.11_all.deb
***************************** Correcto

****************** Subiendo el inventario del software... ******************
***************************** Correcto

*************** Capturando información sobre el hardware... ****************
***************************** Correcto

**************** Enviando información sobre el hardware... *****************
***************************** Correcto

************************* Operaciones completadas **************************
```

Veremos que comenzará a instalar los paquetes necesarios, y si observamos en nuestro panel de administración veremos que se hay notificaciones sobre la inclusión de sistemas a gestionar:

![](/assets/images/Imagenes/Pasted image 20241011172530.png)

![](/assets/images/Imagenes/Pasted image 20241011172630.png)

Vemos que se ha registrado un sistema ha dado de alta la plataforma Linux, y que ha añadido el proyecto ubuntu-20-04.

Si entramos a la sección de ordenadores superior veremos la lista de los que están registrados.

![](/assets/images/Imagenes/Pasted image 20241011173039.png)

---

## Sincronización de paquetes

Para captar los despliegues que se lancen debemos lanzar el comando:

```bash
migasfree -u
```