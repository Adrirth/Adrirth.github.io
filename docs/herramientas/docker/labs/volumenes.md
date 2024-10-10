---
title: Volúmenes
layout: default
nav_order:
parent: Laboratorios
---

# Volúmenes
{:.no_toc}

---

1. List
{:toc}

---

## Montando un volumen en un contenedor

Para montar un volumen en un contenedor debemos indicarlo con el parámetro **-v** junto el comando **docker run**, para poner en marcha el contenedor con el volumen. Debemos indicar la ruta del volumen.

```bash
docker run -v /mnt tomcat:latest
```



