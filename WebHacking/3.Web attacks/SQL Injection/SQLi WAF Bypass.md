# SQLi WAF Bypass

## Table of contents

- [SQLi WAF Bypass](#sqli-waf-bypass)
  - [SQL injection with filter bypass via XML encoding](#sql-injection-with-filter-bypass-via-xml-encoding)

## SQL injection with filter bypass via XML encoding

vamos a instalar un plugins de burpsuite llamado [hackvertor](https://github.com/portswigger/hackvertor):

tenemos una consulta de stock disponible que si lo interceptamos se ve asi:

![[Pasted image 20230213223557.png]]

a parte de que podriamos intentar un **XXE injection**, estos campos puede que se esten pasando a una consulta SQL, por lo que podemos intentar una inyeccion:

![[Pasted image 20230213224430.png]]

>[!note]
>El payload no necesita **'** porque es un valor entero, `select * from product where storeId=1 union select...-- -` no es necesario cerrar comillas simples

por la respuesta puede que un WAF este protegiendo y haya detectado nuestra peticion como un intento de ataque.

con **hackvertor** podemos ofuscar cargas utiles para evadir WAF:

- seleccionamos nuestro payload
- damos click derecho
- extensions > hackvertor > encode > hex entities

![[Pasted image 20230213224725.png]]

por que seleccionamos esa opcion? pues las tecnicas de encode son buenas para la evasion del WAF, puede intentar con todas ellas y ver cual le funciona, esta funciono para el caso.

Esto transformara el payload y lograra la evasion:

![[Pasted image 20230213224907.png]]

con esto podemos obtener la contraseña del administrador:

![[Pasted image 20230213224953.png]]

