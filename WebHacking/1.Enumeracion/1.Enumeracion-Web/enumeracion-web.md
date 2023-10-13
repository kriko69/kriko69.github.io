# ENUMERACION WEB

## Table Obsidian

- [[#Table of contents|Table of contents]]
- [[#ENUMERACION DE TECNOLOGIAS|ENUMERACION DE TECNOLOGIAS]]
	- [[#Wappalyzer|Wappalyzer]]
	- [[#NETCRAFT|NETCRAFT]]
	- [[#Consultar el metodo HTTP OPTIONS y TRACE|Consultar el metodo HTTP OPTIONS y TRACE]]
	- [[#HEADERS|HEADERS]]
	- [[#LinkedIn|LinkedIn]]
- [[#GITHUB OSINT|GITHUB OSINT]]
- [[#ENUMERACION DE WAF|ENUMERACION DE WAF]]
	- [[#Wafw00f|Wafw00f]]
- [[#ENUMERACION DE RECURSOS EN LA NUBE|ENUMERACION DE RECURSOS EN LA NUBE]]
	- [[#ENUMERACION DE SERVIDORES DE UNA EMPRESA|ENUMERACION DE SERVIDORES DE UNA EMPRESA]]
	- [[#GOOGLE DORKS|GOOGLE DORKS]]
		- [[#AWS|AWS]]
		- [[#AZURE|AZURE]]
	- [[#DOMAIN.GLASS|DOMAIN.GLASS]]
	- [[#GRAYHAT WARFARE|GRAYHAT WARFARE]]
- [[#ENUMERACION DE HEADERS|ENUMERACION DE HEADERS]]
	- [[#Security Headers|Security Headers]]
	- [[#HTTP Headers|HTTP Headers]]
- [[#ENUMERACION DE PARAMETROS|ENUMERACION DE PARAMETROS]]
	- [[#wfuzz|wfuzz]]
	- [[#ParamSpider|ParamSpider]]
	- [[#gospider|gospider]]
	- [[#Arjun|Arjun]]
- [[#ENUMERACION DE HISTORIAL DE PAGINAS WEB|ENUMERACION DE HISTORIAL DE PAGINAS WEB]]
	- [[#WAYBACK MACHINE|WAYBACK MACHINE]]
		- [[#EJEMPLO|EJEMPLO]]
	- [[#WAYBACKURLS|WAYBACKURLS]]
- [[#ENUMERACION DE SUBDOMINIOS|ENUMERACION DE SUBDOMINIOS]]
	- [[#DICCIONARIOS|DICCIONARIOS]]
	- [[#enumeracion pasiva mediante links (OSINT)|enumeracion pasiva mediante links (OSINT)]]
	- [[#shodan|shodan]]
	- [[#Content Security Policy (CSP)|Content Security Policy (CSP)]]
	- [[#SSL Certificate|SSL Certificate]]
	- [[#DNSdumpster|DNSdumpster]]
	- [[#Certificate transparency name extractor (CT.py)|Certificate transparency name extractor (CT.py)]]
	- [[#DNS resolver|DNS resolver]]
	- [[#VIRUS TOTAL|VIRUS TOTAL]]
	- [[#Censys|Censys]]
	- [[#Transferencia de zona|Transferencia de zona]]
	- [[#DIG|DIG]]
	- [[#NSLOOKUP|NSLOOKUP]]
		- [[#DNS RECORDS|DNS RECORDS]]
	- [[#DNSRecon|DNSRecon]]
	- [[#Descubriendo el espacio IP (ASN)|Descubriendo el espacio IP (ASN)]]
	- [[#MURMURHASH (BUSQUEDA POR FAVICON.ICO)|MURMURHASH (BUSQUEDA POR FAVICON.ICO)]]
	- [[#google dorks|google dorks]]
	- [[#Archivos bien conocidos|Archivos bien conocidos]]
	- [[#AMASS|AMASS]]
	- [[#SubDomainizer|SubDomainizer]]
	- [[#gobuster|gobuster]]
	- [[#wfuzz|wfuzz]]
	- [[#Feroxbuster|Feroxbuster]]
	- [[#subbrute|subbrute]]
	- [[#CRT.SH|CRT.SH]]
	- [[#OBTENER LAS IP DE LOS SERVIDORES DE DOMINIO|OBTENER LAS IP DE LOS SERVIDORES DE DOMINIO]]
		- [[#COMBINANDO CON SHODAN PARA MAS INFORMACION|COMBINANDO CON SHODAN PARA MAS INFORMACION]]
	- [[#THE HARVESTER|THE HARVESTER]]
	- [[#knock|knock]]
	- [[#subfinder|subfinder]]
	- [[#assetfinder|assetfinder]]
	- [[#bbot|bbot]]
	- [[#ReconFTW|ReconFTW]]
	- [[#aquatone|aquatone]]
- [[#VALIDACION DE DOMINIOS WEB|VALIDACION DE DOMINIOS WEB]]
	- [[#HTTPROBE & HTTPX|HTTPROBE & HTTPX]]
	- [[#Httpx + subfinder|Httpx + subfinder]]
- [[#ENUMERACION DE USUARIOS|ENUMERACION DE USUARIOS]]
	- [[#linkedin2username|linkedin2username]]
- [[#ENUMERACION DE PUERTOS WEB|ENUMERACION DE PUERTOS WEB]]
	- [[#masscan|masscan]]
	- [[#nmap|nmap]]
- [[#WHOIS|WHOIS]]
- [[#Subdomain takeover|Subdomain takeover]]
	- [[#subjack|subjack]]
	- [[#takeover|takeover]]
- [[#Metodologia analisis web externo|Metodologia analisis web externo]]
- [[#FUENTE|FUENTE]]


## Table of contents

- [ENUMERACION WEB](#enumeracion-web)
  - [ENUMERACION DE TECNOLOGIAS](#enumeracion-de-tecnologias)
    - [Wappalyzer](#wappalyzer)
    - [NETCRAFT](#netcraft)
    - [Consultar el metodo HTTP OPTIONS y TRACE](#consultar-el-metodo-http-options-y-trace)
  - [ENUMERACION DE WAF](#enumeracion-de-waf)
    - [Wafw00f](#wafw00f)
  - [ENUMERACION DE HEADERS](#enumeracion-de-headers)
    - [Security Headers](#security-headers)
    - [HTTP Headers](#http-headers)
  - [ENUMERACION DE PARAMETROS](#enumeracion-de-parametros)
    - [wfuzz](#wfuzz)
    - [ParamSpider](#paramspider)
    - [gospider](#gospider)
    - [Arjun](#arjun)
  - [ENUMERACION DE HISTORIAL DE PAGINAS WEB](#enumeracion-de-historial-de-paginas-web)
    - [WAYBACK MACHINE](#wayback-machine)
      - [EJEMPLO](#ejemplo)
    - [WAYBACKURLS](#waybackurls)
  - [ENUMERACION DE SUBDOMINIOS](#enumeracion-de-subdominios)
    - [DICCIONARIOS](#diccionarios)
    - [enumeracion pasiva mediante links (OSINT)](#enumeracion-pasiva-mediante-links-osint)
    - [shodan](#shodan)
    - [Content Security Policy (CSP)](#content-security-policy-csp)
    - [DNSdumpster](#dnsdumpster)
    - [Certificate transparency name extractor (CT.py)](#certificate-transparency-name-extractor-ctpy)
    - [DNS resolver](#dns-resolver)
    - [VIRUS TOTAL](#virus-total)
    - [Censys](#censys)
    - [Transferencia de zona](#transferencia-de-zona)
    - [DIG](#dig)
    - [NSLOOKUP](#nslookup)
      - [DNS RECORDS](#dns-records)
    - [DNSRecon](#dnsrecon)
    - [Descubriendo el espacio IP (ASN)](#descubriendo-el-espacio-ip-asn)
    - [MURMURHASH (BUSQUEDA POR FAVICON.ICO)](#murmurhash-busqueda-por-faviconico)
    - [google dorks](#google-dorks)
    - [Archivos bien conocidos](#archivos-bien-conocidos)
    - [AMASS](#amass)
    - [gobuster](#gobuster)
    - [wfuzz](#wfuzz)
    - [Feroxbuster](#feroxbuster)
    - [subbrute](#subbrute)
    - [CRT.SH](#crtsh)
    - [THE HARVESTER](#the-harvester)
    - [knock](#knock)
    - [subfinder](#subfinder)
    - [assetfinder](#assetfinder)
    - [ReconFTW](#reconftw)
    - [aquatone](#aquatone)
  - [VALIDACION DE DOMINIOS WEB](#validacion-de-dominios-web)
    - [HTTPROBE & HTTPX](#httprobe--httpx)
    - [Httpx + subfinder](#httpx--subfinder)
  - [ENUMERACION DE USUARIOS](#enumeracion-de-usuarios)
    - [linkedin2username](#linkedin2username)
  - [ENUMERACION DE PUERTOS WEB](#enumeracion-de-puertos-web)
    - [masscan](#masscan)
    - [nmap](#nmap)
  - [WHOIS](#whois)
  - [Metodologia analisis web externo](#metodologia-analisis-web-externo)
  - [FUENTE](#fuente)

## ENUMERACION DE TECNOLOGIAS

### Wappalyzer

También podemos instalar [Wappalyzer](https://www.wappalyzer.com/) como una extensión del navegador. Tiene una funcionalidad similar a Whatweb, pero los resultados se muestran mientras se navega por la URL de destino.

![[Pasted image 20221216220532.png]]

### NETCRAFT

[Netcraft](https://www.netcraft.com/) puede ofrecernos información sobre los servidores sin siquiera interactuar con ellos, y esto es algo valioso desde el punto de vista de la recopilación pasiva de información. Podemos usar el servicio visitando `https://sitereport.netcraft.com` e ingresando el dominio de destino.

![[Pasted image 20221216215327.png]]

- [https://searchdns.netcraft.com/](https://searchdns.netcraft.com/)

### Consultar el metodo HTTP OPTIONS y TRACE

```bash
curl -X OPTIONS http://10.10.1.10

curl -X TRACE http://10.10.1.10
```

### HEADERS

- x-powered-by
- server

### LinkedIn

- Ofertas de trabajo de la empresa, pueden revelar que tecnologias usan segun los criterios de busqueda de talento.
- Empleados de la empresa (sobre todo desarrolladores) pueden tener sus repositorios Github o Gitlab publicos y pueden tener informacion de la empresa.

## GITHUB OSINT

- [https://medium.com/@cuncis/leveraging-github-for-open-source-intelligence-osint-tools-and-techniques-f63fb2a1066](https://medium.com/@cuncis/leveraging-github-for-open-source-intelligence-osint-tools-and-techniques-f63fb2a1066)

## ENUMERACION DE WAF

### Wafw00f

bueno para detectar WAF:

```bash
sudo apt install wafw00f -y
wafw00f -v https://www.tesla.com
```

## ENUMERACION DE RECURSOS EN LA NUBE

El uso de la nube, como **AWS** , **GCP** , **Azure** y otros, es ahora uno de los componentes esenciales para muchas empresas en la actualidad. Al fin y al cabo, todas las empresas quieren poder hacer su trabajo desde cualquier lugar, por lo que necesitan un punto central para toda la gestión. Es por eso que los servicios de **Amazon( AWS)**, **Google( GCP)** y **Microsoft( Azure)** son ideales para este propósito.

Esto a menudo comienza con `S3 buckets` (AWS), `blobs` (Azure) y `cloud storage` (GCP), al que se puede acceder sin autenticación si se configura incorrectamente.

### ENUMERACION DE SERVIDORES DE UNA EMPRESA

```BASH
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```

Esta es una simple consulta de direcciones IP de una lista de subdominios de una empresa, puede que alguno tenga una nomenclatura propia de alguna empresa cloud como AWS:

![[Pasted image 20230619230029.png]]

### GOOGLE DORKS

#### AWS

```bash
intext:nombre_empresa inurl:amazonaws.com
```

![[Pasted image 20230619230253.png]]

#### AZURE

```bash
intext:nombre_empresa inurl:blob.core.windows.net
```

![[Pasted image 20230619230548.png]]

**Dicho contenido también suele incluirse en el código fuente de las páginas web, desde donde se cargan las imágenes, los códigos JavaScript o CSS. Este procedimiento a menudo libera al servidor web y no almacena contenido innecesario.**

![[Pasted image 20230619230706.png]]

### DOMAIN.GLASS

Una vez que encontramos un dominio cloud podemos consultarlo a [https://domain.glass/](https://domain.glass/) y nos retornara mucha informacion.

### GRAYHAT WARFARE

- [https://buckets.grayhatwarfare.com/](https://buckets.grayhatwarfare.com/)

**_Puedes enumerar buckets o files almacenados en buckets bajo un cierto criterio de busqueda (nombre de la empresa)_**

Necesitas crearte una cuenta.

![[Pasted image 20230619231532.png]]

## ENUMERACION DE HEADERS

### Security Headers

```bash
pip3 install shcheck

shcheck.py https://www.example.com/ -i
```

```bash
 -p PORT, --port=PORT -> definir un puerto en especifico por defecto 80
 -c COOKIE_STRING, --cookie=COOKIE_STRING -> definir una cookie                    
 -a HEADER_STRING, --add-header=HEADER_STRING -> agregar una cabecera -> 'Header: value'
 -d, --disable-ssl-check -> deshabilita la verificacion del certificado SSL
 -g, --use-get-method  usar metodo GET en lugar de metodo HEAD
 -j, --json-output     imprimir salida en formato json
 -i, --information     mostrar informacion de la cabecersa
 -x, --caching         Display caching headers
 --proxy=PROXY_URL     definir un proxy (Ex: http://127.0.0.1:8080)
 --hfile=PATH_TO_FILE  cargar una lista de hosts
```

### HTTP Headers

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

## ENUMERACION DE PARAMETROS

diccionario: [https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/burp-parameter-names.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/burp-parameter-names.txt)

### wfuzz

```bash
wfuzz -c -t 100 --hw=854 --hc=400,404 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u "http://10.10.182.126/api/items?FUZZ=hola" -X POST #glitch THM
```

### ParamSpider

```bash
git clone https://github.com/devanshbatham/ParamSpider
cd ParamSpider
pip3 install -r requirements.txt
```

uso

```bash
python3 paramspider.py --domain herbalife.com.bo --exclude woff,css,js,png,svg,php,jpg

# prueba varios parametros: https://hackerone.com/test.php?p=test&q=FUZZ
python3 paramspider.py --domain hackerone.com --level high
```

>[!tip]
>El `exclude` es para que no busque en archivos css, png, jpg, etc...

![[Pasted image 20230602004734.png]]

### gospider

```bash
apt install gospider
```

uso:

```bash
gospider -s "https://google.com/" -o output -c 10 -d 1

# Also get URLs from 3rd party (Archive.org, CommonCrawl.org, VirusTotal.com, AlienVault.com)
gospider -s "https://google.com/" -o output -c 10 -d 1 --other-source

#gospider blacklisted .(jpg|jpeg|gif|css|tif|tiff|png|ttf|woff|woff2|ico) as default
gospider -s "https://google.com/" -o output -c 10 -d 1 --blacklist ".(woff|pdf)"
```

![[Pasted image 20230602010211.png]]

### Arjun

Herramienta para descubrir parametros ocultos en endpoints:

- [https://github.com/s0md3v/Arjun](https://github.com/s0md3v/Arjun)

```bash
apt install arjun
```

uso:

```bash
#escaneo a un unico target
arjun -u https://api.example.com/endpoint

#especificando el metodo
arjun -u https://api.example.com/endpoint -m POST
```

exportar resultado;

```bash
-oJ result.json
-oT result.txt
-oB 127.0.0.1:8080
```

## ENUMERACION DE HISTORIAL DE PAGINAS WEB

### WAYBACK MACHINE

[Internet Archive](https://en.wikipedia.org/wiki/Internet_Archive) es una biblioteca digital estadounidense que brinda acceso público gratuito a materiales digitalizados, incluidos sitios web, recopilados automáticamente a través de sus rastreadores web.

Podemos acceder a varias versiones de estos sitios web utilizando [Wayback Machine](http://web.archive.org/) para encontrar versiones antiguas que pueden tener comentarios interesantes en el código fuente o archivos que no deberían estar allí. Esta herramienta se puede utilizar para encontrar versiones anteriores de un sitio web en un momento dado.

![[Pasted image 20221216215511.png]]

![[Pasted image 20221216215528.png]]

#### EJEMPLO

Tomemos un sitio web que ejecuta WordPress, por ejemplo. Es posible que no encontremos nada interesante mientras lo evaluamos con métodos manuales y herramientas automatizadas, por lo que lo buscamos con Wayback Machine y encontramos una versión que utiliza un complemento específico (ahora vulnerable). Volviendo a la versión actual del sitio, encontramos que el complemento no se eliminó correctamente y aún se puede acceder a través del directorio `wp-content`. Luego podemos utilizarlo para obtener la ejecución remota de código en el host y una buena recompensa.

### WAYBACKURLS

>[!note]
>Es necesario tener instalado y configurado Golang.

También podemos usar la herramienta [waybackurls](https://github.com/tomnomnom/waybackurls) para inspeccionar las URL guardadas por Wayback Machine y buscar palabras clave específicas.

```bash
wget https://github.com/tomnomnom/waybackurls/releases/download/v0.1.0/waybackurls-linux-amd64-0.1.0.tgz

tar -xvf waybackurls-linux-amd64-0.1.0.tgz 

#obtener una lista de URL rastreadas de un dominio con la fecha en que se obtuvo
waybackurls -dates https://facebook.com > waybackurls.txt
cat waybackurls.txt
```

![[Pasted image 20230601234835.png]]

## ENUMERACION DE SUBDOMINIOS

### DICCIONARIOS

Puedes usar los de seclist que no estan mal.

https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS

o puedes crear uno en base a lsa palabras que etnga la pagina web a auditar:

```bash
cewl -d 3 -m 2 -w dict.txt https://example.com
```

* -d: profundidad en la busqueda de palabras, mas profundo mas especifico en la recoleccion.
* -m: minima longitud de cada palabra.
*  -w: para exportar la salida.

puedes recuperar correos de una pagina web:

```bash
cewl https://esgeeks.com/ -n -e
```

* -e: para correos.
* -n: para no generar una lista de palabras.

https://esgeeks.com/como-utilizar-cewl/

### enumeracion pasiva mediante links (OSINT)

Esta es un listado de paginas que puedes usar para encontrar informacion de un dominio en especifico.

* [https://spyse.com/tools/dns-lookup](https://spyse.com/tools/dns-lookup)
* [https://centralops.net/co/DomainDossier.aspx](https://centralops.net/co/DomainDossier.aspx)
* [https://whois.domaintools.com/](https://whois.domaintools.com/)
* [https://domainbigdata.com/bdp.com.bo](https://domainbigdata.com/bdp.com.bo)
* [https://search.censys.io/](https://search.censys.io/)
* [https://crt.sh/?](https://crt.sh/?) busqueda a traves de certificados SSL
* [https://securitytrails.com/](https://securitytrails.com/)
* [https://dnsdumpster.com/](https://dnsdumpster.com/)
* [https://www.virustotal.com/gui/home/search](https://www.virustotal.com/gui/home/search)
* [https://searchdns.netcraft.com/](https://searchdns.netcraft.com/)
* [https://www.nmmapper.com/](https://www.nmmapper.com/)
* [https://suip.biz/?act=subfinder](https://suip.biz/?act=subfinder)

**IP Address o Dominio**

* [https://whois.domaintools.com/](https://whois.domaintools.com/)

**Tecnologias de una web**

* [https://sitereport.netcraft.com/](https://sitereport.netcraft.com/)
  
**DNS information**

* [https://www.robtex.com](https://www.robtex.com)

### shodan

desde la terminal:

se necesita python

```bash
#instalacion
pip install -U --user shodan
pip3 install -U --user shodan

#verificacion
shodan

#agregar api key
shodan init YOUR_API_KEY

#enumerar subdominios
shodan domain xyz.com
```

[api de shodan](https://help.shodan.io/command-line-interface/0-installation)

puedes automatizar todo esto mediante un script en bash y jugar con las API o curl para obtener la informacion.

### Content Security Policy (CSP)

La política de seguridad de contenido (CSP) define el header HTTP `Content-Security-Policy`, que le permite crear una lista blanca de fuentes de contenido confiable e indica al navegador que solo ejecute o represente recursos de esas fuentes. Enumerará un montón de fuentes (dominios) que podrían ser de interés para nosotros como atacantes. Hay formas obsoletas de header CSP, `X-Content-Security-Policy` y `X-Webkit-CSP`.

```bash
curl -s -I -L "https://www.herbalife.com.bo/" | grep -Ei '^Content-Security-Policy:' | sed "s/;/;\\n/g"

curl -s -I -L "https://www.herbalife.com.bo/" | grep -Ei '^Content-Security-Policy:' | sed "s/;/;\\n/g"  tr " " "\n"
```

![[Pasted image 20230603002756.png]]

### SSL Certificate

El primer punto de presencia en Internet puede ser el `SSL certificate` de la web principal de la empresa que podemos examinar. A menudo, dicho certificado incluye más que solo un subdominio, y esto significa que el certificado se usa para varios dominios, y estos probablemente aún estén activos.

![[Pasted image 20230612222700.png]]



### DNSdumpster

- [https://dnsdumpster.com/](https://dnsdumpster.com/)

![[Pasted image 20230603000106.png]]

da mucha mas informacion.

### Certificate transparency name extractor (CT.py)

```bash
wget https://raw.githubusercontent.com/blechschmidt/massdns/master/scripts/ct.py
```

```bash
python ct.py quimiza.com
```

![[Pasted image 20230603000613.png]]

### DNS resolver

podemos usan **massdns** para intentar resolver (encontrar) la IP de los subdominios:

**ct.py + massdns**:

```bash
apt install massdns

wget https://raw.githubusercontent.com/blechschmidt/massdns/master/lists/resolvers.txt

python ct.py quimiza.com | massdns -r resolvers.txt -t A -q
```

![[Pasted image 20230603001328.png]]

### VIRUS TOTAL

VirusTotal mantiene su servicio de replicación de DNS, que se desarrolla conservando las resoluciones de DNS realizadas cuando los usuarios visitan las URL proporcionadas por ellos. Para recibir información sobre un dominio, escriba el nombre de dominio en la barra de búsqueda y haga clic en la pestaña "Relaciones".

![[Pasted image 20221216161210.png]]

### Censys

Otra fuente de información interesante que podemos utilizar para extraer subdominios son los certificados SSL/TLS. La razón principal es Certificate Transparency (CT), un proyecto que requiere que cada certificado SSL/TLS emitido por una Autoridad de certificación (CA) se publique en un registro de acceso público.

-   [https://censys.io](https://censys.io/)

Podemos navegar a https://search.censys.io/certificates e introducir el nombre de dominio de nuestra organización objetivo para empezar a descubrir nuevos subdominios.

![[Pasted image 20221216162103.png]]


### Transferencia de zona

La transferencia de zona es cómo un servidor DNS secundario recibe información del servidor DNS primario y la actualiza. El enfoque maestro-esclavo se usa para organizar servidores DNS dentro de un dominio, y los esclavos reciben información DNS actualizada del DNS maestro. El servidor DNS maestro debe configurarse para permitir transferencias de zona desde servidores DNS secundarios (esclavos), aunque esto podría estar mal configurado.


- [https://hackertarget.com/zone-transfer/](https://hackertarget.com/zone-transfer/)

![[Pasted image 20221216222608.png]]

### DIG

obtener nameserver:

```bash
dig ns dominio @IP

#si es una pagina publica en internet
#puedes probar el DNS publico de google u otro que se sepa que resuelva la pagina
dig ns inlanefright.htb @8.8.8.8

#se coloca la IP de la maquina en caso de estar dentro de una red privada
dig ns inlanefright.htb @10.129.33.176
```

![[Pasted image 20221216230456.png]]

realizamos transferencia de zona:

```bash
dig axfr dominio @nameserver

#en caso de red privada es necesario configurar 
#el dominio y el namserver en el /etc/hosts
dig axfr inlanefright.htb @10.129.33.176

#caso contrario y si se tiene salida a internet
dig axfr google.com @ns1.google.com.

dig +multi AXFR @ns1.insecuredns.com insecuredns.com
```

![[Pasted image 20221216230922.png]]

usando dig:

```bash
dig @bdp.com.bo axfr
```

puede ir la direccion IP en lugar del nombre de dominio.

### NSLOOKUP


1. Identificacion de Nameservers

```bash
nslookup -type=NS zonetransfer.me

#output
Server:		10.100.0.1
Address:	10.100.0.1#53

Non-authoritative answer:
zonetransfer.me	nameserver = nsztm2.digi.ninja. #nameserver 1
zonetransfer.me	nameserver = nsztm1.digi.ninja. #nameserver 1
```

2. Prueba de transferencia de zona ANY y AXFR

Esto por cada nameserver

```bash
nslookup -type=any -query=AXFR zonetransfer.me nsztm2.digi.ninja
nslookup -type=any -query=AXFR zonetransfer.me nsztm1.digi.ninja
```

>[!note]
>Si logramos realizar una transferencia de zona exitosa para un dominio, no es necesario continuar enumerando este dominio en particular, ya que esto extraerá toda la información disponible.

otros ejemplos:

```bash
nslookup www.facebook.com

#consultando registros A
nslookup -query=A www.facebook.com

#consultando registros PTR
nslookup -query=PTR 31.13.92.36

#consultando cualquier registro
nslookup -query=ANY www.facebook.com
```

```bash
nslookup www.facebook.com

#obtener¡mos la IP del comando anterior y consultamos con whois
whois 157.240.199.35
```

```bash
dig www.facebook.com @1.1.1.1

#consultando registros A
dig a www.facebook.com @1.1.1.1

#consultando registros PTR
dig -x 31.13.92.36 @1.1.1.1

#consultando cualquier registro
dig any google.com @8.8.8.8
dig any cloudflare.com @8.8.8.8
```

#### DNS RECORDS

- [https://www.cloudflare.com/es-es/learning/dns/dns-records/](https://www.cloudflare.com/es-es/learning/dns/dns-records/)

### DNSRecon

>[!note]
>Ya esta instalado en Kali.

```bash
dnsrecon -t brt -d herbalife.com.bo
```

![[Pasted image 20230602232021.png]]

si conocemos el nameserver:

encontrar nameserver: [[#NSLOOKUP]]

```bash
dnsrecon -t brt -d herbalife.com.bo -n nameserver
```

### Descubriendo el espacio IP (ASN)

**ASN** (Número de sistema autónomo) es un identificador único de ciertos prefijos de IP. Organizaciones muy grandes como Apple, Github, Tesla tienen su propio espacio IP significativo.

buscamos aqui: [https://bgp.he.net/](https://bgp.he.net/)

![[Pasted image 20230602232422.png]]

Ahora que hemos averiguado el número de ASN, lo siguiente que necesitamos es averiguar los rangos de IP dentro de ese ASN.

```bash
whois -h whois.radb.net  -- '-i origin AS6568' | grep -Eo "([0-9.]+){4}/[0-9]+" | uniq
```

![[Pasted image 20230602232547.png]]

otra forma seria con nmap:

```bash
nmap --script targets-asn --script-args targets-asn.asn=6568
```

Ahora que conocemos los rangos de direcciones IP desde el ASN de una organización, podemos realizar consultas PTR en las direcciones IP y verificar si hay hosts válidos. (**Registros PTR - DNS inverso**)

**¿Qué es el DNS inverso?**

Cuando un usuario intenta llegar a un nombre de dominio en su navegador, se produce una búsqueda de DNS que hace coincidir el nombre de dominio (ejemplo.com) con la dirección IP (como 192.168.1.1). Una búsqueda inversa de DNS es lo opuesto a este proceso: es una consulta que comienza con la dirección IP y busca el nombre de dominio.

Esto significa que, dado que ya conocemos el espacio IP de una organización, podemos realizar una consulta inversa de las direcciones IP y encontrar los dominios válidos.

**ENTEL** tiene el ASN **ASN6568** que representa el rango de IP **181.115.189.0/24.** (enter otros rangos) Entonces, veamos que hay que realizar el DNS inverso.

PRIMERO INSTALAMOS:

- mapcidr
- dnsx

```bash
wget https://github.com/projectdiscovery/mapcidr/releases/download/v1.1.2/mapcidr_1.1.2_linux_amd64.zip

unzip mapcidr_1.1.2_linux_amd64.zip
cp mapcidr /usr/bin/mapcidr

apt install dnsx
```

```bash
#Yse guarda en output.txt
echo 181.115.189.0/24 | mapcidr -silent | dnsx -ptr -resp-only -o output.txt
```

![[Pasted image 20230602234050.png]]

podemos hacer todo con un oneliner:

```bash
whois -h whois.radb.net  -- '-i origin AS6568' | grep -Eo "([0-9.]+){4}/[0-9]+" | uniq | mapcidr -silent | dnsx -ptr -resp-only
```

### MURMURHASH (BUSQUEDA POR FAVICON.ICO)

La imagen/icono que se muestra en el lado izquierdo de una pestaña se denomina **favicon.ico**. Este ícono generalmente se obtiene de una fuente/CDN diferente. Por lo tanto, podemos encontrar este enlace de favicon desde el código fuente del sitio web.

**¿Cómo encontrar el enlace favicon.ico?**

-   Visite cualquier sitio web que ya posea un favicon. ( [https://www.microsoft.com/en-in](https://www.microsoft.com/en-in) )
-   Ahora, vea el código fuente y busque la palabra clave " **favicon**" en el código fuente.
-   Encontrarás el enlace donde está alojado el favicon. ( [https://cs-microsoft.com/favicon.ico?v2](https://c.s-microsoft.com/favicon.ico?v2) )

**MurMurHash** es una herramienta simple que se usa para generar hash para el favicon dado.

```bash
git clone https://github.com/Viralmaniar/MurMurHash.git
cd MurMurHash/
pip3 install -r requirements.txt
```

![[Pasted image 20230602235636.png]]

Ahora consultamos a Shodan `http.favicon.hash:<hash>` con ese hash de favicon.

![[Pasted image 20230602235745.png]]

estamos buscando las paginas que cargan el mismo flavicon.

### google dorks

- [[Google_Dorks]]
- [[Google hacking]]

```google
site:example.com
```

```url
site:*.domain.com -www
site:domain.com filetype:pdf
site:domain.com inurl:'&'
site:domain.com inurl:login,register,upload,logout,redirect,redir,goto,admin
site:domain.com ext:php,asp,aspx,jsp,jspa,txt,swf
site:*.*.domain.com
```

### Archivos bien conocidos

Busque archivos conocidos a los que pueda acceder a través del navegador; es posible que existan en el objetivo:

- **robots.txt:** acceda al archivo como http://TARGET_IP/robots.txt. 
- **.well-know:** estos puntos finales pueden contener muchos otros detalles. 
- **.git:** pocos sitios web pueden exponer accidentalmente su código fuente a través de este punto final.

### AMASS

Puede descargar el release desde su repo de github:

- [https://github.com/owasp-amass/amass/releases/tag/v3.23.2](https://github.com/owasp-amass/amass/releases/tag/v3.23.2)

Descomprimimos el zip:

```bash
unzip amass.zip
```

>[!note]
>muchas veces ya viene instalado en kali

ahora descargamos su archivo de configuracion y configuramos las [[API Keys]]:

- [https://github.com/owasp-amass/amass/blob/master/examples/config.ini](https://github.com/owasp-amass/amass/blob/master/examples/config.ini)

luego ejecutamos:

```bash
amass enum -passive -d owasp.org -src -config config.ini

amass enum -passive -d owasp.org -src
```

![[Pasted image 20230531231747.png]]

```bash
amass enum -active -d owasp.org -src -config config.ini

amass enum -active -d owasp.org -src
```

![[Pasted image 20230531232141.png]]

podemos usar un diccionario personalizado:

```bash
amass enum -aw subdomains-top1million-5000.txt -d femenina.com.bo
```

![[Pasted image 20230531232742.png]]

### SubDomainizer

```bash
git clone https://github.com/nsonaniya2010/SubDomainizer.git
cd SubDomainizer
pip install -r requirements.txt
python SubDomainizer.py -u https://www.quimiza.com/
```

![[Pasted image 20230605150925.png]]

### gobuster

```bash
apt install gobuster
```

```bash
gobuster dns -d erev0s.com -w awesome_wordlist.txt -i

gobuster vhost -u erev0s.com -w awesome_wordlist.txt

# Omitir autenticación básica
gobuster dir -u mytarget.com -w path/to/my/awesome/wordlist.txt -U BasicAuthUser -P BasicAuthPass
```

![[Pasted image 20230531235205.png]]

### wfuzz

```bash
wfuzz -c -t 200 --hc=404 --hw=28,73 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -H "Host: FUZZ.localhost.com" http://localhost.com
```

![[Pasted image 20230531235610.png]]

### Feroxbuster

[FEROXBUSTER](https://github.com/epi052/feroxbuster)

```bash
apt install feroxbuster
```

```bash
./feroxbuster --url http://10.10.10.10 #default values

./feroxbuster --url http://10.10.10.10 --depth 2 --wordlist /usr/share/wordlist/rockyou.txt

./feroxbuster --url http://10.10.10.10 -x jsp,java,class
```

enumerar directorio o archivos. Extensiones comunes de archivos backup:

```bash
.bak
.bac
.old
.000
.01
._bak
.001
.inc
.Xxx
```

### subbrute

```bash
git clone https://github.com/TheRook/subbrute
python2 subbrute.py domain.example.com > subdomains.txt
```

![[Pasted image 20230601002737.png]]

### CRT.SH

- [https://crt.sh/](https://crt.sh/)

podemos buscar subdominios con el caracter `%`:

```bash
%.example.com
```

podemos usar la herramienta de este repositorio tambien:

- [https://github.com/az7rb/crt.sh](https://github.com/az7rb/crt.sh)

```bash
git clone https://github.com/az7rb/crt.sh.git
cd crt.sh
chmod +x crt.sh

apt install jq

./crt.sh -d example.com

curl -s https://crt.sh/\?q\=herbalife.com.bo\&output\=json | jq .

curl -s https://crt.sh/\?q\=herbalife.com.bo\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

esto nos lo exporta en un archivo el resultado.

![[Pasted image 20230601233626.png]]

usando curl:

```bash
export TARGET="facebook.com"

curl -s "https://crt.sh/?q=${TARGET}&output=json" | jq -r '.[] | "\(.name_value)\n\(.common_name)"' | sort -u > "${TARGET}_crt.sh.txt"

head -n20 facebook.com_crt.sh.txt
```

### OBTENER LAS IP DE LOS SERVIDORES DE DOMINIO

supongamos que hemos encontrado varios subdominios de la empresa `herbalife.com` usando diferentes tecnicas de enumeracion y las tenemos guardadas en un archivo (`/tmp/sub1.txt`), podemos usar el comando host para intentar resolver la direccion IP del servidor:

```bash
#sub1.txt
aem.herbalifenutrition.com
herbalife.ca
*.herbalife.com.bo
herbalife.com.bo
referral.herbalife.ca
referral.herbalife.com.bo
www.herbalife.com.bo
www.herbalife.com.mx
```

```bash
for i in $(cat /tmp/sub1.txt);do host $i | grep "has address" | grep herbalife.com | cut -d" " -f1,4;done
```

![[Pasted image 20230612224201.png]]

#### COMBINANDO CON SHODAN PARA MAS INFORMACION

```bash
for i in $(cat /tmp/sub1.txt);do host $i | grep "has address" | grep herbalife.com | cut -d" " -f4 >> /tmp/ip-addr.txt;done

for i in $(cat /tmp/ip-addr.txt);do shodan host $i;done
```

![[Pasted image 20230612224711.png]]

### THE HARVESTER

[TheHarvester](https://github.com/laramies/theHarvester) es una herramienta fácil de usar pero poderosa y efectiva para pruebas de penetración en etapas tempranas y compromisos de equipo rojo. Podemos usarlo para recopilar información para ayudar a identificar la superficie de ataque de una empresa. La herramienta recopila `emails`, `names`, `subdomains`, `IP addresses` y `URLs` de varias fuentes de datos públicos para la recopilación pasiva de información. Por ahora, usaremos los siguientes módulos:

|**Modulo**|**Descripcion**|
|:-------:|:-------:|
|[Baidu](http://www.baidu.com/)|Motor de búsqueda Baidu.|
|`Bufferoverun`|Utiliza datos de Project Sonar de Rapid7 - [www.rapid7.com/research/project-sonar/](http://www.rapid7.com/research/project-sonar/)|
|[Crtsh](https://crt.sh/)|Búsqueda de certificados de Comodo.|
|[hackertarget](https://hackertarget.com/)|Escáneres de vulnerabilidades en línea e inteligencia de red para ayudar a las organizaciones.|
|`Otx`|Intercambio abierto de amenazas de AlienVault: [https://otx.alienvault.com](https://otx.alienvault.com/)|
|[rapiddns](https://rapiddns.io/)|Herramienta de consulta de DNS, que facilita la consulta de subdominios o sitios que utilizan la misma IP.|
|[Sublista3r](https://github.com/aboul3la/Sublist3r)|Herramienta de enumeración rápida de subdominios para probadores de penetración|
|[threatcrowd](http://www.threatcrowd.org/)|Inteligencia de amenazas de código abierto.|
|[threatminer](https://www.threatminer.org/)|Minería de datos para inteligencia de amenazas.|
|`Trello`|Buscar tableros de Trello (utiliza la búsqueda de Google)|
|[Urlscan](https://urlscan.io/)|Un sandbox para la web que es un escáner de URL y sitios web.|
|`Vhost`|Búsqueda de hosts virtuales de Bing.|
|[virustotal](https://www.virustotal.com/gui/home/search)|Búsqueda de dominio.|
|[zoomeye](https://www.zoomeye.org/)|Una versión china de Shodan.|

```bash
cat sources.txt

baidu
bufferoverun
crtsh
hackertarget
otx
projecdiscovery
rapiddns
sublist3r
threatcrowd
trello
urlscan
vhost
virustotal
zoomeye
```

```bash
export TARGET="facebook.com"
cat sources.txt | while read source; do theHarvester -d "${TARGET}" -b $source -f "${source}_${TARGET}";done
```

Cuando finalice el proceso, podemos extraer todos los subdominios encontrados y ordenarlos mediante el siguiente comando:

```bash
cat *.json | jq -r '.hosts[]' 2>/dev/null | cut -d':' -f 1 | sort -u > "${TARGET}_theHarvester.txt"
```

Ahora podemos fusionar todos los archivos de reconocimiento pasivo a través de:

```bash
cat facebook.com_*.txt | sort -u > facebook.com_subdomains_passive.txt

#cantidad de dominios encontrados
cat facebook.com_subdomains_passive.txt | wc -l
```

### knock

```bash
git clone https://github.com/guelfoweb/knock

cd knock/knockpy
python setup.py install
pip install -r requiremnts.txt

knockpy google.com
```

![[Pasted image 20230602000749.png]]

### subfinder

```bash
apt install subfinder

#ver el listado de fuentes, los que tienen * necesitan API key o token
subfinder -ls
```

![[Pasted image 20230602001222.png]]

configuracion de api keys:

```bash
cat /root/.config/subfinder/config.yaml
```

![[Pasted image 20230602001706.png]]

```bash
subfinder -d example.com -o /tmp/results_subfinder.tx
```

![[Pasted image 20230602001615.png]]

### assetfinder

```bash
apt install assetfinder
```

descubrimiento de subdominios:

```bash
assetfinder --subs-only herbalife.com.bo
```

![[Pasted image 20230602002332.png]]

### bbot

```bash
pip install bbot
bbot -t quimiza.com -f subdomain-enum
```

![[Pasted image 20230605153735.png]]

### ReconFTW

>[!note]
>Require Golang> **1.15.0+** instalado y variables de entorno correctamente seteadas (**\$GOPATH**, **\$GOROOT**)

```bash
git clone https://github.com/six2dez/reconftw
cd reconftw/
./install.sh
```

```bash
./reconftw.sh -d target.com -r
```

## SCREENSHOT

Podemos tener una lista de subdominios en diferentes servidores web, esos servidores web pueden tener muchos puertos http abiertos (80,443,8080,8081,8888,etc) lo que nos da una gran cantidad de opciones que revisar. Para tener un primer pantallazo de cada puerto de cada servidor de cada subdominio, podemos usar herramientas como **aquatone** o **eyewitness** que toma capturas depantalla de cada opciones que existe (pagina web) y lo documenta y filtra en un reporte para facilitarnos las cosas.

### Recoleccion de informacion con NMAP

supongamos que tenemos la siguient lista:

```bash
cat scope_list
```

![[Pasted image 20230623224517.png]]

podemos usar nmap para enumerar los puertos web mas comunes y exportarlo en formato XML, ya que ese formato podemos pasarlo a las herramientas de screenshot:

>[!note]
>Los puertos los podemos definir nosotros en base a nuestras necesidades.

```bash
nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list 
```

### eyewitness

instalacion:

```bash
sudo apt install eyewitness
```

tomamos capturas los servicios que enumeró nmap en formato XML:

```bash
eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness
```

![[Pasted image 20230623225606.png]]

### aquatone

descargarmos el release mas estable:

- [https://github.com/michenriksen/aquatone/releases/tag/v1.7.0](https://github.com/michenriksen/aquatone/releases/tag/v1.7.0)

>[!note]
>Nos pedira tambien instalar chromium para las screenshot:
>
>```bash
>sudo apt install chromium
>```

```bash
unzip aquatone.zip
mv aquatone /usr/bin

#pasando una lista de subsdominios
cat sub.txt | aquatone -out Reporte

#ejemplo
#sub.txt
https://www.quimiza.com
```

podemos tambien pasarle el XML de nmap:

```bash
cat web_discovery.xml | ./aquatone -nmap
```

![[Pasted image 20230623230729.png]]

![[Pasted image 20230623230808.png]]

## VALIDACION DE DOMINIOS WEB 

Una vez que realizamos la enumeracion de subdominios, podemos usar herramientas que nos ayudaran a detectar cuales de ellos son paginas web. Esto nos ayuda a filtrar aquellos en los que podemos ingresar y probar vulnerabilidades web.

### HTTPROBE & HTTPX

Ahora tenemos toda una lista de subdominios de una empresa, pero con las herramientas **httprobe** y **httpx** podemos ver cuales de ellos soportan **https** o **http** (son paginas web):

- [https://github.com/tomnomnom/httprobe](https://github.com/tomnomnom/httprobe)
- [https://github.com/projectdiscovery/httpx](https://github.com/projectdiscovery/httpx)

```bash
sudo apt install httprobe
sudo apt install httpx
```

ahora a una lista de subdominios (resultado de crt.sh) usamos estas herramientas:

```bash
cat crt.sh/output/domain.bancosol.com.bo.txt | httpx -status-code

cat crt.sh/output/domain.bancosol.com.bo.txt | httprobe
```

![[Pasted image 20230308162006.png]]

![[Pasted image 20230308162040.png]]

otra forma de ejecutar:

```bash
httpx -l hosts.txt -silent -probe
cat hosts.txt | httpx
```

### Httpx + subfinder

```bash
subfinder -d hackerone.com | httpx -title -tech-detect -status-code
```

## ENUMERACION DE USUARIOS

### linkedin2username

- [https://github.com/initstring/linkedin2username](https://github.com/initstring/linkedin2username)

## ENUMERACION DE PUERTOS WEB

###  masscan

Este es un escáner de puertos a escala de Internet. Puede escanear toda la Internet en menos de 5 minutos, transmitiendo 10 millones de paquetes por segundo, desde una sola máquina.

sirve cuando el scope es demasiado grande

```bash
sudo apt-get --assume-yes install git make gcc
git clone https://github.com/robertdavidgraham/masscan
cd masscan
make
```

[documentacion masscan](https://github.com/robertdavidgraham/masscan)

### nmap

```bash
nmap -sV --script=http-enum <target>
```

```bash
nmap --script dns-brute --script-args dns-brute.domain=foo.com,dns-brute.threads=6,dns-brute.hostlist=./hostfile.txt,newtargets -sS -p 80
```

## WHOIS

Podemos considerar a [WHOIS](https://en.wikipedia.org/wiki/WHOIS) como las "páginas blancas" de los nombres de dominio. Es un protocolo de consulta/respuesta orientado a transacciones basado en TCP que escucha en el puerto TCP 43 de forma predeterminada. Podemos usarlo para consultar bases de datos que contienen nombres de dominio, direcciones IP o sistemas autónomos y proporcionar servicios de información a los usuarios de Internet.

```bash
whois.exe facebook.com #windows
whois facebook.com #linux
```

## Subdomain takeover

### subjack

```bash
apt install subjack

subjack -w sub.txt -v

#sub.txt
#www.quimiza.com
#app.quimiza.com
```

![[Pasted image 20230605152838.png]]

### takeover

- [https://github.com/m4ll0k/takeover](https://github.com/m4ll0k/takeover)

```bash
git clone https://github.com/m4ll0k/takeover.git
cd takeover
python3 setup.py install
```

```bash
python3 takeover.py -d www.domain.com -v 
python3 takeover.py -d www.domain.com -v -t 30
python3 takeover.py -d www.domain.com -p http://127.0.0.1:8080 -v 
python3 takeover.py -d www.domain.com -o <output.txt> or <output.json> -v 
python3 takeover.py -l uber-sub-domains.txt -o output.txt -p http://xxx.xxx.xxx.xxx:8080 -v 
python3 takeover.py -d uber-sub-domains.txt -o output.txt -T 3 -v 
```

## Metodologia analisis web externo

* Conocer el objetivo (en su totalidad - arquitectura si es posible)
* Reconocer subdomios
* Detectar subdominios web
* Reconocer direcciones IP (rangose IP de los DNS)
* Reconocer tecnologias usadas
* analisis de directorios
	* wfuzz, ffuf, gobuster, burpsuite
* web banner disclosure
	* burpsuite
* security headers validation
* brute force de parametros
* JS files finder
* subdomain takeover
* verificar uso de CMS
	* WPScan, Drupscan, Joomscan
* reconFTW
* enumeracion de usuarios con linkedin
* enumeracion de puertos en servidores web

## FUENTE

- [https://0xsanz.medium.com/web-enumeration-methodology-8b44e52730d6](https://0xsanz.medium.com/web-enumeration-methodology-8b44e52730d6)
- [https://www.bugcrowd.com/blog/discovering-subdomains/](https://www.bugcrowd.com/blog/discovering-subdomains/)
- [https://thegrayarea.tech/bug-hunting-101-directory-enumeration-authentication-bypass-1b92b3c87ef9](https://thegrayarea.tech/bug-hunting-101-directory-enumeration-authentication-bypass-1b92b3c87ef9)
- [https://debprasadbanerjee502.medium.com/the-secret-trick-for-subdomain-enumeration-91b28be2b957](https://debprasadbanerjee502.medium.com/the-secret-trick-for-subdomain-enumeration-91b28be2b957)
- [https://medium.com/@nynan/the-most-underrated-tool-in-bug-bounty-and-the-filthiest-one-liner-possible-cab14ef7faeb](https://medium.com/@nynan/the-most-underrated-tool-in-bug-bounty-and-the-filthiest-one-liner-possible-cab14ef7faeb)
- [https://apexvicky.medium.com/bug-bounty-methodology-horizontal-enumeration-89f7cd172e6e](https://apexvicky.medium.com/bug-bounty-methodology-horizontal-enumeration-89f7cd172e6e)
- [https://blog.appsecco.com/a-penetration-testers-guide-to-sub-domain-enumeration-7d842d5570f6](https://blog.appsecco.com/a-penetration-testers-guide-to-sub-domain-enumeration-7d842d5570f6)
- [https://gist.github.com/m4ll0k/31ce0505270e0a022410a50c8b6311ff](https://gist.github.com/m4ll0k/31ce0505270e0a022410a50c8b6311ff)
- []()
- []()
- []()