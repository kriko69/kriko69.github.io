# HTTP Verb Tampering

## Table of contents

- [HTTP Verb Tampering](#http-verb-tampering)
  - [INTRODUCCION](#introduccion)
  - [CAUSAS](#causas)
    - [Configuraciones inseguras](#configuraciones-inseguras)
    - [Codificación insegura](#codificacin-insegura)
  - [BYPASS BASIC AUTHENTICATION](#bypass-basic-authentication)
  - [BYPASS FILTROS DE SEGURIDAD](#bypass-filtros-de-seguridad)
  - [PREVENCION](#prevencion)
    - [Codificación insegura](#codificacin-insegura)

## INTRODUCCION

Si bien los programadores consideran principalmente los dos métodos HTTP más utilizados `GET` y `POST`, cualquier cliente puede enviar cualquier otro método en sus solicitudes HTTP y luego ver cómo el servidor web maneja estos métodos. Suponga que tanto la aplicación web como el servidor web back-end están configurados solo para aceptar solicitudes `GET` y `POST`. En ese caso, enviar una solicitud diferente hará que se muestre una página de error del servidor web, lo que no es una vulnerabilidad grave en sí misma (aparte de proporcionar una mala experiencia de usuario y potencialmente conducir a la divulgación de información). Por otro lado, si las configuraciones del servidor web no están restringidas para aceptar solo los métodos HTTP requeridos por el servidor web (p. ej . `GET`/ `POST`), y la aplicación web no está desarrollada para manejar otros tipos de solicitudes HTTP (p. ej. `HEAD`, `PUT`), entonces es posible que podamos explotar esta configuración insegura para obtener acceso a funcionalidades a las que no tenemos acceso, o incluso eludir ciertos controles de seguridad.

|**Verbo**|**Descripción**|
|:-----:|:-----:|
|`HEAD`|Idéntico a una solicitud GET, pero su respuesta solo contiene el `headers`, sin el cuerpo de la respuesta|
|`PUT`|Escribe la carga útil de la solicitud en la ubicación especificada|
|`DELETE`|Elimina el recurso en la ubicación especificada|
|`OPTIONS`|Muestra diferentes opciones aceptadas por un servidor web, como verbos HTTP aceptados|
|`PATCH`|Aplicar modificaciones parciales al recurso en la ubicación especificada|

## CAUSAS

### Configuraciones inseguras

Las configuraciones inseguras del servidor web causan el primer tipo de vulnerabilidades de manipulación de verbos HTTP. La configuración de autenticación de un servidor web puede estar limitada a métodos HTTP específicos, lo que dejaría algunos métodos HTTP accesibles sin autenticación.

```xml
<Limit GET POST>
    Require valid-user
</Limit>
```

### Codificación insegura

Las prácticas de codificación inseguras causan el otro tipo de vulnerabilidades de manipulación de verbos HTTP (aunque es posible que algunos no consideren esta manipulación de verbos). Esto puede ocurrir cuando un desarrollador web aplica filtros específicos para mitigar vulnerabilidades particulares sin cubrir todos los métodos HTTP con ese filtro.

```php
$pattern = "/^[A-Za-z\s]+$/";

if(preg_match($pattern, $_GET["code"])) {
    $query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'";
    ...SNIP...
}
```

Podemos ver que el filtro de desinfección solo se está probando en el parámetro `GET`. Si las solicitudes GET no contienen caracteres incorrectos, se ejecutará la consulta. Sin embargo, cuando se ejecuta la consulta, `$_REQUEST["code"]`se están utilizando los parámetros, que también pueden contener parámetros `POST`.

## BYPASS BASIC AUTHENTICATION

La explotación de las vulnerabilidades de manipulación de verbos HTTP suele ser un proceso relativamente sencillo. Solo necesitamos probar métodos HTTP alternativos para ver cómo los manejan el servidor web y la aplicación web.

Puede que algunas acciones de una pagina web requieran autenticacion web basica para poder completarla, necesitamos identificar qué páginas están restringidas por esta autenticación, una ves identificadas podemos capturar la peticion en intentar cambiar de metodo.

Podemos identificar que verbos admite la peticion consultando el verbo **OPTIONS**. (Si es que lo permite.)

```bash
curl -i -X OPTIONS http://SERVER_IP:PORT/

HTTP/1.1 200 OK
Date: 
Server: Apache/2.4.41 (Ubuntu)
Allow: POST,OPTIONS,HEAD,GET #metodos que admite
Content-Length: 0
Content-Type: httpd/unix-directory
```

## BYPASS FILTROS DE SEGURIDAD

Imaginemos el siguiente escenario:

> Una pagina que es vulnerable a Command Injection al crear un item mediante una peticion GET, usa proteccion de caracteres especiales y blacklist de comandos.

Si la pagina esta mal configurada, podemos probar si aplicó esas mismas protecciones en otros verbos como `HEAD` o `PUT`

>[!tip]
>Es bueno probar el metodo `HEAD` como reemplazo de un metodo `GET`, la diferenia solo sera la respuesta.

>[!tip]
>Intente cambiar los verbos:
>- de `GET` a `POST`
>- de `POST` a `GET`
>- etc.

## PREVENCION

### Codificación insegura

Se recomienda usar las variables de tipo **REQUEST** que obtiene la informacion enviada en la peticion en cualquier verbo. Esto a diferencia de usar, por ejemplo en PHP, `$_POST` o `$_GET`:

|Lenguaje|Función|
|:-------:|:------:|
|PHP|`$_REQUEST['param']`|
|Java|`request.getParameter('param')`|
|C#|`Request['param']`|

