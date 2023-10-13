# WEB ENUMERATION

## technology discover

whatweb:

```bash
whatweb http://10.10.10.10

whatweb www.facebook.com
```

```bash
whatweb -a 1 <URL> #Stealthy
whatweb -a 3 <URL> #Aggresive
webtech -u <URL>
```

wappalyzer

**ENUMERACION DE VIRTUAL HOSTING**

## Herramientas

gobuster:

```bash
gobuster dns -d erev0s.com -w awesome_wordlist.txt -i

gobuster vhost -u erev0s.com -w awesome_wordlist.txt

# Omitir la verificación SSL

gobuster dir -u mytarget.com -w path/to/my/awesome/wordlist.txt -k

# Omitir autenticación básica

gobuster dir -u mytarget.com -w path/to/my/awesome/wordlist.txt -U BasicAuthUser -P BasicAuthPass
```

wfuzz:

```bash
wfuzz -c -t 200 --hc=404 --hw=28,73 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -H "Host: FUZZ.localhost.com" http://localhost.com
```

subbrute:

```bash
git clone https://github.com/TheRook/subbrute
python subbrute.py domain.example.com > subdomains.txt
```

subfinder:

```bash
go get github.com/subfinder/subfinder
./Subfinder/subfinder --set-config PassivetotalUsername='USERNAME',PassivetotalKey='KEY'
./Subfinder/subfinder --set-config RiddlerEmail="EMAIL",RiddlerPassword="PASSWORD"
./Subfinder/subfinder --set-config CensysUsername="USERNAME",CensysSecret="SECRET"
./Subfinder/subfinder --set-config SecurityTrailsKey='KEY'
./Subfinder/subfinder -d example.com -o /tmp/results_subfinder.tx
```

aquatone:

```bash
apt install ruby
gem install aquatone

aquatone-discover -d ggogle.com --disable-collectors dictionary
aquatone-discover --domain example.com
aquatone-discover --domain example.com --threads 2

aquatone-scan --domain example.com
aquatone-scan --domain example.com --ports 80,443,3000,8080
aquatone-scan --domain example.com --ports large
```

-   **small**: 80, 443
-   **medium**: 80, 443, 8000, 8080, 8443 (same as default)
-   **large**: 80, 81, 443, 591, 2082, 2087, 2095, 2096, 3000, 8000, 8001, 8008, 8080, 8083, 8443, 8834, 8888
-   **xlarge**: 80, 81, 300, 443, 591, 593, 832, 981, 1010, 1311, 2082, 2087, 2095, 2096, 2480, 3000, 3128, 3333, 4243, 4567, 4711, 4712, 4993, 5000, 5104, 5108, 5800, 6543, 7000, 7396, 7474, 8000, 8001, 8008, 8014, 8042, 8069, 8080, 8081, 8088, 8090, 8091, 8118, 8123, 8172, 8222, 8243, 8280, 8281, 8333, 8443, 8500, 8834, 8880, 8888, 8983, 9000, 9043, 9060, 9080, 9090, 9091, 9200, 9443, 9800, 9981, 12443, 16080, 18091, 18092, 20720, 28017


**METODOLOGIA PENTESTING WEB**

[https://book.hacktricks.xyz/pentesting/pentesting-web](https://book.hacktricks.xyz/pentesting/pentesting-web)

**BURP SUITE**

[[Burp suite]]

**WORDPRESS - JOOMLA - DRUPAL**

[[Wordpress Hacking]]

[DRUPAL](https://book.hacktricks.xyz/pentesting/pentesting-web/drupal)

[JOOMLA](https://book.hacktricks.xyz/pentesting/pentesting-web/joomla)

**ENUMERACION DE DIRECTORIOS**

[[enumeracion web]]

[[wfuzz]]

**VALIDACIONES**

- Client Side Validation.
- SQL injection.
- File Upload y descarga de algun archivo.

```bash
www.example.com/document.pdf
```

- Cross Site Scripting (XSS).
- Redirecciones. (Puede pasar que al interceptar una aplicacion nos de un 302 pero se puede cambiar a 200 y mostrar el contenido)
- Authentication bypass y fuerza bruta. (En logins)
- Mensajes de error que muestren informacion.
- Web Banner disclosure.
- IDOR.
- CGI (shellshock)
- enumerar directorio o archivos. Extensiones comunes de archivos backup:

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

[cURL uso](https://www.geeksforgeeks.org/curl-command-in-linux-with-examples/#:~:text=curl%20is%20a%20command%20line,to%20work%20without%20user%20interaction.)