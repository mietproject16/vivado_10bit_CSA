## vivado_10bit_CSA
## REG NUM :25019176
## NAME :Arun M
## EXPERIMENT – 5  10-BIT CARRY SELECT ADDER (CSA)
##  TITLE
Implementation of a 10-bit Carry Select Adder (CSA) on Spartan-7 Boolean Board using Vivado

## AIM
To design, simulate, synthesize, and implement a 10-bit Carry Select Adder (CSA) using Verilog HDL and display the output on LEDs of the Boolean Board (Spartan-7 FPGA).

## TOOLS REQUIRED
Hardware
Spartan-7 Boolean Board
USB-JTAG Cable
DC Power (via USB)
Software
Xilinx Vivado Design Suite
Waveform Simulator (Vivado XSIM)

## THEORY

Addition is one of the most important operations in digital systems.
Ripple Carry Adder (RCA) is slow because the carry must propagate through all bits.

Carry Select Adder (CSA) solves this by:
 Splitting the input bits into blocks
 Computing sum for carry = 0 and carry = 1 in parallel
 Using a multiplexer to select the correct result

This reduces delay because long carry chains are avoided.
For a 10-bit CSA, we divide bits as:
Lower 5 bits → Ripple Carry Adder (direct)
Upper 5 bits → Two RCAs (one for Cin=0, one for Cin=1)
Mux → Selects correct sum depending on carry-out of lower block

## PROCEDURE
A. Vivado Steps
Open Vivado → Create New Project
Add Verilog source file (CSA.v, top.v)
Add Simulation sources (testbench)
Add XDC constraints file
Run behavioral simulation
Run Synthesis → Implementation → Generate Bitstream
Connect Boolean board and Program Device

## Boolean Board Steps
SW[9:0] → Input A
SW[19:10] → Input B
LED[9:0] → Output Sum
LED[10] → Carry Out

## VERILOG CODE
(A) Ripple Carry Adder – 5 bit
```
module rca5(input [4:0] A, B, input cin,
            output [4:0] S, output cout);

    assign {cout, S} = A + B + cin;

endmodule
```
## 10-bit Carry Select Adder
```
module csa10(
    input  [9:0] A,
    input  [9:0] B,
    output [9:0] SUM,
    output Cout
);

wire [4:0] s_low;
wire c_low;

wire [4:0] s_high0, s_high1;
wire c_high0, c_high1;

// Lower 5 bits RCA
rca5 RCA_LOW (A[4:0], B[4:0], 1'b0, s_low, c_low);

// Upper 5 bits RCA (Cin=0)
rca5 RCA_HIGH0 (A[9:5], B[9:5], 1'b0, s_high0, c_high0);

// Upper 5 bits RCA (Cin=1)
rca5 RCA_HIGH1 (A[9:5], B[9:5], 1'b1, s_high1, c_high1);

// Select correct result
assign SUM[4:0]  = s_low;
assign SUM[9:5]  = (c_low == 1'b0) ? s_high0 : s_high1;
assign Cout      = (c_low == 1'b0) ? c_high0 : c_high1;

endmodule
```
## Top Module for Boolean Board
```
module top(
    input  [9:0] SW_A,      // switches A
    input  [9:0] SW_B,      // switches B
    output [9:0] LED,       // Sum
    output LEDC             // Carry out
);

csa10 U1(
    .A(SW_A),
    .B(SW_B),
    .SUM(LED),
    .Cout(LEDC)
);

endmodule
```
## TESTBENCH
```
module tb_csa10;

reg [9:0] A, B;
wire [9:0] SUM;
wire Cout;

csa10 dut(A, B, SUM, Cout);

initial begin
    A = 10'd25;  B = 10'd17;  #10;  // 25 + 17 = 42
    A = 10'd100; B = 10'd50;  #10;  // 100+50=150
    A = 10'd512; B = 10'd300; #10;  // 512+300=812
    A = 10'd1023;B = 10'd1;   #10;  // overflow case
    $stop;
end

endmodule
```
## XDC FILE (Boolean Board)
Input SW_A[9:0]
Input SW_B[19:10]
LED[9:0] = SUM
LED[10] = Carry
```
# A switches
set_property -dict { PACKAGE_PIN V2 IOSTANDARD LVCMOS33 } [get_ports {SW_A[0]}]
set_property -dict { PACKAGE_PIN U2 IOSTANDARD LVCMOS33 } [get_ports {SW_A[1]}]
set_property -dict { PACKAGE_PIN U1 IOSTANDARD LVCMOS33 } [get_ports {SW_A[2]}]
set_property -dict { PACKAGE_PIN T2 IOSTANDARD LVCMOS33 } [get_ports {SW_A[3]}]
set_property -dict { PACKAGE_PIN T1 IOSTANDARD LVCMOS33 } [get_ports {SW_A[4]}]
set_property -dict { PACKAGE_PIN R2 IOSTANDARD LVCMOS33 } [get_ports {SW_A[5]}]
set_property -dict { PACKAGE_PIN R1 IOSTANDARD LVCMOS33 } [get_ports {SW_A[6]}]
set_property -dict { PACKAGE_PIN P2 IOSTANDARD LVCMOS33 } [get_ports {SW_A[7]}]
set_property -dict { PACKAGE_PIN P1 IOSTANDARD LVCMOS33 } [get_ports {SW_A[8]}]
set_property -dict { PACKAGE_PIN N2 IOSTANDARD LVCMOS33 } [get_ports {SW_A[9]}]
# B switches
set_property -dict { PACKAGE_PIN N1 IOSTANDARD LVCMOS33 } [get_ports {SW_B[0]}]
set_property -dict { PACKAGE_PIN M2 IOSTANDARD LVCMOS33 } [get_ports {SW_B[1]}]
...
(continue similar for SW_B[2] to SW_B[9])
# LEDs for SUM
set_property -dict { PACKAGE_PIN G1 IOSTANDARD LVCMOS33 } [get_ports {LED[0]}]
set_property -dict { PACKAGE_PIN G2 IOSTANDARD LVCMOS33 } [get_ports {LED[1]}]
...
(continue up to LED[9])
# Carry LED
set_property -dict { PACKAGE_PIN E6 IOSTANDARD LVCMOS33 } [get_ports {LEDC}]
```
## OUTPUT
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/329247d1-dc14-48b8-9306-66adbf3505e4" />

## RESULT
The 10-bit Carry Select Adder was successfully designed, simulated, synthesized, and implemented on the Spartan-7 Boolean board.
