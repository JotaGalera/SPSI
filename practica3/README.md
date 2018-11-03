# Practica 3

## 1.Generar un archivo sharedDSA.pem que contenga los parámetros. Mostrar los valores:

Utilizaremos el comando "dsaparam" que nos permite generar y manipular parámetros asociados al algoritmo DSA: primos ___p___ y ___q____ y el generador ___g___.

~~~~
openssl dsaparam -out sharedDSA.pem 901

~~~~
Así se generará el resultado con un tamaño de 901 bits.

La consola nos mostrará algo como esto:
![](ej1/generacion.png)

Los valores obtenidos en hexadecimal tras generar el archivo son estos:
![](ej1/resultado.png)

## 2.Generar dos parejas de claves para los parámetros anteriores. Las claves se almacenarán en los archivos <nombre>DSAkey.pem y <apellido>DSAkey.pem. No es necesario protegerlas con contraseña.

Para generar la pareja de claves necesitamos del archivo anterior (sharedDSA.pem) el cual actuará como parámetro. Esta vez utilizaremos el comando de openssl ***gedsa***.

En primer lugar generamos el par para Javier:
~~~~
openssl gendsa -out JavierDSAkey.pem sharedDSA.pem
~~~~

Por último, las generamos para Galera:
~~~~
openssl gendsa -out GaleraDSAkey.pem sharedDSA.pem
~~~~

![](ej2/pareja.png)

Ya tenemos generadas las dos parejas de claves.

## 3."Extraer" la clave privada contenida en el archivo <nombre>DSAkey.pem a otro archivo que tenga por nombre <nombre>DSApriv.pem. Este archivo deberá estar protegido por contraseña. Mostrad sus valores. Haced lo mismo para el archivo <apellido>DSAkey.pem

Para realizarlo utilizaremos el comando de openssl ***dsa***. El cual nos permitirá sustraer la clave public y/o privada.Además utilizaremos como cifrado simétrico AES-128 y como contraseña: 0123456789

De esta forma extraeremos y cifraremos la clave privada de Javier:
~~~~
openssl dsa -in JavierDSAkey.pem -out JavierDSApriv.pem -aes128
~~~~
![](ej3/Javier.png)

Del mismo modo para Galera:
![](ej3/Galera.png)

Por último mostraremos el resultado de dichas claves, a través del comando:
~~~~
openssl dsa -in <nombre>DSApriv.pem -text -noout
~~~~

De ***Javier***:
![](ej3/showJavier.png)

De ***Galera***:
![](ej3/showGalera.png)

## 4.Extraer en <nombre>DSApub.pem la clave pública contenida en el archivo <nombre>DSAkey.pem. De nuevo <nombre>DSApub.pem no debe estar cifrado ni protegido. Mostrad sus valores. Lo mismo para el archivo <galera>DSAkey.pem

Al igual que en el apartado anterior, utilizaremos el comando de openssl ***dsa***.

Para la extracción de la clave pública utilizaremos el comando:
~~~~
openssl dsa -in <nombre>DSAkey.pem -out <nombre>DSApub.pem -pubout
~~~~

Tanto para Javier como para Galera:
![](ej4/pub.png)

Por último mostraremos los valores de las claves:

Para ***Javier***:

![](ej4/Javier.png)

Para ***Galera***:

![](ej4/Galera.png)

## 5. Coger un archivo que actuará como entrada, de al menos 128 bytes. En adelante, dicho archivo será denominado como "message".

Para crear el archivo utilizaremos:
~~~~
dd if=/dev/zero of=message count=1024 bs=1
~~~~

El parámetro count recibe el número de bits del archivo que se creará.
Hemos introducido 1024, pues 128 bytes * 8 bits/byte = 1024 bits.

## 6.Firmar directamente el archivo message empleando el comando openssl pkeyutl sin calcular los valores hash, la firma deberá almacenarse en un archivo llamado message.sign. Mostrad el archivo con la firma.
