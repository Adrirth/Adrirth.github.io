---
title: Criptografía
layout: default
parent: Hacking
---

# Criptografía
{:.no_toc}

---

1. List
{:toc}

---

Una clave es una cadena de caracteres que se utiliza de un algoritmo de encriptación para alterar los datos de forma que parezcan aleatorios, de modo que solo el que tenga la "llave" correcta puede desencriptar la cadena.

Los datos originales se conocen como _texto plano_, y los datos después de que la clave los cifre se conocen como _texto cifrado_.

La fórmula:

	texto plano  + clave  = texto cifrado

---

## Cifrado simétrico

También conocida como cifrado de clave secreta utilizada la misma clave cifrada para encriptar y desencriptar la información. Esto significa que el transmisor y el destinatario deben tener la misma clave para desencriptar la información correctamente.

Si la clave secreta es compartida o perdida, la seguridad de la información ya no está garantizada. Ejemplos de algoritmos de ésta son **AES (Advanced Encryption Standard)** y **DES (Data Encryption Standard)**, siendo AES más segura a día de hoy.

---

## Cifrado Asimétrico

También conocido como cifrado de clave-pública, es un método que utiliza dos claves diferentes.

- una clave **privada**
- una clave **pública**


La clave púbica es usada para cifrar la información, mientras que la privada es usada para descifrarla. Esto significa que cualquier puede usar una clave pública para cifrar información para alguien, pero solo el receptor puede, con su clave privada descifrar la información. 

Algunos ejemplos de esto son RSA, PGP y ECC, y algunas aplicaciones como:

| E-Signatures | SSL/TLS | VPNs  |
| ------------ | ------- | ----- |
| SSH          | PKI     | Cloud |

---

## Algoritmos usados

| **Algoritmo**                                | **Acrónimo** | **Seguridad**                                                              |
| -------------------------------------------- | ------------ | -------------------------------------------------------------------------- |
| `Diffie-Hellman`                             | `DH`         | Relatively secure and computationally efficient                            |
| `Rivest–Shamir–Adleman`                      | `RSA`        | Widely used and considered secure, but computationally intensive           |
| `Elliptic Curve Diffie-Hellman`              | `ECDH`       | Provides enhanced security compared to traditional Diffie-Hellman          |
| `Elliptic Curve Digital Signature Algorithm` | `ECDSA`      | Provides enhanced security and efficiency for digital signature generation |

---

## Bibliografía

[**Página HTB**](https://app.hackthebox.com/)

[**Intercambio de claves**](https://academy.hackthebox.com/module/34/section/1875)

[**Criptografía**](https://academy.hackthebox.com/module/34/section/2026)

[**Intercambio de claves Diffie Hellman**](https://www.computerweekly.com/es/definicion/Intercambio-de-claves-Diffie-Hellman-intercambio-de-claves-exponencial)

[**Transport Layer Security**](https://www.cloudflare.com/es-es/learning/ssl/transport-layer-security-tls/)

[**AES**](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)

[**DES**](https://en.wikipedia.org/wiki/Data_Encryption_Standard)
