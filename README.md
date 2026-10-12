# Barrera-post1-u3

Parte 1:
Checkpoint 2:
La salida del comando D tiene tres columnas. La primera es la dirección de memoria en formato segmento:desplazamiento (por ejemplo 0724:0200), donde 0724 es el 
segmento y 0200 es el desplazamiento del primer byte de esa línea; en cada línea siguiente el desplazamiento aumenta en 10h porque se muestran 16 bytes por fila. 
La segunda columna contiene esos 16 bytes en hexadecimal, separados por espacios, con un guion entre el octavo y el noveno byte para facilitar la lectura; aquí se 
ve el patrón AB CD EF repetido, que es lo que se escribió con el comando F. La tercera columna es la representación en ASCII de cada byte, donde los caracteres 
imprimibles se muestran como letras o símbolos y los que no lo son se reemplazan por un punto. En este caso toda la columna aparece como puntos (................) 
porque los valores AB, CD y EF no corresponden a caracteres ASCII imprimibles.

Checkpoint Decisión técnica:
Para confirmar que el Paso 11 dejó 78 56 en 0724:0300 y 0724:0301, necesito leer la memoria sin cambiar nada, y entre R, E, F y D el único que cumple esto es D.

Invocar E 300 sin lista de bytes no es seguro porque DEBUG entra en modo interactivo: muestra el valor actual del byte y espera una entrada. Si por error escribo 
un valor o pulso una tecla equivocada, sobrescribo ese byte, así que la verificación se convierte en una nueva modificación. En cambio, E 300 78 56 con la lista 
completa se ejecuta de una vez, escribe exactamente esos dos bytes y termina sin pedir nada más. Esa es la escritura que quiero comprobar, no una forma de
comprobarla.

F tampoco sirve, aunque trabaje sobre un rango igual que D, porque su función es escribir: rellena todo el rango con el patrón indicado. Usarlo para “ver” la 
región destruiría justamente los bytes 78 56 que quiero confirmar y los reemplazaría por el patrón nuevo.

R trabaja con los registros de la CPU, no con la memoria, así que no muestra el contenido de DS:0300. Además, R seguido de un nombre de registro (por ejemplo R 
AX) permite cambiar su valor, de modo que tampoco es una herramienta puramente de consulta.

D, en cambio, solo lee y muestra el contenido de la memoria (bytes en hexadecimal y su representación ASCII) sin escribir nada ni alterar registros. Puedo 
repetirlo cuantas veces quiera con el mismo resultado, y por eso es el comando adecuado para verificar sin riesgo.

Checkpoint decisión técnica:
Aunque MOV AX,0005 (B8 05 00) y MOV AX,[0300] (A1 00 03) ocupan los mismos 3 bytes, el modo directo a memoria es el que requiere un ciclo adicional de acceso al 
bus. En el inmediato, el valor 0005 viaja dentro del propio flujo de instrucción: el procesador lo obtiene al leer la instrucción y no necesita ir a buscar nada 
más. En el directo, los bytes 00 03 no son el dato sino una dirección (0300); el procesador debe resolverla y hacer una lectura adicional en DS:0300 para traer el 
valor que irá a AX. Por eso, en un procesador como el 8086, la forma inmediata es más rápida que la de memoria.

Preferiría direccionamiento directo a memoria cuando el valor puede cambiar en tiempo de ejecución y no es una constante conocida al ensamblar. Es el caso del 
Paso 11: los bytes 78 56 se escribieron en 0300 con el comando E, y la instrucción MOV AX,[0300] siempre lee lo que haya ahí en ese momento. Con inmediato, el 
valor quedaría fijo dentro del código y habría que reensamblar la instrucción para cambiarlo.

Con U puedo confirmar la codificación sin ejecutar nada: el desensamblado muestra los bytes de cada instrucción junto a su mnemónico. U 320 324 muestra A10003 
para el modo directo, mientras que el programa del Paso 10 mostraba B80500 para el inmediato, lo que permite distinguir los dos modos por su opcode (A1 frente a 
B8) y por lo que contienen sus operandos.

Parte 2:

Checkpoint 1:
Tabla de trazas completa:
Instrucción	AX	BX	CX	IP siguiente	ZF	CF	SF
MOV AX,000A	000A	0000	0000	0103	0 (NZ)	0 (NC)	0 (PL)
MOV BX,0005	000A	0005	0000	0106	0 (NZ)	0 (NC)	0 (PL)
MOV CX,0003	000A	0005	0003	0109	0 (NZ)	0 (NC)	0 (PL)
ADD AX,BX	000F	0005	0003	010B	0 (NZ)	0 (NC)	0 (PL)
ADD AX,CX	0012	0005	0003	010D	0 (NZ)	0 (NC)	0 (PL)

Checkpoint 2:
Tabla de trazado completa:
Iteración	Instrucción	AX después	CX después	IP siguiente	¿LOOP salta?
0	MOV AX,0000	0000	0000	0103	no
0	MOV CX,0004	0000	0004	0106	no
1	ADD AX,+02	0002	0004	0109	no
1	LOOPW 0106	0002	0003	0106	Sí (CX ≠ 0)
2	ADD AX,+02	0004	0003	0109	no
2	LOOPW 0106	0004	0002	0106	Sí (CX ≠ 0)
3	ADD AX,+02	0006	0002	0109	no
3	LOOPW 0106	0006	0001	0106	Sí (CX ≠ 0)
4	ADD AX,+02	0008	0001	0109	no
4	LOOPW 0106	0008	0000	010B	no (CX = 0)

Checkpint:
Para este bucle contador simple, donde CX solo sirve para contar las iteraciones, recomiendo LOOP. En bytes, LOOP 0106 ocupa 2 (E2 FB) frente a los 3 de DEC CX + 
JNZ 0206 (49 75 FA), así que el programa completo ocupa 13 bytes con LOOP y 14 con DEC/JNZ. En instrucciones, LOOP hace el control del bucle con una sola 
instrucción por iteración, mientras que DEC/JNZ obliga al procesador a extraer y ejecutar dos. Menos código y menos instrucciones por vuelta hacen que LOOP sea la 
opción más compacta y directa cuando la condición de salida es simplemente CX ≠ 0.

Preferiría DEC/JNZ cuando el control del bucle necesite algo que LOOP no ofrece. LOOP solo trabaja con CX y no modifica las banderas, mientras que DEC sí 
actualiza ZF, SF y otras, y puede aplicarse a cualquier registro. Por eso lo usaría si el cuerpo del bucle necesitara CX para otra operación aritmética y el 
contador tuviera que ir en otro registro, o si la condición de salida no fuera “CX ≠ 0” sino el resultado de una comparación con CMP. También conviene tener 
presente que en procesadores modernos LOOP suele ser más lento que DEC/JNZ, así que el ahorro de bytes no siempre implica ahorro de tiempo.

Para comparar el tamaño sin ejecutar nada, usaría U sobre ambos programas. En U 100 10B la última instrucción (INT 20) empieza en 010B y el programa termina en 
010C, es decir, ocupa 13 bytes. En U 200 20D termina en 020D, es decir, 14 bytes. La diferencia de un byte se ve directamente en las direcciones y en la columna 
de código máquina.

Checkpoint:
Con CX = 0x0064, G 20C es más práctico que T porque ejecuta las 100 iteraciones a velocidad completa y se detiene en 020C, mientras que T exigiría unas 300 
pulsaciones con sus volcados de registros. Lo que se pierde es el detalle intermedio: no se ven los valores de AX, CX y las banderas en cada instrucción, solo el 
estado final. Por eso T fue realmente necesario en el Paso 11, donde había que completar una tabla de traza instrucción por instrucción, una granularidad que solo 
T ofrece. Después de G 20C, usaría el comando R, que muestra los registros sin modificarlos y permite confirmar AX = 0008 sin haber observado ningún paso 
intermedio.

Checkpoint 3:
Iteración	Instrucción	AX después	CX después	IP siguiente	¿JNZ salta?
0	MOV AX,0000	0000	0002*	0203	no
0	MOV CX,0004	0000	0004	0206	no
1	ADD AX,+02	0002	0004	0209	no
1	DEC CX	0002	0003	020A	no
1	JNZ 0206	0002	0003	0206	Sí (ZF = 0)
2	ADD AX,+02	0004	0003	0209	no
2	DEC CX	0004	0002	020A	no
2	JNZ 0206	0004	0002	0206	Sí (ZF = 0)
3	ADD AX,+02	0006	0002	0209	no
3	DEC CX	0006	0001	020A	no
3	JNZ 0206	0006	0001	0206	Sí (ZF = 0)
4	ADD AX,+02	0008	0001	0209	no
4	DEC CX	0008	0000	020A	no
4	JNZ 0206	0008	0000	020C	no (ZF = 1, ZR)
0	INT 20	0008	0000	-	Fin del programa

Versión	Preparación	Cuerpo (4 iteraciones)	INT 20	Total
LOOP	2	2 × 4 = 8	1	11
DEC/JNZ	2	3 × 4 = 12	1	15

Decisión técnica del Paso 10:
Para este bucle contador simple recomiendo LOOP. En código máquina, LOOP 0106 ocupa 2 bytes (E2 FB), mientras que DEC CX + JNZ 0206 ocupan 3 (49 75 FA), de modo 
que el programa completo pasa de 13 a 14 bytes. En ejecución, LOOP necesita una instrucción por iteración para controlar el bucle y DEC/JNZ necesita dos, lo que 
se refleja en el conteo total: 11 instrucciones ejecutadas con LOOP frente a 15 con DEC/JNZ. Preferiría DEC/JNZ si el contador no pudiera ser CX, si el cuerpo del 
bucle necesitara CX para otra operación aritmética o si la condición de salida dependiera de una comparación con CMP, porque DEC actualiza las banderas y funciona 
con cualquier registro, mientras que LOOP solo usa CX y no modifica banderas. Con U se compara sin ejecutar nada: el programa con LOOP va de 0100 a 010C y el de 
DEC/JNZ de 0200 a 020D, es decir, 13 y 14 bytes respectivamente.

Decisión técnica del Paso 12:
Con CX = 0x0064, G 20C es más práctico que T porque ejecuta las 100 iteraciones a velocidad completa y se detiene en la dirección 020C, mientras que T obligaría a 
unas 300 pulsaciones, cada una con su volcado de registros que nadie leería. Lo que se pierde al usar G es el detalle intermedio: no se observan AX, CX, IP ni las 
banderas en cada instrucción, solo el estado final al llegar al punto de parada. Por eso T fue realmente necesario en el Paso 11, donde había que completar una 
tabla de traza instrucción por instrucción, una granularidad que solo T ofrece. Después de G 20C, el estudiante usaría el comando R sin argumentos, que solo 
muestra los registros sin modificarlos, para confirmar que AX = 0008 sin haber observado ningún paso intermedio.
