# SSH TO RCE

- [[#LECTURA DE ID_RSA|LECTURA DE ID_RSA]]
- [[#MODIFICACION DEL AUTHORIZED_KEYS|MODIFICACION DEL AUTHORIZED_KEYS]]


## LECTURA DE ID_RSA

Es posible que con un LFI, XSS, un SQLi u otro ataque puedas leer el id_rsa de un usuario del servidor. O simplemente por FTP.

puedes conectarte a SSH con usuario y contraseña o proporcionando la llave privada. Normalmente cuando se crean las llaves ssh con el comando:

```
ssh-keygen
```

Esto crea dos llaves RSA una publica y otra privada:

- id_rsa: clave privada.
- id_rsa.pub: clave publica.

Y normalmente se puede encontrar una o ambas en las siguientes rutas:

```
/home/username/.ssh/id_rsa
~/.ssh/id_rsa
```

donde username es un usuario del sistema. (pueden comprobar en el /etc/passwd).

o buscarlo:

```bash
find / -name authorized_keys 2> /dev/null

find / -name id_rsa 2> /dev/null
```

Si lo consigues y puedes ver el contenido dice que tipo de clave es tambien:

![private key](https://raw.githubusercontent.com/kriko69/Pentesting/master/hacking_web/images%20ssh%20to%20rce/1.png)

Lo pasamos a nuestro equipo de atacante y le asignamos el siguiente permiso:

```
chmod 600 id_rsa
```

supongamos que es la id_rsa del usuario pepito, ahora nos podemos conectar al servidor usando su id_rsa **(siempre y cuando esa id_rsa se encuentre en la carpeta de authorized_keys)**:

```
ssh pepito@10.10.10.123 -i id_rsa -p 22
```

## MODIFICACION DEL AUTHORIZED_KEYS

O quizas hayas conseguido una conexion reversa con un exploit y quieras conectarte por ssh a un equipo.

Puedes generar las llaves RSA de ssh:

```
ssh-keygen
```

puede salir uno de los siguientes mensajes:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa):
```

```
/home/username/.ssh/id_rsa already exists.
Overwrite (y/n)?
```

Si elige sobrescribir la clave en el disco, ya **no** podrá autenticar usando la clave anterior. Tenga mucho cuidado al convalidar la operación, ya que este es un proceso destructivo que no puede revertirse.

A continuación, se le solicitará que introduzca una frase de contraseña para la clave. Esta es una frase de contraseña opcional que puede usarse para cifrar el archivo de clave privada en el disco. (Puede dejarlo en blanco)

```
Created directory '/home/username/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

Ahora dispondrá de una clave pública y privada que puede usar para realizar la autenticación. El siguiente paso es ubicar la clave pública en su servidor, a fin de poder utilizar la autenticación basada en la clave SSH para iniciar sesión.

```
Your identification has been saved in /home/username/.ssh/id_rsa.
Your public key has been saved in /home/username/.ssh/id_rsa.pub.
The key fingerprint is:
a9:49:2e:2a:5e:33:3e:a9:de:4e:77:11:58:b6:90:26 username@remote_host
The key's randomart image is:
+--[ RSA 2048]----+
|     ..o         |
|   E o= .        |
|    o. o         |
|        ..       |
|      ..S        |
|     o o.        |
|   =o.+.         |
|. =++..          |
|o=++.            |
+-----------------+
```

Ahora tenemos una clave publica y otra privada, vamos a crear un archivo llamado **authorized_keys**, dentro de la carpeta **.ssh** que es tambien donde estan nuestras llaves, donde vamos a agregar la clave publica. Este archivo va a dejar autorizar a todo aquel que proporcione su clave par (privada) a la hora de autenticarse.

```
touch authorized_keys

cat id_rsa.pub

echo "id_rsa.pub output" > authorized_keys
```

Y con eso deberiamos copiarnos la llave privada a nuestra maquina de atacantes, darle el permiso 600 y autenticarnos:

```
chmod 600 id_rsa

ssh -i id_rsa pepito@10.129.112.108

<enter>

<colocamos el passphrase>
```
