# Barrera-post1-u3

Checkpoint 2:
La salida del comando D tiene tres columnas. La primera es la dirección de memoria en formato segmento:desplazamiento (por ejemplo 0724:0200), donde 0724 es el 
segmento y 0200 es el desplazamiento del primer byte de esa línea; en cada línea siguiente el desplazamiento aumenta en 10h porque se muestran 16 bytes por fila. La 
segunda columna contiene esos 16 bytes en hexadecimal, separados por espacios, con un guion entre el octavo y el noveno byte para facilitar la lectura; aquí se ve el 
patrón AB CD EF repetido, que es lo que se escribió con el comando F. La tercera columna es la representación en ASCII de cada byte, donde los caracteres imprimibles 
se muestran como letras o símbolos y los que no lo son se reemplazan por un punto. En este caso toda la columna aparece como puntos (................) porque los 
valores AB, CD y EF no corresponden a caracteres ASCII imprimibles.
