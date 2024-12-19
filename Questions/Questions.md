1. What is synthesis?
   
   - Verilog synthesis is the process of transforming high-level verilog code, which describes digital circuit, into a lower-level representation that can be implemented in hardware.This trasformation typically results in a netlist, which consists of logical components like gates and flip-flops that can be physically realized in an FPGA or ASIC.

2. 什么是网表？网表有什么用处？
   
   - 网表是描述电路中元件及其相互连接关系的文件
       元件实例、端口/引脚、网络、属性参数
   - 网表的分类：
     - 逻辑网表
         描述电路的逻辑结构。仅仅描述电路中各元件的逻辑连接，而不涉及物理布局
     - 物理网表
         电路中每个元件的位置和实际连接，常用于版图设计
     - 层次网表
   - 网表的用途
       逻辑验证与仿真、版图设计、制造测试
   - 网表优化
     - 布局优化
         提高速度、降低功耗
     - 布线优化
         调整布线层次和宽度，减少布线间的串扰和电阻，确保信号稳定
     - 电源网络优化
     - 时序和功耗优化

3. RTL行为级仿真、综合后门级功能仿真（前仿真）、时序仿真（后仿真）
   
   - RTL行为级仿真，又称功能仿真。用于检查代码中的语法错误以及代码行为的正确性。此阶段的仿真不包含延时信息。
   - 前仿真 综合工具产生一个标准网表文件，也可产生Verilog或者VHDL网表，其中标准网表文件用来在各个工具之间传递设计数据，不能用来仿真，而Verilog/VHDL网表可以用来仿真。综合工具缠身的仿真网表与生产厂商器件底层元件模型对应，因此进行综合后仿真必须在仿真过程中添加生产厂商的器件库，并且需要对仿真器进行一些必要的配置，否则仿真器无法识别底层元件。
   - 时序仿真 布局布线之后可以提供一个时序仿真模型，模型中包含了器件的信息，同时提供一个SDF时序标注文件（Standard Delay Format Timing Anotation）。

4. 如果不使用initial块，能否产生测试时钟？
    不能产生测试时钟。未初始化，时钟信号始终为x
   
    ![仿真图片](https://github.com/Spider-Viper/Picture/blob/main/clock_tb_wave.PNG "仿真波形图")

5. initial块和always块的不同
   
    There are mainly two types of procedual blocks in Verilog - initial and always.
   
   - initial块
     
     * 不可被综合
     
     * 主要用于仿真过程中的变量初始化
     
     * 多个initial块并行执行
     
     * initial块在仿真过程中仅执行一次
       
       ***notes:***
       
       ```verilog
         module sim();
       
             reg a, b;
       
             initial begin
                 a = 1'b0;
                 #20 b = 1'b0;
             end
       
             initial begin
                 #10 a = 1'b1;
                 #40 b = 1'b1;
             end
       
             initial begin
                 #30 $finish;    // $finish system task
                                 // terminate the current simulation
             end
         endmodule
       
         // 以上模块中，经过30个单位时间之后，执行system task $finish,此时，会结束其他尚未执行完毕的initial块。
       ```
   
   - always块
     
     * sensitivity list
       
       - no sensitivity list
         
         ```verilog
         // 用于仿真，产生时钟信号：
             always #10 clk = ~clk;
         // If there is no timing control statements within an always block ,the simulation will hang because of a zero-delay infinite loop.
         // Explicit delays are not synthesizable into logic gates.
         ```
     
     * 用于组合逻辑电路和时序逻辑电路，可被综合
     
     * To be synthesis, fllow one of the fllowing templates:
       
       ```verilog
       // Template #1: Use for combinational logic,all inputs mentioned in sensitivity list.
       always @(all_inputs) begin
           // combinational logic
       end
       
       // Template #2: Use of a if condition without else.
       // Cause a latch
       always @(all_inputs) begin
           if(enable) begin
       
           end
       end
       
       // Template #3: Use clock in sensitivity list for sequential elements.
       always @(posedge clk) begin
           // behavior to do at posedge clock
       end
       
       // Template #4: Use clock and async reset in sensitivity list.
       always @(posedge clk or negedge rstn) begin
           if(!rstn) begin
               // behavior to do during reset
           end else begin
               // behavior when not in reset
           end
       end
       ```

6. 模块中可以产生逻辑的结构：
   
   - always块
   
   - assign语句
   
   - 例化

7. 