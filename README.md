# verilog-divider


## Principle ##

![diagram](https://github.com/risclite/verilog-divider/blob/master/diagram.PNG)

Divider is an iteration of a serial operation of shift and subtraction. 

Here is the main code of this operation:

```verilog
wire [i:0]      m = dividend[i]>>(XLEN-i-1);
             
wire [i:0]      n = divisor[i];
    
wire            q = (|(divisor[i]>>(i+1))) ? 1'b0 : ( m>=n );
     
wire [i:0]      t = q ? (m - n) : m;

wire [XLEN-1:0] u = dividend[i]<<(i+1);
			    
wire [XLEN+i:0] d = {t,u}>>(i+1);
```

"m" is the shift result of dividend.

"n" is the shift result of divisor.

"q" is the comparison result, which is the "XLEN-i-1" bit of quotient.

"d" is the remainder of this operation or the dividend of the next operation.

Every iteration we get one bit of quotient. Every iteration is the operation of "i"-length subtraction. So the whole calcuation is a serial of XLEN incremental-length subtractions,  which are from 1 to XLEN.

## divfunc.v ##

Two parameters:
* XLEN --- the length of operators, such as 8,16,32,64. take 32 as an example.

* STAGE_LIST --- To put one register after the subtraction. For examples:
     
     32'b0000_0000_0000_0000_0000_0000_0000_0000 : the longest critical path: a chain of "1+2+3+...32"-length subtractions.
     
     32'b0000_0000_0000_0001_0000_0000_1000_0001 : there "1" means there stages. a chain of "1+2+3+...16" is the first stage; "17+...25" is the second stage; 26+27+...32" is the third stage.
     
 
 Input ports:
 
 * a (bit length is XLEN) : dividend
 
 * b (bit length is XLEN) : divisor
 
 * vld ( bit length is 1) : valid signal of this operation. "1" is valid.
 
 
 output ports:
 
 * quo (bit length is XLEN) : quotient
 
 * rem (bit length is XLEN) : remainder
 
 * ack ( bit length is 1) : acknowledge signal corresponds with "vld". It depends how many stages you put with "STAGE_LIST".
 
 
## tb.v ##

A simple verilog testbench file to verify "divfunc.v".

## How to use this divfunc.v ##

First please set STAGE_LIST to all-zero. You get a long critical path: N for (1+2+3+...+32 subtractions). You name an assumed critical path: M. Try to set "STAGE_LIST" with "1" in different bit poisition until your new ciritical path is less than M.

For example: A synthesis on DE2-115 FPGA:

   STAGE_LIST is 32'b0: We get that a critical path is 110 ns.
   
   My assumed critical path is 25 ns. I will try to set STAGE_LIST to 32'b0000_0000_1000_0001_0000_0010_0001_0001.
   
   It is not linear and try to make even. It is possible to use only 5~6 stages to complete a 32 bit/32 bit calculation.
     
