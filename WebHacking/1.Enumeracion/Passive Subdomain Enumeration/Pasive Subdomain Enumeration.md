# Pasive Subdomain Enumeration

## VIRUS TOTAL

VirusTotal mantiene su servicio de replicación de DNS, que se desarrolla conservando las resoluciones de DNS realizadas cuando los usuarios visitan las URL proporcionadas por ellos. Para recibir información sobre un dominio, escriba el nombre de dominio en la barra de búsqueda y haga clic en la pestaña "Relaciones".

![[Pasted image 20221216161210.png]]

## CERTIFICADOS

Otra fuente de información interesante que podemos utilizar para extraer subdominios son los certificados SSL/TLS. La razón principal es Certificate Transparency (CT), un proyecto que requiere que cada certificado SSL/TLS emitido por una Autoridad de certificación (CA) se publique en un registro de acceso público.

-   [https://censys.io](https://censys.io/)
-   [https://crt.sh](https://crt.sh/)

Podemos navegar a https://search.censys.io/certificates o https://crt.sh e introducir el nombre de dominio de nuestra organización objetivo para empezar a descubrir nuevos subdominios.

![[Pasted image 20221216162103.png]]

![[Pasted image 20221216162109.png]]

Realicemos una solicitud de curl al sitio web de destino solicitando una salida JSON, ya que esto es más manejable para que lo procesemos. Podemos hacer esto a través de los siguientes comandos:

```bash
export TARGET="facebook.com"

curl -s "https://crt.sh/?q=${TARGET}&output=json" | jq -r '.[] | "\(.name_value)\n\(.common_name)"' | sort -u > "${TARGET}_crt.sh.txt"

head -n20 facebook.com_crt.sh.txt
```

## OPENSSL

```bash
export TARGET="facebook.com"

export PORT="443"

openssl s_client -ign_eof 2>/dev/null <<<$'HEAD / HTTP/1.0\r\n\r' -connect "${TARGET}:${PORT}" | openssl x509 -noout -text -in - | grep 'DNS' | sed -e 's|DNS:|\n|g' -e 's|^\*.*||g' | tr -d ',' | sort -u
```

## THE HARVESTER

[TheHarvester](https://github.com/laramies/theHarvester) es una herramienta fácil de usar pero poderosa y efectiva para pruebas de penetración en etapas tempranas y compromisos de equipo rojo. Podemos usarlo para recopilar información para ayudar a identificar la superficie de ataque de una empresa. La herramienta recopila `emails`, `names`, `subdomains`, `IP addresses`y `URLs`de varias fuentes de datos públicos para la recopilación pasiva de información. Por ahora, usaremos los siguientes módulos:

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

