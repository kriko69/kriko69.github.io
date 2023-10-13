## INDEX

- [Bypass authentication](#bypass-authentication)
    - [OTP bypass](#otp-bypass)
        - [Ejemplo 1](#ejemplo-1)
        - [Ejemplo 2](#ejemplo-2)
- [codigo cuando se introduce OTP valido](#codigo-cuando-se-introduce-otp-valido)
- [codigo cuando se introduce OTP valido](#codigo-cuando-se-introduce-otp-valido)
- [codigo cuando se introduce OTP invalido](#codigo-cuando-se-introduce-otp-invalido)
- [codigo cuando se introduce OTP invalido](#codigo-cuando-se-introduce-otp-invalido)
        - [Referencias](#referencias)

# Bypass authentication

## OTP bypass

OTP (One Time Password) sistemas proporcionan un mecanismo para iniciar sesión en una red o servicio utilizando **una contraseña única que sólo puede ser utilizado una sola vez**, como su nombre indica.

Normalmente en el registro de un usuario de una pagina web que contiene este tipo de mecanismo le pide un numero de celular para que pueda enviar el codigo OTP de 4 digitos y completar el registro. 

### Ejemplo 1

Para realizar un bypass de este mecanismo debemos usar un proxy web para escuchar las peticiones que se mandan a la pagina y analizar las respuestas del servidor. En este caso lo mejor es BurpSuite.

Si introducimos un codigo OTP invalido veremos la respuesta invalida, por ejemplo:

```bash
{"status":"0","message":"Incorrect OTP. Please try again."}
```

Podemos intuir que si el estado es 0 es que es un codigo OTP invalido y si es 1 es valido. El mensaje dependera del estado por lo que podemos modificar esta respuesta con BurpSuite y modificarla de la siguiente manera:

```bash
{"status":"1","message":"Correct OTP."}
```

Y reenviar la respuesta. Con esto podemos bypassear el codigo OTP.

### Ejemplo 2 

Otra manera, en caso de no ser tan intuitiva la respuesta del servidor como el anterio ejemplo, podemos introducir un codigo OTP valido en la pagina y obtener con el BurpSuite la respuesta cuando es un codigo correcto. Por ejemplo

```bash
# codigo cuando se introduce OTP valido

{"Data": {"det": {}, "block": []}, "gsc": "700"}
```

En un segundo intento colocar un OTP invalido para conocer su respuesta:

```bash
# codigo cuando se introduce OTP invalido

{"gsc":"615","message":"Invalid verification code"}
```

En este segundo intento tendriamos que editar la respuesta del servidor y colocar la respuesta cuando es un codigo valido como si lo fuera y reenviar la peticion.  Con esto podemos bypassear el codigo OTP.

### Referencias

[https://infosecwriteups.com/otp-bypass-on-indias-biggest-video-sharing-site-e94587c1aa89](https://infosecwriteups.com/otp-bypass-on-indias-biggest-video-sharing-site-e94587c1aa89)