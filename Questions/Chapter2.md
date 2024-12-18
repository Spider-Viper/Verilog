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
5. initial块和always块的不同
