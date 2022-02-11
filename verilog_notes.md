# Verilog notes

(This lumps Verilog and SystemVerilog together for the moment.)

## Terminology

* **Combinational circuits** output values that are strictly a function of their
  inputs. Thus, for any input, there is only one possible output.

## Language

### Modules and assignment

```
module my_and(input a, input b, output x);
  assign x = a & b;
endmodule
```


# TODO

* operators
* `wire`
* vectors
  * packed vs unpacked arrays
  * vector operators (`&vec`, `|vec`, etc.)
  * dynamic indexing
    ```verilog
    # https://hdlbits.01xz.net/wiki/Mux256to1
    module top_module( 
        input [255:0] in,
        input [7:0] sel,
        output out );

        assign out = in[sel];
    endmodule
    ```
  * bit slicing - https://stackoverflow.com/a/25125609
* module instances
  * instance arrays and the vector shortcut (https://stackoverflow.com/a/1380171)
  ```verilog
  // https://hdlbits.01xz.net/wiki/Exams/m2014_q4j
  // a 4 bit ripple carry adder with a 5-bit result, implemented using instance
  // arrays

  // a single bit full adder
  module fadd(input a, b, cin, output cout, sum);
    assign sum = cin? !(a ^ b) : (a ^ b);
    assign cout = cin? (a | b) : (a & b);
  endmodule

  // a 4-bit ripple carry adder
  module top_module (input [3:0] x, input [3:0] y, output [4:0] sum);
    // the carry vector serves double duty as both cout and cin, as well as the
    // highest bit of the sum output
    wire [4:0] carry;
    assign carry[0] = 0;  // cin to the first adder is always 0
    assign sum[4] = carry[4];  // the final cout is our 5th sum bit

    // this instantiates an array of `add` modules
    fadd add[3:0](
      // notice we pass the x and y vectors here directly. since they're the
      // same size as the array, verilog automatically connects the appropriate
      // vector wire to the corresponding instance
      .a(x), .b(y),
      .cin(carry[3:0]),
      .cout(carry[4:1]),
      .sum(sum[3:0])
    );
  endmodule
  ```
* `always @(*)`
  * if, case, for, etc.
* generate blocks
  ```verilog
  // https://hdlbits.01xz.net/wiki/Bcdadd100
  module top_module(
      input [399:0] a, b,
      input cin,
      output cout,
      output [399:0] sum );

      wire [99:0] c;
      assign cout = c[99];

      bcd_fadd add(a[3:0], b[3:0], cin, c[0], sum[3:0]);

      generate
          genvar i;
          for (i = 1; i <= 99; i++) begin: adds
              bcd_fadd i_add(
                  .a(a[i*4+3:i*4]),
                  .b(b[i*4+3:i*4]),
                  .cin(c[i - 1]),
                  .cout(c[i]),
                  .sum(sum[i*4+3:i*4])
              );
          end
      endgenerate
  endmodule
  ```
* logic gate symbols
* `x = '1` sets all of x's bits to 1 regardless of size
