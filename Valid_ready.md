Certainly! Below is an example of a simple valid-ready interface in SystemVerilog, commonly used in digital design for handshaking between producer and consumer components. This example features a producer that sends data to a consumer, utilizing the valid and ready signals to control the flow of data.

### Valid-Ready Example

#### Valid-Ready Module

This module represents a producer that sends data to a consumer. The `valid` signal indicates when valid data is available, and the `ready` signal from the consumer indicates when it is ready to receive the data.

```systemverilog
module valid_ready_interface (
    input logic clk,
    input logic reset,
    output logic valid,           // Signal to indicate valid data
    output logic ready,           // Signal to indicate ready to receive
    output logic [31:0] data_out  // Data output
);

    logic [31:0] data_buffer;     // Buffer to hold data
    int count;

    // Data producer logic
    always_ff @(posedge clk or posedge reset) begin
        if (reset) begin
            valid <= 1'b0;
            count <= 0;
        end else begin
            if (count < 10) begin
                data_buffer <= count; // Produce some data
                valid <= 1'b1;         // Assert valid
                count <= count + 1;
            end else begin
                valid <= 1'b0;         // Deassert valid when done
            end
        end
    end

    // Ready logic
    always_ff @(posedge clk) begin
        if (valid && ready) begin
            data_out <= data_buffer; // Send data if both valid and ready
        end
    end

endmodule
```

#### Consumer Module

The consumer will read the data from the producer when it is valid and will assert the `ready` signal when it is prepared to receive data.

```systemverilog
module consumer (
    input logic clk,
    input logic reset,
    input logic valid,          // Signal indicating valid data
    input logic [31:0] data_in, // Data input
    output logic ready          // Signal indicating ready to receive
);

    always_ff @(posedge clk or posedge reset) begin
        if (reset) begin
            ready <= 1'b0;
        end else begin
            if (valid) begin
                ready <= 1'b1; // Assert ready when valid data is present
                $display("Received data: %d", data_in);
            end else begin
                ready <= 1'b0; // Deassert ready if no valid data
            end
        end
    end

endmodule
```

#### Testbench Code

Here’s a testbench that simulates both the producer and the consumer:

```systemverilog
module tb_valid_ready;

    logic clk;
    logic reset;
    logic valid;
    logic ready;
    logic [31:0] data_out;

    // Instantiate the valid-ready interface (producer)
    valid_ready_interface producer (
        .clk(clk),
        .reset(reset),
        .valid(valid),
        .ready(ready),
        .data_out(data_out)
    );

    // Instantiate the consumer
    consumer cons (
        .clk(clk),
        .reset(reset),
        .valid(valid),
        .data_in(data_out),
        .ready(ready)
    );

    // Clock generation
    initial begin
        clk = 0;
        forever #5 clk = ~clk; // 100 MHz clock
    end

    // Test sequence
    initial begin
        // Initialize signals
        reset = 1;
        #10 reset = 0;

        // Run for some time to observe interactions
        #100;

        // Finish simulation
        #20 $finish;
    end
endmodule
```

### Explanation of the Code

1. **Valid-Ready Interface (Producer)**:
   - This module produces data and asserts the `valid` signal to indicate when data is available.
   - The producer generates simple sequential data (0 to 9) and sets the `valid` signal accordingly.
   - The `data_out` is updated with the `data_buffer` when both `valid` and `ready` are asserted.

2. **Consumer Module**:
   - The consumer checks for valid data and asserts the `ready` signal when data is present.
   - It displays the received data through the `$display` statement.

3. **Testbench**:
   - The testbench instantiates both the producer and consumer modules.
   - It generates a clock signal and resets the system.
   - The test runs for a defined period to allow for interaction between the producer and consumer.

### Running the Testbench

To run this example:
1. Save the valid-ready interface, consumer module, and testbench code in separate `.sv` files.
2. Use a SystemVerilog simulator to compile and run the simulation.
3. Observe the output to confirm that the valid-ready handshake works correctly and that the consumer successfully receives the data.

This example can be extended or modified to fit more complex scenarios. If you have specific requirements or need more examples, let me know!

