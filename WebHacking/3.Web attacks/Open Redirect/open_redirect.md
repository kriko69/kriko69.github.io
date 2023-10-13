# OPEN REDIRECT

## Table of contents

- [OPEN REDIRECT](#open-redirect)
  - [DORKS](#dorks)
  - [EJEMPLOS](#ejemplos)
  - [PRACTICA](#practica)
  - [FUENTE](#fuente)

La vulnerabilidad de redirección abierta ocurre cuando una aplicación web acepta una entrada que no es de confianza que podría hacer que la aplicación redirija la solicitud a una URL contenida en una entrada que no es de confianza. Por ejemplo: en términos generales, implementamos algunas funciones básicas en los archivos php para redirigir a los visitantes a otra página web, pero a veces nos olvidamos de verificar la validación, lo que provoca una vulnerabilidad de **open redirect**. Pero los atacantes se aprovechan de esto y redirigen a los usuarios o visitantes a su página de phishing y roban las credenciales de los usuarios.

## DORKS

Asegúrese de verificar los siguientes parámetros de URL cuando busque errores, a menudo los verá en las páginas de inicio de sesión. Ejemplo: `/login.php?redirect=dashboard`

```bash
/login?to=
?redirect_url=
?image_url=
?url=
?rurl=
?dest=
?target=
?link=
?redirect=
?redirecturl=
?redirect_uri=
?return=
?return_to=
?returnurl=
?go=
?goto=
?exit=
?exitpage=
?fromurl=
?fromuri=
?redirect_to=
?next=
?newurl=
?redir=
```

## EJEMPLOS

podemos utilizar los links en:

- texto plano
- base64
- url encoded

```bash
# Some examples
verification-success?redirectTo=https%3a%2f%2fgoogle%2ecom%5c%2ewww%2eupwork%2ecom%2f&flowName=client_high_potential

redirect_after_login=https%3a%2f%2f%63%61%72%64s%2etwitter%2ecom%2fcards%2f18ce54su0k1%2f6tc5h

launchauth/?webbasedpurchasing=1&transid=2614806773554997295&authurl=https%3A%2F%2Fduckduckgo.com%2Forb%2Forb%3FACTION%3DDO_START%26REF%3D000000003700003159190000100001%26MAC%3D7UYgidiXPXlSwepsEkIkt2cjtzUjBN4cskq05erf%252Bhk%253D&s=5b57c3c66fea1edb71950f1b

login?u=2&to=YXdheS5waHA/dG89aHR0cDovL2FtcC5ncy9Wb3VxJnBvc3Q9LTIwNjI5NzI0XzExNjAyMDImY2Nfa2V5P #away.php?to=http://amp.gs/Vouq&post=-20629724_1160202&cc_key en base64
```

## PRACTICA

- bWAPP - Unvalidated redirects & forwards

## FUENTE

- [https://secnhack.in/open-redirection-vulnerability-exploiting-and-mitigation/](https://secnhack.in/open-redirection-vulnerability-exploiting-and-mitigation/)