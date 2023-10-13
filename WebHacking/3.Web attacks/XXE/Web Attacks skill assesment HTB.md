# Web Attacks skill assesment HTB

## COMBINANDO ATAQUES IDOR + HTTP VERB TAMPERING + XXE

Al interactuar con la pagina encontramos un endpoint que permite la enumeracion de cuentas de usuario:

![[Pasted image 20230125131856.png]]

como primero tenemos que escalar privilegios, vamos a mandar la peticion al intruder y vamos a ver si hay algun usuario administrador:

> Se uso un **Grep - Match** con la palabra **admin**.

y encontramos un usuario que parece ser administrador:

![[Pasted image 20230125132122.png]]

en la pagina web tambien hay un formulario para actualizar credenciales:

![[Pasted image 20230125132206.png]]

vamos a interceptar la peticion y ver como viaja:

- Primero vemos que hay una peticion que solicita un token. (Vulnerable a IDOR):

![[Pasted image 20230125132340.png]]

- Luego esta la peticion de cambio de contraseña pasando el token:

![[Pasted image 20230125132427.png]]

podemos intentar solicitar el token del usuario con **uid: 52** que es el administrador y luego cambiar su contraseña. Pero al intentar nos muestra el siguiente mensaje:

![[Pasted image 20230125132626.png]]

vemos que la peticion es un POST, intentemos cambiar el verbo y mandar nuevamente:

![[Pasted image 20230125132734.png]]

Ahora si tuvimos exito.

Vamos a entrar con la nueva cuenta:

![[Pasted image 20230125132915.png]]

vemos una nueva opcion de agregar evento:

![[Pasted image 20230125132940.png]]

los datos se envian en XML:

![[Pasted image 20230125133039.png]]

vamos a intentar realizar un ataque de XXE:

![[Pasted image 20230125133206.png]]

como es posible, vamos a leer la flag:

![[Pasted image 20230125133358.png]]


