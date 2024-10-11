---
title: Instalación compose
layout: default
parent: Docker compose
---

# Instalación Docker compose
{:.no_toc}

---

1. List
{:toc}

---

## Comprobar docker instalado

Primero debemos asegurar que tenemos docker instalado y funcionando, podemos asegurarnos con el comando:

```bash
docker --version
```

![](/assets/images/Imagenes/Pasted image 20241010124100.png)

Una vez comprobado debemos descargar la última versión de compose con el comando curl.

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

![](/assets/images/Imagenes/Pasted image 20241010124241.png)

Una vez instalado deberemos darle permisos de ejecución al archivo:

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

Finalmente verificaremos la instalación.

```bash
docker-compose --version
```

![](/assets/images/Imagenes/Pasted image 20241010124721.png)

