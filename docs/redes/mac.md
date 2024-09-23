---
layout: default
title: Dirección MAC
parent: Redes
nav_order: 2
---

# MAC 
{:.no_toc}

---

1. List
{:toc}

---

**Medium Access Control Address**, representada en formato hexadecimal, dirección física de nuestras interfaces de red. Estándares:

- Ethernet (IEEE 802.3)
- BLuetooth (IEEE 802.15)
- WLAN (IEEE 802.11)

MAC address:

- `DE:AD:BE:EF:13:37`
- `DE-AD-BE-EF-13-37`
- `DEAD.BEEF.1337`

|**Representation**|**1st Octet**|**2nd Octet**|**3rd Octet**|**4th Octet**|**5th Octet**|**6th Octet**|
|---|---|---|---|---|---|---|
|Binary|1101 1110|1010 1101|1011 1110|1110 1111|0001 0011|0011 0111|
|Hex|DE|AD|BE|EF|13|37|

Los tres primeros octetos (3 bytes / 24 bit), son los llamados Unique Identifier (OUI), definidos por el IEEE (Institute of Electrical and Electronics Engineers).

La otra mitad corresponde la interfaz de red (NIC) o Individual Address Part / Network Interface Controller

## MAC Multicast

|**Representation**|**1st Octet**|**2nd Octet**|**3rd Octet**|**4th Octet**|**5th Octet**|**6th Octet**|
|---|---|---|---|---|---|---|
|Binary|`0000 0001`|`0000 0000`|`0101 1110`|1110 1111|0001 0011|0011 0111|
|Hex|`01`|`00`|`5E`|EF|13|37|

## MAC Broadcast

|**Representation**|**1st Octet**|**2nd Octet**|**3rd Octet**|**4th Octet**|**5th Octet**|**6th Octet**|
|---|---|---|---|---|---|---|
|Binary|`1111 1111`|`1111 1111`|`1111 1111`|`1111 1111`|`1111 1111`|`1111 1111`|
|Hex|`FF`|`FF`|`FF`|`FF`|`FF`|`FF`|

## Biliografía

[**macchanger**](https://github.com/alobbs/macchanger)

