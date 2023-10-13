# Login Brute Force

## Table of contents

- [Login Brute Force](#login-brute-force)
  - [INTRODUCCION](#introduccion)
  - [metodos de ataque de fuerza bruta](#metodos-de-ataque-de-fuerza-bruta)
  - [DEFAULT PASSWORDS](#default-passwords)
  - [BRUTE FORCE ATTACK CON NOMBRE DE USUARIO](#brute-force-attack-con-nombre-de-usuario)
  - [CUSTOM WORDLIST](#custom-wordlist)
    - [CUPP](#cupp)
    - [USERNAME ANARCHY](#username-anarchy)

## INTRODUCCION

Muchos servidores web o contenidos individuales en los servidores web todavía se usan a menudo con el esquema [básico de autenticación HTTP .](https://tools.ietf.org/html/rfc7617) Como en nuestro caso, encontramos un servidor web con esa ruta, lo que debería despertar cierta curiosidad.

La especificación HTTP proporciona dos mecanismos de autenticación paralelos:

1.  `Basic HTTP AUTH` se utiliza para autenticar al usuario en el servidor HTTP.
2.  `Proxy Server Authentication` se utiliza para autenticar al usuario en un servidor proxy intermedio.

El esquema de autenticación HTTP básica utiliza ID de usuario y contraseña para la autenticación. El cliente envía una solicitud sin información de autenticación con su primera solicitud. La respuesta del servidor contiene el header `WWW-Authenticate`, que solicita al cliente que proporcione las credenciales.

## metodos de ataque de fuerza bruta

|**tipo**|**descripcion**|
|:-----:|:------:|
|Online brute force attack|Atacar una aplicación en vivo a través de la red, como HTTP, HTTPs, SSH, FTP y otros|
|Offline brute force attack|También conocido como descifrado de contraseñas sin conexión, en el que intenta descifrar un hash de una contraseña cifrada.|
|Inverse brute force attack|También conocido como fuerza bruta de nombre de usuario, donde prueba una única contraseña común con una lista de nombres de usuario en un determinado servicio.|
|Hibrid brute force attack|Atacar a un usuario mediante la creación de una lista de palabras de contraseña personalizada, construida utilizando inteligencia conocida sobre el usuario o el servicio.|

## DEFAULT PASSWORDS

Lo primero que podemos probar es realizar este ataque con contraseñas por defecto. Podemos usar estos diccionarios:

- [https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials)

Basic authentication:

```bash
hydra -C /opt/useful/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt 178.211.23.155 -s 31099 http-get /
```

>[!note]
>El parametro `-C` de hydra se usa solamente si el diccionario tiene palabras combinadas (user:pass)

## BRUTE FORCE ATTACK CON NOMBRE DE USUARIO

Con hydra podemos realizar un ataque pasando 2 diccionarios unos de usuarios y uno de contraseñas:

```bash
hydra -L /opt/useful/SecLists/Usernames/Names/names.txt -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt -u -f 178.35.49.134 -s 32901 http-get /
```

>[!note]
>agregaremos el indicador "-u", de modo que pruebe todos los usuarios con cada contraseña, en lugar de probar las 14 millones de contraseñas en un usuario, antes de pasar al siguiente.

>[!tip]
>Podemos indicarle `hydra` que se detenga después del primer inicio de sesión exitoso especificando la marca `-f`.

## CUSTOM WORDLIST

### CUPP

- [https://github.com/Mebus/cupp](https://github.com/Mebus/cupp)

```bash
sudo apt install cupp
```

uso:

```bash
cupp -i
```

Y como resultado, obtenemos nuestra lista de palabras de contraseña personalizada guardada como `william.txt`.

La lista de palabras de contraseñas personalizadas que generamos tiene unas aprox 43.000 líneas, podemos filtrar solo con las que cumplan la politica de contraseña de la pagina.

Politica:

1.  8 caracteres o más
2.  contiene caracteres especiales
3.  contiene números

filtrado

```bash
sed -ri '/^.{,7}$/d' william.txt            # remove shorter than 8
sed -ri '/[!-/:-@\[-`\{-~]+/!d' william.txt # remove no special chars
sed -ri '/[0-9]+/!d' william.txt            # remove no numbers
```

### USERNAME ANARCHY

- [https://github.com/urbanadventurer/username-anarchy](https://github.com/urbanadventurer/username-anarchy)

También deberíamos considerar crear una lista de palabras de nombre de usuario personalizada basada en los detalles disponibles de la persona. Por ejemplo, el nombre de usuario de la persona podría ser `b.gates` o `gates` o `bill`, y muchas otras posibles variaciones. Existen varios métodos para crear la lista de posibles nombres de usuario, el más básico de los cuales es simplemente escribirlo manualmente.

uso:

```bash
#genrea la lista de combinaciones con el nombre bill gates
./username-anarchy Bill Gates > bill.txt
```