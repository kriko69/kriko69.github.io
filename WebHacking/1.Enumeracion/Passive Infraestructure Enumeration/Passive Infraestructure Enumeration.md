# Passive Infraestructure Enumeration

## NETCRAFT

[Netcraft](https://www.netcraft.com/) puede ofrecernos información sobre los servidores sin siquiera interactuar con ellos, y esto es algo valioso desde el punto de vista de la recopilación pasiva de información. Podemos usar el servicio visitando `https://sitereport.netcraft.com` e ingresando el dominio de destino.

![[Pasted image 20221216215327.png]]

## WAYBACK MACHINE

[Internet Archive](https://en.wikipedia.org/wiki/Internet_Archive) es una biblioteca digital estadounidense que brinda acceso público gratuito a materiales digitalizados, incluidos sitios web, recopilados automáticamente a través de sus rastreadores web.

Podemos acceder a varias versiones de estos sitios web utilizando [Wayback Machine](http://web.archive.org/) para encontrar versiones antiguas que pueden tener comentarios interesantes en el código fuente o archivos que no deberían estar allí. Esta herramienta se puede utilizar para encontrar versiones anteriores de un sitio web en un momento dado.

![[Pasted image 20221216215511.png]]

![[Pasted image 20221216215528.png]]

### EJEMPLO

Tomemos un sitio web que ejecuta WordPress, por ejemplo. Es posible que no encontremos nada interesante mientras lo evaluamos con métodos manuales y herramientas automatizadas, por lo que lo buscamos con Wayback Machine y encontramos una versión que utiliza un complemento específico (ahora vulnerable). Volviendo a la versión actual del sitio, encontramos que el complemento no se eliminó correctamente y aún se puede acceder a través del directorio `wp-content`. Luego podemos utilizarlo para obtener la ejecución remota de código en el host y una buena recompensa.

## WAYBACKURLS

>[!note]
>Es necesario tener instalado y configurado Golang.

También podemos usar la herramienta [waybackurls](https://github.com/tomnomnom/waybackurls) para inspeccionar las URL guardadas por Wayback Machine y buscar palabras clave específicas.

```bash
go get github.com/tomnomnom/waybackurls

#obtener una lista de URL rastreadas de un dominio con la fecha en que se obtuvo
waybackurls -dates https://facebook.com > waybackurls.txt
cat waybackurls.txt
```