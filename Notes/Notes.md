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
		
		```verilog
		// 循环左移
		module shift_left(
			output 	[7:0] 	out,
			input			clk,
			input			rstn
		);
			reg [7:0] shifter;
			always @(posedge clk) begin
				if(!rstn)
					shifter <= 8'b1000_0000;
				else
					shifter <= {shifter[6:0],shifter[7]};
			end
			assign out = shifter;
		endmodule
		```
		```verilog
		// testbench
		`timescale 1ns / 1ns
		module tb;
			reg 		clk, rstn;
			wire [7:0]	out;
			
			shift_left UUT_0(
				.out(out),
				.clk(clk),
				.rstn(rstn)
			);
			
			initial begin
				clk <= 1'b0;
				rstn <= 1'b0;
				#300
				rstn = 1'b1;
			end
			always #10 clk = ~clk;
			
		endmodule
		```
		```verilog
		// 循环右移
		module shift_right(
			output 	[7:0]	out,
			input			clk,
			input			rstn
		);
			reg [7:0] shifter;
			always @(posedge clk) begin
				if(!rstn)
					shifter <= 8'b0000_0001;
				else
					shifter <= {shifter[0],shifter[7:1]};
			end
			assign out = shifter;
		endmodule
		```
##### Operator:"{}"
8. Concatenation Operator:


9. Replication Operator:
	- When the same expression has to be repeated for a number of times, a *replication constant* is used which needs to be non-negative number and cannot be X, Z or any variable.
	- Replication expressions cannot appear on the left hand side of any assignment and *cannot be connected to output or inout ports*.
	- Operands will be evaluated only once when the replication expression is executed even if the constant is zero.
	- The Verilog replication operator **{}** is commonly used in digital design to create bit patterns for initializing registers, memory arrays, or looup tables.
	
	
10. Sign Extension

	In Verilog, sign extension is a way of extending a signed number with fewer bits to a wider signed number by replicating the sign bit. Basically, it is used when performing arithmetic or
	logical operations on numbers with different bit widths.
	
	For example, let's say we have a 4-bit two's complement number, -3, represented as 1101. 
	If we want to add this number to another 8-bit two's complement number, say -10, represented as 11110110, 
	we first need to sign extend the 4-bit number to 8 bits to make it compatible with the 8-bit number. 
	To sign extend the 4-bit number, we replicate its most significant bit (the sign bit) to fill the additional bits, resulting in 11111101. 
	We can then add this sign-extended 4-bit number to the 8-bit number using normal Verilog arithmetic operations. 
	
	Example: (sign extend a 4-bit signed number to an 8-bit signed number)
	```verilog
	module sign_extension(
		output reg 	signed [7:0] output_num,
		input		signed [3:0] input_num
	);
		always @(*) begin
			if(input_num[3] == 1'b1) begin	// if the sign bit is 1
				output_num = {{8{1'b1}},input_num}; 	// Error: {8{1'b1},input_num}
			end else begin	// if the sign bit is 0
				output_num = {{8{1'b0}},input_num};
			end
		end
	endmodule
	```

11. Conditional Operator

- Nested Conditional Operators

Conditional operators can be nested to any level but it can affect readability of code.

```verilog
// "y" is assigned to "out" when both "a < b" and "x % 2" are true
// "z" is assigned to "out" when "a < b" is true and "x % 2" is false
// 0 is assigned to "out" when "a < b" is false
assign out = (a < b) ?  (x % 2) ? y : z : 0;
// step 1 -> (a < b) ? []:0;
// step 2 -> []
	(x % 2)? y : z
```

### Always Block

	- An "always block" can be used to realize combinational or sequantial elements. 
	- always块内被赋值的变量必须为reg类型
	- What happens if there is no sensitivity list?
		If there are no timing control statments within an always block, 
		the simulation will hang because of a zero-delay infinite loop.	
```verilog
// There is no time control, and hence it will stay and
// be repeated at 0 time units only. This continues
// in a loop and simulation will hang
always clk = ~clk;

always #10 clk = ~clk;
// Note: Explicit delays are not synthesizable into logic gates
```
	Hence real Verilog design code always require a sensitivity list.

### Combinational Logic with Always Block
- Example #1 : Half Adder
	```verilog
	module ha(
		output reg sum, cout,
		input a, b
	);
		always @(*) begin
			{cout,sum} = a + b;
		end
		
		/************************************/
		// wire cout,sum;
		// assign {cout,sum} = a + b;
	endmodule
	```
	
- Example #2 : Full Adder
	```verilog
	module fa(
		output reg sum, cout,
		input a, b, cin
	);
		always @(*) begin
			{cout,sum} = a + b + cin;
		end
	endmodule
	```

### Sequential Logic with Always
- JK Flip-Flop(typically implemented using NAND gates)
	```verilog
	// code
	module jk_ff(
		output reg q,
		input j,k,clk,rstn
	);	
		always @(posedge clk or negedge rstn) begin
			if(!rstn)
				q <= 1'b0;
			else
				q <= (j & ~q) | (~k & q);
		end
	endmodule

	// testbench
	`timescale 1ns / 1ns
	module tb;
		reg j, k, clk, rstn;
		wire q;
		integer i;
		
		reg [2:0] delay;
		
		jk_ff UUT_0(
			.q(q),
			.j(j),
			.k(k),
			.clk(clk),
			.rstn(rstn)
		);
		
		always #10 clk = ~clk;
		
		initial begin
			{j,k,clk,rstn} = 0;
			#40
			rstn = 1'b1;
			
			for(i = 0; i < 100; i = i + 1) begin
				delay = $random;
				#(delay) j = $random;
				#(delay) k = $random;
			end
		end
		
	endmodule	
	```
- Modulo-10 counter
	```verilog
	// code
	module modulo_10_counter(
		output reg [3:0]	cnt,
		input				clk,
		input				rstn
	);
		always @(posedge clk) begin
			if(!rstn)
				cnt <= 4'b0000;
			else begin
				if(cnt == 4'd9)
					cnt <= 4'b0000;
				else
					cnt <= cnt + 1'b1;
			end
		end
	endmodule
		
	//testbench
	`timescale 1ns / 1ns
	module tb;
		reg clk, rstn;
		wire [3:0] cnt;
		
		modulo_10_counter UUT_0(
			.cnt(cnt),
			.clk(clk),
			.rstn(rstn)
		);
		
		initial begin
			{clk,rstn} = 2'b0;
			#50
			rstn = 1'b1;
		end
		always #10 clk = ~clk;
	endmodule
	```
### Initial Block
	1. There are mainly two types of procedural block in Verilog: always and initial
	2. An initial block is not synthesizable and hence cannot be converted into a hardware schematic with digital elements. These blocks are primarily used to initialize variables and drive design ports with specific values
### Generate Block
	略
### Behavioral Modeling
There are two kinds of block statements: sequential and parallel
- sequential
	- begin...end
- parallel
	- fork...join
	```verilog
	initial begin
		#10 data = 8'hfe;
		fork
			#20 data = 8'h11;
			#10 data = 8'h00;
		join
	end
	```
	```verilog
	initial begin
		#10 data = 8'hfe;
		fork
			#10 data = 8'h11;
			begin
				#20 data = 8'h00;
				#30 data = 8'haa;
			end
		join
	end
	```
- naming of blocks
	```verilog
	begin : block_name
		//
	end

	fork : block_name
		//
	join

	// By doing so, the block can be referenced in a "disable" statement
	```
### Assignments

Placing values onto nets and variables are called assignments. There are three basic forms: **Procedural**,**Continuous**,**Procedural continous**
- Procedural Assignment

Procedural assignments occur within procedures such as **always**,**iniital**,**task**,and **functions** and are used to place values onto variables.The variable will hold the value until the next 
assignment to the same variable.
	- Variable declaration assignment
	```verilog
	// If the variables is initialized during declaration and at time 0 in an initial block as shown below, the order of evaluation is not guaranteed, and hence can have either 8'h05 or 8'h33.
	module design;
		reg [7:0] addr = 8'h05;
		initial
			addr = 8'hee;
	endmodule

	reg [3:0] array [3:0] = 0; 	// Illegal
	```
	
- Continuous Assignment

This is used to assign values onto scalar and vector nets. It provides a way to model combinational logic without specifying an interconnection of gates  and make it easier to drive the net with
logic expressions.
- Procedural Continuous Assignment

These are procedural statements that allow expressions to be continuously assigned to nets or variables and are of two types.
- assign...deassign
	This will override all procedural assignments to a variable and is deactivated by using the same signal with deassign. 
	The value of the variable will remain same until the variable gets a new value through a procedural or procedural continuous assignment. 
	The LHS of an assign statement cannot be a bit-select, part-select or an array reference but can be a variable or a concatenation of variables.
	```verilog
	reg q;
	initial begin
		assign q = 0;
		#10 deassign q;
	end
	```	
- force...release
	These are similar to the assign - deassign statements but can also be applied to nets and variables. 
	The LHS can be a bit-select of a net, part-select of a net, variable or a net but cannot be the reference to an array and bit/part select of a variable. 
	The force statment will override all other assignments made to the variable until it is released using the release keyword.	
	```verilog
	reg out, a, b;
	initial begin
		force out = a & b;
		
		release out;
	end
	```
		
### Blocking and Non-Blocking

- Non-Blocking

```verilog
module tb;
  reg [7:0] a, b, c, d, e;

  initial begin
    a <= 8'hDA;
    $display ("[%0t] a=0x%0h b=0x%0h c=0x%0h", $time, a, b, c);
    b <= 8'hF1;
    $display ("[%0t] a=0x%0h b=0x%0h c=0x%0h", $time, a, b, c);
    c <= 8'h30;
    $display ("[%0t] a=0x%0h b=0x%0h c=0x%0h", $time, a, b, c);
  end

  initial begin
    d <= 8'hAA;
    $display ("[%0t] d=0x%0h e=0x%0h", $time, d, e);
 	e <= 8'h55;
    $display ("[%0t] d=0x%0h e=0x%0h", $time, d, e);
  end
endmodule
```
```verilog
/*
|__ Spawn Block1: initial
|      |___ Time #0ns : a <= 8'DA, is non-blocking so note value of RHS (8'hDA) and execute next step
|      |___ Time #0ns : $display() is blocking, so execute this statement: But a hasn't received new values so a=8'hx
|      |___ Time #0ns : b <= 8'F1, is non-blocking so note value of RHS (8'hF1) and execute next step
|      |___ Time #0ns : $display() is blocking, so execute this statement. But b hasn't received new values so b=8'hx
|      |___ Time #0ns : c <= 8'30, is non-blocking so note value of RHS (8'h30) and execute next step
|      |___ Time #0ns : $display() is blocking, so execute this statement. But c hasn't received new values so c=8'hx
|      |___ End of time-step and initial block, assign captured values into variables a, b, c
|
|__ Spawn Block2: initial
|      |___ Time #0ns : d <= 8'AA, is non-blocking so note value of RHS (8'hAA) and execute next step
|      |___ Time #0ns : $display() is blocking, so execute this statement: But d hasn't received new values so d=8'hx
|      |___ Time #0ns : e <= 8'55, is non-blocking so note value of RHS (8'h55) and execute next step
|      |___ Time #0ns : $display() is blocking, so execute this statement. But e hasn't received new values so e=8'hx
|      |___ End of time-step and initial block, assign captured values into variables d and e
|
|__ End of simulation at #0ns
*/
```

```verilog
module tb;
  reg [7:0] a, b, c, d, e;

  initial begin
    a <= 8'hDA;
    $display ("[%0t] a=0x%0h b=0x%0h c=0x%0h", $time, a, b, c);
    #10 b <= 8'hF1;
    $display ("[%0t] a=0x%0h b=0x%0h c=0x%0h", $time, a, b, c);
    c <= 8'h30;
    $display ("[%0t] a=0x%0h b=0x%0h c=0x%0h", $time, a, b, c);
  end

  initial begin
    #5 d <= 8'hAA;
    $display ("[%0t] d=0x%0h e=0x%0h", $time, d, e);
 	#5 e <= 8'h55;
    $display ("[%0t] d=0x%0h e=0x%0h", $time, d, e);
  end
endmodule
```
```verilog
/*
|__ Spawn Block1 at #0ns: initial
|      |___ Time #0ns : a <= 8'DA, is non-blocking so note value of RHS (8'hDA) and execute next step
|      |___ Time #0ns : $display() is blocking, so execute this statement: But a hasn't received new values so a=8'hx
|      |___ End of time-step : Assign captured value to variable a, and a is now 8'hDA
|      |___ Wait until time advances by 10 time-units to #10ns
|	
|      |___ Time #10ns : b <= 8'F1, is non-blocking so note value of RHS (8'hF1) and execute next step
|      |___ Time #10ns : $display() is blocking, so execute this statement. But b hasn't received new values so b=8'hx
|	   |___ Time #10ns : c <= 8'30, is non-blocking so note value of RHS (8'h30) and execute next step
|      |___ Time #10ns : $display() is blocking, so execute this statement. But c hasn't received new values so c=8'hx
|      |___ End of time-step and initial block, assign captured values into variables b, c
|	
|__ Spawn Block2 at #0ns: initial
|      |___ Wait until time advances by 5 time-units to #5ns
|	
|      |___ Time #5ns : d <= 8'AA, is non-blocking so note value of RHS (8'hAA) and execute next step
|      |___ Time #5ns : $display() is blocking, so execute this statement: But d hasn't received new values so d=8'hx
|      |___ End of time-step : Assign captured value to variable d, and d is now 8'hAA
|      |___ Wait until time advances by 5 time-units to #5ns
|	
|      |___ Time #10ns : e <= 8'55, is non-blocking so note value of RHS (8'h55) and execute next step
|      |___ Time #10ns : $display() is blocking, so execute this statement. But e hasn't received new values so e=8'hx
|      |___ End of time-step and initial block, assign captured values to variable e, and e is now 8'h55
|
|__ End of simulation at #10ns
*/
```

### Control Blocks

##### if-else-if

- 注意优先级！！！
- False includes : zero, X, Z
- Hardware Implementation
	- if without else
	- if else if

		In the following example, the design module has a 4-bit output q that is incremented when mode is 1 and decrements when mode is 2 with if else construct. 
		Note that the description does not specify what has to be done if mode is 0 or 3 which are valid values for a 2-bit variable. 
		It is assumed that the circuit does nothing when mode is 1 and 3, but maintain exiting value of q.
		It is not recommended to leave such ambiguity in real design code, but is shown here to highlight the possibility.	
	```verilog
	module des (
		output reg [3:0] q,
		input [1:0] mode,
		input clk,
		input rstn
	);
		always @ (posedge clk) begin
			if (!rstn)
			  q <= 0;
			else begin			// 缺少mode == 0 和 mode == 3 的判断
			  if (mode == 1)
				q <= q + 1;
			  else if (mode == 2)
				q <= q - 1;
			end
		end
	endmodule
	```	
	![](https://github.com/Spider-Viper/Picture/blob/main/if-else-if-1.PNG)	

	```verilog
	module des ( 
		output reg [3:0] q,
		input mode,
		input clk,
		input rstn
	);
		always @ (posedge clk) begin
			if (! rstn)
				q <= 0;
			else begin
				if (mode)
					q <= q + 1;
				else
					q <= q - 1;
			end
		end
	endmodule
	```	
	![](https://github.com/Spider-Viper/Picture/blob/main/if-else-if-2.PNG)
		
##### case statement
- case,casez,casex
- Difference between if-else-if and case
	- Expressions given in a if-else block are more general while in a case block, a single expression is matched with multiple items.
	- Case will provide a definitive result when there are X and Z values in an expression.

##### for loop
- Design Example
	Let us take a look at how an 8-bit left shift register can be implemented in Verilog without a **for** loop and then compare it with the code using a **for** loop just to appreciate the utility of a looping construct.
	```verilog
	module left_shift_register(
		output reg 	[7:0] 	out,
		input		[7:0]	load_val,	// Load Value
		input				load_en,	// Load Enable
		input				clk,
		input				rstn
	);
	// If rstn is high, load new value to out port if load_en=1
	// If rstn is high, and load_en=0 shift register to left
	always @(posedge cld) begin
		if(!rstn)
			out <= 8'b0;
		else begin
			if(load_en)
				out <= load_val;
			else begin
				out[0] <= out[7];
				out[1] <= out[0];
				out[2] <= out[1];
				out[3] <= out[2];
				out[4] <= out[3];
				out[5] <= out[4];
				out[6] <= out[5];
				out[7] <= out[6];
			end
		end
	end
	
	endmodule
	```
	The same behavior can be implemented using a **for** loop which will reduce the code and make it scalable for different register widths. If the width of the register is made a verilog parameter,
	the design module will become scalable and the same parameter can be used inside the **for** loop.
	```verilog
	module left_shift_register(
		output reg 	[7:0]	out,
		input		[7:0]	load_val,
		input				load_en,
		input				clk,
		input				rstn
	);
		integer i;
		always @(posedge clk) begin
			if(!rstn)
				out <= 8'b0;
			else begin
				if(load_en)
					out <= load_val;
				else begin
					for(i = 0; i < 8; i = i + 1) begin
						out[i + 1] <= out[i];
					end
					out[0] <= out[7];
				end
			end
		end
	endmodule
	```
	```verilog
	// Testbench
	`timescale 1ns / 1ns
	module tb;
		reg [7:0] load_val;
		reg	load_en, clk, rstn;
		wire [7:0] out;
		
		left_shift_register UUT_0(
			.out(out),
			.load_val(load_val),
			.load_en(load_en),
			.clk(clk),
			.rstn(rstn)
		);
		
		always #10 clk = ~clk;
		
		initial begin
			clk <= 1'b0;
			rstn <= 1'b0;
			load_val <= 8'b0000_0001;
			load_en <= 1'b0;
			
			// Apply reset to design
			repeat (2) @(posedge clk);
			rstn <= 1;
			repeat (5) @(posedge clk);
			
			// Set load_en for 1 clock so that load_val is loaded
			load_en <= 1'b1;
			repeat (1) @(posedge clk);
			load_en <= 1'b0;

			// Let design run for 20 clocks and then finish
			repeat (20) @(posedge clk);
			$finish;
		end
	endmodule
	// 使用非阻塞赋值?????
	```
##### forever loop

##### repeat loop
- This will execute statements a fixed number of times. If the expression evaluates to an X or Z, then it will be treated as zero and will not be executed at all.
	```verilog
	repeat ([num_of_times]) begin
		[statements]
	end

	repeat ([num_of_times]) @ ([some_event]) begin
		[statements]
	end
	```
	
##### while loop

### Parameters
Parameters are Verilog constructs that allow a module to be reused with a different
specification.

Parameters are basically constants and hence it's illegal to modify their value at runtime.

##### Module Parameters
Module parameters can be used to override parameter definitions within a module and this makes the module have a different set of parameters at compile time.

A parameter can be modified with the **defparam** statement or in the module instance statement.
- style:
```verilog
// 1995 style
module design_ip(
	addr, wdata, write, sel, rdata
);
	parameter 	BUS_WIDTH = 32,
	parameter	DATA_WIDTH = 64,
	parameter	FIFO_DEPTH = 512;

	input addr;
	input wdata;
	...
endmodule
```
```verilog
// new style
module design_ip #(
	parameter BUS_WIDTH = 32,
	parameter DATA_WIDTH = 64,
	parameter FIFO_DEPTH = 512
)(
	input [BUS_WIDTH-1:0]	addr,
	...
);

endmodule
```
- overriding parameters
	```verilog
	module design;
		// 方式一：
		design_ip #(BUS_WIDTH = 64, DATA_WIDTH = 128) d0 (...);
		// 方式二：
		defparam d0.FIFO_DEPTH = 128;
	endmodule
	```
	Example:
	```verilog
	// 2-bit up-counter
	module counter #(
		parameter N = 2,
		parameter DOWN = 0
	)(
		output reg [N-1:0] out,
		input	en,
		input	rstn,
		input	clk
	);
		always @(posedge clk) begin
			if(!rstn)
				out <= 0;
			else begin
				if(en) begin
					if(DOWN)
						out <= out - 1;
					else
						out <= out + 1;
				end
				else
					out <= out;
			end
		end
	endmodule
	// top module
	module design_top(
		output [1:0] out,
		input	en,
		input	rstn,
		input	clk
	);
		counter #(.N(2)) u0(
			.out(out),
			.en(en),
			.rstn(rstn),
			.clk(clk)
		);
	endmodule
	```
	```verilog
	// 4-bit down-counter
	module design_top(
		output [3:0] out,
		input	en,
		input	rstn,
		input	clk
	);
		counter #(.N(4),.DOWN(1)) u0(
			.out(out),
			.en(en),
			.rstn(rstn),
			.clk(clk)
		);
	endmodule
	```
##### Local Parameters
Local parameters cannot be modified or overridden from outside the module using defparam. This feature ensures that the values assigned to localparams remain constant throughout the module's lifetime, providing a way to protect critical design values from accidental changes during instantiation.
```verilog
module design #(
	parameter WIDTH = 8
)(
	output [WIDTH-1:0] out,
	input	[WIDTH-1:0] a,
	input	[WIDTH-1:0] b
);
	// Define a local parameter that cannot be modified externally
	localparam LENGTH = 4;
	
	assign out = a + b;
endmodule

module tb;
	reg [15:0] a;
	reg [15:0] b;
	wire [15:0] out;

	// Instantiate the design module
	design #(.WIDTH(16)) inst(
		.out(out),
		.a(a),
		.b(b)
	);

	// Illegal
	// LENGTH is a localparam
	design #(.WIDTH(16),.LENGTH(5)) inst(....);
endmodule
```
However,local parameters can be assigned constant expressions containing parameters, which can be modified with defparam statements or module instance parameter value assignments.
```verilog
module design #(
	parameter WIDTH = 8,
				PARAM_LENGTH = 4
)(
	output [WIDTH-1:0] out,
	input	[WIDTH-1:0] a,
	input	[WIDTH-1:0] B
);
	localparam LENGTH = 2 + PARAM_LENGTH; // modified

	assign out = a + b;
endmodule

module tb;
	// Instantiate design
	design #(.WIDTH(16),.PARAM_LENGTH(5)) u0(....);
endmodule
```
Bit-selects and part-selects of local parameters that are not of type real shall be allowed.
```verilog
module design #(parameter WIDTH = 8, PARAM_LENGTH = 4) (
	output	[WIDTH-1:0] out,
    input	[WIDTH-1:0] a,
    input	[WIDTH-1:0] b,    
);

  // Assign a part-select of original parameter
  localparam LENGTH = 2 + PARAM_LENGTH[1:0];

  assign out = a + b;

  initial begin
    $display("Length = %0d", LENGTH);
  end
endmodule

module tb;
  // Statements

  // Update parameter length to 6, and localparam should get 2 + 2'b10 = 4
  design #(.WIDTH(16), .PARAM_LENGTH(6)) add_instance2 ( ... );
endmodule
```
##### specify parameters
略
##### Difference between parameter and localparam
### Functions








   
   
