Certainly! Below are examples of different types of arbiters in SystemVerilog, including Round Robin, Priority, and First-Come, First-Served (FCFS) arbiters.

### 1. Round Robin Arbiter

This arbiter grants access in a rotating order.

```systemverilog
module round_robin_arbiter (
    input logic clk,
    input logic reset,
    input logic [N-1:0] request,
    output logic [N-1:0] grant
);
    parameter N = 4; // Number of requesters
    logic [$clog2(N)-1:0] current_priority;

    always_ff @(posedge clk or posedge reset) begin
        if (reset) begin
            grant <= '0;
            current_priority <= '0;
        end else begin
            // Grant the current priority request
            grant <= (request & (1'b1 << current_priority)) ? (1'b1 << current_priority) : '0';

            // Move to the next requester
            current_priority <= (current_priority + 1) % N;
        end
    end
endmodule
```

### 2. Priority Arbiter

This arbiter grants access based on priority levels.

```systemverilog
module priority_arbiter (
    input logic clk,
    input logic reset,
    input logic [N-1:0] request,
    output logic [N-1:0] grant
);
    parameter N = 4; // Number of requesters
    logic [N-1:0] priority_mask;

    always_ff @(posedge clk or posedge reset) begin
        if (reset) begin
            grant <= '0;
        end else begin
            // Grant the highest priority request
            grant <= (request & priority_mask) ? (1'b1 << $clog2(request)) : '0';
        end
    end

    // Assign priority_mask to give higher priority to lower indices
    assign priority_mask = {1'b1, 1'b1, 1'b1, 1'b1}; // Example: All requests have equal priority
endmodule
```

### 3. First-Come, First-Served (FCFS) Arbiter

This arbiter serves requests in the order they are received.

```systemverilog
module fcfs_arbiter (
    input logic clk,
    input logic reset,
    input logic [N-1:0] request,
    output logic [N-1:0] grant
);
    parameter N = 4; // Number of requesters
    logic [N-1:0] queued_requests;

    always_ff @(posedge clk or posedge reset) begin
        if (reset) begin
            grant <= '0;
            queued_requests <= '0;
        end else begin
            // Check for new requests
            queued_requests <= queued_requests | request;

            // Grant to the first request in the queue
            if (queued_requests) begin
                grant <= (1'b1 << $clog2(queued_requests)); // Grant the first valid request
                queued_requests <= queued_requests & ~grant; // Remove granted request from queue
            end else begin
                grant <= '0;
            end
        end
    end
endmodule
```

### 4. Weighted Round Robin Arbiter

This arbiter grants access based on predefined weights.

```systemverilog
module weighted_round_robin_arbiter (
    input logic clk,
    input logic reset,
    input logic [N-1:0] request,
    output logic [N-1:0] grant
);
    parameter N = 4; // Number of requesters
    int weight[N] = '{2, 1, 1, 1}; // Example weights for each requester
    int count[N];
    logic [$clog2(N)-1:0] current_priority;

    always_ff @(posedge clk or posedge reset) begin
        if (reset) begin
            grant <= '0;
            current_priority <= '0;
            count <= '0;
        end else begin
            // Check the request and grant based on weight
            if (request[current_priority]) begin
                grant[current_priority] <= 1;
                count[current_priority] <= count[current_priority] + 1;

                // Rotate priority based on weights
                if (count[current_priority] >= weight[current_priority]) begin
                    count[current_priority] = 0; // Reset count
                    current_priority <= (current_priority + 1) % N;
                end
            end else begin
                current_priority <= (current_priority + 1) % N; // Move to next requester
            end
        end
    end
endmodule
```

### Conclusion

These examples cover various arbiter types commonly used in digital design. Depending on your application's requirements, you can choose the appropriate arbiter to ensure efficient resource allocation and fair access to shared resources.
