# SUMADOR DE 1 BIT

Este módulo implementa un sumador de 1 bit que toma dos bits de entrada (`A` y `B`), un bit de acarreo de entrada (`Ci`) que produce un bit de suma (`Sum`) y un bit de acarreo de salida (`Cout`).

## Descripción del Módulo

```verilog
module sum1b(
    input A, 
    input B, 
    input Ci,
    output Cout,
    output Sum
);

reg [1:0] result;

assign Sum = result[0];
assign Cout = result[1];

always@(*) begin
  result = A + B + Ci;
end

endmodule
```

- **Entradas:**
  - `A`: Primer bit a sumar.
  - `B`: Segundo bit a sumar.
  - `Ci`: Carry entrada (carry-in)

- **Salidas:**
  - `Sum`: Resultado de la suma de los bits de entrada.
  - `Cout`: Bit de acarreo de salida (carry-out), que indica si hay un acarreo hacia el siguiente bit más significativo.

- **`result`**: Un registro de 2 bits (`reg [1:0]`) que se utiliza para almacenar el resultado completo de la suma de `A`, `B`, y `Ci`.

### Funcionamiento del Módulo

1. **Asignaciones Directas (`assign`):**
   - `Sum = result[0];` asigna el bit menos significativo de `result` a la salida `Sum`.
   - `Cout = result[1];` asigna el bit más significativo de `result` a la salida `Cout`.

2. **Bloque `always`:**
   - El bloque `always@(*)` se activa siempre que alguna de las señales `A`, `B`, o `Ci` cambie de valor.
   - Dentro de este bloque, se realiza la operación de suma de `A`, `B`, y `Ci`. El resultado es almacenado en el registro `result`, que tiene dos bits para acomodar tanto el resultado de la suma como el acarreo generado.

### Cómo funciona?

- **Suma de 1 Bit**: 
  - La operación de suma entre `A`, `B`, y `Ci` puede generar un resultado entre `0` y `3`, lo cual se puede representar en dos bits.
  - El bit menos significativo (`result[0]`) se asigna a `Sum`, que es el resultado de la suma de los bits de entrada.
  - El bit más significativo (`result[1]`) se asigna a `Cout`, que indica si la suma generó un acarreo hacia el siguiente bit.

### Tabla de verdad

| `A` | `B` | `Ci` | `Cout` | `Sum` |
|:---:|:---:|:---:|:------:|:-----:|
|  0  |  0  |  0  |    0   |   0   |
|  0  |  0  |  1  |    0   |   1   |
|  0  |  1  |  0  |    0   |   1   |
|  0  |  1  |  1  |    1   |   0   |
|  1  |  0  |  0  |    0   |   1   |
|  1  |  0  |  1  |    1   |   0   |
|  1  |  1  |  0  |    1   |   0   |
|  1  |  1  |  1  |    1   |   1   |

### Diagrama

