# Broken Access Control

## Table of contents

- [Broken Access Control](#broken-access-control)
  - [INTRODUCCION](#introduccion)
  - [Controles de acceso verticales](#controles-de-acceso-verticales)
  - [Controles de acceso horizontales](#controles-de-acceso-horizontales)
  - [Controles de acceso dependientes del contexto](#controles-de-acceso-dependientes-del-contexto)
  - [Explotacion](#explotacion)
    - [Escalada de privilegios verticales (user to admin)](#escalada-de-privilegios-verticales-user-to-admin)
      - [Unprotected admin functionality](#unprotected-admin-functionality)
      - [Unprotected admin functionality with unpredictable URL](#unprotected-admin-functionality-with-unpredictable-url)
      - [Métodos de control de acceso basados ​​en parámetros](#mtodos-de-control-de-acceso-basados-en-parmetros)
        - [User role controlled by request parameter](#user-role-controlled-by-request-parameter)
        - [User role can be modified in user profile (MASS ASSIGMENT)](#user-role-can-be-modified-in-user-profile-mass-assigment)
      - [Mala configuración de la plataforma](#mala-configuracin-de-la-plataforma)
        - [El control de acceso basado en URL](#el-control-de-acceso-basado-en-url)
        - [El control de acceso basado en Metodo HTTP (Verb Tampering)](#el-control-de-acceso-basado-en-metodo-http-verb-tampering)
    - [Escalada de privilegios horizontales (user to other user)](#escalada-de-privilegios-horizontales-user-to-other-user)
      - [User ID controlled by request parameter (IDOR)](#user-id-controlled-by-request-parameter-idor)
      - [User ID controlled by request parameter, with unpredictable user IDs](#user-id-controlled-by-request-parameter-with-unpredictable-user-ids)
      - [ID de usuario controlado por parámetro de solicitud con fuga de datos en redirección](#id-de-usuario-controlado-por-parmetro-de-solicitud-con-fuga-de-datos-en-redireccin)
    - [Escalada de privilegios de horizontal a vertical](#escalada-de-privilegios-de-horizontal-a-vertical)
      - [ID de usuario controlado por parámetro de solicitud con divulgación de contraseña](#id-de-usuario-controlado-por-parmetro-de-solicitud-con-divulgacin-de-contrasea)
    - [Referencias a objetos directos inseguros (IDOR)](#referencias-a-objetos-directos-inseguros-idor)
    - [Vulnerabilidades de control de acceso en procesos de varios pasos](#vulnerabilidades-de-control-de-acceso-en-procesos-de-varios-pasos)
      - [Proceso de varios pasos sin control de acceso en un solo paso](#proceso-de-varios-pasos-sin-control-de-acceso-en-un-solo-paso)
    - [Referer-based access control](#referer-based-access-control)

## INTRODUCCION

El control de acceso (o autorización) es la aplicación de restricciones sobre quién (o qué) puede realizar intentos de acciones o acceder a los recursos que han solicitado. En el contexto de las aplicaciones web, el control de acceso depende de la autenticación y la gestión de sesiones:

- **La autenticación**: identifica al usuario y confirma que es quien dice ser.
- **La gestión de sesiones**: identifica qué solicitudes HTTP subsiguientes está realizando ese mismo usuario.
- **El control de acceso**: determina si el usuario puede realizar la acción que está intentando realizar.

## Controles de acceso verticales

Los controles de acceso vertical son mecanismos que restringen el acceso a funciones sensibles que no están disponibles para otros tipos de usuarios.

Ejemplo:

- Proteger un panel de administrador a un usuario normal.

## Controles de acceso horizontales

Los controles de acceso horizontal son mecanismos que restringen el acceso a los recursos a los usuarios que tienen permiso específico para acceder a esos recursos.

Ejemplo:

- Proteger el acceso como editor al perfil de un usuario diferente al que somos.

## Controles de acceso dependientes del contexto

Los controles de acceso dependientes del contexto evitan que un usuario realice acciones en el orden incorrecto. 

Ejemplo: 

- Un sitio web podría evitar que los usuarios modifiquen el contenido de su carrito de compras después de haber realizado el pago.

## Explotacion

### Escalada de privilegios verticales (user to admin)

#### Unprotected admin functionality

En su forma más básica, la escalada vertical de privilegios surge cuando una aplicación no impone ninguna protección sobre la funcionalidad confidencial.

A traves del archivo robots.txt podemos encontrar rutas existentes:

![[Pasted image 20230214160416.png]]

puede que esten desprotegidas y puedas realizar acciones administrativas:

![[Pasted image 20230214160502.png]]

#### Unprotected admin functionality with unpredictable URL

En algunos casos, la funcionalidad confidencial no está protegida de manera sólida, sino que se oculta dándole una URL menos predecible: la llamada **seguridad por oscuridad** (**security by obscurity**).

Por ejemplo, considere una aplicación que hospeda funciones administrativas en la siguiente URL:

```bash
https://insecure-website.com/admin-rg0yq9
```

Es posible que un atacante no pueda adivinarlo directamente. Sin embargo, la aplicación aún podría filtrar la URL a los usuarios. Por ejemplo, la URL podría divulgarse en JavaScript que construye la interfaz de usuario según el rol del usuario:

![[Pasted image 20230214161121.png]]

![[Pasted image 20230214161219.png]]

#### Métodos de control de acceso basados ​​en parámetros

Algunas aplicaciones determinan los derechos de acceso o la función del usuario al iniciar sesión y luego almacenan esta información en una ubicación controlable por el usuario, como un campo oculto, una cookie o un parámetro de cadena de consulta preestablecido. La aplicación toma decisiones de control de acceso posteriores en función del valor enviado. 

Por ejemplo:

```bash
https://insecure-website.com/login/home.jsp?admin=true
https://insecure-website.com/login/home.jsp?role=1
```

##### User role controlled by request parameter

al loguearnos y consultar un panel de administracion `/admin` vemos que la peticion viaja asi:

![[Pasted image 20230214161712.png]]

la cooike **Admin** controla el acceso, con el valor `false` recibimos un **401 Unauthorize**, pero si modificamos a `true` recibimos un **200 ok**:

![[Pasted image 20230214161941.png]]

![[Pasted image 20230214161959.png]]

>[!tip]
>Puede que en todas las acciones (requests) que necesitemos ser "admin" debamos modificar esta cookie con burpsuite.

##### User role can be modified in user profile (MASS ASSIGMENT)

Podemos ver que la pagina tiene una opcion de cambio de contraseña:

![[Pasted image 20230214163840.png]]

si la interceptamos nos muestra los atributos de la cuenta modificados:

![[Pasted image 20230214163936.png]]

actualizamos el atributo **email** de nuestro usuario, pero tiene otros como: **username, apikey, roleid**, podemos intentar mandarlos en el JSON del body para ver si  se actualizan:

![[Pasted image 20230214164113.png]]

si los actualiza y puede que ese u otro roleId tenga permisos administrativos:

![[Pasted image 20230214161959.png]]

#### Mala configuración de la plataforma

Algunas aplicaciones imponen controles de acceso en la capa de la plataforma restringiendo el acceso a URL y métodos HTTP específicos según la función del usuario.

```bash
DENY: POST, /admin/deleteUser, administrators
```

Esta regla deniega el acceso al método `POST` en la URL `/admin/deleteUser` para los usuarios del grupo de administradores.

Algunos frameworks web admiten headers HTTP no estándar que se pueden usar para anular la URL en la solicitud original, como:

- `X-Original-URL`
- `X-Rewrite-URL`

```http
POST / HTTP/1.1 

X-Original-URL: /admin/deleteUser 
...
```

>[!note]
>Estos headers se concatenan al URI consultado `X-Original-URL+URI`

```http
POST /?id=1 HTTP/1.1 

X-Original-URL: /admin/deleteUser 
...

URL = /admin/deleteUser/?id=1
```

##### El control de acceso basado en URL

Un panel de administrador se encuentra en la siguiente ruta `/admin` pero si consultamos esto nno tenemos permisos:

![[Pasted image 20230214171642.png]]

podemos intentar usar el header `X-Original-URL` para intentar evadir este control de accesos:

- Hacemos un GET hacia la raiz `/`
- En el header colocamos la ruta a la cual no tenemos accesos `/admin`

![[Pasted image 20230214171844.png]]

![[Pasted image 20230214172025.png]]

Para realizar una accion en ese pnel, como eliminar un usuario, debemos realizar la misma accion para que funcione:

```bash
#peticion original

/admin/delete/?username=carlos
```

![[Pasted image 20230214172451.png]]

##### El control de acceso basado en Metodo HTTP (Verb Tampering)

una accion de modificacion de roles por POST con la cookie de sesion de un usuario sin privilegios nos muestra un mensaje de **Unauthorize**:

![[Pasted image 20230214174802.png]]

pero si intentamos cambiar el verbo de la peticion a GET y pasar lor parametros por URL lo admite:

![[Pasted image 20230214174905.png]]

### Escalada de privilegios horizontales (user to other user)

#### User ID controlled by request parameter (IDOR)

Por ejemplo, un usuario normalmente puede acceder a su propia página de cuenta usando una URL como la siguiente:

```bash
https://0aec005803d8ec9ac078909000040034.web-security-academy.net/my-account?id=wiener
```

![[Pasted image 20230214231231.png]]

si modifica el parametro GET **id** con el nombre de un usuario existente puede ver la informacion del otro usuario:

![[Pasted image 20230214231339.png]]

#### User ID controlled by request parameter, with unpredictable user IDs

En algunas aplicaciones, el parámetro explotable no tiene un valor predecible. Por ejemplo, en lugar de un número creciente, una aplicación podría usar identificadores únicos globales (GUID) para identificar a los usuarios. Es posible que un atacante no pueda adivinar el identificador de otro usuario. Sin embargo, los GUID que pertenecen a otros usuarios pueden divulgarse en otra parte de la aplicación donde se hace referencia a los usuarios, **como mensajes de usuario o reseñas.**

una pagina de blogs tiene publicaciones hachas por diferentes usuarios:

![[Pasted image 20230214232329.png]]

al revisar cada uno de ellos podemos ver el autor de la publicacion y a nivel de codigo fuente ver su identificar (GUID):

![[Pasted image 20230214232417.png]]

![[Pasted image 20230214232448.png]]

al ir a nuestro perfil vemos un parametro que es nuestro GUID:

![[Pasted image 20230214232549.png]]

podemos reemplazar el que encontramos en el blog y ver informacion de otra persona:

![[Pasted image 20230214232704.png]]

#### ID de usuario controlado por parámetro de solicitud con fuga de datos en redirección

En algunos casos, una aplicación detecta cuándo el usuario no tiene permiso para acceder al recurso y devuelve una redirección a la página de inicio de sesión. Sin embargo, la respuesta que contiene la redirección aún puede incluir algunos datos confidenciales que pertenecen al usuario objetivo, por lo que el ataque aún tiene éxito.

al querer acceder a una cuenta que no tenemos acceso, en la redireccion al login puede haber un body con informacion sensible:

![[Pasted image 20230214233104.png]]

### Escalada de privilegios de horizontal a vertical

Esta es una combinacion de una tecnica horizontal mas una tecnica vertical.

Un ataque de escalada de privilegios horizontal puede convertirse en una escalada de privilegios vertical, al comprometer a un usuario con más privilegios. 

Por ejemplo:

una escalada horizontal podría permitir que un atacante restablezca o capture la contraseña que pertenece a otro usuario. Si el atacante se dirige a un usuario administrativo y compromete su cuenta, entonces puede obtener acceso administrativo y así realizar una escalada de privilegios verticales.

#### ID de usuario controlado por parámetro de solicitud con divulgación de contraseña

puede que a traves de un IDOR podamos acceder a la informacion de una cuenta de administrador:

![[Pasted image 20230214235154.png]]
si cambiamos el ID por el nombre de un usuario administrador:

![[Pasted image 20230214235320.png]]

hay un campo prellenado con la contraseña del usuario, esta enmascarado por un input de tipo **password** pero para verlo solo hay que manipular el HTML, con eso podemos iniciar sesion con la cuenta de administrador y escalar privilegios.

### Referencias a objetos directos inseguros (IDOR)

Las referencias directas a objetos inseguros (IDOR) son una subcategoría de vulnerabilidades de control de acceso. IDOR surge cuando una aplicación utiliza la entrada proporcionada por el usuario para acceder a los objetos directamente y un atacante puede modificar la entrada para obtener acceso no autorizado.

por ejemplo, un historial de chat carga los mensajes desde un archivo de texto:

![[Pasted image 20230215000046.png]]

el nombre del archivo para ser la referencia a un objeto y es predecible, podemos llamar a uno anterior y ver otro chat:

![[Pasted image 20230215000154.png]]

### Vulnerabilidades de control de acceso en procesos de varios pasos

Muchos sitios web implementan funciones importantes en una serie de pasos. Esto se hace a menudo cuando es necesario capturar una variedad de entradas u opciones, o cuando el usuario necesita revisar y confirmar los detalles antes de realizar la acción. Por ejemplo, la función administrativa para actualizar los detalles del usuario podría implicar los siguientes pasos:

1.  Cargue el formulario que contiene detalles para un usuario específico.
2.  Presentar cambios.
3.  Revise los cambios y confirme.

A veces, un sitio web implementará controles de acceso rigurosos sobre algunos de estos pasos, pero ignorará otros.

#### Proceso de varios pasos sin control de acceso en un solo paso

existe una funcion administrativa (realizada por un usuario administrador) para asignar roles a usuarios, el proceso tiene 2 pasos:

1. eleccion de role en usuario

![[Pasted image 20230215001144.png]]

![[Pasted image 20230215001250.png]]

2. confirmacion de la accion

![[Pasted image 20230215001318.png]]

![[Pasted image 20230215001419.png]]

en este ultimo paso, si nos loguearamos como un usuario normal y copiamos su **cookie de sesion** y se asigna a si mismo (**en el username**) lo deja realizar sin problemas:

![[Pasted image 20230215001730.png]]

### Referer-based access control

Algunos sitios web basan los controles de acceso en el header `Referer` enviado en la solicitud HTTP. El header `Referer` generalmente se agrega a las solicitudes de los navegadores para indicar la página desde la que se inició una solicitud.

Por ejemplo:

Supongamos que una aplicación impone un control de acceso sólido sobre la página administrativa principal en `/admin`, pero para subpáginas como `/admin/deleteUser` solo inspecciona el header `Referer`. Si el header `Referer` contiene la URL principal `/admin`, se permite la solicitud.

![[Pasted image 20230215003607.png]]

![[Pasted image 20230215003513.png]]


