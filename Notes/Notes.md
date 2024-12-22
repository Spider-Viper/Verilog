### Data Types

1. Strings
   
   - every character in the string take 1-byte to be stored
     
     > "hello world" // String with 12 characters -> require 12 bytes
     > 
     > "x + z"       // String with 5 characters -> require 5 bytes
     > 
     > "How are you
     > 
     > feeling today?" // Illegal
   
   - Strings are stored in reg. If the size of the variable is smaller the string, then Verilog truncates the leftmost bits of the string. If the size of the variable is large than the string, then Verilog adds zeros to the left of the string.
     
     - ```verilog
       module testbench;
           reg [8*11:1] str1 = "Hello World";    // Variable can store 11 bytes, str = "Hello World"
           reg [8*5:1]  str2 = "Hello World";    // Variable stores only 5 bytes(rest is truncated), str = "World"
           reg [8*20:1] str3 = "Hello World";    // Variable can store 20 bytes(rest is padded with zeros), str = "         Hello World" )
       endmodule
       ```

2. Almost all data-types can only have one of the four different values as given below
   
   | x | represents an unknown logic value(can be 0 or 1)
   
   | z | represents a high-impedance state(also use "?")
   
   | 0 | 
   
   | 1 |
   
   * What does the verilog value-set imply?
     
     * X or x means that the value is simply unknown at the time, and could be either 0 or 1. This is quite different from the way X is treated in Boolean logic, where it means "Don't Care".
     
     * As with any incomplete electric circuit, the wire that is not connected to anything will have a high-impedance at that node and is represented by Z or z. Even in Verilog, any unconnected wire will result in a high impedance.
     
     * 高阻，或者说“阻高”，是一种电路分析时的特殊情况。简单来说，它可以被看作是具有极高的输入（输出）电阻，在极限状态下就成为了断路。
       
       严谨地说，高阻态（High_impedance state）、高输出态（High output state）和低输出态（Low output state）共同组成了三态逻辑。
       
       这里要说明一下：真实电路中电平信号是连续变化的，我们定义VCC代表电路电压（也就是电压上限值），而数字电路则对 0~VCC 的范围进行了区间划分。例如：0 ~ 0.3VCC 为低电平（0），0.7VCC ~ VCC 为高电平（1），0.3VCC ~ 0.7VCC 之间的则不属于工作电压，而高阻态往往被认为是理想的 0 电压。 **因此，低输出态不等于没有输出；高阻则在行为上更接近于没有输出。**
       
       因此，如果两个输出端口被导线直接相连，且二者一个处于高电平状态，另一个处于低电平状态，此时相连的导线上电平就会出现混乱，即 1 + 0 = ?。但处于高阻态的端口不会影响另一个端口的输出结果，即 z + 0 = 0, z + 1 = 1。这一特性使得三态门被广泛应用与总线（Bus）结构之中。

3. Nets and Variables are the two main groups of data types which represent different hardware structures and differ in the way they are assigned and retain values.
   
   - Nets
     
     - **wire**: Nets are used to connect between hardware entities like logic gates and hence do not store any value on its own. There are different types of nets each with different characteristics, but most popular and widely used net in digital design is of type ***wire***. A ***wire*** is a Verilog data-type used to connect elements and to connect nets that are driven by a single gate or continuous assignment. The ***wire*** is similar to the electrical wire. 在 Verilog中，wire类型为默认数据类型.
       
       | wire | 0   | 1   | x   | z   |
       | ---- | --- | --- | --- | --- |
       | 0    | 0   | x   | x   | 0   |
       | 1    | x   | 1   | x   | 1   |
       | x    | x   | x   | x   | x   |
       | z    | 0   | 1   | x   | z   |
     
     - Note: wire 变量不能进行初始化
   
   - Variables
     
     - A variable is an abstraction of a data storage element and can hold values.
     
     - **reg**: Verilog data-type ***reg*** can be used to model hardware registers since it can hold values between assignments. *Note that a reg need not always represents a flip-flop because it can also be used to represent combinational logic.* reg关键字只是声明了一个在always语句中进行赋值的信号。如果 always描述的是组合逻辑，那么reg就会综合成一根线，如果always描述的是时序逻辑，那么reg才会综合成一个寄存器。
     
     - integer: 32-bits; singed number; 可以被当做32bit的寄存器
     
     - time; realtime; real

### Scalar and Vector

1. A net or reg declaration without a range specification is considered 1-bit wide and is a ***scalar***. If a range is specified, then the net or reg becomes a multibit entity known as a ***vector***.

2. Note that the MSB and LSB should be a constant expression and cannot be substituted by a variable. But they can be any integer value - positive, negative or zero; and the LSB value can be greater than, equal to or less than MSB value.
   
   ```verilog
   wire [MSB:LSB] name;
   integer msb;
   wire [15:0] addr;
   wire [msb:2] data;    // illegal
   ```

3. **Bit-selects**
   
   - If the bit-select is out of bounds or the bit-select is x or z, then the value returned will be x.

4. **Part-selects**
   
   - > [<start_bit> +: <width>]    // part-select increments from start_bit
     > 
     > [<start_bit> -: <width>]    // part-select decrements from start_bit
   
   - Common Errors
     
     ```verilog
     module tb;
         reg [15:0] data;
         initial begin
             $display("data[0:9] = 0x%0h", data[0:9]);    // Error: Reversed part-select index expression ordering
         end
     endmodule
     ```

### Array and Memories

1. Arrays are allowed in Verilog for reg, wire, integer, and real data types.
   
   ```verilog
   reg        y1 [11:0];    //y1 is an scalar reg array of depth=12, each 1-bit wide
   wire [0:7] y2 [3:0];     //y2 is an 8-bit vector net with a depth of 4
   reg  [7:0] y3 [0:1][0:3] //y3 is a 2D array rows=2, cols=4 each 8-bit wide
   ```

2. Note that a memory of n 1-bit reg is not the same as an n-bit vector reg.
   
   ```verilog
   y1 = 0;    // illegal
   reg [7:0] mem1;    // reg vector 8-bit wide
   reg [7:0] mem2 [0:3];    // 8-bit wide vector reg array with depth=4
   reg [15:0] mem3 [0:3][0:1];    // 16-bit wide vector 2D array with rows=4, cols=2
   ```

3. Memories:
   
   - Memories are digital storage elements that help store a data and information in digital circuits. RAMs and ROMs are good examples of such memory elements. Storage elements can be modeled using one-dimensional arrays of type reg and is called a memory. 
     
     ```verilog
     // Register Vector
     reg [15:0] register;
     ```
     
     ![](https://github.com/Spider-Viper/Picture/blob/main/Register%20Vector.PNG)
     
     ```verilog
     // Memory
     reg [15:0] register [0:3];
     ```
     
     ![](https://github.com/Spider-Viper/Picture/blob/main/Memory.PNG)

### Strength

    略

### Module

1. list of ports is optional
   
   ```verilog
   // A module can have an empyt portlist
   module name;
       // Contents of the module
   endmodule
   ```

2. Hierarchical Names
   
   A hierarchical structure is formed when module can be instantiated inside one another, and hence the top level module is called the root. Since each lower module instantiations within a given module is required to have different identifier names, there will not be any ambiguity in accessing signals. A hierarchical name is constructed by a list of these identifiers separated by dots for each level of the hierarchy. Any signal can be accessed within any module using the hierarchical path to that paricular signal.
   
   ```verilog
   design.mod_inst1
   design.mod_inst1.y
   testbench.d0._net;
   ```

### Ports

1. Ports are by default considered as nets of type wire.

2. **Signed ports**
   
   The *signed* attribute can be attached to a port declaration or a net/reg declaration or both. Implicit nets are by default **unsigned**.
   
   ```verilog
   module mod_name (
       output c,
       input a, b
   );
       // prots a, b, and c are by default unsigned
   endmodule
   ```
   
   If either the net/reg declaration has a *signed* attribute, then the other shall also be considered signed.
   
   ```verilog
   module mod_name(
       output c,
       input signed a, b
   );
       wire a, b;    // a, b are signed from port declaration
       reg signed c;    // c is signed from reg declaration
   endmodule
   ```

3. **Port Variations**
   
   ```verilog
   // Verilog 1995
   module test(a, b, c);
       output [7:0] c,    // output "c" by default is a wire
       input  [7:0] a,    // input "a" and "b" are wires
       input  [7:0] b
   
       // Still, you can declare them again as wires to avoid confusion
       wire [7:0] a;
       wire [7:0] b;
       wire [7:0] c;
   endmodule
   ```
   ```verilog
   module test(a, b, c);
   
       output [7:0] c;    // By default c is of type wire
       input [7:0] a, b;
       
       // port "c" is changed to a reg type
       reg [7:0] c;
   
   endmodule
   ```
   ```verilog
   // Verilog 2001
   /***********************************************************************/
   // If a port declaration includes a net or variable type, then that port
   // is considered to be completely declared. It is illegal to redeclare the same
   // port in a net or variable type declaration.
   module test(
       output reg [7:0] e,
       input      [7:0] a
   );
       wire signed [7:0] a;     // illegal - declaration of a is already complete -> simulator dependent
       wire        [7:0] e;     // illegal - declaration of e is already complete
   endmodule
   /***********************************************************************/
   // If the port declaration does not include a net or variable type, then the port
   // can be declared in a net or variable type declaration again.
   module test(
       output [7:0] e,
       input  [7:0] a
   );
       reg [7:0] e;     // Okay
   endmodule
   ```
### Module Instantiations

1. **Unconnected/Floating Ports**
   
   Ports that are not connected to any wire in the instantiating module will have a value of high-impendance - z.
   
   ```verilog
   module mydesign (
       output o,
       input x, y, z
   );
   endmodule
   ```
   ```verilog
   module tb_top;
   
       wire [1:0] a;
       wire       c;    // output
       
       mydesign d0 (
           .x(),    // x is an input and not connected, hence a[0] will be z
           .y(a[1]),
           .z(a[1]),
           .o()    // o has valid value in mydesign but since it is not connected to "c" in tb_top, c will be z
       );
   
   endmodule
   ```
![](https://github.com/Spider-Viper/Picture/blob/main/Unconnected-Floating%20Ports.PNG)

Note that output from instances u1 and u2 are left unconnected in the RTL schematic obtained after synthesis. Since the input d to instances u2 and u3 are now connected to nets that are not being driven by anything it is grounded.

In simulations, such unconnected ports will be denoted as high impedance.

All port declarations are implicity declared as wire and hence the port direction is sufficient in that case. However output ports that need to store values should be declared as reg data type and _can be used in a procedural block like always and initial only._

Ports of type input or inout cannot be declared as reg because they are being driven from outside continuously and should not store values, rather reflect the changes in the external signals as soon as possible.

*It is perfectly legal to connect two ports with varying vector sizes, but the one with lower vector size will prevail and the remaining bits of the other port with a higher width will be ignored.*

```verilog
// Case #1 : Inputs are by default implicitly declared as type "wire"
module des0_1(input wire clk);    // wire neet not be specified here
module des0_2(input clk);        // By default clk is of type wire


// Case #2 : Inputs connot be of type reg
module des1(input reg clk);    // Illegal : inputs cannot be of type reg


// Case #3 : Take two module here with varying port widths
module des2(output [3:0] data);
module des3(input [7:0] data);


module top();
    wire [7:0] net;
    des2 u0(.data(net));    // Upper 4-bits of net are undriven
    des3 u1(.data(net));     
endmodule


// Case #4 : Outputs cannot be connected to reg in parent module
module top();
    reg [3:0] data_reg;
    des2 u0(.data(data_reg));    // Illegal: data output port is connected to a reg type signal "data_reg"
                                 // 顶层模块进行各子模块的连接，当然需要用导线连接！
endmodule
```

### Assign Statement

1. syntax
   
   The drive strenth and delay are optional and are mostly used for dataflow modeling than synthesizing into real harware.
   
   > assign <net_expression> = \[drive_strenth\]\[delay\]\<expression of different signals or constant value\>
   
   Delay values are useful for specifying delays for gates and are used to module timing behavior in real hardware because the value dictates when the net should be assigned with the evaluated value.

2. rules:
   
   - LHS should always be a scalar or vector net or a concatenation of scalar or vector nets and *never a scalar or vector register*.
   
   - RHS can contain scalar or vector registers and function calls.
   
   - Whenever any operand on the RHS changes in value, LHS will be updated with the new value.
   
   - **assign** statements are also called continuous assignments and are always active.

3. ```verilog
   module xyz(
       output [4:0] z,
       input  [3:0] x,    // x = 'b1100
       input        y     // y = 'b1
   );
       wire [1:0] a;
       wire       b;
   
       // Case #1 : 4-bits of x and 1 bit of y is concatenated to get a 5-bit net
       // and is assigned to the 5-bit nets of z. So value of z='b11001 or z='h19
       assign z = {x,y};
   
       // Case #2 : 4-bits of x and 1 bit of y is concatenated to get a 5-bit net
       // and is assigned to selected 3-bits of net z. Remaining 2 bits of z remains
       // undriven and will be high-imp. So value of z='bZ001Z
       assign z[3:1] = {x,y};    // {x,y} = 5'b11001
   
       // Case #3 : The same statement is used but now bit4 of z is driven with a 
       // constant value of 1. Now z='b1001Z because only bit0 remains undriven
       assign z[3:1] = {x,y};
       assign z[4] = 1;
   
       // Case #4 : Assume bit3 is driven instead, but now there are two drivers for bit3, 
       // and both are dirving the same value of 0. So there should be no contention
       // and value of z='bZ001Z
       assign z[3:1] = {x,y};
       assign z[3] = 0;
   
       // Case #5 : Assume bit3 is instead driven with value 1, so now there are two
       // drivers with different values, where the first line is driven with the value
       // of X which at the time is 0 and the second assignment where it is driven
       // with value 1, so now it becomes unknown which will win. So, z='bZX01Z
       assign z[3:1] = {x,y};
       assign z[3] = 1;
   
       // Case #6 : Partial selection fo operands on RHS is also possible and say only
       // 2-bits are chosen from x, then z='b00001 because z[4:3] will be driven with 0
       assign z = {x[1:0],y}
   
       // Case #7 : Say we explicitly assign only 3-bits of z and leave remaining
       // unconnected the z='bZZ001
       assign z[2:0] = {x[1:0],y};
   
       // Case #8 : Same variable can be used multiple times as well and z='b00111
       // 3{y} is the same as {y,y,y}
       assign z = {3{y}};
   
       // Case #9 : LHS can also be concatenated: a is 2-bit vector and b is scala
       // RHS is evaluated to 11001 and LHS is 3-bit wide so first 3 bits from LSB
       // of RHS will be assigned to LHS. So a='b00 and b='b1
       assign {a,b} = {x,y};
   
       // Case #10 : If we reverse order on LHS keeping RHS same, we get a='b01 and
       // b='b0
       assign {b,a} = {x,y};
   endmodule
   ```

4. assign reg variables
   
   It is illegal to drive or assign **reg** type variable with an _assign statement_. 
   This is because a **reg** variable is capable of storing data and does not require to be driven continuously. 
   **reg** signals can only be driven in procedual blocks like **initial** and **always**.

### Combinational Logic with Assign

   1. Example #1: Simple combinational logic
   
   ```verilog
   // code
   module combo(
       output z,
       input  a, b, c, d, e
   );
       assign z = ((a & b) | (c ^ d) & ~e);
   endmodule
   
   //Testbench
   module tb;
       reg a, b, c, d, e;
       wire z;
       integer i;
       // Instantiate the design
       combo UUT_0(
           .a(a),
           .b(b),
           .c(c),
           .d(d),
           .e(e),
           .z(z)
       );
   
       initial begin
            // At the beginning of time, initialize all inputs of the design to a
            // known value.
            a <= 0;
            b <= 0;
            c <= 0;
            d <= 0;
            e <= 0;

            // Use a $monitor task to print any change in the signal to simulation console
            $monitor("a=%0b b=%0b c=%0b d=%0b e=%0b z=%0b", a,b,c,d,e,z);

            // Because there are 5 inputs, there can be 32 different input combinations
            // So use an iterator "i" to increment from 0 to 32 and assign the value
            // to tesebench variables so that it drives the design inputs
            for(i = 0; i < 32; i = i + 1) begin
                {a,b,c,d,e} = i;
                #10;
            end
       end
   endmodule
   ```
   2. Example #2 : Half Adder
   ```verilog
   // code
   module ha(
        output sum, cout,
        input a, b
   );
        assign sum = a ^ b;
        assign cout = a & b;
   endmodule

   // testbench
   module tb;
        // declare testbench variables
        reg a, b;
        wire sum, cout;
        integer i;

        ha UUT_0(
            .a(a),
            .b(b),
            .sum(sum),
            .cout(cout)
        );

        initial begin
            a <= 0;
            b <= 0;

            $monitor("a=%0b b=%0b sum=%0b cout=%0b", a,b,sum,cout);

            for(i = 0; i < 4; i = i + 1) begin
                {a,b} = i;
                #10;
            end
        end
   endmodule
   ```
   3. Example #3 : Full Adder
   ```verilog
   // code
   module fa(
        output sum, cout,
        input  a, b, cin
   );
        assign sum = (a ^ b) ^ cin;
        assign cout = (a & b) | ((a ^ b) & cin);
   endmodule

   // testbench
   module tb;
        reg a, b, cin;
        wire sum, cout;
        integer i;

        fa UUT_0(
            .a(a),
            .b(b),
            .cin(cin),
            .sum(sum),
            .cout(cout)
        );

        initial begin
            a <= 0;
            b <= 0;

            $monitor("a=%0b b=%0b cin=%0b sum=%0b cout=%0b", a, b, cin, sum, cout);

            for(i = 0; i < 7; i = i + 1) begin
                {a,b,cin} = i;
                #10;
            end
			
        end
   endmodule
   ```
   4. Example #4 : 2to1 MUX
   ```verilog
   // code
   module mux_2to1(
        output out,
        input  a, b, sel
   );

        assign out = sel ? b : a;
        
   endmodule

   // testbench
   module tb;
        reg a, b, sel;
        wire out;
        integer i;

        mux_2to1 UUT_0(
            .out(out),
            .a(a),
            .b(b),
            .sel(sel)
        );

        initial begin
            a <= 0;
            b <= 0;
            sel <= 0;

            $monitor("a=%0b b=%0b sel=%0b out=%0b", a, b, sel, out);

            for(i = 0; i <= 3; i = i + 1) begin
				{a, b, sel} = i;
				#10;
            end
			
        end
   endmodule
   ```
   5. Example #5 : 1to4 Demultiplexer
   ```verilog
   // code
   module demux_1to4(
		output a, b, c, d,
		input  in,
		input [1:0] sle
   );
		assign a = in & ~sel[1] & ~sel[0];
		assign b = in & sel[1] & ~sel[0];
		assign c = in & ~sel[1] & sel[0];
		assign d = in & sel[1] & sel[0];
   endmodule
   
   // testbench
   module tb;
		reg in;
		reg [1:0] sel;
		wire a, b, c, d;
		integer i;
		
		demux_1to4 UUT_0(
			.a(a),
			.b(b),
			.c(c),
			.d(d),
			.sel(sel),
			.in(in)
		);
		
		initial begin
			f <= 0;
			sel <= 0;
			
			$monitor("in=%0b sel=%0b a=%0b b=%0b c=%0b d=%0b", in, sel, a, b, c, d);
			
			for(i = 0; i < 8; i = i + 1) begin
				{in,sel} = i;
				#10;
			end
			
		end
   endmodule
   ```
   6. Example #6 : 4to16 Decoder
   ```verilog
   // code
   module dec_4to16(
		output [15:0] out,
		input  [3:0]  in
   );
		assign out = 1 << in;
   endmodule
   
   //testbench
   module tb;
		reg 	[3:0] 	in;
		wire 	[15:0] 	out;
		integer 	i;
		
		dec_4to16 UUT_0(
			.out(out),
			.in(in)
		);
		
		initial begin
			in <= 0;
			
			$monitor("in=0x%0h out=0x%0h", in, out);
			
			for(i = 0; i <= 15; i = i + 1) begin
				in = i;
				#10;
			end
			
		end
   endmodule
   ```
### Operator
1. If the second operand of a division or modulus operator is zero, then the result will be X.
   
	| Operator | Description     |
	|----------|-----------------|
	| a ** b   | 3**2 = 3^2 = 9  |
2. Relation Operators:
   
	If either of the operands is X or Z, then the result will be X.
4. Equality Operators(==;!=;===;!==):

	If either of the operands of logical-equality(==) or logical-inequality(!=) is X or Z, then the result will be X.
	
	| Operator | Description |
	| -------- | ----------- |
	| a === b  | a equal to b, including x and z |
	| a !== b  | a not equal to b, including x and z |
5. Logical Operators(&&,||,!):
	
	If either of the operands is X, then the result will be X as well. 
	
	The logical negation (!) operator will convert a non-zero or true operand into 0 and a zero or false operand into 1, while an X will remain as an X.
6. Bitwise Operators:

	| &    | 0    | 1    | x    | z    |
	| ---- | ---- | ---- | ---- | ---- |
	| 0    | 0    | 0    | 0    | 0    |
	| 1    | 0    | 1    | x    | x    |
	| x    | 0    | x    | x    | x    |
	| z    | 0    | x    | x    | x    |
	
	| \|   | 0    | 1    | x    | z    |
	| ---- | ---- | ---- | ---- | ---- |
	| 0    | 0    | 1    | x    | z    |
	| 1    | 1    | 1    | 1    | 1    |
	| x    | x    | 1    | x    | x    |
	| z    | x    | 1    | x    | x    |
7. Shift Operators:
	- logical shift operator << and >>
	- arithmetic shift operator <<< and >>>
	
		算术移位需要考虑符号位，而逻辑移位则无需考虑符号位。
		


   
   
   
