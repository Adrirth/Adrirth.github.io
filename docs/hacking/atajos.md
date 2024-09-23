---
title: Atajos
layout: default
parent: Hacking
---

# Atajos
{:.no_toc}

---

{: .note }

Estas funciones son originalmente creadas por [s4vitar](https://github.com/s4vitar), sirviendo para realizar funciones de escaneo de puertos y demás de forma más rápida y eficaz.

---

1. List
{:toc}

---


## Escáner de puertos


```bash
#!/bin/bash

function ctrl_c(){
	echo -e "\n\n[!] Saliendo...\n"
	tput cnorm; exit 1
}

#Ctrl+C
trap ctrl_c INT

tput civis

for port in $(seq 1 65535); do 
	timeout 1 bash -c "echo '' > /dev/tcp/INSERT.IP.NUMBER/$port" 2>/dev/null && echo "[+] Port $port - OPEN" & done; wait

tput down
```

---


## Extraer puertos de un escaneo nmap con un archivo grep

````python
# Extract nmap information
    
function extractPorts(){
    
	 ports="$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr ' ' ',')"
     ip_address="$(cat $1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' | sort -u | head -n 1)"
     echo -e "\n[*] Extracting information...\n" > extractPorts.tmp
     echo -e "\t[*] IP Address: $ip_address"  >> extractPorts.tmp
     echo -e "\t[*] Open ports: $ports\n"  >> extractPorts.tmp
     echo $ports | tr -d '\n' | xclip -sel clip
     echo -e "[*] Ports copied to clipboard\n"  >> extractPorts.tmp
     cat extractPorts.tmp; rm extractPorts.tmp
    
}
````

---

## Crear tres directorios para organizar trabajo

```bash
mkt () {
	mkdir {nmap,content,scripts}
}
```

---

## Pequeño programa que funcione como nmap

```bash
#!/bin/bash

function ctrl_c(){
	echo -e "\n\n[!] Saliendo...\n"
	tput cnorm; exit 1
}

#Ctrl+C
trap ctrl_c INT

tput civis

for port in $(seq 1 65535); do 
	timeout 1 bash -c "echo '' > /dev/tcp/INSERT.IP.NUMBER/$port" 2>/dev/null && echo "[+] Port $port - OPEN" & done; wait

tput down
```

---


## Fijar ip de la maquina vulnerable en la polybar

```bash
settarget () {
	if [ $# -eq 1 ]
	then
		echo $1 > ~/.config/polybar/shapes/scripts/target
	elif [ $# -gt 2 ]
	then
		echo "settarget [IP] [NAME] | settarget [IP]"
	else
		echo $1 $2 > ~/.config/polybar/shapes/scripts/target
	fi
}

```

---

## Averiguar sistema en base a TTL de escaneo nmap

```python
#!/usr/bin/python3
#coding: utf-8

import re, sys, subprocess

# python3 wichSystem.py 10.10.10.188
    
if len(sys.argv) != 2:
	print("\n[!] Uso: python3 " + sys.argv[0] + " <direccion-ip>\n")
	sys.exit(1)
    
def get_ttl(ip_address):
    
	proc = subprocess.Popen(["/usr/bin/ping -c 1 %s" % ip_address, ""], stdout=subprocess.PIPE, shell=True)
    (out,err) = proc.communicate()
    
     out = out.split()
     out = out[12].decode('utf-8')
    
     ttl_value = re.findall(r"\d{1,3}", out)[0]

     return ttl_value

def get_os(ttl):
    
	 ttl = int(ttl)
    
     if ttl >= 0 and ttl <= 64:
         return "Linux"
     elif ttl >= 65 and ttl <= 128:
         return "Windows"
     else:
         return "Not Found"
    
if __name__ == '__main__':
    
     ip_address = sys.argv[1]
    
     ttl = get_ttl(ip_address)

     os_name = get_os(ttl)
    
     print("\n%s (ttl -> %s): %s\n" % (ip_address, ttl, os_name))
 ```