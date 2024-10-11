---
title: Dockerfile
layout: default
nav_order: 4
parent: Docker
---

# Dockerfile
{:.no_toc}

---

1. List
{:toc}

---

Archivo de texto diseñado para el uso con dockers que contiene una serie de instrucciones que docker utiliza para construir una imagen. No tiene extensión, se procesa secuencialmente de arriba hacia abajo y en el orden que encuentre la instrucción se realizará antes o después la acción.

Es recomendable que esté en el repositorio del código fuente para especificar su construcción.

## Multi-stage build

Funcionalidad agregada desde la versión 17.05, con la que poder simplificar el proceso de construcción. Con el podemos reducir el tamaño de la imagen y aumentar la seguridad, pudiendo crear múltiples etapas de construcción en un mismo dockerfile, pudiendo compilar o generar archivos en las primeras etapas pero conservando solo lo necesario en la imagen final, descartando lo innecesario.