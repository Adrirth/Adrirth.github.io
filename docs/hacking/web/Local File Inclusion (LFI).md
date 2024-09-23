---
title: Local File Inclusion
parent: Hacking Web
layout: default
nav_order: 2
---

# Local File Inclusion
{:.no_toc}

---

1. List
{:toc}

---

Muchos lenguajes modernos back-end, como PHP, Javascript o Java usan parámetros HTTP especificando que se muestra en una página web, lo que permite construir páginas dinámicas, reduciendo el tamaño total de código y simplificándolo. En muchos casos, los parámetros son usados para especificar que recurso se muestra en la página.

Si esas funcionalidades no son securizadas correctamente, un atacante puede manipular estos parámetros para mostrar el contenido de cualquier **archivo local (local file)**, en el servidor host, llevando a una vulnerabilidad **Local File Inclusion**.

El sitio más común para encontrar LFI es dentro de los motores de plantillas (templating engines). Para hacer que la mayoría de la aplicación web se vea de la misma forma entre páginas, una plantilla de motor muestra una página que enseña las partes estáticas comunes, como el header, la nav bar y el footer, y después carga el contenido dinámico que cambia entre páginas. Si no, para cada página necesitaría ser modificada cuando se aplican cambios en las partes estáticas.

Es por esto que se pueden ver parámetros como **/index.php?page=about** donde **index.php** establece el contenido estático (header, footer), y después vuelca el contenido dinámico especificado en el parámetro, que en este caso puede ser leído  de un archivo llamada **about.php**.

Mientras tengamos control de la porción de la petición es posible hacer que el servidor "coja" otros archivos y enseñarlos en la página. Bajo unas condiciones concretas, será posible ejectuar código en el servidor remoto.

---
## Ejemplos de código vulnerable
&nbsp;

Todos los framework de desarrollo tienen una forma diferente de acceder a archivos locales, pero todos tienen en común que cargan el archivo desde una ruta específica. 

Puede que una página tenga el parámetro **GET ?language** y si un usuario cambia el idioma desde un menú desplegable entonces la página volvería a salir pero con un parámetro de lenguaje diferente (**?language=es**). 

En ese caso se cambiará el lenguaje cambiando el directorio del que la aplicación web está cargando las páginas (**/en/**,**/es/**).

Si tenemos control sobre la ruta cargada, entonces podremos legar a explotar esta vulnerabilidad para leer otros archivos y potencialmente conseguir ejecución de código remota (**[[Remote Code Execution (RCE)]]**).

---
### PHP
&nbsp;

En PHP, puede que usemos la función **include()** para cargar un archivo local o remoto mientras cargamos la página. SI la ruta pasada por **include()** es recogida por un parámetro controlado por el usuario, como **GET**, y el código **no filtra ni sanitiza explícitamente el input del usuario**, entonces el código se convierte vuelve vulnerable a inclusión de código. Por ejemplo:

```php
if (isset($_GET['language'])) {
    include($_GET['language']);
}
```
&nbsp;

Vemos que el parámetro `language` se pasa directamente a la función `include()`. Por lo tanto, cualquier ruta que pasemos en el parámetro `language` se cargará en la página, incluidos cualquier archivo local en el servidor back-end. Esto no es exclusivo de la función `include()`, ya que hay muchas otras funciones en PHP que llevarían a la misma vulnerabilidad si tuviéramos control sobre la ruta que se les pasa. Dichas funciones incluyen `include_once()`, `require()`, `require_once()`, `file_get_contents()`, entre otras.

---
### NodeJS
&nbsp;

Al igual que en el caso de PHP, los servidores web de NodeJS también pueden cargar contenido basado en parámetros HTTP. El siguiente es un ejemplo básico de cómo un parámetro GET `language` se usa para controlar qué datos se escriben en una página:


```javascript
if(req.query.language) {
    fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
        res.write(data);
    });
}
```
&nbsp;

Como podemos ver, cualquier parámetro pasado desde la URL se utiliza en la función `readFile`, que luego escribe el contenido del archivo en la respuesta HTTP. Otro ejemplo es la función `render()` en el marco de trabajo `Express.js`. El siguiente ejemplo muestra cómo se usa el parámetro `language` para determinar de qué directorio debe extraer la página `about.html`:


```js
app.get("/about/:language", function(req, res) {
    res.render(`/${req.params.language}/about.html`);
});
```
&nbsp;

A diferencia de nuestros ejemplos anteriores donde los parámetros GET se especificaban después de un carácter (`?`) en la URL, el ejemplo anterior toma el parámetro de la ruta de la URL (por ejemplo, `/about/en` o `/about/es`). Dado que el parámetro se usa directamente dentro de la función `render()` para especificar el archivo renderizado, podemos cambiar la URL para mostrar un archivo diferente.

---
### Java
&nbsp;

El mismo concepto se aplica a muchos otros servidores web. Los siguientes ejemplos muestran cómo las aplicaciones web para un servidor web Java pueden incluir archivos locales basados en el parámetro especificado, utilizando la función `include`:

```jsp
<c:if test="${not empty param.language}">
    <jsp:include file="<%= request.getParameter('language') %>" />
</c:if>
```
&nbsp;

La función `include` puede tomar un archivo o una URL de página como su argumento y luego renderiza el objeto en la plantilla del front-end, similar a los ejemplos que vimos anteriormente con NodeJS. La función `import` también se puede usar para renderizar un archivo local o una URL, como en el siguiente ejemplo:

```jsp
<c:import url="<%= request.getParameter('language') %>"/>
```

---
### .NET
&nbsp;

Finalmente, tomemos un ejemplo de cómo pueden ocurrir vulnerabilidades de inclusión de archivos en aplicaciones web .NET. La función `Response.WriteFile` funciona de manera muy similar a todos nuestros ejemplos anteriores, ya que toma una ruta de archivo como entrada y escribe su contenido en la respuesta. La ruta puede ser recuperada de un parámetro GET para la carga dinámica de contenido, de la siguiente manera:

```cs
@if (!string.IsNullOrEmpty(HttpContext.Request.Query['language'])) {
    <% Response.WriteFile("<% HttpContext.Request.Query['language'] %>"); %> 
}
```
&nbsp;

Además, la función `@Html.Partial()` también se puede usar para renderizar el archivo especificado como parte de la plantilla del front-end, similar a lo que vimos anteriormente:

```cs
@Html.Partial(HttpContext.Request.Query['language'])
```
&nbsp;

Finalmente, la función `include` puede ser utilizada para renderizar archivos locales o URLs remotas, y también puede ejecutar los archivos especificados.

```cs
<!--#include file="<% HttpContext.Request.Query['language'] %>"-->
```
&nbsp;


| **Función**                  | **Leer Contenido** | **Ejecutar** | **URL Remota** |
| ---------------------------- | :----------------: | :----------: | :------------: |
| **PHP**                      |                    |              |                |
| `include()`/`include_once()` |         ✅          |      ✅       |       ✅        |
| `require()`/`require_once()` |         ✅          |      ✅       |       ❌        |
| `file_get_contents()`        |         ✅          |      ❌       |       ✅        |
| `fopen()`/`file()`           |         ✅          |      ❌       |       ❌        |
| **NodeJS**                   |                    |              |                |
| `fs.readFile()`              |         ✅          |      ❌       |       ❌        |
| `fs.sendFile()`              |         ✅          |      ❌       |       ❌        |
| `res.render()`               |         ✅          |      ✅       |       ❌        |
| **Java**                     |                    |              |                |
| `include`                    |         ✅          |      ❌       |       ❌        |
| `import`                     |         ✅          |      ✅       |       ✅        |
| **.NET**                     |                    |              |                |
| `@Html.Partial()`            |         ✅          |      ❌       |       ❌        |
| `@Html.RemotePartial()`      |         ✅          |      ❌       |       ✅        |
| `Response.WriteFile()`       |         ✅          |      ❌       |       ❌        |
| `include`                    |         ✅          |      ✅       |       ✅        |

Con esto ya podemos intentar acceder a archivos del sistema más críticos en el caso de la página cargue algún recurso, como **/etc/passwd** en Linux o **C:\Windows\boot.ini**.

![](/assets/images/Imagenes/Pasted image 20240602203916.png)

---
## Path Traversal
&nbsp;

En este caso, leemos un archivo al especificar la ruta absoluta (**/etc/passwd**). Esto funcionaría si todo el input fuera usada por la función **include()** sin más elementos:

```php
include($_GET['language']);
```
&nbsp;

Sin embargo en otras ocasiones, los desarrolladores web pueden añadir o quitar una cadena al parámetro **language**. Por ejemplo que el parámetro **language** sea usado por el nombre del archivo y pueda ser añadido después de un directorio:

```php
include("./languages/" . $_GET['language']);
```
&nbsp;

En este caso, si intentamos acceder mediante ruta absoluta (**/etc/passwd**), entonces la ruta pasada por la función **include()** sería (**./languages//etc/passwd**), y este archivo no existe, y nos seremos capaces de leer nada.

![](/assets/images/Imagenes/Pasted image 20240602204515.png)

&nbsp;

Podemos rebasar esta restricción recorriendo directorios usando **rutas relativas**. Para hacer eso, podemos añadir **../** antes del nombre de nuestro archivo, refiriéndose al directorio padre. Por ejemplo, la ruta completa del directorio de lenguajes es **/var/www/html/languages**, entonces usando **../index.php** se referiría al archivo **index.php** en el directorio padre (****/var/www/html/index.php**).

Con este truco podemos retroceder varios directorios hasta que lleguemos la ruta raíz (**/**), y después especificar nuestra ruta absoluta del archivo (**../../../../../../etc/passwd**), y el archivo debería existir y mostrarse:

![](/assets/images/Imagenes/Pasted image 20240602205026.png)

&nbsp;

Para ser más eficientes, la distancia que debemos recorrer para llegar al directorio raíz es de tres saltos, por lo que con usar **../** deberemos poder realizarlo.

---
## Prefijo del nombre del archivo
&nbsp;

En algunas ocasiones nuestra entrada de datos puede adjuntarse con una cadena diferente. Por ejemplo puede ser usado con un prefijo para conseguir el nombre completo del archivo:

```php
include("lang_" . $_GET['language']);
```
&nbsp;

En este caso sin intentáramos llegar al directorio con **../../../etc/passwd**, la cadena final sería  **lang_../../../etc/passwd**, que sería inválido:

![](/assets/images/Imagenes/Pasted image 20240603123714.png)

&nbsp;

En vez de eso podemos usar un prefijo con nuestro **payload** (carga de datos), y esto podría considerar el prefijo como un directorio, y después podremos sobrepasar el nombre del archivo y ser capaces de recorrer directorios.

![](/assets/images/Imagenes/Pasted image 20240603124134.png)

---
## Extensiones ligadas
&nbsp;

Otra forma común de una extensión ligada a una entrada de archivo es la siguiente:

```php
include($_GET['language'].".php");
```
&nbsp;

Es muy común, como en este caso, no tendríamos que escribir la extensión cada vez que necesitemos cambiar el lenguaje. Esto también puede ser más seguro ya que nos restringe a incluir solo archivos PHP. En este caso, si intentamos leer */etc/passwd*, entonces el archivo incluido sería */etc/passwd.php*, el cuál no existe:

![](/assets/images/Imagenes/Pasted image 20240618230742.png)

---
## Ataques de Segundo Orden (Second-Order Attacks)
&nbsp;

Este ocurre cuando muchas funcionalidades de aplicaciones web pueden estar de forma insegura cogiendo archivos de la parte del servidor basándose en parámetros controlados por el usuario.

Por ejemplo, una aplicación web puede permitirnos descargar nuestro avatar a través  a través de una URL como */profile/$username/avatar.png*. Si creamos un LFI de nombre de usuario malicioso (*../../../../etc/passwd*), entonces es posible poder cambiar el archivo que está siendo llamado por otro archivo local en el servidor y coger ese en vez de nuestro avatar.

En este caso estaríamos envenenando una entrada de base de datos con una carga LFI maliciosa con nuestro usuario. Entonces, otra funcionalidad de aplicación web utilizaría esta entrada envenenada para realizar nuestro ataque, por eso se le llama Ataque de Segundo Orden.

---
## Sorteamientos Básicos (Bypass)
&nbsp;

En los casos que la aplicación web aplique varias protecciones contra file inclusion, nuestra carga maliciosa no funcionaría. Si embargo, a no ser que la web esté segurizada apropiadamente contra LFI de entrada usuario maliciosos, puede que seamos capaces de sobrepasar las protecciones creadas y llegar a ejecutarlo.

---
## Filtros de Path Traversal No Recursivos
&nbsp;

Uno de los filtros más básicos contra LFI es un filtro de búsqueda y reemplazamiento, donde simplemente elimina partes de la cadena (*../*) para evitar path traversals. Por ejemplo:

```php
$language = str_replace('../','', $_GET['language']);
```
&nbsp;

Este código tiene la intención de evitar el path traversal, dejando intentos de LFI inútiles. SI intentamos las cargas LFI hechas anteriormente, tendríamos lo siguiente:

![](/assets/images/Imagenes/Pasted image 20240618233807.png)

&nbsp;

Vemos que todas las subcadenas *../* fueron eliminadas, resultando que la ruta final fuera *./languages/etc/passwd*. Sin embargo, este filtro es muy inseguro, ya que no está removiendo la subcadena *../* de forma recursiva. Por ejemplo, si usamos *....//* como nuestra payload, entonces el filtro quitaría *../* y la cadena resultante sería *../*, lo que significa que todavía podemos realizar un path traversal.

![](/assets/images/Imagenes/Pasted image 20240618234142.png)

&nbsp;

Aparte de esta, podemos usar otras como *.././* o *...\/* y otra serie de payloads.

---
## Codificación
&nbsp;

Algunos filtros web pueden prevenir filtros de entrada que incluyan ciertos caracteres relacionados con LFIs, como un punto *.* o una barra */*, usados para path traversals. Sin embargo, algunos de estos filtros pueden ser burlados mediante codificación URL de nuestro input, con lo que no incluiría estos caracteres, pero seguiría estando decodificado de vuelta a nuestra cadena de path traversal una vez alcance la función vulnerable.

Los filtros fundamentales de PHP en versiones anteriores a la 5.3.4 eran específicamente vulnerables a esta sorteamiento, pero incluso en versiones más nuevas es posible encontrar filtros propios que pueden ser sorteados mediante codificación URL.

Si el cliente de la aplicación no lo permite *.* ni */* en nuestra input, podemos codificar por URL *../* a *%2e%2e%2f*, que puede sortear el filtro. Para hacerlo, podemos usar cualquier utilidad codificación online a URL o usar la herramienta Burp Suite Decode, de la siguiente forma:

![](/assets/images/Imagenes/Pasted image 20240621110905.png)

---

{: .highlight }

**Nota**: Para que esto funcione debemos codificar todos los caracteres, incluyendo los puntos. Algunos codificador pueden no codificar puntos al ser considerados parte del esquema URL

---

![](/assets/images/Imagenes/Pasted image 20240621111341.png)

Como se puede ver, podemos de forma exitosa sortear el filtro y usar path traversal para leer el archivo */etc/passwd*. 

---

## Rutas aprovadas
&nbsp;

Algunas aplicaciones web pueden usar **Expresiones Regulares** para asegurar que el archivo que está siendo incluido esta sobre una ruta específica. Por ejemplo, la aplicación web que hemos estado probando solo acepta rutas que están en el directorio *./languages*:

```php
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
    include($_GET['language']);
} else {
    echo 'Illegal path specified!';
}
```
&nbsp;

Para buscar la ruta permitida, podemos examinar las peticiones enviadas por formularios existentes, y ver que ruta están usando para un funcionamiento web correcto. Podemos además, "rastrear" (*fuzz*) directorios web que estén en la misma ruta, y probar diferentes hasta conseguir uno compatible (*match*). Para sortear esto, podemos usar path traversal y empezar nuestro payload con la ruta permitida, y luego usar *../* para volver al directorio raíz y leer el archivo que especifiquemos.

En pocas palabras, rastrear rutas existentes que podamos aprovechar para inyectar nuestra carga de código, sorteando el filtro de rutas específicas.

![](/assets/images/Imagenes/Pasted image 20240621112420.png)

&nbsp;

Algunas aplicaciones web pueden aplicar este filtro junto con otros filtros anteriores, por lo que podemos combinar ambas técnicas primero con el directorio permitido que hayamos averiguado, y después con la codificación URL de nuestro payload o usando payload recursivo.

---

{: .highlight }

**Nota**: Todas las técnicas mencionadas deberían funcionar con cualquier vulnerabilidad LFI, sin importar el lenguaje de desarrollo back-end o el framework.

---

## Extensión Añadida
&nbsp;

Como se discutió en la sección anterior, algunas aplicaciones web añaden una extensión a nuestra cadena de entrada (por ejemplo, .php) para asegurarse de que el archivo que incluimos tiene la extensión esperada. Con versiones modernas de PHP, es posible que no podamos evitar esto y estaremos restringidos a leer solo archivos con esa extensión, lo cual aún puede ser útil, como veremos en la siguiente sección (por ejemplo, para leer código fuente).

Existen un par de técnicas adicionales que podemos usar, pero son obsoletas con las versiones modernas de PHP y solo funcionan con versiones anteriores a PHP 5.3/5.4. Sin embargo, aún puede ser beneficioso mencionarlas, ya que algunas aplicaciones web pueden estar ejecutándose en servidores más antiguos, y estas técnicas pueden ser los únicos métodos posibles para eludir las restricciones.

## Truncamiento de Ruta
&nbsp;

En versiones anteriores de PHP, las cadenas definidas tienen una longitud máxima de 4096 caracteres, probablemente debido a la limitación de los sistemas de 32 bits. Si se pasa una cadena más larga, simplemente se truncará, y cualquier carácter después de la longitud máxima será ignorado. Además, PHP también solía eliminar barras inclinadas finales y puntos simples en los nombres de ruta, por lo que si llamamos a (/etc/passwd/.) entonces el /. también sería truncado, y PHP llamaría a (/etc/passwd). PHP, y los sistemas Linux en general, también ignoran múltiples barras inclinadas en la ruta (por ejemplo, ////etc/passwd es lo mismo que /etc/passwd). De manera similar, un atajo de directorio actual (.) en medio de la ruta también sería ignorado (por ejemplo, /etc/./passwd).

Si combinamos ambas limitaciones de PHP, podemos crear cadenas muy largas que se evalúan como una ruta correcta. Cada vez que alcancemos la limitación de 4096 caracteres, la extensión añadida (.php) sería truncada, y tendríamos una ruta sin una extensión añadida. Finalmente, es importante notar que también necesitaríamos comenzar la ruta con un directorio inexistente para que esta técnica funcione.

Un ejemplo de tal carga útil sería el siguiente:

`?language=directorio_no_existente/../../../etc/passwd/./././.[./ REPETIDO ~2048 veces]`
&nbsp;

Por supuesto, no tenemos que escribir manualmente *./* 2048 veces (un total de 4096 caracteres), pero podemos automatizar la creación de esta cadena con el siguiente comando:

```bash
$ echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
```
&nbsp;

`directorio_no_existente/../../../etc/passwd/./././<RECORTADO>././././`

También podemos aumentar la cantidad de ../, ya que añadir más aún nos llevaría al directorio raíz, como se explicó en la sección anterior. Sin embargo, si usamos este método, deberíamos calcular la longitud total de la cadena para asegurarnos de que solo se trunque .php y no el archivo solicitado al final de la cadena (/etc/passwd). Por esta razón, sería más fácil usar el primer método.

---
## Bytes Nulos 
&nbsp;

Las verisiones anteriores de PHP a la 5.5 eran vulnerables a **inyección de byte nulo**, que significa que añadiendo un byte nulo (*%00*) al final de la cadena haría que no se considerara nada después de este. Esto es debido a como las cadenas son guardadas en la memoria de bajo nivel, donde las cadenas en memoria deben usar un byte para indicar el final de la cadena, como en Assembly, C o lenguajes C++.

Para explotar esta vulnerablilidad, podemos acabar nuestra entrada con un byte nulo (ej. */etc/passwd%00*), con lo que la ruta final pasada a la función **include()** sería (*/etc/passwd%00.php*). De esta forma, aunque *.php* sea adjunto a nuestra cadena, cualquier cosa después del byte nulo será cortado, con lo que la ruta final usada sería */etc/passwd*, llevandonos a sortear la extensión adjunta.


```
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --sc 200,202,204,301,302,307,403 http://example.com/uploads/FUZZ
```

---

## Filtros PHP
&nbsp;

Muchas aplicaciones web son desarrolladas en PHP, además de varias aplicaciones personalizadas construidas con frameworks de PHP, como Laravel o Symfony. Si identificamos una vulnerabilidad en aplicaciones web con PHP, podemos utilizar diferentes [Wrappers](https://www.php.net/manual/en/wrappers.php.php) (envoltorios) de PHP para extender nuestra explotación LFI, e incluso alcanzar potencialmente un RCE (Remote Code Execution).

Estos nos permiten acceder a diferentes secuencias de entrada y salida de datos (I/O streams) a nivel de aplicación, como entradas y salidas estándar o secuencias de memoria. Para desarrolladores PHP esto tiene muchos usos, y en nuestro caso utilizarlo apara extender nuestros ataques de explotación y ser capaces de leer código fuente de archivos PHP o incluso ejecutar comandos de sistema.

Esto no es solo útil para ataques LFI, pero también para otros ataques como XXE (XML External Entity).

---

## Filtros de entrada
&nbsp;

Los filtros PHP son una serie de envoltorios (wrappers), donde podemos introducir distintos tipos de entrada y filtrar por el filtro que especifiquemos. Para usarlos, podemos utilizar el esquema *php://* en nuestra cadena, y podemos acceder al filtro *wrapper* con *php://filter/*.

El filtro de wrapper puede tener varios parámetros, pero los principales requeridos para nuestro ataque son **resource** y **read**. El parámetro *resource* es requerido para algunos filtros wrapper, y con el podemos especificar el flujo (ejem. archivo local), mientras que el parámetro *read* puede aplicar varios filtros en el recurso de entrada, por lo que podemos usarlo para especificar que filtro queremos aplicar a nuestro recurso.

Hay cuatro tipos diferentes de filtros disponibles para usar, que son [**Filtros de cadenas**](https://www.php.net/manual/en/filters.string.php), [**FIltros de conversión**](https://www.php.net/manual/en/filters.convert.php), [**Filtros de compresión**](https://www.php.net/manual/en/filters.compression.php) y [**Filtros de encriptación**](https://www.php.net/manual/en/filters.encryption.php).