# Registro de Evidencias Fotográficas (Capturas DOSBox)

En este directorio se almacenan las capturas de pantalla tomadas durante la ejecución real de las sesiones del depurador `DEBUG` en DOSBox.

### Checkpoints de la Parte 1
* `CP1_registros.png`: Salida completa del comando `R` mostrando todos los registros iniciales y la modificación selectiva de `AX = 1234h`.
* `CP2_volcado_memoria.png`: Volcado hexadecimal con `D 200 L40` evidenciando el patrón `AB CD EF` rellenado con `F`.
* `CP3_ensamblado_desensamblado.png`: Ensamblado del programa de 4 instrucciones con `A 100` y verificación de código máquina con `U 100 109`.
* `CP4_memoria_direccionamiento.png`: Modificación puntual con `E 300 78 56`, instrucción directa `MOV AX, [0300]` y ejecución con `T` (`AX = 5678h`).

### Checkpoints de la Parte 2
* `CP1_traza_suma.png`: Ensamblado y traza paso a paso con `T` del programa de suma de tres operandos finalizando con `AX = 0012h`.
* `CP2_traza_loop.png`: Traza de las iteraciones del bucle con `LOOP 0106` decrementando `CX` hasta que `AX = 0008h`.
* `CP3_traza_dec_jnz.png`: Traza del bucle alternativo con `DEC CX` / `JNZ 0206` y ejecución directa con punto de ruptura usando `G 20C`.
* `CP4_resumen_final.png`: Vista consolidada de la sesión de trabajo y verificación final en DOSBox.
