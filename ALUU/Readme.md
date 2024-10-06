# ALU (Unidad Aritmética Lógica) con Control de Multiplicación y Salida a 7 Segmentos

Este módulo describe una Unidad Aritmética Lógica (ALU) de 4 bits que realiza operaciones básicas como suma, resta, multiplicación y operación AND lógica. Además, se utiliza para controlar un display de 7 segmentos, mostrando el resultado en un solo display o en múltiples displays.

## Descripción

### Entradas
- **A [3:0]**: Primer operando de 4 bits.
- **B [3:0]**: Segundo operando de 4 bits.
- **opcode [1:0]**: Señal de control de 2 bits que selecciona la operación a realizar.
  - `00`: Suma (`A + B`).
  - `01`: Resta (`A - B`).
  - `10`: Multiplicación (`A[2:0] * B[2:0]`).
  - `11`: Operación lógica AND (`A & B`).
- **clk**: Señal de reloj utilizada para la operación de multiplicación.
- **reset**: Señal de reinicio que inicializa los valores internos.

### Salidas
- **result [3:0]**: Resultado de la operación seleccionada.
- **Cout**: Carry o borrow de la operación realizada.
- **Sseg [6:0]**: Salida para un solo display de 7 segmentos, que muestra el resultado de la ALU.
- **Sseg_multi [20:0]**: Salida para múltiples displays de 7 segmentos.

## Componentes Internos

- **Sumador de 4 bits**: Calcula la suma de los operandos `A` y `B`, generando el resultado y el carry.
- **Restador de 4 bits**: Realiza la resta de `A` y `B`, devolviendo el resultado y el borrow.
- **Multiplicador Secuencial**: Toma los 3 bits menos significativos de `A` y `B` para calcular el producto y generar una señal de finalización.
- **Decodificador BCD a 7 segmentos**: Convierte el resultado de la ALU en un código BCD para mostrarlo en un display de 7 segmentos.
- **Extensión de resultado a 16 bits**: El resultado de la ALU se extiende a 16 bits para ser decodificado por el controlador de múltiples displays.

## Operaciones

Dependiendo de la señal `opcode`, la ALU realiza una de las siguientes operaciones:

- `00` (Suma): Realiza la suma de `A` y `B`, almacenando el resultado en `result` y el carry en `Cout`.
- `01` (Resta): Realiza la resta de `A` menos `B`, almacenando el resultado en `result` y el borrow en `Cout`.
- `10` (Multiplicación): Inicia la multiplicación de los 3 bits menos significativos de `A` y `B`. Una vez que la operación ha finalizado, el resultado se almacena en `result` y el bit más significativo del resultado en `Cout`.
- `11` (AND): Realiza la operación lógica AND entre `A` y `B`.

## Ejecución

La ALU se actualiza en cada flanco positivo del reloj (`clk`). Cuando `reset` está activo, todos los registros internos se inicializan a cero. La multiplicación se realiza de manera secuencial y requiere varios ciclos de reloj para completarse, mientras que las operaciones de suma, resta y AND son instantáneas.

## Decodificación de Displays

- **Display simple**: El resultado de la ALU se muestra en un solo display de 7 segmentos utilizando el módulo `BCD2Sseg`.
- **Displays múltiples**: El resultado extendido a 16 bits se puede visualizar en múltiples displays de 7 segmentos mediante el módulo `decoder`.

## Ejemplo de Uso

1. Suma: Si `A = 4'b0011` y `B = 4'b0001`, el `opcode = 2'b00` devuelve `result = 4'b0100` y `Cout = 0`.
2. Resta: Si `A = 4'b0100` y `B = 4'b0011`, el `opcode = 2'b01` devuelve `result = 4'b0001` y `Cout = 0`.
3. Multiplicación: Si `A = 4'b0101` y `B = 4'b0010`, el `opcode = 2'b10` devuelve `result = 4'b1010` después de varios ciclos de reloj.
4. AND: Si `A = 4'b1100` y `B = 4'b1010`, el `opcode = 2'b11` devuelve `result = 4'b1000`.

## Dependencias

Este módulo requiere los siguientes submódulos:
- `sum4b`: Módulo sumador de 4 bits.
- `restador_4bits`: Módulo restador de 4 bits.
- `multiplicador`: Módulo de multiplicación secuencial.
- `BCD2Sseg`: Decodificador BCD a 7 segmentos.
- `decoder`: Decodificador para múltiples displays.
# Tablas de Verdad de la ALU

A continuación se presentan las tablas de verdad para las operaciones que puede realizar la ALU: **Suma**, **Resta**, **Multiplicación** y **AND**. Estas operaciones se seleccionan mediante la señal de control `opcode`.

## 1. Tabla de Verdad para la Suma (`opcode = 00`)

| **A**   | **B**   | **Cout** | **Resultado (A + B)** |
|---------|---------|----------|-----------------------|
| 0000    | 0000    | 0        | 0000                  |
| 0001    | 0001    | 0        | 0010                  |
| 0010    | 0001    | 0        | 0011                  |
| 0011    | 0011    | 0        | 0110                  |
| 0100    | 0100    | 0        | 1000                  |
| 1100    | 0011    | 1        | 1111                  |
| 1111    | 1111    | 1        | 1110                  |

En esta tabla, `Cout` representa el bit de acarreo.

## 2. Tabla de Verdad para la Resta (`opcode = 01`)

| **A**   | **B**   | **Borrow (Cout)** | **Resultado (A - B)** |
|---------|---------|-------------------|-----------------------|
| 0000    | 0000    | 0                 | 0000                  |
| 0001    | 0001    | 0                 | 0000                  |
| 0010    | 0001    | 0                 | 0001                  |
| 0011    | 0011    | 0                 | 0000                  |
| 0100    | 0010    | 0                 | 0010                  |
| 0011    | 0100    | 1                 | 1111                  |
| 1111    | 1111    | 0                 | 0000                  |

En esta tabla, `Borrow` (Cout) indica si hay una deuda (prestamo) durante la resta.

## 3. Tabla de Verdad para la Multiplicación (`opcode = 10`)

| **A [2:0]** | **B [2:0]** | **Resultado (A * B)** |
|-------------|-------------|-----------------------|
| 000         | 000         | 000000                |
| 001         | 001         | 000001                |
| 010         | 001         | 000010                |
| 011         | 010         | 000110                |
| 100         | 011         | 001100                |
| 111         | 111         | 110001                |

En esta tabla, la multiplicación solo utiliza los 3 bits menos significativos de `A` y `B`.

## 4. Tabla de Verdad para la Operación Lógica AND (`opcode = 11`)

| **A**   | **B**   | **Resultado (A & B)** |
|---------|---------|-----------------------|
| 0000    | 0000    | 0000                  |
| 0001    | 0001    | 0001                  |
| 0011    | 0001    | 0001                  |
| 0010    | 0010    | 0010                  |
| 0100    | 0010    | 0000                  |
| 1100    | 0110    | 0100                  |
| 1111    | 1010    | 1010                  |

La operación AND realiza una comparación bit a bit entre `A` y `B`, devolviendo 1 solo si ambos bits son 1.

## Resumen de las Operaciones

| **opcode** | **Operación**       | **Descripción**                    |
|------------|---------------------|------------------------------------|
| 00         | Suma                | A + B                              |
| 01         | Resta               | A - B                              |
| 10         | Multiplicación       | A[2:0] * B[2:0]                    |
| 11         | AND                 | A & B                              |

