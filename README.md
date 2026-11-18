# Laboratorio Post-contenido: Manejo del DEBUG — Exploración, Ensamblado y Ejecución Paso a Paso en DOSBox

**Asignatura:** Arquitectura de Computadores  
**Unidad:** Unidad 3 · Arquitectura del Procesador y Modos de Direccionamiento  
**Estudiante:** Eduard Cantillo  
**Institución:** Universidad Francisco de Paula Santander (UFPS)  
**Entorno de Ejecución:** DOSBox 0.74-3+ con el depurador estándar `DEBUG.EXE` en modo real x86 (16 bits)

---

## 📋 Resumen del Repositorio

El presente repositorio consolida la ejecución práctica y el análisis documental del laboratorio de la Unidad 3. En esta actividad se interactúa de manera directa con los registros internos de la CPU, la memoria segmentada y las instrucciones en código máquina de la arquitectura x86 utilizando la herramienta `DEBUG` sobre el emulador **DOSBox**.

El trabajo se estructura en dos partes fundamentales:
1. **Parte 1 (Exploración con DEBUG en DOSBox):** Inspección del estado inicial del procesador con `R`, inicialización de bloques con `F`, análisis de volcados hexadecimales con `D`, exploración del Program Segment Prefix (PSP), desensamblado con `U`, modificación selectiva de memoria con `E` y verificación de direccionamiento directo a memoria (`MOV AX, [dirección]`).
2. **Parte 2 (Ensamblado y Ejecución Paso a Paso):** Ensamblado en memoria con `A`, ejecución secuencial paso a paso con `T` registrando tablas de traza analíticas, estudio del mecanismo de bucle por hardware `LOOP` junto con la aritmética de saltos relativos, comparación técnica contra un bucle equivalente implementado con `DEC` y `JNZ`, y evaluación de la ejecución a velocidad completa con puntos de interrupción mediante `G`.

---

## 📂 Estructura del Repositorio

```plaintext
cantillo-post1-u3/
├── capturas/
│   ├── README.md                      ; Descripción de las capturas de pantalla requeridas
│   ├── CP1_registros.png              ; Parte 1 - Checkpoint 1: Estado inicial y modificación con R
│   ├── CP2_volcado_memoria.png        ; Parte 1 - Checkpoint 2: Volcado D 200 L40 con patrón AB CD EF
│   ├── CP3_ensamblado_desensamblado.png ; Parte 1 - Checkpoint 3: Ensamblado con A y desensamblado con U
│   ├── CP4_memoria_direccionamiento.png ; Parte 1 - Checkpoint 4: Modificación E y direccionamiento [0300]
│   ├── CP1_traza_suma.png             ; Parte 2 - Checkpoint 1: Traza completa del programa de suma
│   ├── CP2_traza_loop.png             ; Parte 2 - Checkpoint 2: Traza completa del bucle LOOP
│   ├── CP3_traza_dec_jnz.png          ; Parte 2 - Checkpoint 3: Traza bucle DEC/JNZ y ejecución con G
│   └── CP4_resumen_final.png          ; Parte 2 - Checkpoint 4: Resumen general de la sesión en DOSBox
└── README.md                          ; Documentación técnica completa, trazas y decisiones técnicas
```

---

## 🛠️ Comandos de la Herramienta DEBUG

El depurador `DEBUG` opera mediante un intérprete de línea de comandos identificado por el prompt guion (`-`). A continuación se detallan los comandos fundamentales trabajados en el laboratorio:

| Comando | Sintaxis General | Descripción y Función en el Laboratorio |
| :---: | :--- | :--- |
| **`R`** | `R [registro]` | **Registers:** Inspecciona todos los registros y banderas, o permite modificar el valor de un registro específico (ej. `R AX`). |
| **`D`** | `D [segmento:]offset [longitud]` | **Dump:** Vuelca el contenido de la memoria en formato hexadecimal y ASCII sin alterar su contenido (solo lectura). |
| **`F`** | `F inicio longitud lista_bytes` | **Fill:** Rellena un rango contiguo de memoria repitiendo de manera cíclica un patrón de bytes. |
| **`E`** | `E dirección [lista_bytes]` | **Enter:** Escribe bytes puntuales en la memoria a partir de una dirección determinada, o permite editarlos de forma interactiva. |
| **`U`** | `U [inicio] [fin / longitud]` | **Unassemble:** Desensambla el contenido de memoria traduciendo los bytes de código máquina a mnemónicos legibles x86. |
| **`A`** | `A [dirección]` | **Assemble:** Ensambla instrucciones directamente en memoria a partir de una dirección dada (por defecto `CS:0100`). |
| **`T`** | `T [=dirección] [instrucciones]` | **Trace:** Ejecuta paso a paso un número determinado de instrucciones, mostrando el estado de registros y banderas tras cada paso. |
| **`G`** | `G [=inicio] [parada]` | **Go:** Ejecuta el programa a velocidad nativa sin interrupciones hasta alcanzar la dirección de parada (*breakpoint*). |
| **`Q`** | `Q` | **Quit:** Finaliza la sesión de depuración y regresa al prompt del sistema operativo DOS. |

---

# PARTE 1: Exploración con DEBUG en DOSBox

### Paso 1 a 3: Configuración del Entorno
Se inicia DOSBox montando la unidad de trabajo y accediendo a la carpeta del laboratorio:
```dos
Z:\> MOUNT C C:\DOSWork
Drive C is mounted as local directory C:\DOSWork\
Z:\> C:
C:\> MD LAB3POST1
C:\> CD LAB3POST1
C:\LAB3POST1> DEBUG
-
```

---

### Paso 4 y 5: Inspección y Modificación de Registros (`R`)
Al invocar `R` sin parámetros se revela el estado inicial del procesador:
```dos
-R
AX=0000  BX=0000  CX=0000  DX=0000  SP=FFFE  BP=0000  SI=0000  DI=0000
DS=1357  ES=1357  SS=1357  CS=1357  IP=0100   NV UP EI PL NZ NA PO NC
1357:0100 CD20          INT     20
```

* **Registros de Propósito General:** `AX`, `BX`, `CX` y `DX` inician en `0000h`.
* **Puntero de Pila (`SP`):** Apunta a `FFFEh`, correspondiente al límite superior del segmento para albergar la pila del sistema.
* **Registros de Segmento (`DS`, `ES`, `SS`, `CS`):** Todos apuntan al mismo párrafo base (`1357h`), propio del modelo de memoria COM (tiny model).
* **Puntero de Instrucción (`IP`):** Situado en `0100h`, justo después de los 256 bytes (`100h`) reservados para el PSP.
* **Banderas:** Indican estado por defecto: No Overflow (`NV`), Up direction (`UP`), Enable Interrupts (`EI`), Plus (`PL`), Not Zero (`NZ`), No Auxiliary Carry (`NA`), Parity Odd (`PO`), No Carry (`NC`).

Para modificar selectivamente `AX` a `1234h`:
```dos
-R AX
AX 0000
:1234
-R
AX=1234  BX=0000  CX=0000  DX=0000  SP=FFFE  BP=0000  SI=0000  DI=0000 ...
```
> **Checkpoint 1:** Evidencia guardada en `capturas/CP1_registros.png`.

---

### Paso 6 a 8: Relleno con `F`, Volcado con `D` y Análisis del PSP

Se inicializa un bloque de 64 bytes (`40h`) a partir de `DS:0200` con el patrón cíclico `AB CD EF`:
```dos
-F 200 L40 AB CD EF
-D 200 L40
1357:0200  AB CD EF AB CD EF AB CD-EF AB CD EF AB CD EF AB  ................
1357:0210  CD EF AB CD EF AB CD EF-AB CD EF AB CD EF AB CD  ................
1357:0220  EF AB CD EF AB CD EF AB-CD EF AB CD EF AB CD EF  ................
1357:0230  AB CD EF AB CD EF AB CD-EF AB CD EF AB CD EF AB  ................
```

#### Descripción de las Columnas de la Salida del Comando `D` (Checkpoint 2):
La salida del comando `D` se compone de tres secciones visuales claramente definidas:
1. **Columna de Dirección (`Segmento:Offset`):** Ubicada a la izquierda (ej. `1357:0200`), indica en formato hexadecimal la dirección lógica base de memoria correspondiente al primer byte de dicha línea de visualización.
2. **Área Central Hexadecimal:** Presenta 16 valores hexadecimales ordenados de izquierda a derecha correspondientes a los 16 bytes consecutivos en memoria. Un guion central (`-`) divide el bloque en dos grupos de 8 bytes para facilitar la lectura visual y la rápida alineación de desplazamientos.
3. **Columna Derecha ASCII:** Proyecta la interpretación alfanumérica de cada byte bajo la tabla de caracteres estándar (rango imprimible de `20h` a `7Eh`). Aquellos valores numéricos que no tienen representación imprimible o son códigos de control se muestran con un punto (`.`), tal como sucede con los bytes `ABh`, `CDh` y `EFh`.

> **Checkpoint 2:** Evidencia guardada en `capturas/CP2_volcado_memoria.png`.

#### Exploración del PSP (Program Segment Prefix):
```dos
-D 0 L20
1357:0000  CD 20 FF 9F 00 9A F0 FE-1D F0 4F 03 B1 05 8A 05  .. .......O.....
1357:0010  B1 05 01 01 01 00 02 FF-FF FF FF FF FF FF FF FF  ................
```
Los dos primeros bytes en el offset `0000h` son `CD 20`, correspondientes a la instrucción `INT 20h`. El DOS ubica esta instrucción al inicio del PSP como salvaguarda de terminación: si un programa COM ejecuta una instrucción de retorno `RET` cuando la pila apunta al inicio, el flujo se desvía a `0000h` y ejecuta `INT 20h`, finalizando el proceso ordenadamente.

---

### Paso 9 y 10: Desensamblado con `U` y Ensamblado de Código
Al desensamblar la memoria antes de ingresar código:
```dos
-U 100 L10
1357:0100 CD20          INT     20
1357:0102 0000          ADD     [BX+SI],AL
1357:0104 0000          ADD     [BX+SI],AL
```
Los bytes nulos `00 00` son decodificados como `ADD [BX+SI],AL`. Esto evidencia un principio clave del modo real: la memoria es lineal y no distingue intrínsecamente entre datos e instrucciones; el procesador interpretará cualquier byte como código ejecutable si `CS:IP` apunta a él.

Se ensambla un programa elemental de 4 instrucciones:
```dos
-A 100
1357:0100 MOV AX, 0005
1357:0103 MOV BX, 0003
1357:0106 ADD AX, BX
1357:0108 INT 20
1357:010A
-U 100 109
1357:0100 B80500        MOV     AX,0005
1357:0103 BB0300        MOV     BX,0003
1357:0106 03C3          ADD     AX,BX
1357:0108 CD20          INT     20
```
* `MOV AX, 0005`: Codificado como `B8 05 00` (Opcode `B8` + inmediato de 16 bits en little-endian: byte bajo `05`, byte alto `00`). Ocupa 3 bytes.
* `MOV BX, 0003`: Codificado como `BB 03 00` (Opcode `BB` + inmediato `0003h`). Ocupa 3 bytes.
* `ADD AX, BX`: Codificado como `03 C3` (Opcode `03` + byte ModR/M `C3`). Ocupa 2 bytes.
* `INT 20`: Codificado como `CD 20`. Ocupa 2 bytes. Total: 10 bytes.

> **Checkpoint 3:** Evidencia guardada en `capturas/CP3_ensamblado_desensamblado.png`.

---

### Paso 11 a 14: Modificación con `E` y Direccionamiento Directo

Se inicializa un bloque en `DS:0300` con ceros y se escriben dos bytes puntuales con `E`:
```dos
-F 300 L10 00
-D 300 L10
1357:0300  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................
-E 300 78 56
-D 300 L10
1357:0300  78 56 00 00 00 00 00 00-00 00 00 00 00 00 00 00  xV..............
```
Los bytes `78h` y `56h` almacenados en `0300h` y `0301h` representan, bajo la convención **little-endian** de la arquitectura x86, el valor entero de 16 bits `0x5678` (byte menos significativo primero).

Se ensambla la instrucción de lectura con direccionamiento directo a memoria en `0320h`:
```dos
-A 320
1357:0320 MOV AX, [0300]
1357:0323 INT 20
1357:0325
-U 320 324
1357:0320 A10003        MOV     AX,[0300]
1357:0323 CD20          INT     20
```
Al ejecutar con `T` estableciendo `IP = 0320`:
```dos
-R IP
IP 0100
:320
-T
AX=5678  BX=0000  CX=0000  DX=0000  SP=FFFE  BP=0000  SI=0000  DI=0000
DS=1357  ES=1357  SS=1357  CS=1357  IP=0323   NV UP EI PL NZ NA PO NC
1357:0323 CD20          INT     20
```
El registro `AX` se carga exactamente con `5678h`, validando la lectura de la memoria física.

> **Checkpoint 4:** Evidencia guardada en `capturas/CP4_memoria_direccionamiento.png`.

---

## 🧠 Decisiones Técnicas Justificadas (Parte 1)

### Decisión Técnica 1: Verificación No Destructiva de una Escritura en Memoria
> **Pregunta:** ¿Cuál de los cuatro comandos (`R`, `E`, `F`, `D`) es el apropiado para confirmar que la escritura de memoria del Paso 11 tuvo el efecto esperado sin arriesgar una modificación accidental del estado del programa?

**Justificación técnica:**  
En el proceso de inspección y depuración de software en bajo nivel, la integridad del estado del procesador y de la memoria depende estrictamente de elegir herramientas de solo lectura para las etapas de verificación. El comando `D` (Dump) constituye la única opción segura y apropiada entre los cuatro comandos disponibles para comprobar que la modificación realizada en `DS:0300` fue exitosa. Invocar el comando `E` sin la lista explícita de argumentos sitúa al depurador en un modo interactivo donde se imprime en pantalla el valor actual del byte y se suspende la ejecución a la espera de una entrada del operador; en este modo, cualquier pulsación fortuita del teclado o el ingreso erróneo de caracteres sobreescribe inmediatamente la celda de memoria antes de avanzar a la siguiente posición, introduciendo mutaciones no deseadas en el espacio de trabajo. Por su parte, el comando `F` (Fill) tampoco es admisible como método de verificación, dado que su propósito operativo es netamente destructivo y de escritura masiva, diseñado para sobrescribir bloques enteros de memoria con un patrón fijo y no para consultar valores existentes. El comando `R` se restringe al banco de registros del procesador y no permite inspeccionar celdas generales de memoria RAM. En contraposición, `D` garantiza una lectura pura y no invasiva: consulta las celdas direccionadas, las formatea en pantalla en representación hexadecimal y ASCII, y preserva intactos tanto el contenido de la memoria como los registros y las banderas del procesador.

---

### Decisión Técnica 2: Modo de Direccionamiento Inmediato vs. Directo a Memoria
> **Pregunta:** Comparando `MOV AX, 0005` (`B8 05 00`) con `MOV AX, [0300]` (`A1 00 03`), ¿cuál requiere un ciclo adicional de acceso al bus de memoria, en qué escenarios se prefiere cada uno y cómo se verifican con `U` sin ejecutar el código?

**Justificación técnica:**  
A pesar de que ambas instrucciones comparten exactamente la misma longitud física de tres bytes en memoria, su comportamiento ante el bus del sistema y sus ciclos de temporización son sustancialmente disímiles debido al modo de direccionamiento implementado. La instrucción `MOV AX, [0300]` con direccionamiento directo a memoria requiere obligatoriamente un ciclo de bus adicional durante su fase de ejecución (*memory read bus cycle*). En la instrucción inmediata `MOV AX, 0005`, el operando fuente viaja embebido dentro del propio flujo de bytes de la instrucción (código de operación `B8` seguido de `05 00`); la Unidad de Interfaz con el Bus (BIU) extrae estos bytes mediante la cola de prebúsqueda (*prefetch queue*) y la Unidad de Ejecución (EU) carga el dato directamente en `AX` sin requerir nuevas transacciones en el bus externo. En cambio, en `MOV AX, [0300]`, los dos bytes finales representan un desplazamiento lineal (`0300h`); una vez decodificada la instrucción, la EU ordena a la BIU realizar un ciclo de lectura sobre el bus externo colocando la dirección efectiva para transferir la palabra desde la memoria RAM hacia el registro `AX`. El direccionamiento directo a memoria es indispensable en escenarios donde los datos son dinámicos, cambian en tiempo de ejecución o residen en estructuras de almacenamiento variables, como buffers de entrada o celdas modificadas previamente con `E`. El direccionamiento inmediato se reserva para constantes matemáticas invariables ya conocidas en tiempo de compilación. Con el comando `U` de `DEBUG`, el estudiante confirma sin ejecutar el código cuál modo fue generado observando los códigos de operación: `B8` identifica una carga inmediata en `AX`, mientras que `A1` denota la optimización de código de máquina para la carga directa del acumulador desde una dirección de memoria.

---

# PARTE 2: Ensamblado y Ejecución Paso a Paso

### Paso 1 a 4: Programa de Suma con Traza Completa

Se ingresa en `CS:0100` un programa que acumula tres sumandos en `AX`:
```dos
-A 100
1357:0100 MOV AX, 000A
1357:0103 MOV BX, 0005
1357:0106 MOV CX, 0003
1357:0109 ADD AX, BX
1357:010B ADD AX, CX
1357:010D INT 20
1357:010F
-U 100 10E
1357:0100 B80A00        MOV     AX,000A
1357:0103 BB0500        MOV     BX,0005
1357:0106 B90300        MOV     CX,0003
1357:0109 03C3          ADD     AX,BX
1357:010B 03C1          ADD     AX,CX
1357:010D CD20          INT     20
```

Se ejecuta paso a paso con el comando `T`, obteniendo la traza detallada:

#### Tabla de Traza 1: Programa de Suma de Tres Operandos
| Paso | Instrucción Ejecutada | AX | BX | CX | IP Sig. | ZF | CF | SF | Observaciones del Estado |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **Inicio** | *(Estado antes de ejecutar)* | `0000` | `0000` | `0000` | `0100` | NZ | NC | PL | Registros en cero tras inicialización |
| **1** | `MOV AX, 000A` | `000A` | `0000` | `0000` | `0103` | NZ | NC | PL | Se carga 10 decimal en AX (no altera banderas) |
| **2** | `MOV BX, 0005` | `000A` | `0005` | `0000` | `0106` | NZ | NC | PL | Se carga 5 decimal en BX |
| **3** | `MOV CX, 0003` | `000A` | `0005` | `0003` | `0109` | NZ | NC | PL | Se carga 3 decimal en CX |
| **4** | `ADD AX, BX` | `000F` | `0005` | `0003` | `010B` | NZ | NC | PL | AX = 10 + 5 = 15 (`000Fh`), resultado no nulo ni negativo |
| **5** | `ADD AX, CX` | `0012` | `0005` | `0003` | `010D` | NZ | NC | PL | AX = 15 + 3 = 18 (`0012h`), resultado final verificado |
| **6** | `INT 20` | `0012` | `0005` | `0003` | `010D` | NZ | NC | PL | Proceso finalizado de forma normal |

> **Checkpoint 1 (Parte 2):** Evidencia guardada en `capturas/CP1_traza_suma.png`.

---

### Paso 5 a 7: Programa con Bucle usando `LOOP` y `CX`

Se ensambla un bucle acumulador que suma el valor `2` cuatro veces consecutivas en `AX`:
```dos
-A 100
1357:0100 MOV AX, 0000
1357:0103 MOV CX, 0004
1357:0106 ADD AX, 0002
1357:010A LOOP 0106
1357:010C INT 20
1357:010E
-U 100 10D
1357:0100 B80000        MOV     AX,0000
1357:0103 B90400        MOV     CX,0004
1357:0106 050200        ADD     AX,+02
1357:0109 E2FB          LOOP    0106
1357:010B CD20          INT     20
```

#### Cálculo del Desplazamiento Relativo del Salto `LOOP`:
La instrucción `LOOP 0106` se codifica en código máquina como `E2 FB`:
* Opcode: `E2` (instrucción `LOOP`).
* Desplazamiento relativo firmado (8 bits): `FBh` (representación en complemento a dos de $-5$).
$$\text{Desplazamiento} = \text{Dirección Destino} - \text{Dirección Siguiente}$$
$$\text{Desplazamiento} = 0106\text{h} - 010B\text{h} = -5\text{d} = \text{FBh}$$

#### Tabla de Traza 2: Bucle con Instrucción `LOOP` (10 Invocaciones)
| N° | Iteración | Instrucción Ejecutada | AX Después | CX Después | IP Sig. | ¿LOOP salta? | Comentario y Estado |
| :---: | :---: | :--- | :---: | :---: | :---: | :---: | :--- |
| **1** | Init | `MOV AX, 0000` | `0000` | `0000` | `0103` | — | Acumulador inicializado en cero |
| **2** | Init | `MOV CX, 0004` | `0000` | `0004` | `0106` | — | Contador de iteraciones fijado en 4 |
| **3** | Iter 1 | `ADD AX, 0002` | `0002` | `0004` | `0109` | — | Primera suma: AX = 2 |
| **4** | Iter 1 | `LOOP 0106` | `0002` | `0003` | `0106` | **Sí** | CX pasa a 3 (CX ≠ 0); salta a 0106h |
| **5** | Iter 2 | `ADD AX, 0002` | `0004` | `0003` | `0109` | — | Segunda suma: AX = 4 |
| **6** | Iter 2 | `LOOP 0106` | `0004` | `0002` | `0106` | **Sí** | CX pasa a 2 (CX ≠ 0); salta a 0106h |
| **7** | Iter 3 | `ADD AX, 0002` | `0006` | `0002` | `0109` | — | Tercera suma: AX = 6 |
| **8** | Iter 3 | `LOOP 0106` | `0006` | `0001` | `0106` | **Sí** | CX pasa a 1 (CX ≠ 0); salta a 0106h |
| **9** | Iter 4 | `ADD AX, 0002` | `0008` | `0001` | `0109` | — | Cuarta suma: AX = 8 |
| **10**| Iter 4 | `LOOP 0106` | `0008` | `0000` | `010B` | **No** | CX pasa a 0; bucle finaliza sin saltar |
| **11**| Fin | `INT 20` | `0008` | `0000` | `010B` | — | Terminación normal verificada (`AX = 8`) |

> **Checkpoint 2 (Parte 2):** Evidencia guardada en `capturas/CP2_traza_loop.png`.

---

### Paso 8: Análisis del Código Máquina con `D`
Se inspecciona la memoria con `D CS:100 L0C`:
```dos
-D CS:100 L0C
1357:0100  B8 00 00 B9 04 00 05 02-00 E2 FB CD 20           ...
```
* `B8 00 00` $\rightarrow$ `MOV AX, 0000` (3 bytes)
* `B9 04 00` $\rightarrow$ `MOV CX, 0004` (3 bytes)
* `05 02 00` $\rightarrow$ `ADD AX, 0002` (3 bytes)
* `E2 FB` $\rightarrow$ `LOOP 0106` (2 bytes)
* `CD 20` $\rightarrow$ `INT 20` (2 bytes)  
*Total del programa con LOOP:* **13 bytes**.

---

### Paso 9 a 12: Bucle Equivalente con `DEC/JNZ` y Comando `G`

Se implementa el bucle manual en la dirección `CS:0200`:
```dos
-A 200
1357:0200 MOV AX, 0000
1357:0203 MOV CX, 0004
1357:0206 ADD AX, 0002
1357:0209 DEC CX
1357:020A JNZ 0206
1357:020C INT 20
1357:020E
-U 200 20D
1357:0200 B80000        MOV     AX,0000
1357:0203 B90400        MOV     CX,0004
1357:0206 050200        ADD     AX,+02
1357:0209 49            DEC     CX
1357:020A 75FA          JNZ     0206
1357:020C CD20          INT     20
```

* `DEC CX`: Opcode `49` (1 byte). Decrementa `CX` y actualiza `ZF` (no altera `CF`).
* `JNZ 0206`: Opcode `75` + desplazamiento `FAh` ($-6$ en complemento a dos: $0206\text{h} - 020C\text{h} = -6$). Ocupa 2 bytes.
* *Control de bucle:* `DEC` + `JNZ` ocupa **3 bytes** en dos instrucciones, frente a **2 bytes** en una sola instrucción con `LOOP`.

#### Tabla de Traza 3: Bucle con `DEC CX` y `JNZ` (15 Invocaciones)
| N° | Iteración | Instrucción Ejecutada | AX Después | CX Después | IP Sig. | ¿JNZ salta? | Comentario y Estado de Banderas |
| :---: | :---: | :--- | :---: | :---: | :---: | :---: | :--- |
| **1** | Init | `MOV AX, 0000` | `0000` | `0000` | `0203` | — | Acumulador inicializado |
| **2** | Init | `MOV CX, 0004` | `0000` | `0004` | `0206` | — | Contador fijado en 4 |
| **3** | Iter 1 | `ADD AX, 0002` | `0002` | `0004` | `0209` | — | AX = 2 |
| **4** | Iter 1 | `DEC CX` | `0002` | `0003` | `020A` | — | CX pasa a 3; ZF=0 (no cero) |
| **5** | Iter 1 | `JNZ 0206` | `0002` | `0003` | `0206` | **Sí** | Salta a 0206h porque ZF=0 |
| **6** | Iter 2 | `ADD AX, 0002` | `0004` | `0003` | `0209` | — | AX = 4 |
| **7** | Iter 2 | `DEC CX` | `0004` | `0002` | `020A` | — | CX pasa a 2; ZF=0 |
| **8** | Iter 2 | `JNZ 0206` | `0004` | `0002` | `0206` | **Sí** | Salta a 0206h porque ZF=0 |
| **9** | Iter 3 | `ADD AX, 0002` | `0006` | `0002` | `0209` | — | AX = 6 |
| **10**| Iter 3 | `DEC CX` | `0006` | `0001` | `020A` | — | CX pasa a 1; ZF=0 |
| **11**| Iter 3 | `JNZ 0206` | `0006` | `0001` | `0206` | **Sí** | Salta a 0206h porque ZF=0 |
| **12**| Iter 4 | `ADD AX, 0002` | `0008` | `0001` | `0209` | — | AX = 8 |
| **13**| Iter 4 | `DEC CX` | `0008` | `0000` | `020A` | — | CX pasa a 0; **ZF=1** (activada) |
| **14**| Iter 4 | `JNZ 0206` | `0008` | `0000` | `020C` | **No** | No salta porque ZF=1; avanza a INT 20 |
| **15**| Fin | `INT 20` | `0008` | `0000` | `020C` | — | Terminación normal verificada (`AX = 8`) |

#### Comparación de Costo de Ejecución:
* **Bucle con `LOOP`:** $2\text{ (init)} + 4 \times (\text{ADD} + \text{LOOP}) + 1\text{ (INT 20)} =$ **11 instrucciones ejecutadas**.
* **Bucle con `DEC/JNZ`:** $2\text{ (init)} + 4 \times (\text{ADD} + \text{DEC} + \text{JNZ}) + 1\text{ (INT 20)} =$ **15 instrucciones ejecutadas**.
* *Diferencia:* Exactamente **4 instrucciones adicionales** en el bucle manual (1 instrucción extra por iteración requerida para evaluar el salto).

#### Demostración con Comando `G`:
```dos
-R IP
IP 020D
:200
-G 20C
AX=0008  BX=0005  CX=0000  DX=0000  SP=FFFE  BP=0000  SI=0000  DI=0000
DS=1357  ES=1357  SS=1357  CS=1357  IP=020C   NV UP EI PL ZR NA PE NC
1357:020C CD20          INT     20
```
El comando `G 20C` ejecuta directamente el bucle completo y se detiene en el *breakpoint* `020Ch`, confirmando instantáneamente `AX = 0008h`.

> **Checkpoint 3 (Parte 2):** Evidencia guardada en `capturas/CP3_traza_dec_jnz.png`.

---

## 🧠 Decisiones Técnicas Justificadas (Parte 2)

### Decisión Técnica 3: Selección de Mecanismo de Control de Bucle (LOOP vs. DEC/JNZ)
> **Pregunta:** Para un bucle contador simple donde `CX` no se utiliza para otro fin, ¿cuál mecanismo se recomienda en términos de bytes e instrucciones por iteración, en qué escenarios se preferiría `DEC/JNZ` y cómo se verifica con `U`?

**Justificación técnica:**  
Para un bucle de iteración simple donde el registro `CX` funciona estrictamente como contador decreciente, la recomendación técnica inequívoca es utilizar la instrucción dedicada `LOOP`. Desde la perspectiva de la densidad de código máquina, `LOOP 0106` se codifica en solo dos bytes (`E2 FB`), mientras que la combinación funcionalmente equivalente de `DEC CX` seguido de `JNZ 0206` demanda tres bytes (`49` y `75 FA` respectivamente). A nivel de la microarquitectura del procesador 8086, `LOOP` requiere una única fase de búsqueda y decodificación (*instruction fetch*) por cada iteración del ciclo para resolver simultáneamente la reducción del contador y la ramificación condicional, reduciendo el tráfico en el bus de instrucciones en comparación con el esquema compuesto de `DEC/JNZ`, el cual obliga a la Unidad de Interfaz con el Bus a buscar dos instrucciones independientes en cada vuelta del ciclo. Sin embargo, el mecanismo `DEC/JNZ` resulta preferible y necesario en contextos donde la condición de control sea más compleja que un simple decremento a cero, cuando se requiera evaluar banderas adicionales generadas por comparaciones explícitas con `CMP`, o cuando el registro `CX` deba ser empleado obligatoriamente en el cuerpo del bucle para otras operaciones especializadas (tales como desplazamientos variables con `CL` o instrucciones de cadena), obligando a utilizar otro registro de propósito general (como `BX` o `DX`) como contador auxiliar. El estudiante comprueba esta diferencia sin ejecutar el programa mediante el comando `U` de `DEBUG`, contrastando visualmente los desplazamientos consecutivos generados por el ensamblador: la distancia entre el inicio de `LOOP` y la siguiente instrucción es de dos bytes, mientras que el bloque `DEC` más `JNZ` abarca tres bytes de memoria.

---

### Decisión Técnica 4: Comando de Verificación para Bucles de Muchas Iteraciones (T vs. G)
> **Pregunta:** Para verificar el resultado final de un bucle de 100 iteraciones, ¿por qué `G` es más práctico que `T`, qué información se pierde con `G`, cuándo fue indispensable `T` y qué comando confirma `AX` tras `G`?

**Justificación técnica:**  
Cuando un algoritmo iterativo involucra un número significativo de ciclos, como un bucle configurado para cien iteraciones (`CX = 0x0064`), la ejecución manual mediante el comando `T` (Trace) resulta completamente ineficiente y propensa al error humano, pues demandaría introducir cientos de comandos individuales para alcanzar la condición de parada. El comando `G` (Go), especificando la dirección de punto de interrupción inmediatamente posterior al bucle (por ejemplo `G 20C`), resulta sustancialmente más práctico porque conmuta la CPU a velocidad de reloj nativa, ejecutando la totalidad de las iteraciones de forma continua y transparente sin pausar la ejecución en cada ciclo intermedio. No obstante, esta velocidad introduce como contrapartida la pérdida total de visibilidad sobre los estados transitorios: el depurador no muestra la evolución gradual del acumulador, la fluctuación de los indicadores de estado en el registro de banderas (`FLAGS`), ni posibles desvíos anómalos que puedan ocurrir en ramas intermedias antes del punto de ruptura. La granularidad fina provista por `T` fue estrictamente indispensable en los Pasos 4, 7 y 11 del laboratorio, dado que el propósito pedagógico y analítico exigía registrar en una tabla de traza formal el comportamiento exacto de los registros y el estado de las banderas (`ZF`, `CF`, `SF`) en cada instrucción ejecutada. Tras completar una ejecución rápida con `G 20C`, el estudiante puede certificar de manera atómica el valor final acumulado en el registro de salida invocando el comando `R AX`, el cual expone de inmediato el valor de 16 bits sin alterar ninguna otra variable del entorno.

---

## 📷 Matriz de Checkpoints y Evidencias Fotográficas

| Checkpoint | Ubicación del Archivo | Descripción del Hito Técnico Verificado |
| :---: | :--- | :--- |
| **CP 1 (P1)** | `capturas/CP1_registros.png` | Inspección inicial con `R` y modificación selectiva de `AX = 1234h`. |
| **CP 2 (P1)** | `capturas/CP2_volcado_memoria.png` | Volcado hexadecimal `D 200 L40` evidenciando el patrón `AB CD EF` generado con `F`. |
| **CP 3 (P1)** | `capturas/CP3_ensamblado_desensamblado.png` | Ensamblado con `A 100` y correspondencia de bytes desensamblados con `U 100 109`. |
| **CP 4 (P1)** | `capturas/CP4_memoria_direccionamiento.png` | Modificación con `E 300 78 56`, instrucción `MOV AX, [0300]` y traza `T` resultando en `AX = 5678h`. |
| **CP 1 (P2)** | `capturas/CP1_traza_suma.png` | Traza completa paso a paso con `T` del programa de suma acumulando `AX = 0012h`. |
| **CP 2 (P2)** | `capturas/CP2_traza_loop.png` | Traza de las 10 ejecuciones del bucle `LOOP 0106` verificando decremento de `CX` hasta `AX = 0008h`. |
| **CP 3 (P2)** | `capturas/CP3_traza_dec_jnz.png` | Traza del bucle manual `DEC/JNZ` (15 pasos) y ejecución directa con punto de parada `G 20C`. |
| **CP 4 (P2)** | `capturas/CP4_resumen_final.png` | Verificación final consolidada de la sesión de trabajo y estado limpio de registros. |

---

## 📜 Conclusiones y Autoría

* La herramienta `DEBUG` permite desmitificar la ejecución en modo real x86, evidenciando que el código y los datos conviven en el mismo espacio lineal y que la interpretación depende exclusivamente del puntero de instrucción `CS:IP`.
* La instrucción especializada `LOOP` optimiza tanto la huella de memoria (2 bytes frente a 3 bytes de `DEC/JNZ`) como los ciclos de búsqueda de instrucciones de la CPU, consolidando la eficiencia del conjunto CISC x86 para bucles contadores elementales.
* La combinación metodológica entre la traza exhaustiva (`T`) y la ejecución con puntos de interrupción (`G`) dota al analista de sistemas de la capacidad de alternar entre la verificación microscópica de banderas y la comprobación rápida de resultados algorítmicos.

**Autor:** Eduard Cantillo  
**Materia:** Arquitectura de Computadores — UFPS  
**Año Académico:** 2026
