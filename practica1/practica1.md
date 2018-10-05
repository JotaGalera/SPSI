# Practica 1 SPSI

En este documento explicaremos paso a paso la resolución de la primera práctica de la asignatura SPSI.

## 1. y 2.Creación de dos archivos binarios de 1024 bits, uno de ellos con todos los bits a 0 y el otro con un bit a 1 entre el 130 y 150.

Mediante:
~~~~
dd if=/dev/zero of=input1.bin count=128 bs=1
~~~~
Conseguimos un archivo de 1024 bits(128 bytes) totalmente a cero.

***Nota: dev/zero es un archivo propio de Unix que provee de tantos caracteres null como necesitemos.***

Para el segundo archivo, utilizando la herramienta "bless" abrimos el archivo input2.bin e introducimos un 1 en la posición indicada.

## 3. Cifrar ambos archivos con AES-256 en modos ECB, CBC y 0FB usando una clave a elegir del tamaño adecuado (32 bytes, 64 caracteres),y con vector de inicialización "0123456789abcdef".

Primero comprobamos que nuestra versión de OpenSSL soporta AES-256

![Imagen AES-256-sopotada](./Ejercicio3/AES-256-sopotada.png)

### modo ECB:

![ECB-teoria](./Ejercicio3/ECB-teoria.png)

~~~~
openssl aes-256-ecb -K
0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
-iv 0123456789abcdef -in input1.bin -out output1.aes

Nota: Hemos inicializado el vector de inicialización, pero ecb no lo utiliza,
por lo que el resultado debe ser el mismo sin importar la cadena de iv que
utilicemos.
~~~~

![Imagen AES-256-ecb](./Ejercicio3/aes-256-ecb.png)

Utilizando xxd output1.aes muestra:

![Imagen-xdd-aes-256-ecb](./Ejercicio3/xdd-aes-256-ecb.png)

Aplicando el mismo proceso para el segundo archivo:
~~~~
openssl aes-256-ecb -K
0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
-iv 0123456789abcdef -in input1.bin -out output1.aes
~~~~

Y mediante xdd:

![Imagen-xdd-aes-256-ecb](./Ejercicio3/xdd-aes-256-ecb-2.png)

En el modo ECB, cada bloque de mensaje que se cifra, se cifra de modo independiente al anterior. Por eso todas las lineas que tengan el mismo contenido, un bloque de ceros, se representan de la misma forma, mientras que la linea donde esta el 1 cambia por completo.
La última linea entiendo que cambia por el fin de archivo.


### modo CBC:

![CBC-teoria](./Ejercicio3/CBC-teoria.png)

~~~~
openssl aes-256-cbc -K
0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
-iv 0123456789abcdef -in input1.bin -out output1-cbc.aes
~~~~

Aplicamos lo mismo para el segundo archivo y mostramos por pantalla(con xxd) el resultado para compararlo:

![CBC-comparacion](./Ejercicio3/CBC-comparacion.png)

En modo CBC el primer bloque del mensaje recibe solo la suma (XOR) con el IV y se le aplica la clave, pero como el primer mensaje no tiene un antecesor, resulta ser un simple XOR con el IV inicial que elegimos. Por esta razón la primera linea de nuestro output1 y output2 coinciden.
Por la misma razón resultan ser todas las linea consiguientes distintas, ya que al modificarse un bit en la segunda linea ya cambia todo el resultado siguiente.


### modo OFB:

![CBC-teoria](./Ejercicio3/OFB-teoria.png)

Aplicamos el cifrado a los archivos: "input1.bin" y "input2.bin" tal que:
~~~~
openssl aes-256-ofb -K
0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
-iv 0123456789abcdef -in input1.bin -out output1-ofb.aes
~~~~
___Repetir con input2.bin___

Una vez cifrado los archivos mostramos los resultados:

![OFB-comparacion](./Ejercicio3/xxd-aes-256-ofb.png)

En este modo es el vector de inicio el que se ve afectado por la clave, es decir, cuando vaya a cifrar el primer bloque, coge el IV y le aplica la clave, dará un resultado "S1", que se convertirá en el proximo IV para el siguiente bloque, así sucesivamente hasta el final.
Cada resultado de aplicar la clave al IV se aplica en XOR con los distintos bloques de nuestro mensaje.
Es por eso que solo se aprecia una diferencia en la segunda linea , que es donde se encuentra el bit a 1.

## 4. Cifrad input.bin e input1.bin con AES-128 en modo ECB,CBC,OFB usando una contraseña a elegir. Explicad los diferentes resultados.

### ECB

Para realizar un cifrado con contraseña en aes-128-ecb simplemente escribimos en nuestra terminal:
~~~~
openssl aes-128-ecb -in input1.bin -out output1-ecb.aes128

Nos pedirá una contraseña y que la verifiquemos, en mi caso he elegido: 1234
~~~~

Mostramos el resultado de ambos:

![ECB-comparación](./Ejercicio4/xdd-aes-128-ecb.png)

Podemos observar en la primera linea: Salted_ ....
En diferencia a cifrar con una clave y un vector inicial elegido por nosotros, openssl se encarga de elegir una ___clave y vector inicial al azar___ de tal forma que el mismo archivo cifrado 2 veces consecutivas con la misma operación(la escrita en el recuadro de antes) mostrará dos resultados completamente diferentes uno de otro vease este ejemplo en el que se muestran dos resultados de cifrar el mismo archivo input1.bin(todo ceros) como indicamos antes:

![ECB-comparacion-input1](./Ejercicio4/xdd-aes-128-ecb-2.png)

Por lo demás sigue la misma fórmula que con aes-256-ecb

### CBC

Mismo proceso que antes:

![aes-128-cbc](./Ejercicio4/creacion-cbc.png)

Contraseña:1234

Mostramos los resultados de ambos:

![CBC-comparacion](./Ejercicio4/xdd-aes-cbc.png)

Y volvemos a comparar ahora los resultados de input1:

![CBC-comparacion-input1](./Ejercicio4/xdd-aes-128-cbc-2.png)

Comprobamos que al igual que con el ECB, los resultados de cifrar input1 de la misma manera dan como resultado dos cifrados completamente distintos. Aun así, se sigue aplicando CBC con total normalidad.

### OFB

Repetimos proceso pero esta vez con OFB:

![aes-128-ofb](./Ejercicio4/creacion-ofb.png)

Mostramos los resultados de ambos:

![OFB-comparacion](./Ejercicio4/xdd-aes-ofb.png)

Y volvemos a comparar, una vez más, los resultados de input1:

![OFB-comparacion-input1](./Ejercicio4/xdd-aes-128-ofb-2.png)

Y, de nuevo, comprobamos que no tienen nada que ver los resultados generados de input1.

Como conclusión, podemos comprobar que openssl establece su propio iv y su propia clave cada vez que se le manda cifrar un archivo, aunque este sea el mismo, lo que si podemos comprobar es que se cifran tal cual viene explicado en el ejercicio anterior.

## 5 Repetir el punto anterior con la opcion -nosalt.

Con la opción -nosalt , evitamos la aleatoriedad a la hora de cifrar el archivo mediante una contraseña, vamos a comprobarlo:

### ECB

Creación de archivos:

![Creación-ECB](./Ejercicio5/creacion-ecb.png)

Comparamos entre los dos:

![](./Ejercicio5/comparacion-ecb-nosalt.png)

Por último comparamos input1 consigo mismo creandolo de nuevo:

![](./Ejercicio5/comparacion-ecb-input1.png)

### CBC

Creación de archivos:

![Creación-CBC](./Ejercicio5/creacion-cbc.png)

Comparamos entre los dos:

![](./Ejercicio5/comparacion-cbc-nosalt.png)

Por último comparamos input1 consigo mismo creandolo de nuevo:

![](./Ejercicio5/comparacion-cbc-input1.png)

### OFB

Creación de archivos:

![Creación-OFB](./Ejercicio5/creacion-ofb.png)

Comparamos entre los dos:

![](./Ejercicio5/comparacion-ofb-nosalt.png)

Por último comparamos input1 consigo mismo creandolo de nuevo:

![](./Ejercicio5/comparacion-ofb-input1.png)


Podemos observar que, evidentemente, ahora elige la misma clave y el mismo iv para el mismo archivo y ya no existe ningun tipo de aleatoriedad.

## 6. Cifrar input1.bin con aes-192 en modo OFB, clave y vector de inicialización a elegir(no contraseña).Supongamos que la salida es output.bin

La clave en aes-192 deberá de ser de 48 carácteres, pues que 192 bits corresponde a 48 caracteres hexadecimales que ocupan 4 bits cada uno.
Para este ejercicio elegiremos como clave K :
___0123456789abcdef0123456789abcdef0123456789abcdef___

Como vector inicial: ___0123456789abcdef___

Para cifrar el archivo utilizamos:
~~~~
openssl aes-192-ofb -K
0123456789abcdef0123456789abcdef0123456789abcdef
-iv 0123456789abcdef -in input1.bin -out output1.aes
~~~~

![Captura-Pantalla](./Ejercicio6/captura-ofb.png)

Mostramos el resultado:

![Captura-resultado](./Ejercicio6/resultado-ofb.png)

## 7. Descifrar output.bin utilizando la misma clave y vector de inicialización que en el ejercicio 6.

Para descifrar un archivo cifrado con ___AES-XXX___ tan solo hace falta utilizar el parámetro -d, añadiendo además como entrada ___-in "el archivo cifrado"___ y como salida ___-out "el archivo descrifrado"___.
Después para visualizar el archivo, basta con utilizar xxd "el nombre del archivo de salida", a continuación se muestra una imagen de como realizarlo:

![Descrifrado](./Ejercicio7/decrypt.png)

## 8. Vuelve a cifrar output.bin(el archivo cifrado) con aes-192 en modo OFB, clave y vector de inicialización del ejercicio 6. Compara el resultado obtenido con el punto 7, explicando el resultado.

Para cifrar de nuevo el archivo basta con pasarle al output el mismo cifrado tal que:

![creacion](./Ejercicio8/creacion.png)

Comparamos el resultado del ejercicio 7 con el 8:
![mostrar](./Ejercicio8/mostrar7.png)
![mostrar](./Ejercicio8/mostrar.png)

Podemos observar que cuando a un archivo ___A___ le aplicamos un cifrado obtenemos ___A'___, y a partir de aqui,tanto si desciframos como si ciframos ___A'___ obtenemos ___A___.
Esto es debido a que el cifrado por OFB realiza la operacion tal que:
~~~~
  IV --> So --> S1 = Ek(So) --> ..... S2 --> .....  Sn
                      |xor            |xor          |xor
                      M1              M2            Mn
~~~~
De tal forma que al aplicarlo sobre nuestro mensaje el cual está a cero en todos los bits resulta que nuestro archivo output esta formado por los distintos bloques de la forma:
~~~~
  S1,S2,...Sn ; Es decir, el resultado de nuestro archivo cifrado es el vector
  inicial aplicandole la clave(S1), S1 aplicandole la clave(S2),....,Sn-1
  aplicandole la clave(Sn)
~~~~
Por tanto, al volver a aplicar, con el mismo IV y clave la operacón XOR al bloque cifrado obtenemos:
~~~~
  S1 xor S1 = 0
  S2 xor S2 = 0
  ....
  Sn xor Sn = 0
~~~~
Esta es la razón por la que al descifrar y al cifrar el primer archivo output obtenemos el mismo resultado.


## 9. Repetir los puntos 6,7 y 8 pero empleando contraseña en lugar de clave y vector de inicialización.

Como contraseña elegiremos al igual que en ejercicios anteriores: 1234

1. Empezaremos realizando el apartado 6(Cifrar input1.bin con aes-192 en modo OFB con pass):

![Creacion-aes-192-pass](./Ejercicio9/output1ofbpass.png)

2. En el apartado 7(Descriframos utilizando la pass del ejercicio 6)

![decrypt](./Ejercicio9/decrypt.png)
![resultado](./Ejercicio9/show.png)

3. Apartado del ejercicio 8(cifrar de nuevo la salida anterior con la misma contraseña,1234):

![decrypt](./Ejercicio9/doble_cifrado.png)
![resultado](./Ejercicio9/show_doble_cifrado.png)

Esta vez, podemos ver que el resultado no es el mismo, esto es por la aleatoriedad a la hora de elegir el IV y la clave al cifrar los archivos, en consecuencia, la primera vez se realizo el XOR con un IV y una Clave, que después fueron distintas, como consecuencia, el resultado es el obtenido.

## 10. Presentar otro algoritmo de cifrado simétrico que aparezca en mi implementación de openssl

#### Camellia

Camellia es un cifrado simétrico cuyo tamaño de bloque es de 128 bits. Y cuya longitud de clave es de 128,192 y 256 bits.
Este cifrado tiene un nivel y habilidad de proceso comparable a AES.

Si nos preguntamos entonces:¿Por qué utilizar AES y no Camellia si son tan parecidos? La respuesta la podemos encontrar en este artículo: [Camellia](https://www.jmest.org/wp-content/uploads/JMESTN42351298.pdf)

En él se nos muestra una tabla de velocidad en función de los distintos tamaños de bloques que hay. Y aunque las primeras medias no son muy lejanas entre ellas, podemos observar como los tiempos se multiplican por 4 en el caso de Camellia, dejando a AES muy por encima en eficiencia. Aunque ambos se consideran a un nivel equiparable el uno del otro a la hora de hablar de seguridad.


Camellia es parte de la TLS, la capa de transporte, es un protrocolo de criptografía diseñado para proporcionar una comunicación segura entre una red de ordenadores, como puede ser internet.

## 11. Repetir el ejercicico 3,4 y 5 con el cifrado presentado en el ejercicio 10

####1. Cifraremos input1.bin e input2.bin con Camellia-256 en los modos ECB,CBC y OFB usando una clave del tamaño adecuado y con un IV inicializado a "0123456789abcdef"

##### ECB

Usaremos la orden:
~~~~
openssl camellia-256-ecb -K
0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
-iv 0123456789abcdef -in input1.bin -out output1-ecb.camelia

Al igual que con AES, nos advierte que el IV no se utilizará para cifrar con ECB
~~~~

![resultado](./Ejercicio11/cam-ecb.png)

Mostraremos ahora el resultado del cifrado con Camellia:

![mostramos](./Ejercicio11/show-ecb.png)

Comparamos con el de AES:

![aes](./Ejercicio11/xdd-aes-256-ecb.png)

Ya podemos comprobar que aunque el cifrado es similar, no es idéntico. Tengamos en cuenta que todos los parámetros han sido idénticos, tanto la clave, como el IV, como el archivo binario "input1.bin" del que partíamos inicialmente.

Cifremos ahora el input2.bin siguiendo los pasos anteriores y mostramos el resultado del archivo:

![mostramos](./Ejercicio11/ecb-camelia-2.png)

Comprobamos que sigue ocurriendo al igual que antes con AES.


##### CBC

Usaremos la orden:
~~~~
openssl camellia-256-cbc -K
0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
-iv 0123456789abcdef -in input1.bin -out output1-cbc.camelia

~~~~

![resultado](./Ejercicio11/cam-cbc.png)

Mostraremos ahora el resultado del cifrado con Camellia:

![mostramos](./Ejercicio11/show-cbc.png)
![mostramos](./Ejercicio11/show-cbc-2.png)

Comparamos con el de AES:

![aes](./Ejercicio11/CBC-comparacion.png)

Efectivamente ocurre lo mismo que con ECB, vamos a comprobar ahora con OFB

##### OFB

Usaremos la orden:
~~~~
openssl camellia-256-ofb -K
0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
-iv 0123456789abcdef -in input1.bin -out output1-ofb.camelia
~~~~

![resultado](./Ejercicio11/cam-ofb.png)

Mostraremos ahora el resultado del cifrado con Camellia:

![mostramos](./Ejercicio11/show-ofb.png)
![mostramos](./Ejercicio11/show-ofb-2.png)

Comparamos con el de AES:

![aes](./Ejercicio11/OFB-comparacion.png)

Efectivamente ocurre lo mismo que con CBC, queda demostrado que Camelia funciona de manera similar que AES al menos con los parámetros elegidos de IV y la clave.

####2. Cifrad input1.bin e input2.bin con camelia-128 en modo ECB,CBC y OFB usando contraseña a elegir. Explicar los diferentes resultados.

Ciframos con el comando:
~~~~
openssl camellia-128-ecb -in input1.bin -out output1-128-ecb.camelia

Utilizaremos la contraseña: 1234
~~~~

##### ECB

![Creacion-ECB](./Ejercicio11/creacion-ecb.png)

##### CBC

![Creacion-CBC](./Ejercicio11/creacion-cbc.png)

##### OFB

![Creacion-OFB](./Ejercicio11/creacion-ofb.png)


___Comparamos los resultados de los tres:___

##### ECB

![Comparamos-ECB](./Ejercicio11/comp-ecb.png)

##### CBC

![Comparamos-CBC](./Ejercicio11/comp-cbc.png)

##### OFB

![Comparamos-OFB](./Ejercicio11/comp-ofb.png)

Para hacer una comparación como en el ___ejercicio 4___ se han creado otra vez los input1 en los 3 modos(ECB,CBC Y OFB) tal que:

![Creacion-de-nuevo](./Ejercicio11/creacion-otra.png)

Y los mostramos:

![mostramos](./Ejercicio11/ecb-1-again.png)
![mostramos](./Ejercicio11/ofb-1-again.png)

Así podemos comprobar que ocurre al igual que con AES, los resultados son diferentes cada vez que aplicamos el cifrado con password, puesto que cada vez que ciframos los valos de la clave y el IV son ___aleatorios___.

####3. Repetir el ejercicio 4 pero usando el parámetro -nosalt.

Crearemos los archivos cifrados con el parámetros -nosalt:
~~~~
openssl camellia-128-ecb -nosalt -in input1.bin -out output1-128-ecb.camelia

Utilizaremos la contraseña: 1234
~~~~
![](./Ejercicio11/ecb-nosalt-creacion.png)

Crearemos también el CBC y el OFB:

![](./Ejercicio11/cbc-nosalt-creacion.png)
![](./Ejercicio11/ofb-nosalt-creacion.png)

Hemos creado la segunda version de la creacion para hacer ahora la comparación, para comprobar si pasa lo mismo que en el ___ejercicio 4___ con AES o no:

#####ECB
![](./Ejercicio11/mostrar-ecb-nosalt.png)
#####CBC
![](./Ejercicio11/mostrar-cbc-nosalt.png)
#####OFB
![](./Ejercicio11/mostrar-ofb-nosalt.png)

En conclusión, podemos decir, que al igual que con AES, utilizar el parámetro -nosalt, produce que los resultados sean siempre los mismos, es decir, la clave y el IV serán los mismos para el mismo objeto.
