// =============================================================================
// INSTITUTO TECNOLÓGICO DE LAS AMÉRICAS (ITLA)
// Asignatura:    Sistemas Digitales
// Profesor:      Wilkins Gabriel Cedano Del Rosario
// Estudiante:    Luis Manuel Trinidad Diaz
// Matrícula:     2021-0684
// Evaluación:    Primer Parcial (Reto Individual Verilog)
// Archivo:       tb_top_20210684.v
// Descripcion:   Testbench de verificación automática para 4096 vectores
//                combinacionales y 12 condiciones temporales obligatorias.
// =============================================================================

`timescale 1ns/1ps

module tb_top_20210684;

    // Señales de estímulo
    reg a, b, c, d;
    reg [3:0] A;
    reg [3:0] B;
    reg clk;
    reg rst;
    reg en;

    // Salidas del diseño bajo prueba (DUT)
    wire U, V;
    wire [3:0] Y;
    wire flag_comb;
    wire [3:0] Q;
    wire flag_q;

    // Variables de control de la simulación automática
    integer i, j, k;
    integer total_tests = 0;
    integer error_count = 0;
    
    // Variables para el modelo de referencia independiente
    reg expected_u, expected_v;
    reg [3:0] expected_Y;
    reg expected_flag;
    reg [1:0] sel_op;

    // Instanciación del módulo superior (System Under Test)
    top_20210684 dut (
        .a(a), .b(b), .c(c), .d(d),
        .A(A), .B(B),
        .clk(clk), .rst(rst), .en(en),
        .U(U), .V(V),
        .Y(Y), .flag_comb(flag_comb),
        .Q(Q), .flag_q(flag_q)
    );

    // Generador de Reloj (Periodo de 20ns = 50MHz)
    always #10 clk = ~clk;

    // Tarea auxiliar para evaluar salidas combinacionales contra el modelo ideal
    task verificar_combinacional;
        begin
            // Modelo de referencia independiente para lógica de control
            expected_u = (5 <= {a,b,c,d} && {a,b,c,d} <= 9) || ({a,b,c,d} >= 12);
            expected_v = ({a,b,c,d} == 1) || ({a,b,c,d} == 2) || ({a,b,c,d} == 3) || 
                         ({a,b,c,d} == 6) || ({a,b,c,d} == 7) || ({a,b,c,d} == 9) || 
                         ({a,b,c,d} == 11) || ({a,b,c,d} == 13) || ({a,b,c,d} == 15);
            
            sel_op = {expected_v, expected_u};

            // Modelo de referencia para la ruta de datos
            case(sel_op)
                2'b00: begin // XOR
                    expected_Y = A ^ B;
                    expected_flag = ^expected_Y;
                end
                2'b01: begin // RESTA
                    expected_Y = A - B;
                    expected_flag = (A < B);
                end
                2'b10: begin // SUMA
                    expected_Y = A + B;
                    expected_flag = ((A + B) > 15);
                end
                2'b11: begin // MAYOR
                    expected_Y = (A >= B) ? A : B;
                    expected_flag = (A == B);
                end
            endcase

            total_tests = total_tests + 1;

            // Verificación estricta contra indeterminaciones X/Z y fallos lógicos
            if ((U !== expected_u) || (V !== expected_v) || (Y !== expected_Y) || (flag_comb !== expected_flag) ||
                $isunknown(U) || $isunknown(V) || $isunknown(Y) || $isunknown(flag_comb)) begin
                $display("ERROR COMBINACIONAL: Ctrl=%d A=%d B=%d | Obtenido: U=%b V=%b Y=%d F=%b | Esperado: U=%b V=%b Y=%d F=%b", 
                          {a,b,c,d}, A, B, U, V, Y, flag_comb, expected_u, expected_v, expected_Y, expected_flag);
                error_count = error_count + 1;
            end
        end
    endtask

    initial begin
        // Inicialización de señales fijas
        clk = 0;
        rst = 0;
        en = 0;
        a = 0; b = 0; c = 0; d = 0;
        A = 0; B = 0;

        $display("=================================================================");
        $display("INICIANDO VERIFICACIÓN EXHAUSTIVA - RETO 15 (MATRÍCULA 2021-0684)");
        $display("=================================================================");

        // Configuración para volcado de ondas (Exigido en los entregables)
        $dumpfile("sim/resultado_simulacion.vcd");
        $dumpvars(0, tb_top_20210684);

        // ---------------------------------------------------------------------
        // ETAPA 1: VERIFICACIÓN COMBINACIONAL PURA (4096 Vectores)
        // ---------------------------------------------------------------------
        for (i = 0; i < 16; i = i + 1) begin
            for (j = 0; j < 16; j = j + 1) begin
                for (k = 0; k < 16; k = k + 1) begin
                    {a, b, c, d} = i;
                    A = j;
                    B = k;
                    #5; // Tiempo de asentamiento combinacional
                    verificar_combinacional();
                end
            end
        end
        $display(">>> Prueba Combinacional Lista. Vectores evaluados: %d", total_tests);

        // ---------------------------------------------------------------------
        // ETAPA 2: COMPROBACIÓN DE LOS 12 CASOS TEMPORALES MÍNIMOS
        // ---------------------------------------------------------------------
        $display("Iniciando casos temporales obligatorios...");
        
        // Configuración inicial fija: Operación SUMA (Ctrl = 8 -> U=1, V=0 -> {v,u}=2'b10)
        {a,b,c,d} = 4'd8; A = 4'd10; B = 4'd2; en = 1; #5;

        // Casos (1, 2 y 3): Dinámica de Reset Asíncrono
        #2 rst = 1; // (1) Activar rst entre flancos de reloj
        #15;        // (2) Mantener rst con en=1 durante un flanco ascendente
        if (Q !== 4'b0000 || flag_q !== 1'b0) begin
            $display("FALLO CASO 2: El reset asíncrono no forzó las salidas registradas a 0.");
            error_count = error_count + 1;
        end
        #3 rst = 0; // (3) Liberar rst de forma asíncrona lejos del flanco

        // Caso (4): Capturar operación SUMA con acarreo (15 + 1)
        A = 4'd15; B = 4'd1; en = 1;
        @(posedge clk); #2; // Espera flanco y asentamiento
        if (Q !== 4'd0 || flag_q !== 1'b1) begin
            $display("FALLO CASO 4: Error al capturar SUMA con acarreo.");
            error_count = error_count + 1;
        end

        // Caso (5): Capturar operación RESTA con préstamo (0 - 1)
        {a,b,c,d} = 4'd1; // Ctrl=1 -> U=0, V=1 -> {v,u}=2'b01 (RESTA)
        A = 4'd0; B = 4'd1;
        @(posedge clk); #2;
        if (Q !== 4'd15 || flag_q !== 1'b1) begin
            $display("FALLO CASO 5: Error al capturar RESTA con préstamo.");
            error_count = error_count + 1;
        end

        // Caso (6): Capturar operación XOR con paridad impar
        {a,b,c,d} = 4'd0; // Ctrl=0 -> U=0, V=0 -> {v,u}=2'b00 (XOR)
        A = 4'b0101; B = 4'b0000; // Y = 4'b0101 (Dos bits en 1, paridad par -> flag_comb = 0)
        @(posedge clk); #2;
        A = 4'b0001; B = 4'b0000; // Y = 4'b0001 (Un bit en 1, paridad impar -> flag_comb = 1)
        @(posedge clk); #2;
        if (Q !== 4'b0001 || flag_q !== 1'b1) begin
            $display("FALLO CASO 6: Error al capturar XOR o su paridad.");
            error_count = error_count + 1;
        end

        // Caso (7): Capturar operación MAYOR con empate
        {a,b,c,d} = 4'd3; // Ctrl=3 -> U=0, V=1 -> Modificado por Karnaugh a {v,u}=2'b11 (MAYOR)
        A = 4'd7; B = 4'd7;
        @(posedge clk); #2;
        if (Q !== 4'd7 || flag_q !== 1'b1) begin
            $display("FALLO CASO 7: Error al capturar MAYOR con empate.");
            error_count = error_count + 1;
        end

        // Caso (8): Deshabilitar en=0, cambiar entradas y verificar retención
        en = 0; A = 4'd12; B = 4'd2;
        @(posedge clk); #2;
        if (Q !== 4'd7 || flag_q !== 1'b1) begin // Debe retener el valor anterior del Caso 7
            $display("FALLO CASO 8: El registro cambió su estado estando deshabilitado (en=0).");
            error_count = error_count + 1;
        end

        // Caso (9): Volver a habilitar en=1 y capturar el nuevo valor
        en = 1;
        @(posedge clk); #2;
        if (Q !== 4'd12 || flag_q !== 1'b0) begin
            $display("FALLO CASO 9: No se capturaron los datos tras volver a activar la habilitación.");
            error_count = error_count + 1;
        end

        // Caso (10): Variar entradas entre flancos con en=1 y comprobar que Q permanece estable
        A = 4'd4; B = 4'd1; // Cambian las entradas combinacionales
        #5;
        if (Q !== 4'd12) begin // Q solo debe alterarse exactamente en el flanco ascendente
            $display("FALLO CASO 10: Salida registrada Q cambió de valor de forma asíncrona ante las entradas.");
            error_count = error_count + 1;
        end

        // Caso (11): Reiniciar asíncronamente entre flancos con un valor guardado distinto de cero
        #2 rst = 1; #2;
        if (Q !== 4'b0000 || flag_q !== 1'b0) begin
            $display("FALLO CASO 11: Falla el reset asíncrono inter-flanco.");
            error_count = error_count + 1;
        end
        rst = 0; #2;

        // Caso (12): Capturar una nueva operación inmediatamente después de liberar el reset
        A = 4'd9; B = 4'd9; en = 1;
        @(posedge clk); #2;
        if (Q !== 4'd9 || flag_q !== 1'b1) begin
            $display("FALLO CASO 12: Error de captura en el primer ciclo post-reset.");
            error_count = error_count + 1;
        end

        // ---------------------------------------------------------------------
        // RESUMEN FINAL DE LA EVALUACIÓN
        // ---------------------------------------------------------------------
        $display("\n=================================================================");
        $display("      RESUMEN DE EJECUCIÓN DE PRUEBAS AUTOMÁTICAS                ");
        $display("=================================================================");
        $display(" Pruebas totales realizadas: %d", total_tests);
        $display(" Errores críticos detectados: %d", error_count);
