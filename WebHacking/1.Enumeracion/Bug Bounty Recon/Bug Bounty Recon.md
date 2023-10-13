# Bug Bounty Recon

## SUBDOMAIN ENUMERATION

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

./crt.sh -d example.com
```

esto nos lo exporta en un archivo el resultado.

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




