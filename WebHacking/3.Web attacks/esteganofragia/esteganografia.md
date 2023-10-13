# ESTEGANOGRAFIA

Es la tecnica de ocultar informacion a traves de archivos multimedia o de un formato diferente. Puedes ocultar informacion en imagenes, sonidos y videos, pero tambien en cualquier tipo de archivo.

## Table of contents

- [ESTEGANOGRAFIA](#esteganografia)
  - [VERIFICACION DE INFORMACION DE UN ARCHIVO](#verificacion-de-informacion-de-un-archivo)
    - [FILE](#file)
    - [STRINGS](#strings)
    - [CMP (COMPARACION)](#cmp-comparacion)
  - [EXTRAER INFORMACION DE ARCHIVOS](#extraer-informacion-de-archivos)
    - [BINWALK](#binwalk)
    - [FOREMOST](#foremost)
    - [EXIFTOOL](#exiftool)
    - [EXIV2](#exiv2)
  - [EXTRAER DATOS OCULTOS EN TEXTOS](#extraer-datos-ocultos-en-textos)
    - [STEGHIDE (JPEG, BMP, WAV, AU)](#steghide-jpeg-bmp-wav-au)
    - [ZSTEG (PNG, BMP)](#zsteg-png-bmp)
    - [STEGOVERITAS (JPG, PNG, GIF, TIFF Y BMP)](#stegoveritas-jpg-png-gif-tiff-y-bmp)
    - [STEGPY (PNG, BMP, GIF, WebP, WAV)](#stegpy-png-bmp-gif-webp-wav)
    - [PNGCHECK](#pngcheck)
  - [FUENTE](#fuente)

## VERIFICACION DE INFORMACION DE UN ARCHIVO

### FILE

Verificar el tipo de archivo:

```bash
file data.txt
```

### STRINGS

Muestra los strings legibles de un binario o archivo, util en binarios:

```bash

strings binary.exe

strings -n 6 file #Extract the strings with min length of 6
strings -n 6 file | head -n 20 #Extract first 20 strings with min length of 6
strings -n 6 file | tail -n 20 #Extract last 20 strings with min length of 6
strings -e s -n 6 file #Extract 7bit strings
strings -e S -n 6 file #Extract 8bit strings
strings -e l -n 6 file #Extract 16bit strings (little-endian)
strings -e b -n 6 file #Extract 16bit strings (big-endian)
strings -e L -n 6 file #Extract 32bit strings (little-endian)
strings -e B -n 6 file #Extract 32bit strings (big-endian)

```

### CMP (COMPARACION)

compara archivos originales con modificados:

```bash
cmp original.jpg stego.jpg -b -l
```

## EXTRAER INFORMACION DE ARCHIVOS

### BINWALK

instalacion: [https://github.com/ReFirmLabs/binwalk](https://github.com/ReFirmLabs/binwalk)

```bash
apt install binwalk
```

uso:

```bash
file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts the data from the given file
binwalk --dd ".*" file #Displays and extracts the data from the given
```

### FOREMOST

instalacion: [https://github.com/korczis/foremost](https://github.com/korczis/foremost)

```bash
apt install foremost
```

se debe configurar el tipo de archivo en **/etc/foremost.conf**

uso:

```bash
foremost -i file
```


### EXIFTOOL

buscar y agregar metadatos:

```bash
apt install exiftool
```

uso:

```bash
exiftool file

exiftool COMMENT="HOLA" file
```

### EXIV2

similar a exiftool: [https://github.com/Exiv2/exiv2](https://github.com/Exiv2/exiv2)

```bash
apt install exiv2
```

uso:

```bash
exiv2 file
```

## EXTRAER DATOS OCULTOS EN TEXTOS

### STEGHIDE (JPEG, BMP, WAV, AU)

instalacion: [https://github.com/StefanoDeVuono/steghide](https://github.com/StefanoDeVuono/steghide)

```bash
apt install steghide
```

uso:

```bash
steghide extract -sf file [--passphrase password]
```

en caso de no saber la contraseña, si la tuviera, se puede aplicar fuerza bruta: [https://github.com/Paradoxis/StegCracker](https://github.com/Paradoxis/StegCracker)

```bash
stegcracker file rockyou.txt
```

### ZSTEG (PNG, BMP)

instalar: [https://github.com/zed-0xff/zsteg](https://github.com/zed-0xff/zsteg)

```bash
gem install zteg
```

uso:

```bash
zteg -a file

zteg -E file
```

### STEGOVERITAS (JPG, PNG, GIF, TIFF Y BMP)

Script de python que tiene muchas buenas caracteristicas: [https://github.com/bannsec/stegoVeritas](https://github.com/bannsec/stegoVeritas)

```bash
stegoveritas.py -h
```

uso para todas las comprobaciones:

```bash
stegoveritas.py file.jpg
```

### STEGPY (PNG, BMP, GIF, WebP, WAV)

- [https://github.com/dhsdshdhk/stegpy](https://github.com/dhsdshdhk/stegpy)

### PNGCHECK

solo para PNG:

```bash
apt install pngcheck
```

uso

```bash
pngcheck stego.png
```

## FUENTE

- [https://book.hacktricks.xyz/stego/stego-tricks](https://book.hacktricks.xyz/stego/stego-tricks)







