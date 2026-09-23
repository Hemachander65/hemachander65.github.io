---
layout: post
author: Hemachander Rubeshkumar
tags: [portfolio, in-progress, embedded systems, computer architecture]
permalink: /27-bit-arch/
title: 27-bit Computer Architecture
---
## Why 27-bits?
This was part of the final exam for my Computer Architecture class, and the brilliant Professor Swartz had setup a script to randomize certain variables for all of us to make sure we all did our independent designs. And that system had delivered me, a 27-bit processor. The numbers were intentionally chosen to be odd to throw off any AI assistance as at the time, they were not sufficiently equipped to handle such an odd design decision. 

## Design Requirements
We were to write and simulate, the memory controller, registers, ALU, and the full processor that combines these components. We were to record the test case instructions into the registers before runtime. Of course, the testbench was also to be written in Verilog. To run this, we used EDA Playground, an online verilog simulator. (It has other capabilities as well, but that was its use here).

### The test programs:
> Program A: A + B = C; C + A = C; # Worth 90% of the grade
> Program B: for (int i = 0; i < 10; i++) {A += 1}; B = A - 2; # Worth the final 10% of the grade
(these aren't the exact test cases, but they are fairly close)

## Instruction Set Architecture
To begin, I decided to start by going through our test conditions, and start with the ALU. I started my design based off the MIPS architecture, and split all potential instructions into R, I, and J-type instructions. R-type were Arithmetic instructions, simple additions, subtractions, etc. I-type instuctions involved memory management, such as read or write instructions. J-type instuctions were reserved for branching. Given the test cases, I could have just chosen a few instructions, but given I did have 27-bits to work with, I decided to make use of them and build out the ISA.

``` Register File {
Address Bus - 12 bits
Address bus has a width of 12 bits (2^12 possible registers),
PC adds 12 to increment instructions,
Data width of 24 bits 
Registers store 24 bits each, or 3 Bytes,
4 Registers,
Registers are addressed by 2 bits,
4 x 24,
}

ALU {
Entry Point of Instructions 0x800 (16^2 * 8 = 2048)
All Programs will be loaded relative to this address
PC starts at 0x800,
2-bit Register Addresses,
}

ISA:
27bit Instructions,
Immediate Field 16 bits,
5 bits for instruction types (2^5 = 32 instructions),

I Type:
[op_code][Address A][Address B][Immediate]
{5 - 2 - 2 - 16 - 2e} [2 extra bits at the end]

R Type:
[op_code][Address A][Address B][Result Address][Function (32 maximum)]
{5 - 2 - 2 - 2 - 5 - 11e}

J Type
[op_code][Immediate]
{5 - 12 - 10e}
*I can jump to any address i want with this, as the addresses are all 12bit, so the immediate, when sign extended, can point to an absolute address anywhere

Op_Code:
00000 - R-type {ALU is Controlled by Function}
    
Would contain upto 32 types of arithmetic operations,
1XXXX - J-Type {Jump}
    
Would contain 16 types of operations,
0XXXX (Excluding 00000) - I-type {Immediate Arithmetic}
    
Would contain 15 types of operations,
32 opcodes, but upto 63 instructions

Important Instructions:
R-type [opcode-function]
[00000-00000 = Add]
[00000-00001 = Subtract]
I Type [opcode]
[01000 = Load Word]
[01100 = Store Word]
J Type [opcode]
[10000 = Jump]
[10010 = BEQ]
[10001 = BNE] ```

Now, for the sake of simplicity, I avoided using a Decoder, as I frankly had more bits than I needed. This meant I could use certain bits directly as control wires. Bits were reserved for the addresses of the inputs A and B, as well as the output C, leaving the rest for the ISA and control. Before I could fully get to deciding the amount of bits to leave to the ALU instruction, I had to determine the actual processor's architecture.

![Info.json](/images/27Arch.png)
<div style="text-align: center;">
    Rough MSPaint Sketch of the Processor Architecture
</div>

With the control wires layed out, it was easy to work out what wires were needed, and hard-code them to a certain index of the instruction. I re-arranged to condense multiple functions to the same bit, which i used to represent if the instruction was R, I, or J type earlier, minimizing the number of wires needed. Then, with the use of a few logic gates to further condense, I now knew how many extra bits I had to work with for the ALU instructions.

![Info.json](/images/27ISA.png)
<div style="text-align: center;">
    ISA Instruction Set Laid out in text
</div>

To be honest, I needed the "7 important instructions," but I wanted to fully flesh out what a 27-bit architecture could look like, and wrote as many as I could think of.

![Info.json](/images/27ALUverilog.png)
<div style="text-align: center;">
    All the ALU operations that were listed above, in Verilog
</div>

## 27-bit Memory Management
27-bits is an odd number, and very incompatible with the convention of Bytes and Nybbles. This makes memory management incredibly inefficient. Powers of 2, in binary, allow for many optimizations on this front. My approach was to simply use buffer bits, but this does mean you waste a lot of potential memory. An additional design constraint was to use Harvard Architecture, meaning having one memory registers for instructions and data, which I simplified by virtually partitioning the storage into 2 halves. This is what the 0x800 and \<0x800 represent in the MSPaint Sketch. 

## Further considerations
This is a very basic architecture, compared to the stuff Intel and AMD use. Decodes, multi-layer processors, pipeline architecture, threading, multi-threading, maintaining backwards compatibility with newer ISAs, etc. are all considerations that were beyond the scope of this particular project, though I would love to tackle them in a future project.
