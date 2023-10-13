# WEB METODOLOGY (CTF)

- Descubrir puertos (80-443-8080-8000-8443) (nmap -sS -Pn -T4 -p- TARGET_IP) (nmap -O -A -Pn -T4 -p80,443 TARGET_IP)
- Interactuar con la pagina
	- LLenar registros de usuarios (crearse cuenta)
- Revisar el codigo fuente
	- revisar comentarios
	- revisar links-urls (subdominios y directorios)
	- revisar scripts .js u otros
- SCRIPT HTTP-ENUM NMAP
- VER CONTENT DISCOVERY FILE
- ver archivos bien conocidos
	- /robots.txt (directorios)
	- /.well-known/example (lista de example -> https://en.wikipedia.org/wiki/Well-known_URI#List_of_well-known_URIs)
	- .git
- Virtual Hosting
	- A veces, visitar la dirección IP de destino no revela nada. (Agregar la ip al /etc/hosts)
- verificacion de un waf
	- whatw00f
- ver las OPCIONES de verbos disponibles de una pagina:

```bash
curl -X OPTIONS http://10.10.1.10
```

- jugar cambiando el verbo de la peticion GET, POST PUT, DELETE (SOBRETODO CON API's)
- decubrimiento de parametros (SOBRETODO CON API's):

```bash
wfuzz -c -t 100 --hw=854 --hc=400,404 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -u "http://10.10.182.126/api/items?FUZZ=hola" -X POST #glitch THM
```

 

- Descubrimiento de directorio (RECURSIVO) **AGREGAR EXTENSIONES PARA ARCHIVOS**
	- dirb
	- wfuzz
	- ffuf
	- dirsearch
	- feroxbuster
	- gobuster

```bash
gobuster dir -u http://10.10.51.62 -x txt,php,tar,zip -w /usr/share/wordlists/dirb/common.txt
```

	    - nmap script http-enum
- descubrimiento de subdominios
	- gobuster
	- wfuzz
	- ffuf
	- security trails
	- google dorks (site:) 
	- reconFTW
	- subbrute
	- domain zone transfer
	- assetfinder (assetfinder --subs-only domain.name) (https://github.com/tomnomnom/assetfinder)
- descubrir tecnologias (POR CADA PAGINA)
	- whatweb
	- wappalyzer
- buscar exploit con las versiones de tecnologias web (POR CADA PAGINA)
	- exploit-db.com
	- searchsploit
	- attackerkb.com
- Ataques de fuerza a inicios de sesion (CTF porque en Real World te bloquean)
	- hydra (con y sin autenticacion basica)

	


