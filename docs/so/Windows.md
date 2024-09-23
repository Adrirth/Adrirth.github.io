---
title: Windows
layout: default
parent: Sistemas
nav_order: 2
---

# Windows
{:.no_toc}

---

1. List
{:toc}

---

Introducido por primera vez el 20 de Noviembre de 1985, la primera versión de éste era una shell gráfica para MS-DOS. Versiones posteriores introdujeron el gestor de archivos, el gestor de programas y programas de gestión de impresión.

Windows 95 fue  la primera integración completa de Windows y DOS ofreciendo soporte internet por primera vez, debutando el buscador Internet Explorer. 

En cuanto a Windows Server, fue lanzado en 1993 con el lanzamiento de WIndows NT3.1 Advanced Server. Este fue recibiendo varias actualizaciones durante los años, añadiendo tecnologías como Internet Information Services (IIS), varios protocolos de red y más. Con el lanzamiento de Windows 2000 se introdujo Actice Directory o Directorio Activo, originalmente con la intención de ayudar administradores de sistemas a configuración de la compartición de archivos, encriptación de datos, VPNs, etc. 

Con nuevas versiones se introdujeron más opciones y mejoras, siendo versiones de Server 2012, Server 2016 y la más reciente Server 2019. Ésta última añade soporte para Kubernetes, contenedores Linux y más opciones de seguridad avanzadas.

## Versiones Windows

| Nombres de sistema operativo         | Número versión |
| ------------------------------------ | -------------- |
| Windows NT 4                         | 4.0            |
| Windows 2000                         | 5.0            |
| Windows XP                           | 5.1            |
| Windows Server 2003, 2003 R2         | 5.2            |
| Windows Vista, Server 2008           | 6.0            |
| Windows 7, Server 2008 R2            | 6.1            |
| Windows 8, Server 2012               | 6.2            |
| Windows 8.1, Server 2012 R2          | 6.3            |
| Windows 10, Server 2016, Server 2019 | 10.0           |

Para saber información sobre nuestro sistema operativo podemos usar el comando **Get-WmiObject**, en concreto con el parámetro **-class win32_OperatingSystem**.

```powershell
PS C:\htb> Get-WmiObject -Class win32_OperatingSystem | select Version,BuildNumber

Version    BuildNumber
-------    -----------
10.0.19041 19041
```

Otras opciones son Win32_Process, Win32_Service, Win32_Bios o ComputerName.


### Remote Desk Protocol (RDP)

El Protocolo de Escritorio Remoto usa una arquitectura cliente/servidor donde una aplicación en el lado del cliente es usada para especificar la IP de el ordenador o host sobre la red donde RDP está activado. Éste host con RDP activo es considerado el servidor.

El protocolo RDP escucha por defecto en el puerto lógico **3389**, recordando que una IP es un identificador de un ordenador en una red y un puerto lógico un identificador asignado a una aplicación, siendo una subnet (red de la empresa) el símil de una calle, la IP el número de la casa y los puertos lógicos las formas de acceder a ésta. 

Podemos aprovechar que al establecerse una conexión pueden guardarse los perfiles con los que se accede, pudiendo buscarlos como archivos **.rdp**.

Para poder conectarse desde linux mediante RDP a un host Windows podemos usar **xfreerdp**, ya que es fácil de usar y eficiente. Otros conocidos son **Remmina** y **rdesktop**.

Para instalar xfreerdp el comando:

```shell
sudo apt-get intall freerdp2-x11
```

Para conectarse a un objetivo este es el sistema de comando

```shell
Adrirth@htb[/htb]$ xfreerdp /v:<IPobjetivo> /u:husuario /p:clave
```

![](/assets/images/Imagenes/Pasted image 20240601200801.png)
### Bibliografía

---

[**Página HTB**](https://academy.hackthebox.com/module/49/section/454)

[**Activar y permitir acceso remoto**](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/remote-desktop-allow-access)

[**xfreerdp**](https://linux.die.net/man/1/xfreerdp)
