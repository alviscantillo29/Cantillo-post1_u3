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


*Trazas adicionales en proceso...*
