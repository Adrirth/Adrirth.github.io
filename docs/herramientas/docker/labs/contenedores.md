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

Son el resultado de haber instanciado una imagen Docker, con la cuál podremos hacer tantos contenedores necesitemos de la misma imagen, siempre que el hardware nos permita.

Cuando necesitemos actualizar un sistema que está en funcionamiento para subirlo de versión deberemos:

- Recrear la imagen docker con el nuevo contenido que queremos poner en producción
- Instalar en el servidor/servdidores necesarios la imagen docker en cuestión
- Eliminar los contenedores antiguos
- Crear los nuevos contenedores de la nueva imagen
- Eliminar en caso necesario, las imágenes antiguas que ya no vayamos a usar más.