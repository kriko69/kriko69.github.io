# BUSINESS LOGIC

## Table of contents

- [BUSINESS LOGIC](#business-logic)
  - [INTRODUCCION](#introduccion)
  - [METODOS DE EXPLOTACION](#metodos-de-explotacion)
    - [Excessive trust in client-side controls](#excessive-trust-in-client-side-controls)
    - [High-level logic vulnerability](#high-level-logic-vulnerability)
    - [Flawed enforcement of business rules](#flawed-enforcement-of-business-rules)
    - [Weak isolation on dual-use endpoint](#weak-isolation-on-dual-use-endpoint)

## INTRODUCCION

Las vulnerabilidades de la lógica empresarial son fallas en el diseño y la implementación de una aplicación que permiten a un atacante provocar un comportamiento no deseado. Esto potencialmente permite a los atacantes manipular la funcionalidad legítima para lograr un objetivo malicioso. Estas fallas generalmente son el resultado de no anticipar estados de aplicación inusuales que pueden ocurrir y, en consecuencia, no manejarlos de manera segura.

Las fallas de lógica a menudo son invisibles para las personas que no las buscan explícitamente, ya que normalmente no estarán expuestas por el uso normal de la aplicación. Sin embargo, un atacante puede explotar las peculiaridades del comportamiento al interactuar con la aplicación de formas que los desarrolladores nunca pretendieron.

## METODOS DE EXPLOTACION

- Confianza excesiva en los controles del lado del cliente, la validacion de parametros solo se la realiza en el frontend y no en el backend, por lo que podemos intenar un IDOR.

### Excessive trust in client-side controls

puede que a la hora de agregar un producto al carrito se envie el precio como un parametro:

![[Pasted image 20230213235232.png]]

podemos manipular esto y comprar cosas a otro precio.

### High-level logic vulnerability

Al quere agregar un producto al carrito podemos intentar pasar valores negativos en cuanto a la cantidad, normalmente la formula para sacar el precio es `cantidad*precio_unitario=precio_total`.

Al final todo lo que agregues al carrito se va sumando, por lo que si agregas valores negativos y despues positivos puede hacer que el valor de un producto baje: 

- Si vemos que podemos comprar cantidades negativas, podemos comprar una cosa y reducirle el precion con la compra negativa:

![[Pasted image 20220928222751.png]]

![[Pasted image 20220928222727.png]]

### Flawed enforcement of business rules

- algunas aplicaciones cometen el error de asumir que, habiendo superado inicialmente estos estrictos controles, se puede confiar en el usuario y sus datos indefinidamente. Los controles de seguridad debe estar de manera consistente en toda la aplicación.

podemos romper la logica de la aplicacion intercambiando entre cupones de descuento, si aplico el mismo 2 veces no meja pero si intercalo me deja:

![[Pasted image 20220928224841.png]]

### Weak isolation on dual-use endpoint

una pagina tiene una opcion de cambio de contraseña:

![[Pasted image 20230214145651.png]]

si lo interceptamos con burp vemos como viaja la peticion:

![[Pasted image 20230214152949.png]]

un parametro es el username, si intentamos cambiarlo a administrador, no nos deja ya que no conocemos su **current password**:

![[Pasted image 20230214153034.png]]

Pero si quitamos el parametro **current password** vemos que si se logra actualizar la contraseña y podemos ingresar como ese usuario al dashboard de administrador:

![[Pasted image 20230214153102.png]]

![[Pasted image 20230214153201.png]]