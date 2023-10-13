# DNS Enumeration

## NSLOOKUP & DIG

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

## DNS RECORDS

- [https://www.cloudflare.com/es-es/learning/dns/dns-records/](https://www.cloudflare.com/es-es/learning/dns/dns-records/)