# Multiplicador

Se realiza un multiplicador secuencial de 3 bits en Verilog, en cual toma el dato de dos números de 3 bits y genera un producto de 6 bits. Para esto se usara una Máquina de Estados para controlar el proceso de multiplicación bit a bit. 


### Entradas:
- `clk`: Señal de reloj.
- `init`: Señal de inicio, a partir de esta inicia el proceso.
- `MR`: Multiplicador
- `MD`: Multiplicando

### Salidas:
- `done`: Indica que la operación de multiplicación ha terminado.
- `pp`: Producto de 6 bits(guarda el resultado de la multiplicacion)

### Registros internos:
- `sh`: Controla si se debe realizar un desplazamiento.
- `rst`: Reinicia el sistema.
- `add`: Controla si se debe sumar el valor actual de `A` al resultado de `pp`.
- `A`: Registro de 6 bits que contiene una versión desplazada del multiplicando (`MD`).
- `B`: Registro de 3 bits que contiene el multiplicador (`MR`).
- `z`: Señal que indica si el valor de `B` es 0 (se usa para finalizar la operación).
- `status`: Registro que guarda el estado actual de la maquina de estado

## Máquina de Estados Finitos (FSM)

- `START`: Necesita que `init` sea 1 para comenzar.
- `CHECK`: Verifica si el bit menos significativo de `B` es 1. Si es así se suma `A`y si no se realiza un desplazamiento.
- `ADD`: Suma el valor de `A` a `pp`.
- `SHIFT`: Desplaza `A` hacia la izquierda y `B` hacia la derecha. Si `B` es 0 el proceso termina.
- `END1`: Estado final que señala que la multiplicación ha terminado (`done = 1`).

## Flujo del Proceso

1. El proceso comienza cuando la señal `init` es 1.
2. Se cargan los valores de `MR` en el registro `B` y de `MD` en el registro `A`.
3. Se revisa el bit menos significativo de `B`. Si es 1 se suma `A` al contador de productos parciales (`pp`).
4. Se desplazan `A` hacia la izquierda y `B` hacia la derecha.
5. El proceso se repite hasta que `B` llegue a 0, es aqui donde la señal `done` termina la multiplicacion. 

## Código Verilog

```verilog
module multiplicador(
    input clk,
    input init,    
    input [2:0] MR, 
    input [2:0] MD, 
    output reg done,
    output reg [5:0] pp
);

    reg sh;
    reg rst;
    reg add;
    reg [5:0] A;
    reg [2:0] B;
    wire z;

    reg [2:0] status;

    // Estados de la FSM
    parameter START = 0, CHECK = 1, ADD = 2, SHIFT = 3, END1 = 4;

    initial begin
        status = START;
        rst = 1'b0;
        pp = 6'b0;
        A = 6'b0;
        B = 3'b0; 
    end

    always @(negedge clk) begin
        case (status)
            START: begin
                sh <= 1'b0;
                add <= 1'b0;
                if (init) begin
                    status <= CHECK;
                    done <= 1'b0;
                    rst <= 1'b1;
                end
            end
            CHECK: begin 
                done <= 1'b0;
                rst <= 1'b0;
                sh <= 1'b0;
                add <= 1'b0;
                status <= (B[0]==1)? ADD : SHIFT;
            end
            ADD: begin
                done <= 1'b0;
                rst <= 1'b0;
                sh <= 1'b0;
                add <= 1'b1;
                status <= SHIFT;
            end
            SHIFT: begin
                done <= 1'b0;
                rst <= 1'b0;
                sh <= 1'b1;
                add <= 1'b0;
                status <= (z==1)? END1 : CHECK;
            end
            END1: begin
                done <= 1'b1;
                rst <= 1'b0;
                sh <= 1'b0;
                add <= 1'b0;
                status <= START;
            end
            default: status <= START;
        endcase 
    end 

    // Registro de desplazamiento para A y B
    always @(posedge clk) begin
        if (rst) begin
            A <= {3'b000, MD};
            B <= MR;
        end
        else begin 
            if (sh) begin
                A <= A << 1;
                B <= B >> 1;
            end
        end
    end 

    // Suma de productos parciales (pp)
    always @(posedge clk) begin
        if (rst) begin
            pp <= 6'b0;
        end
        else begin 
            if (add) begin
                pp <= pp + A;
            end
        end
    end

    // Comparador
    assign z = (B == 0)? 1'b1 : 1'b0;

endmodule

