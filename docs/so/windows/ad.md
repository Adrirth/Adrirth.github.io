---
title: Active Directory
layout: default
parent: Windows
nav_order: 1
---

# Active Directory
{:.no_toc}

---

1. List
{:toc}

---

**Active Directory** o Directorio Activo es un servicio de directorio para entornos de red de Windows. Es una estructura jerárquica, distribuida que permite administrar de forma centralizada los recursos de una organización, incluyendo usuarios, ordenadores, grupos, dispositivos de red, compartición de archivos, políticas de grupos, dispositivos y fideicomisos. 

AD proporciona funciones de autenticación y autorización dentro de un entorno de dominio Windows. Se ha convertido cada vez más recientemente en un objetivo de ataque. Está diseñado para ser retro compatible, y muchas de sus funciones son no seguras por defecto, pudiendo ser además fácilmente mal configurado.

Esta debilidad puede ser aprovechada para moverse de forma lateral y vertical dentro de una red y ganar acceso no autorizado. AD es esencialmente una base de datos accesible a todos los usuarios dentro del dominio, sin importar el nivel de privilegios. Una cuenta básica de usuario de AD sin privilegios añadidos puede enumerar la mayoría de objetos dentro del Directorio Activo.

Esto hace que sea extremadamente importante asegurar correctamente una implementación de AD porque cualquier cuenta de usuario, sin importar el nivel de privilegios, puede ser usada para enumerar el dominio o la caza de malas configuraciones y de fallas. Además, muchos ataques puede ser realizados solo con una cuenta de usuario estándar, mostrando la importancia de una estrategia de defensa profunda y planeando con cuidado enfocándose en seguridad y fortaleciendo el AD, la segmentación de la red, y los menos privilegios posibles. Un ejemplo es el ataque [noPac](https://www.secureworks.com/blog/nopac-a-tale-of-two-vulnerabilities-that-could-end-in-ransomware) que fue lanzado en Diciembre de 2021.

El Directorio Activo hace que la información sea fácil de buscar y usar para administradores y usuarios. AD es fácilmente escalable, permite millones de objetos por dominio, y permite la creación de dominios adicionales a medida que una organización crece.

![](/assets/images/Imagenes/Pasted image 20240623125448.png)

---

Se estima que alrededor del 95% de las empresas de [Fortune 500](https://en.wikipedia.org/wiki/Fortune_500) utilizan Active Directory, lo que convierte a AD en un objetivo clave para los atacantes. Un ataque exitoso, como un phishing que coloque a un atacante dentro del entorno de AD como un usuario de dominio estándar, les daría suficiente acceso para comenzar a mapear el dominio y buscar vías de ataque. Como profesionales de la seguridad, encontraremos entornos de AD de todos los tamaños a lo largo de nuestra carrera. Es esencial comprender la estructura y función de AD para estar mejor informados tanto como atacantes como defensores.

Los operadores de ransomware han estado apuntando cada vez más a Active Directory como una parte clave de sus rutas de ataque. El [Conti Ransomware](https://www.cisa.gov/sites/default/files/publications/AA21-265A-Conti_Ransomware_TLP_WHITE.pdf), que se ha utilizado en más de 400 ataques en todo el mundo, ha demostrado aprovechar recientes fallas críticas de Active Directory como [PrintNightmare (CVE-2021-34527)](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) y [Zerologon (CVE-2020-1472)](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2020-1472) para escalar privilegios y moverse lateralmente en una red objetivo. Comprender la estructura y función de Active Directory es el primer paso en una carrera para encontrar y prevenir este tipo de fallas antes que los atacantes. Los investigadores están encontrando continuamente nuevos ataques de altísimo riesgo que afectan los entornos de Active Directory y que a menudo requieren no más que un usuario de dominio estándar para obtener control administrativo completo sobre todo el dominio. Hay muchas excelentes herramientas de código abierto para los testers de penetración para enumerar y atacar Active Directory. Sin embargo, para utilizarlas de manera más efectiva, debemos entender cómo funciona Active Directory para identificar fallas evidentes y sutiles. Las herramientas son tan efectivas como su operador es conocedor. Así que tomémonos el tiempo para entender la estructura y función de Active Directory antes de pasar a módulos posteriores que se centrarán en la enumeración manual y basada en herramientas, ataques, movimiento lateral, post-explotación y persistencia.

Este módulo sentará las bases para comenzar a enumerar y atacar Active Directory. Cubriremos en profundidad la estructura y función de AD, discutiremos los diversos objetos de AD, los derechos y privilegios de los usuarios, las herramientas y procesos para gestionar AD, e incluso recorreremos ejemplos de cómo configurar un pequeño entorno de AD.

---

## Historia de Active Directory

LDAP, la base de Active Directory, se introdujo por primera vez en los [RFCs](https://en.wikipedia.org/wiki/Request_for_Comments) ya en 1971. Active Directory fue precedido por el concepto de unidad organizativa [X.500](https://en.wikipedia.org/wiki/X.500), que fue la primera versión de todos los sistemas de directorios creados por Novell y Lotus y lanzado en 1993 como [Novell Directory Services](https://en.wikipedia.org/wiki/NetIQ_eDirectory).

Active Directory se introdujo por primera vez a mediados de los 90, pero no se convirtió en parte del sistema operativo Windows hasta el lanzamiento de Windows Server 2000. Microsoft intentó proporcionar servicios de directorio por primera vez en 1990 con el lanzamiento de Windows NT 3.0. Este sistema operativo combinaba características del protocolo [LAN Manager](https://en.wikipedia.org/wiki/LAN_Manager) y los sistemas operativos [OS/2](https://en.wikipedia.org/wiki/OS/2), que Microsoft creó inicialmente junto con IBM bajo la dirección de [Ed Iacobucci](https://en.wikipedia.org/wiki/Ed_Iacobucci), quien también dirigió el diseño de [IBM DOS](https://en.wikipedia.org/wiki/IBM_PC_DOS) y posteriormente cofundó Citrix Systems. El sistema operativo NT evolucionó a lo largo de los 90, adaptando protocolos como LDAP y Kerberos con elementos propietarios de Microsoft. La primera versión beta de Active Directory fue en 1997.

El lanzamiento de Windows Server 2003 vio una funcionalidad extendida y una administración mejorada, y añadió la función `Forest`, que permite a los administradores de sistemas crear "contenedores" de dominios separados, usuarios, computadoras y otros objetos, todo bajo el mismo paraguas. [Active Directory Federation Services (ADFS)](https://en.wikipedia.org/wiki/Active_Directory_Federation_Services) se introdujo en Server 2008 para proporcionar Single Sign-On (SSO) a sistemas y aplicaciones para usuarios en sistemas operativos Windows Server. ADFS hizo más simple y fluido que los usuarios iniciaran sesión en aplicaciones y sistemas que no están en su misma LAN.

ADFS permite a los usuarios acceder a aplicaciones a través de límites organizacionales utilizando un solo conjunto de credenciales. ADFS utiliza el modelo de autorización de Control de Acceso basado en [claims](https://en.wikipedia.org/wiki/Claims-based_identity), que intenta garantizar la seguridad a través de aplicaciones identificando a los usuarios mediante un conjunto de claims relacionados con su identidad, que son empaquetados en un token de seguridad por el proveedor de identidad.

El lanzamiento de Server 2016 trajo aún más cambios a Active Directory, como la capacidad de migrar entornos de AD a la nube y mejoras adicionales en seguridad como el monitoreo del acceso de usuarios y [Group Managed Service Accounts (gMSA)](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/service-accounts-group-managed). gMSA ofrece una forma más segura de ejecutar tareas automáticas específicas, aplicaciones y servicios, y a menudo es una mitigación recomendada contra el infame ataque Kerberoasting.

2016 vio un empuje más significativo hacia la nube con el lanzamiento de Azure AD Connect, que fue diseñado como un método de Single Sign-On para usuarios que se están migrando al entorno de Microsoft Office 365.

Active Directory ha sufrido diversas configuraciones incorrectas desde 2000 hasta la actualidad. Se descubren nuevas vulnerabilidades regularmente que afectan a Active Directory y otras tecnologías que interactúan con AD, como Microsoft Exchange. A medida que los investigadores de seguridad continúan descubriendo nuevas fallas, las organizaciones que ejecutan Active Directory necesitan mantenerse al día con los parches e implementar soluciones. Como testers de penetración, nuestra tarea es encontrar estas fallas para nuestros clientes antes que los atacantes.

Por esta razón, debemos tener una sólida base en los fundamentos de Active Directory y entender su estructura, función, los diversos protocolos que utiliza para operar, cómo se gestionan los derechos y privilegios de los usuarios, cómo los administradores de sistemas administran AD y la multitud de vulnerabilidades y configuraciones incorrectas que pueden estar presentes en un entorno de AD. Administrar AD no es una tarea fácil. Un cambio/arreglo puede introducir problemas adicionales en otra parte. Antes de comenzar a enumerar y luego atacar Active Directory, cubramos conceptos fundamentales que nos seguirán a lo largo de nuestras carreras en infosec.

Como se dijo antes, el 95% de las empresas de Fortune 500 utilizan Active Directory, y Microsoft tiene casi un monopolio completo en el espacio de servicios de directorio. Aunque muchas empresas están haciendo la transición a entornos en la nube e híbridos, el AD on-premise no desaparecerá para muchas empresas. Si estás realizando compromisos de pruebas de penetración de red, puedes estar casi seguro de encontrar AD de alguna manera en casi todos ellos.

Este conocimiento fundamental nos hará mejores atacantes y nos dará una visión sobre AD que será extremadamente útil al proporcionar consejos de remediación a nuestros clientes. Una comprensión profunda de AD hará que desentrañar sus capas sea menos desalentador, y tendremos la misma confianza al abordar un entorno con 10,000 hosts que con uno con 20.

---

## Estructura AD

Active Directory está organizado como una estructura jerárquica en forma de árbol, con un bosque en la cima conteniendo uno o más dominios, que a su vez pueden contener subdominios. Un bosque es un límite de seguridad donde todos los objetos están bajo control administrativo. Un bosque puede contener múltiples dominios, y un dominio puede contener un dominio hijo o subdominios. Un dominio es una estructura en la que objetos contenidos (usuarios, ordenadores y grupos) son accesibles. 

Tiene Unidades Organizacionales (Organizational Units, OUs), como Controladores de Dominio, Usuarios, Ordenadores, y nuevas OUs que pueden ser creadas por requerimiento. Las OUs pueden contener objetos y sub-OUs, permitiendo asignar diferentes políticas de grupo.

De forma muy simple, una estructura de AD es como se muestra:

```shell
INLANEFREIGHT.LOCAL/
├── ADMIN.INLANEFREIGHT.LOCAL
│   ├── GPOs
│   └── OU
│       └── EMPLOYEES
│           ├── COMPUTERS
│           │   └── FILE01
│           ├── GROUPS
│           │   └── HQ Staff
│           └── USERS
│               └── barbara.jones
├── CORP.INLANEFREIGHT.LOCAL
└── DEV.INLANEFREIGHT.LOCAL
```

Aquí podríamos decir que **INLANEFREIGHT.LOCAL** es el dominio raíz y contiene los subdominios (dominios hijo o raíz del árbol) **ADMIN.INLANEFREIGHT.LOCAL**, **CORP.INLANEFREIGHT.LOCAL** y **DEV.LANEFREIGHT.LOCAL** además de los otros objetos que hacen un dominio como usuarios, grupos, ordenadores y más. 

Es común ver varios dominios (o bosques) unidos vía relaciones de confianza en organizaciones que ejecutan muchas adquisiciones. La mayoría de las veces es más fácil y rápido crear una relación de confianza con otros dominios/bosques que recrear todos los usuarios de un dominio ya existente. Claro que esto introduce huecos de seguridad si no se administra de forma apropiada.

![](/assets/images/Imagenes/Pasted image 20240623131722.png)

En el gráfico siguiente se muestran dos bosques, **INFRAEFREIGHT.LOCAL** y **FREIGHTLOGISTICS.LOCAL**. La flecha de dos direcciones representa una confianza mutua entre los dos bosques, con lo que los usuarios de **INFRAEFREIGHT.LOCAL** pueden acceder a los recursos de **FREIGHTLOGISTICS.LOCAL**. y viceversa. También podemos ver múltiples dominios hijos bajo cada dominio raíz. En este ejemplo podemos ver que el dominio raíz confía en cada uno de los dominios hijo, pero los dominios hijo del bosque A no tienen establecido necesariamente una confianza con los dominios hijo del bosque B.

Esto significa que un usuario que es parte de **admin.dev.freightlogistics.local** NO sería capaz de autenticarse en máquinas del dominio **wh.corp.inlanfreight.local** por defecto aunque exista una relación de confianza entre los dominios con nivel más alto **inlanefreight.local** y **freightlogistics.local**. Para permitir la comunicación directa desde **admin.dev.freightlogistics.local** y **wh.corp.inlanfreight.local**, se debería configurar otra relación de confianza.

![](/assets/images/Imagenes/Pasted image 20240623132629.png)

---

# Terminología AD

## Objeto

Cualquier recurso presente en el entorno de Active Directory como OUs, impresoras, usuarios, controladores de dominio.

## Atributos

Cualquier objeto en el Directorio Activo tiene asociado un número de [atributos](https://learn.microsoft.com/en-us/windows/win32/adschema/attributes-all) usados para definir características de un objeto dado. Un objeto ordenador contiene atributos como el nombre de host y el nombre DNS. Todos los atributos en AD tienen asociado un nombre LDAP que puede ser usado para peticiones LDAP como **displayName** para el nombre con apellidos o **given name** para el nombre.

## Esquema

El [esquema](https://learn.microsoft.com/en-us/windows/win32/ad/schema) de Active Directory es esencialmente un plano de todo el entorno empresarial. Define que tipos de objetos pueden existir dentro de la base de datos del AD y sus atributos asociados. Lista las definiciones correspondientes a los objetos de AD y guarda información sobre cada objeto. Por ejemplo, los usuarios en AD pertenecen a la clase "usuario" y los ordenadores a "ordenador", y así. Cada objeto tiene su propia información (alguna es necesita ser configurada y otra opcional) que es guardada en Atributos. Cuando un objeto es creado en base a una clase, esto se llama instanciación, y cuando un objeto es creado sobre una clase en específico se le llama instancia de un clase. Por ejemplo, con un ordenador RDS01. Este objeto ordenador es una instancia de la clase de Directorio Activo "ordenador".

## Dominio

Es un grupo lógico de objetos como ordenadores, usuarios, grupos, OUs, etc. Podemos pensar en cada dominio como una ciudad diferente dentro de un estado o país. Los dominios pueden operar de forma totalmente independiente entre ellos o conectarse mediante relaciones de confianza.

## Árbol

Un árbol es una colección de dominios de Directorio Activo que pertenecen a un solo dominio raíz. Un bosque es una colección de árboles de Directorio Activo. Cada dominio en un árbol comparte un vínculo con los otros dominios. Una relación de confianza entre padre e hijo es formada cuando un dominio es añadido bajo otro dominio dentro de un árbol. Dos árboles en un mismo bosque no pueden compartir un nombre (namespace). Digamos que tenemos dos árboles en un bosque de Directorio Activo: **inlanefreight.local** y **ilfreight.local**. Un dominio hijo del primero sería **corp.inlanfreight.local** mientras que dominio hijo del segundo sería **corp.ilfreight.local**. Todos los dominios de un árbol comparten un Catálogo Global estándar que contiene toda la información sobre objetos que pertenecen al árbol.

## Contenedor

Los objetos contenedor contienen otros objetos y tienen un lugar definido en la jerarquía del directorio de subárboles de directorio.

## Hoja (Leaf)

Los objetos leaf no contienen otros objetos y se encuentran al final de la jerarquía de subárboles.