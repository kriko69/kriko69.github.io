# Skill Assesment HTB

Interactuando con la pagina se encontro un parametro `?page=` que parece ser vulnerable a LFI, el servido soporta PHP pero el parametro carga contenido sin agregar extension, por lo que internamente se debe estar agregando una extension:

```php
include($_GET['language'] . ".php");
```

![[Pasted image 20230126123147.png]]

Se intento obtener archivo mediante path traversal pero no se tuvo ningun exito. Se evidencio que es posible leer archivos en base64:

```bash
?page=php://filter/convert.base64-encode/resource=index
```

>[!note]
>No se agrega la extension por lo anterior mencionado.

![[Pasted image 20230126123353.png]]

lo decodificamos para ver el contenido del `index.php` y vemos una ruta que no identificamos inicialmente:

![[Pasted image 20230126124615.png]]

si vamos ahi vemos un panel de logs que tiene otro parametro que parece vulnerable a LFI:

![[Pasted image 20230126124706.png]]

aqui si podemos hacer un path traversal:

![[Pasted image 20230126124738.png]]

por la respuesta del servidor, vemos que esta usando nginx:

![[Pasted image 20230126125435.png]]

para obtener un RCE vamos a abusar de su archivo de logs y el user-agent:

```bash
/var/log/nginx/access.log
```

abusamos de esto y podemos ver el nombre del archivo flag y con el LFI podemos obtener su contenido:

![[Pasted image 20230126125749.png]]
