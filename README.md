# FPGA / RTL Design Interview Handbook
### Based on Morris Mano (5th Edition) + Modern FPGA Industry Practices

---

# Purpose

This repository contains interview-focused notes for FPGA Design, RTL Design,
ASIC Design, Digital Logic Design and Hardware Design interviews.

Unlike traditional textbook notes, this guide explains every concept from

- Beginner level
- Engineering level
- FPGA implementation level
- RTL Design level
- Interview level

with emphasis on

- Verilog HDL
- FPGA Architecture
- RTL Coding
- Timing Analysis
- Static Timing Analysis (STA)
- Synthesis
- Verification
- Debugging
- Design Trade-offs
- Industry Best Practices

---

# Primary Reference

Digital Design
M. Morris Mano
Michael D. Ciletti
5th Edition

---

# Additional References

- Xilinx Vivado Design Suite User Guides
- AMD/Xilinx UG901 (Synthesis)
- AMD/Xilinx UG949 (Design Methodology)
- Intel Quartus Prime Handbook
- NPTEL Digital Electronics
- IEEE Papers
- Glassdoor Interview Experiences
- AmbitionBox Interview Experiences
- Reddit FPGA Communities
- ASIC Design Blogs
- FPGA4Student
- ZipCPU
- Clifford Cummings SNUG Papers

---

# Interview Focus

This guide is intended for

- FPGA Design Engineer
- RTL Design Engineer
- ASIC Design Engineer
- ASIC Verification Engineer
- Hardware Design Engineer
- Embedded Hardware Engineer

Companies

- Mettlesemi
- AMD
- Intel
- Qualcomm
- NVIDIA
- Texas Instruments
- NXP
- Microchip
- Synopsys
- Cadence
- Siemens EDA
- Samsung
- Broadcom

---

# How to Study

Do NOT memorize.

For every topic ask yourself

1. What problem does it solve?

2. How does hardware implement it?

3. How would I write it in Verilog?

4. How does FPGA synthesize it?

5. What timing issues occur?

6. How do I verify it?

7. What mistakes are commonly made?

8. What follow-up questions can be asked?

If you can answer these questions, you understand the topic.

---

# Handbook Structure

01_Combinational_Logic.md

Boolean Algebra

Logic Gates

K-Maps

Hazards

MUX

DEMUX

Encoder

Decoder

Comparator

Adders

ALU

Tri-state Buffers

...

---

02_Sequential_Logic.md

Latches

Flip-Flops

Registers

Clocking

Setup

Hold

Metastability

CDC

...

---

03_FSM.md

Moore FSM

Mealy FSM

RTL Coding

State Encoding

Optimization

Debugging

Interview Questions

...

---

04_Counters.md

Ripple Counter

Synchronous Counter

Ring Counter

Johnson Counter

MOD-N Counter

Frequency Divider

FPGA Applications

---

05_FlipFlops.md

SR

JK

D

T

Master Slave

Reset

Enable

RTL Coding

---

06_Timing_STA.md

Setup Time

Hold Time

Propagation Delay

Contamination Delay

Critical Path

Slack

Timing Closure

STA

False Paths

Multi-cycle Paths

---

07_Verilog_Coding_Guidelines.md

RTL Coding Style

always_comb

always_ff

always_latch

Blocking vs Non-blocking

Latch Inference

Coding Standards

---

08_FPGA_Architecture.md

LUT

CLB

Slices

Carry Chains

DSP

BRAM

Clock Networks

Routing

Bitstream

---

09_FPGA_Debugging.md

Simulation

Waveforms

Assertions

ILA

Vivado Reports

Common RTL Bugs

---

10_Interview_Questions.md

Company-wise Questions

Design Problems

Follow-up Questions

Numericals

Timing Problems

---

11_Mettlesemi_Preparation.md

Expected Topics

Revision Checklist

Last-minute Revision Notes

HR + Technical Preparation

---

# Learning Philosophy

Digital Design is not about drawing gates.

It is about transforming

Problem

↓

Boolean Logic

↓

RTL

↓

Synthesis

↓

Netlist

↓

Timing

↓

Bitstream

↓

FPGA Hardware

Understanding this complete flow is what differentiates an FPGA Design Engineer from a student preparing only for university examinations.

---

# Estimated Length

~300+ pages

100+ Interview Questions

150+ Figures

100+ Timing Diagrams

70+ Verilog Examples

40+ Design Problems

20+ Mini Projects

This handbook is intended to be sufficient for internship interviews and entry-level FPGA/RTL Design roles.
