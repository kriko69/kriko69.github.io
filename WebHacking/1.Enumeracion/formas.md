# INFORMATION GATHERING

Obtener informacion de direcciones IP, servidores, sistemas operativos, DNS, etc.
Existen herramientas online para este trabajo.

**IP Address o Dominio**

* [https://whois.domaintools.com/](https://whois.domaintools.com/)

**Tecnologias de una web**

* [https://sitereport.netcraft.com/](https://sitereport.netcraft.com/)
  
**DNS information**

* [https://www.robtex.com](https://www.robtex.com)

**Descubrir subdominios**

Usar el siguiente script de python 2.7 ubicado en GitHub

* [https://github.com/guelfoweb/knock](https://github.com/guelfoweb/knock)

entrar a la carpeta clonada knock/knockpy

```
pip install dnspython

python knockpy.py www.google.com

```

**Descubrir directorios mas archivos**

```
dirb http://192.168.112.140/fotos/

```

para mas informacion de dirb

```
man dirb

```

**Herramienta todo en uno**

Maltego

```
maltegoce

```

* Instalar passiveTotal
* Crear a new page
* infraestructura > Doomain
* Modificar el atributo domain name al gusto
* Click derecho en el icono del mundo y elegir un analisis dando click en el boton play

Verificar herramientas OSINT

```bash

# https://spyse.com/tools/dns-lookup

AS26210
web.bdp.com.bo -> https://administradorweb.bdp.com.bo/ -> 190.181.63.106
www.bdp.com.bo -> 190.181.63.106
ventana.bdp.com.bo -> 190.181.63.108
encuentro.bdp.com.bo -> 190.181.63.109
aula.bdp.com.bo -> aula.bdp.com.bo
mail.bdp.com.bo -> 190.181.63.110 -> lets encrypt

# https://centralops.net/co/DomainDossier.aspx

responsible: Richard Sandoval
web.bdp.com.bo -> 190.181.63.106

# https://securitytrails.com/

bdp.com.bo -> 190.181.63.106
mail2.bdp.com.bo
aula.bdp.com.bo -> 190.181.63.108
proxy.bdp.com.bo -> 190.181.63.109
correo.bdp.com.bo -> 190.181.63.110
reclamosweb.bdp.com.bo -> 190.181.63.106
web.bdp.com.bo -> 190.181.63.106
vpn.bdp.com.bo -> 200.105.173.90
ventana.bdp.com.bo -> 190.181.63.108
complejidades.bdp.com.bo -> 190.181.63.108
www.bdp.com.bo -> 190.181.63.106
mail.bdp.com.bo  -> 190.181.63.110
encuentro.bdp.com.bo -> 190.181.63.108
root.bdp.com.bo
bdpnet.com.bo -> 190.181.63.109

```

### http probe



* xss
	* [https://github.com/s0md3v/XSStrike](https://github.com/s0md3v/XSStrike)
	
	```bash
	git clone https://github.com/s0md3v/XSStrike.git
	cd XSStrike
	pip3 install -r requirements.txt
	python3 xsstrike.py -u https://www.expla.com?name=chris -t 5 #GET
	python3 xsstrike.py -u https://www.expla.com --data "name=chris" #POST
	```
	
	```bash
	-u --url -> definir url
	-t --threads -> hilos
	--params -> Encuentre parámetros ocultos analizando HTML y fuerza bruta.
	--crawl -> Comience a rastrear desde la página web de destino en busca de destinos y pruébelos.
	```
	
	[https://github.com/s0md3v/AwesomeXSS](https://github.com/s0md3v/AwesomeXSS)
	
* sql injection
	* SQLMap

[mas formas](https://blog.appsecco.com/a-penetration-testers-guide-to-sub-domain-enumeration-7d842d5570f6)



apt install php
apt install apache
cisco talos reputation
zip -r as.zip carpta
sudo scp carp.ip root@ip:/var/www/html

fijar permisos

mxtoolbox MX

definir promocion con limite de tiempo