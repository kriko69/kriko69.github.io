## GOOGLE DORKS

###  "comllas"

Fuerza una búsqueda de coincidencia exacta. Usa esto para refinar los resultados de las búsquedas ambiguas, o para excluir los sinónimos en la búsqueda de palabras sueltas.

```
"passwords"
```

###  OR

Busca X o Y. Esto devolverá resultados relacionados con X ó Y, o ambos. Nota: El operador de barra vertical (|) puede también ser utilizado en lugar de “OR”.

```
passwords OR contraseñas

passwords | contraseñas
```

###  AND

Busca X y Y. Esto solo generará resultados relacionados con X y Y. Nota: En realidad no hay mucha diferencia con las búsquedas regulares, pues Google agrega por defecto el “Y” de todos modos. Pero es muy útil cuando se combina con otros operadores.

```
passwords AND contraseñas
```

###  - (menos)

Excluye un término o frase. En nuestro ejemplo, las páginas devueltas estarán relacionadas con jobs, pero _no_ con Apple (la empresa).

```
jobs -apple
```

###  * (asterisco)

Excluye un término o frase. En nuestro ejemplo, las páginas devueltas estarán relacionadas con jobs, pero _no_ con Apple (la empresa).

```
jobs * apple
```

### (parentesis)

Agrupa varios términos u operadores de búsqueda para controlar cómo se ejecuta la búsqueda.

```
(iPad OR iPhone) apple
```

### $

Busca precios. También funciona para el Euro (€), pero no para la Libra Esterlina (£)

```
ipad $ 230
```

### define:

Es un diccionario integrado en Google, básicamente. Esto mostrará el significado de una palabra

```
define: cracking
```

### cache:

Devuelve la versión en caché más reciente de una página web (siempre que la página esté indexada, por supuesto).

```
cache: apple.com
```

### filetype:

Restringe los resultados a un determinado tipo de archivo. Por ejemplo, PDF, DOCX, TXT, PPT, etc. Nota: El operador “ext:” puede también ser utilizado- los resultados son idénticos.

```
filetype: pdf

ext: pdf
```

### site:

Limita los resultados a los de un sitio web específico.

```
site: apple.com

site: .com.bo
```

### related:

Encuentra sitios relacionados a un determinado dominio.

```
related: apple.com
```

### intitle:

Encuentra páginas con una determinada palabra (o palabras) en el título. En nuestro ejemplo, se mostrará cualquier resultado que contenga “apple” o “iphone” en la etiqueta del título.

```
intitle: apple iphone
```

### inalltitle:

Similar a “intitle”, pero sólo se devolverán los resultados que contengan _todas_ las palabras especificadas en la etiqueta del título.

```
inalltitle: apple iphone
```

### inurl:

Encuentra páginas con una determinada palabra (o palabras) en la URL. Para este ejemplo, se mostrarán todos los resultados que contengan la palabra “apple” en la URL.

```
inurl: apple iphone
```

### allinurl:

Similar a “inurl”, pero sólo se mostrarán los resultados que contienen todas las palabras especificadas en la URL.

```
allinurl: apple iphone
```

### intext:

Encuentra páginas conteniendo una cierta palabra (o palabras) en algún lugar del contenido. Para este ejemplo, será mostrado cualquier resultado que contenga ya sea “apple” o “iphone” en el contenido de la página.

```
intext: apple iphone
```

### allintext:

Similar a “intext”, pero sólo se mostrarán los resultados que contengan _todas_ las palabras especificadas en algún lugar de la página.

```
allintext: apple iphone
```

### AROUND(X)

Es una búsqueda de proximidad. Encuentra páginas que contengan dos palabras o frases a X palabras de distancia una de la otra. Para este ejemplo, las palabras “apple” e “iphone” deben estar presentes en el contenido, y no a más de cuatro palabras de distancia.

```
apple AROUND(4) iphone
```

### weather:

Busca el clima para un lugar específico. Esto se muestra en un snippet de tiempo, pero también devuelve los resultados de otros sitios web sobre el “estado del tiempo”.

```
weather: La Paz
```

### stocks:

Muestra la información bursátil (es decir, el precio, etc.) para una empresa específica.

```
stocks: apple
```

### map:

Hace que Google muestre resultados de los mapas para una búsqueda de localización.

```
map: valle de las animas
```

### movie:

Encuentra información sobre una película específica. También encuentra los horarios si la película se muestra actualmente cerca de ti.

```
movie: apple
```

### in

Convierte una unidad a otra. Funciona con monedas, pesos, temperaturas, etc.

```
$350 in GBP
```

### source:

Encuentra resultados de noticias de una determinada fuente en Google Noticias.

```
apple source:the_verge
```

### _ (barra baja)

No es exactamente un operador de búsqueda, sino que actúa como un comodín para la función de Autocompletar de Google.

```
apple CEO_jobs
```

### #..#

Busca un rango de números. En el siguiente ejemplo, se muestran las búsquedas relacionadas con “WWDC vídeos” para los años 2010–2014, pero no para el 2015 y años posteriores.

```
wwdc vdeo 2010..2014
```

### inanchor:

Encuentra páginas que están siendo vinculadas con un anchor text específico. Para este ejemplo, se devolverá cualquier resultado con enlaces entrantes que contengan ya sea “apple” o “iphone” en el anchor text (texto ancla).

```
inanchor: apple iphone
```

### allinanchor:

Similar a “inanchor”, pero sólo se mostrarán los resultados que incluyan _todas_ las palabras especificadas en el anchor text del enlace entrante.

```
allinanchor: apple iphone
```

### blogurl:

Encuentra URLs de blog bajo un dominio específico. Este se utilizaba en la búsqueda de blogs de Google, pero he encontrado que genera algunos resultados en búsqueda normal.

```
blogurl: apple.com
```


### EJEMPLOS

```
site: .bo

intitle: admin

intext: password

site: .pe filetype: pdf

site: .bo filetype: xls

site: .pe filetype: txt
```

```
# Log files
allintext:username filetype:log

# Vulnerable web servers
inurl:/proc/self/cwd

# Open FTP servers
intitle:"index of" inurl:ftp

# ENV files
"index of" ".env"
intitle:"index of" .env

# SSH private keys
filetype:log username putty

# Email lists
filetype:xls inurl:"email.xls"
site:.edu filetype:xls inurl:"email.xls"

# Live cameras
inurl:top.htm inurl:currenttime
intitle:"webcamXP 5"
inurl:"lvappl.htm"

# MP3, Movie, and PDF files
intitle: index of mp3
intitle: index of pdf
intext: .mp4

# Weather
intitle:"Weather Wing WS-2"

# Zoom videos
inurl:zoom.us/j and intext:scheduled for

# SQL dumps
"index of" "database.sql.zip"

# WordPress Admin
intitle:"Index of" wp-admin

# Apache2
intitle:"Apache2 Ubuntu Default Page: It works"

# phpMyAdmin
"Index of" inurl:phpmyadmin

# JIRA/Kibana
inurl:Dashboard.jspa intext:"Atlassian Jira Project Management Software"
inurl:app/kibana intext:Loading Kibana

# cPanel password reset
inurl:\_cpanel/forgotpwd

# Government documents
allintitle: restricted filetype:doc site:gov

```