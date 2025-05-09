// File: traffic_light_controller_with_tb.v
`timescale 1ns/1ps

module traffic_light_controller (
    input clk,
    input reset,
    output reg red,
    output reg yellow,
    output reg green
);

    // Define states
    typedef enum reg [1:0] {
        RED     = 2'b00,
        GREEN   = 2'b01,
        YELLOW  = 2'b10
    } state_t;

    state_t current_state, next_state;
    reg [3:0] timer;  // 4-bit counter for state duration

    // State transition logic
    always @(posedge clk or posedge reset) begin
        if (reset) begin
            current_state <= RED;
            timer <= 0;
        end else begin
            if (timer == 4) begin // adjust value for simulation speed
                current_state <= next_state;
                timer <= 0;
            end else begin
                timer <= timer + 1;
            end
        end
    end

    // Next state logic
    always @(*) begin
        case (current_state)
            RED:    next_state = GREEN;
            GREEN:  next_state = YELLOW;
            YELLOW: next_state = RED;
            default: next_state = RED;
        endcase
    end

    // Output logic
    always @(*) begin
        red    = 0;
        yellow = 0;
        green  = 0;
        case (current_state)
            RED:    red    = 1;
            GREEN:  green  = 1;
            YELLOW: yellow = 1;
        endcase
    end
endmodule

// Testbench for Traffic Light Controller
module tb_traffic_light;

    reg clk, reset;
    wire red, yellow, green;

    // Instantiate the DUT
    traffic_light_controller uut (
        .clk(clk),
        .reset(reset),
        .red(red),
        .yellow(yellow),
        .green(green)
    );

    // Clock generation
    always #5 clk = ~clk; // 10ns clock period

    initial begin
        // Dump waveform
        $dumpfile("dump.vcd");
        $dumpvars(1, tb_traffic_light);

        // Initialize signals
        clk = 0;
        reset = 1;
        #10;
        reset = 0;

        // Run simulation for 200ns
        #200;

        $finish;
    end
endmodule
