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
