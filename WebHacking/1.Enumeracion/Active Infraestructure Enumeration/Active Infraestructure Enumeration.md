# Active Infraestructure Enumeration

## HTTP Headers

Muchas veces encontramos versiones y servicios en las caberas de respuesta de las peticiones.

También hay otras características a tener en cuenta al tomar las huellas digitales de los servidores web en los encabezados de respuesta. Estos son:

- **X-Powered-By Header:** este encabezado puede decirnos qué está usando la aplicación web. Podemos ver valores como PHP, ASP.NET, JSP, etc.
- **Cookies:** las cookies son otro valor atractivo a tener en cuenta, ya que cada tecnología por defecto tiene sus propias cookies. Algunos de los valores predeterminados de las cookies son:
	- **.NET:** `ASPSESSIONID<RANDOM>=<COOKIE_VALUE>`
	- **PHP:** `PHPSESSID=<COOKIE_VALUE>`
	- **JAVA:** `JSESSION=<COOKIE_VALUE>`

```bash
curl -I "http://${TARGET}"

#output
HTTP/1.1 200 OK
Date: Thu, 23 Sep 2021 15:10:42 GMT
Server: Apache/2.4.25 (Debian)
X-Powered-By: PHP/7.3.5
Link: <http://192.168.10.10/wp-json/>; rel="https://api.w.org/"
Content-Type: text/html; charset=UTF-8
```

## WhatWeb

```bash
whatweb -a3 https://www.facebook.com -v

#mas agresivo
whatweb -a4 https://www.facebook.com -v
```

## Wappalyzer

También podemos instalar [Wappalyzer](https://www.wappalyzer.com/) como una extensión del navegador. Tiene una funcionalidad similar a Whatweb, pero los resultados se muestran mientras se navega por la URL de destino.

![[Pasted image 20221216220532.png]]

## Wafw00f

bueno para detectar WAF:

```bash
sudo apt install wafw00f -y
wafw00f -v https://www.tesla.com
```

## Aquatone

[Aquatone](https://github.com/michenriksen/aquatone) es una herramienta para la inspección automática y visual de sitios web en muchos hosts y es conveniente para obtener rápidamente una descripción general de las superficies de ataque basadas en HTTP al escanear una lista de puertos configurables, visitar el sitio web con un navegador Chrome sin interfaz gráfica de usuario y tomar una captura de pantalla.

**instalacion**

```bash
sudo apt install golang chromium-driver
go get github.com/michenriksen/aquatone
export PATH="$PATH":"$HOME/go/bin"
```

**uso**

Podemos hacer uso de una lista de subdominios generado mediante [[Pasive Subdomain Enumeration]]:

```bash
cat facebook_aquatone.txt | aquatone -out ./aquatone -screenshot-timeout 1000
```

Cuando termine, tendremos un archivo llamado `aquatone_report.html`donde podemos ver capturas de pantalla, tecnologías identificadas, encabezados de respuesta del servidor y HTML.

![[Pasted image 20221216220927.png]]