`timescale 1ns / 1ps

// 1. BAUD RATE GENERATOR

module baud_rate_generator (
    input  wire clk,
    input  wire reset,
    output wire tx_enb,
    output wire rx_enb
);
    reg [4:0] tx_counter;
    reg [0:0] rx_counter;

    
    always @(posedge clk) begin
        if (reset)
            tx_counter <= 5'd0;
        else if (tx_counter == 5'd15)
            tx_counter <= 5'd0;
        else
            tx_counter <= tx_counter + 1'b1;
    end

    
    always @(posedge clk) begin
        if (reset)
            rx_counter <= 1'b0;
        else
            rx_counter <= 1'b0;
    end

    assign tx_enb = (tx_counter == 5'd15);
    assign rx_enb = 1'b1; 
  
endmodule



// 2. UART TRANSMITTER

module uart_sender (
    input  wire       clk,
    input  wire       rst,
    input  wire       enb,
    input  wire       wr_enb,
    input  wire [7:0] data_in,
    output reg        tx,
    output wire       busy
);
    localparam IDLE_STATE  = 2'b00;
    localparam START_STATE = 2'b01;
    localparam DATA_STATE  = 2'b10;
    localparam STOP_STATE  = 2'b11;

    reg [7:0] data;
    reg [2:0] index;
    reg [1:0] state;

    always @(posedge clk) begin
        if (rst) begin
            state <= IDLE_STATE;
            tx    <= 1'b1;
            index <= 3'd0;
            data  <= 8'd0;
        end else begin
            case (state)
                IDLE_STATE: begin
                    tx <= 1'b1;
                    if (wr_enb) begin
                        data  <= data_in;
                        index <= 3'd0;
                        state <= START_STATE;
                    end
                end

                START_STATE: begin
                    tx <= 1'b0; 
                    if (enb) begin
                        state <= DATA_STATE;
                    end
                end

                DATA_STATE: begin
                    tx <= data[index]; 
                    if (enb) begin
                        if (index == 3'd7) begin
                            state <= STOP_STATE;
                        end else begin
                            index <= index + 1'b1;
                        end
                    end
                end

                STOP_STATE: begin
                    tx <= 1'b1; 
                    if (enb) begin
                        state <= IDLE_STATE;
                    end
                end

                default: begin
                    state <= IDLE_STATE;
                    tx    <= 1'b1;
                end
            endcase
        end
    end

    assign busy = (state != IDLE_STATE);
endmodule



// 3. UART RECEIVER

module uart_receiver (
    input  wire       clk,
    input  wire       rst,
    input  wire       rx,
    input  wire       clk_en,
    input  wire       rdy_clr,
    output reg        rdy,
    output reg  [7:0] data_out
);
    localparam S_IDLE  = 2'b00;
    localparam S_START = 2'b01;
    localparam S_DATA  = 2'b10;
    localparam S_STOP  = 2'b11;

    reg [1:0] state;
    reg [2:0] index;
    reg [3:0] sample;
    reg [7:0] temp_register;

    always @(posedge clk) begin
        if (rst) begin
            state         <= S_IDLE;
            index         <= 3'd0;
            sample        <= 4'd0;
            temp_register <= 8'd0;
            data_out      <= 8'd0;
            rdy           <= 1'b0;
        end else begin
            if (rdy_clr)
                rdy <= 1'b0;

            if (clk_en) begin
                case (state)
                    S_IDLE: begin
                        if (rx == 1'b0) begin 
                            sample <= 4'd0;
                            state  <= S_START;
                        end
                    end

                    S_START: begin
                        if (sample == 4'd7) begin 
                            if (rx == 1'b0) begin
                                sample <= 4'd0;
                                index  <= 3'd0;
                                state  <= S_DATA;
                            end else begin
                                state  <= S_IDLE; 
                            end
                        end else begin
                            sample <= sample + 1'b1;
                        end
                    end

                    S_DATA: begin
                        if (sample == 4'd15) begin
                            sample <= 4'd0;
                            if (index == 3'd7) begin
                                state <= S_STOP;
                            end else begin
                                index <= index + 1'b1;
                            end
                        end else begin
                            if (sample == 4'd7) begin 
                                temp_register[index] <= rx;
                            end
                            sample <= sample + 1'b1;
                        end
                    end

                    S_STOP: begin
                        if (sample == 4'd15) begin
                            state    <= S_IDLE;
                            data_out <= temp_register;
                            rdy      <= 1'b1;
                            sample   <= 4'd0;
                        end else begin
                            sample <= sample + 1'b1;
                        end
                    end

                    default: state <= S_IDLE;
                endcase
            end
        end
    end
endmodule


// 4. UART TOP MODULE

module uart_top (
    input  wire       clk,
    input  wire       rst,
    input  wire       wr_en,
    input  wire [7:0] data_in,
    input  wire       rdy_clr,
    output wire       tx,
    output wire       busy,
    output wire       rdy,
    output wire [7:0] dout
);
    wire tx_tick, rx_tick;
    wire loopback_wire;

    baud_rate_generator baud_gen (
        .clk    (clk),
        .reset  (rst),
        .tx_enb (tx_tick),
        .rx_enb (rx_tick)
    );

    uart_sender transmitter (
        .clk     (clk),
        .rst     (rst),
        .enb     (tx_tick),
        .wr_enb  (wr_en),
        .data_in (data_in),
        .tx      (loopback_wire),
        .busy    (busy)
    );

    assign tx = loopback_wire;

    uart_receiver receiver (
        .clk      (clk),
        .rst      (rst),
        .rx       (loopback_wire),
        .clk_en   (rx_tick),
        .rdy_clr  (rdy_clr),
        .rdy      (rdy),
        .data_out (dout)
    );
endmodule
