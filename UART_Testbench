`timescale 1ns / 1ps

module uart_top_tb;
    reg        clk;
    reg        rst;
    reg        wr_en;
    reg        rdy_clr;
    reg  [7:0] data_in;
    wire       tx;
    wire       busy;
    wire       rdy;
    wire [7:0] dout;

    uart_top dut (
        .clk     (clk),
        .rst     (rst),
        .wr_en   (wr_en),
        .data_in (data_in),
        .rdy_clr (rdy_clr),
        .tx      (tx),
        .busy    (busy),
        .rdy     (rdy),
        .dout    (dout)
    );

    initial clk = 0;
    always #5 clk = ~clk; 

    task send_byte(input [7:0] din);
        begin
            @(negedge clk);
            data_in = din;
            wr_en   = 1'b1;
            @(negedge clk);
            wr_en   = 1'b0;
        end
    endtask

    task clear_ready;
        begin
            @(negedge clk);
            rdy_clr = 1'b1;
            @(negedge clk);
            rdy_clr = 1'b0;
        end
    endtask

    initial begin
        {rst, wr_en, rdy_clr, data_in} = 0;

        @(negedge clk);
        rst = 1'b1;
        repeat (3) @(negedge clk);
        rst = 1'b0;

        
        send_byte(8'h41);
        wait (rdy);
        $display("[TIME: %0t ns] Received Byte 1 = 0x%h (Expected: 0x41)", $time, dout);
        clear_ready;
        wait (!busy);

        
        send_byte(8'h55);
        wait (rdy);
        $display("[TIME: %0t ns] Received Byte 2 = 0x%h (Expected: 0x55)", $time, dout);
        clear_ready;
        wait (!busy);

        #100;
        $finish;
    end

    initial begin
        $dumpfile("dump.vcd");
        $dumpvars(0, uart_top_tb);
    end
endmodule
