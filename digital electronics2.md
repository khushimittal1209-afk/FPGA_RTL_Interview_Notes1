# FPGA / RTL Design Internship — Interview Notes
**Target roles:** Mettlesemi, AMD, Qualcomm, Intel, NVIDIA, Texas Instruments (RTL/FPGA/Digital Design Intern)
**Core reference:** Morris Mano, *Digital Design with an Introduction to the Verilog HDL*
**Supplementary flow:** NPTEL Digital Design with Verilog · NPTEL VLSI Circuits · AMD Vivado Synthesis Guide · Nandland · HDLBits

**How to read this document:** every topic follows the same interview arc —
`Logic → RTL → Synthesis → Timing → Verification`. This is the order interviewers actually probe in, and the order you should answer in.

---

## Priority Legend
- ★★★★★ Must-know — asked in almost every FPGA/RTL screen
- ★★★★☆ Very important — asked once fundamentals are cleared
- ★★★☆☆ Good to know — occasional, adds polish to answers

---

## Table of Contents
1. [Combinational Logic](#1-combinational-logic-) ★★★★★
2. [Sequential Logic](#2-sequential-logic-) ★★★★★
3. [Finite State Machines](#3-finite-state-machines-fsms-) ★★★★★
4. [Counters](#4-counters-) ★★★★★
5. [Flip-Flops](#5-flip-flops-) ★★★★★
6. [Timing](#6-timing-) ★★★★★
7. [Final Prep Kit](#7-final-prep-kit)
   - One-page cheat sheet
   - 50 most important interview questions
   - 20 common fresher mistakes
   - 3-day revision plan
   - Mock technical interview (Mettlesemi / AMD / Qualcomm / Intel / NVIDIA / TI style)

---
## 1. Combinational Logic ★★★★★

### 1.1 Beginner → Interview Explanation
Combinational logic is a circuit whose output depends **only on the current inputs**, with no memory of the past. No clock, no feedback loop, no state.
- Interview framing: "output = f(inputs), instantaneously (after gate delay)."
- Built from Boolean algebra → simplified (K-map / SOP / POS) → mapped to gates or a `case`/`assign` block in Verilog → synthesized into LUTs on an FPGA.
- The interview-ready way to describe *any* combinational block: **"For a given set of inputs, there is exactly one valid output, and if I ever see the output depend on a `always @(posedge clk)` without a state element, something is wrong."**

### 1.2 Why It Matters in FPGA/ASIC/RTL Design
- Every datapath (ALU, MUX trees, address decoders, priority encoders) is combinational logic underneath.
- Interviewers use it to check if you can go from **word problem → truth table → simplified logic → RTL** without hand-holding — this is the single most common "whiteboard" exercise.
- Bad combinational coding (latches, incomplete `case`, multi-driven nets) is the #1 source of first-week bugs for freshers, so interviewers probe it hard.

### 1.3 Practical Hardware Examples
- Address decoders in memory-mapped I/O.
- ALU adder/subtractor/comparator blocks.
- Priority encoders for interrupt controllers.
- MUX-based bus arbitration / data selection.
- Parity generators/checkers, CRC combinational trees.

### 1.4 Real Semiconductor-Project Usage
- Datapath ALUs in a CPU core.
- Combinational "next-state logic" feeding into an FSM's state register (the FSM itself is sequential, but its next-state function is pure combinational logic).
- Bus decoders in SoC interconnects (AXI/AHB address decode).
- Glue logic between IP blocks (glitch-sensitive — must be handled carefully near clock domain boundaries).

### 1.5 Common Fresher Mistakes
- Leaving a `case`/`if` incomplete → **unintended latch inference**.
- Using `always @(a or b)` instead of `always @(*)` → sensitivity list bugs (stale outputs in simulation).
- Assigning the same signal from two `always` blocks → multi-driver error.
- Confusing combinational hazards (real, physical glitches) with functional bugs.
- Forgetting a `default:` in `case` statements.
- Using blocking (`=`) incorrectly is *correct* here (blocking IS correct for combinational) — but freshers often mix it with nonblocking (`<=`) by habit from sequential code.

### 1.6–1.7 Interview Questions (Easy → Advanced) with Answers

**Q1 (Easy).** What defines combinational logic?
*A:* Output is a pure function of current inputs only; no memory, no clock.

**Q2 (Easy).** Implement a 2:1 MUX in Verilog.
*A:*
```verilog
assign y = sel ? a : b;
```

**Q3 (Easy).** Design a 1-bit full adder. Give sum/carry equations.
*A:* `sum = a ^ b ^ cin`, `cout = (a & b) | (cin & (a ^ b))`.

**Q4 (Medium).** Why does an incomplete `case` statement infer a latch?
*A:* Synthesis must preserve the previous output value when no branch matches (since RTL simulation "remembers" the last driven value) — the tool inserts a latch to hold that value. Fix: add `default:` or preassign the output at the top of the block.

**Q5 (Medium).** Difference between SOP and POS forms, and when would you prefer one?
*A:* SOP = OR of AND terms (natural for `0`-heavy truth tables solved via minterms); POS = AND of OR terms (natural when maxterms/zeros dominate). Preference depends on which minimizes literals/gates for a given function.

**Q6 (Medium).** What is a static hazard? How do you eliminate it?
*A:* A momentary, unwanted output glitch caused by unequal gate delays along different paths for a single input transition, even though the steady-state logic value doesn't change. Fixed by adding a redundant "consensus" term covering the transition (extra K-map grouping) or by ensuring the receiving logic is clocked (registered) so the glitch is not sampled.

**Q7 (Hard).** You see `output` toggling for one cycle in simulation but the RTL is "logically correct." Is this a bug?
*A:* Likely a combinational glitch/hazard from unequal path delays, not a functional bug — but it's a problem if downstream logic is asynchronous or level-sensitive (e.g., feeds a latch or async reset), because the glitch can be captured. Standard fix: don't let async logic depend on a glitchy combinational signal — register it first.

**Q8 (Hard).** Design a priority encoder (4-to-2) and explain how it differs from a plain encoder.
*A:* A plain encoder assumes only one input is active; priority encoder resolves multiple simultaneous active inputs by picking the highest-priority one. RTL uses a priority `casez`/`if-else` chain rather than parallel `assign`s.

```verilog
always @(*) begin
  casez (in[3:0])
    4'b1???: {y, valid} = {2'b11, 1'b1};
    4'b01??: {y, valid} = {2'b10, 1'b1};
    4'b001?: {y, valid} = {2'b01, 1'b1};
    4'b0001: {y, valid} = {2'b00, 1'b1};
    default: {y, valid} = {2'b00, 1'b0};
  endcase
end
```

### 1.8 Numerical Problems
1. Minimize `F(A,B,C) = Σm(1,3,5,6,7)` using a K-map. *(Answer: F = C + AB — walk through grouping in the interview, don't just state it.)*
2. A 4-input priority encoder has inputs asserted as `1010`. What is the encoded output and why?
   *(Answer: highest-priority set bit is bit 3 → output = `11` (binary for 3), because priority favors MSB.)*
3. Compute worst-case propagation delay of a 4-bit ripple-carry adder if each full adder's carry delay is 1.2 ns. *(Answer: 4 × 1.2 = 4.8 ns critical path through the carry chain — this is exactly why carry-lookahead/carry-chain hardware exists on FPGAs.)*

### 1.9 Timing Diagrams
Combinational logic has **no clock edge** — output reacts after propagation delay `tpd` following an input change:
```
input   a  ──┐                  ┌──────
              └──────────────────┘
                     |<--tpd-->|
output  y  ──────────┐          ┌───────
                       └──────────┘
```
Key point to say out loud in interviews: *"the output settles some finite delay after the input changes — that delay is what eats into my timing slack downstream."*

### 1.10 Verilog Implementation
Preferred style — `assign` for simple logic, `always @(*)` with full case coverage for complex logic:
```verilog
module mux4to1 (
    input  [1:0] sel,
    input  [7:0] a, b, c, d,
    output reg [7:0] y
);
  always @(*) begin
    case (sel)
      2'b00: y = a;
      2'b01: y = b;
      2'b10: y = c;
      default: y = d;   // covers 2'b11 AND prevents latch inference
    endcase
  end
endmodule
```

### 1.11 FPGA Implementation Considerations
- Maps directly to **LUTs** (Look-Up Tables). A 4-input LUT can implement any 4-input Boolean function in one level.
- Wide MUXes/decoders may need multiple LUT levels or dedicated MUX resources — check the tool's resource utilization report.
- Carry-chain-heavy logic (adders, comparators) should use the FPGA's dedicated **carry chain** hardware rather than generic LUTs, for both speed and area — inferred automatically from standard `+`/`-`/`>` operators in most tools, but worth mentioning you're aware of it.

### 1.12 Timing-Related Issues
- Combinational depth (number of logic levels between two registers) directly determines the **critical path** and hence max clock frequency.
- Deep combinational chains (e.g., a 32-bit priority encoder or wide adder tree) can become the bottleneck; pipelining (inserting registers mid-chain) is the standard fix.
- Glitches don't matter for timing closure math (STA uses settled values) but do matter for power (glitch = extra dynamic power) and for any asynchronous consumer of the signal.

### 1.13 Synthesis Considerations
- Synthesis vs simulation mismatch is the #1 danger zone: incomplete sensitivity lists were a classic Verilog-1995 pitfall (fixed by `always @(*)`), but incomplete `case`/`if` branches **still** silently create latches today — always state you check the synthesis log for "latch inferred" warnings.
- Tool will flatten/optimize Boolean expressions — you don't need to hand-minimize logic for real designs, but you should be able to do it on a whiteboard.

### 1.14 Verification/Debugging Considerations
- Exhaustive truth-table testing is feasible for small combinational blocks (≤ ~20 input bits) — mention **directed + exhaustive testbenches** for small MUX/decoder blocks.
- Watch waveforms for: outputs that "hold" a stale value after an input changes (classic latch symptom) and X-propagation on unconnected `case` branches.
- Assertions (`assert` / SVA) are great for combinational invariants, e.g., "exactly one-hot output for a decoder."

### 1.15 Common Follow-up Questions
- "How would you prove your MUX has zero glitches on the select line changing?"
- "What's the difference between a functional bug and a hazard, and does synthesis care?"
- "If I gave you a 64-input priority encoder, how would you structure the RTL for both readability and timing?"

### 1.16 Connections to Computer Architecture / RTL
- The ALU is the canonical combinational block in a datapath; its output feeds a pipeline register.
- Address decode logic (which memory/peripheral a bus transaction targets) is pure combinational logic gating control signals.
- In a pipelined CPU, combinational logic sits **between** pipeline stage registers — its total delay must fit inside one clock period, directly setting the achievable clock frequency (this is the bridge into the Timing section).

---
## 2. Sequential Logic ★★★★★

### 2.1 Beginner → Interview Explanation
Sequential logic has **memory** — output depends on current inputs *and* past state, stored in flip-flops/latches and updated on a clock edge.
- Interview framing: "next_state = f(current_state, inputs); output either equals state (Moore-like) or f(state, inputs) (Mealy-like)."
- Every register, counter, FSM, pipeline stage, and shift register is sequential logic.

### 2.2 Why It Matters in FPGA/ASIC/RTL Design
- All "real" digital systems are sequential — combinational logic alone can't remember anything across clock cycles.
- Interviewers use this topic to check clock discipline: do you understand **why** we use synchronous design, why nonblocking assignments matter, and how registers interact with timing closure.

### 2.3 Practical Hardware Examples
- Registers/pipeline flops.
- Shift registers (SIPO/PISO) for serial interfaces (SPI, UART).
- Memory elements: register files, FIFOs.
- Debounce circuits for mechanical switches.

### 2.4 Real Semiconductor-Project Usage
- Pipeline registers between combinational stages in a CPU/DSP datapath.
- FIFOs at clock-domain-crossing boundaries.
- Control/status registers (CSRs) in a peripheral IP block.
- Shift registers implementing serial protocol shift-in/shift-out logic.

### 2.5 Common Fresher Mistakes
- Mixing blocking (`=`) and nonblocking (`<=`) assignments in the same sequential `always` block.
- Not understanding **why** nonblocking is mandatory for sequential logic (race conditions / simulation-synthesis mismatch).
- Missing/incorrect reset logic — forgetting whether reset is synchronous or asynchronous, active-high or active-low.
- Inferring a latch instead of a flip-flop by not gating with a proper clock edge (`@(posedge clk)`).
- Multiple always blocks driving the same register.

### 2.6–2.7 Interview Questions with Answers

**Q1 (Easy).** Why must sequential logic use nonblocking assignments (`<=`)?
*A:* Nonblocking assignments schedule updates to happen simultaneously at the end of the time step, matching real hardware where all flip-flops sample their D input on the same clock edge and update together. Blocking assignments execute immediately/sequentially, which can create order-dependent race conditions and mismatches between simulation and synthesized hardware.

**Q2 (Easy).** Write a simple D flip-flop with synchronous active-high reset.
```verilog
always @(posedge clk) begin
  if (rst) q <= 1'b0;
  else     q <= d;
end
```

**Q3 (Medium).** What's the difference between a latch and a flip-flop?
*A:* A latch is level-sensitive (transparent while enable is high — output follows input); a flip-flop is edge-sensitive (captures input only at the clock edge). FPGA/ASIC synchronous design almost always wants flip-flops; latches are generally avoided (glitch-prone, harder to time).

**Q4 (Medium).** Synchronous vs asynchronous reset — tradeoffs?
*A:* Synchronous reset only takes effect on a clock edge — clean, glitch-free, but requires the clock to be running to reset, and can need a longer reset pulse to guarantee capture. Asynchronous reset takes effect immediately (independent of the clock) — resets even without a clock, but risks **reset removal/recovery** timing violations if the reset deassertion isn't synchronized to the clock, potentially causing metastability. Best practice: **asynchronous assert, synchronous deassert** (reset synchronizer).

**Q5 (Hard).** What happens if two `always @(posedge clk)` blocks both drive the same reg?
*A:* Multiple-driver conflict — synthesis error/unpredictable simulation. Only one always block (or single continuous assignment) should drive any given signal.

**Q6 (Hard).** Explain why nonblocking assignment order doesn't matter but blocking order does, with an example of swapping two registers.
*A:*
```verilog
// Correct 2-flop swap using nonblocking:
always @(posedge clk) begin
  a <= b;
  b <= a;
end
// Both RHS values are sampled BEFORE either LHS updates -> true swap.

// Broken with blocking:
always @(posedge clk) begin
  a = b;   // a updated immediately
  b = a;   // now b gets the NEW a, not the old one -> NOT a swap
end
```

### 2.8 Numerical Problems
1. A shift register has an initial value `1000` and shifts left, shifting in `1` from the right each cycle. What is the value after 3 clock edges? *(Answer: `0011`... work through cycle by cycle: 1000→0001+shift-in=0001? — walk through explicitly on the whiteboard with the exact convention stated.)*
2. If a synchronous reset must guarantee reset over 2 clock cycles at 200 MHz, what is the minimum reset pulse width? *(Answer: 2 × (1/200MHz) = 10 ns.)*

### 2.9 Timing Diagrams
```
clk    __|‾|__|‾|__|‾|__|‾|__
d      ──X===X===X===X───────   (d changes anytime)
q      ──────X===X===X===X───   (q updates only at posedge clk, one cycle later)
              ^        ^
           captures  captures
```
Key interview line: *"q always lags d by exactly one clock edge — that lag is the entire point of a synchronous register."*

### 2.10 Verilog Implementation
```verilog
module sync_reg #(parameter W=8) (
    input             clk,
    input             rst_n,   // async active-low reset, synchronously deasserted upstream
    input      [W-1:0] d,
    output reg [W-1:0] q
);
  always @(posedge clk or negedge rst_n) begin
    if (!rst_n) q <= {W{1'b0}};
    else        q <= d;
  end
endmodule
```

### 2.11 FPGA Implementation Considerations
- Every flip-flop on an FPGA fabric already has a hardware reset pin and often a clock-enable pin — use them instead of manually gating the clock ("clock gating" via logic is bad practice on FPGAs).
- Prefer synchronous-enable style (`if (en) q <= d;`) over gating the clock signal itself, which creates skew/glitch problems.

### 2.12 Timing-Related Issues
- Setup/hold at every flip-flop input (see Timing section for the deep dive).
- Reset recovery/removal timing for asynchronous resets — the deassertion edge must be synchronized (2-flop synchronizer) so it doesn't violate setup/hold on some flops and not others.

### 2.13 Synthesis Considerations
- Synthesis tools infer a flip-flop when they see an edge-triggered `always` block with proper `<=` usage; mixing styles (blocking in a clocked block) can generate warnings or unintended latches/combinational loops.
- Reset style (sync vs async) changes the flip-flop primitive chosen and can affect max frequency — synchronous reset adds reset logic into the D-input's combinational path, potentially reducing Fmax slightly vs async reset.

### 2.14 Verification/Debugging Considerations
- Always check reset behavior first in simulation — a design that "mostly works" but glitches on reset deassertion is a classic first bug.
- Watch for X-propagation: an uninitialized register in simulation showing `X` for many cycles usually means a missing reset path.

### 2.15 Common Follow-up Questions
- "Why is async-assert/sync-deassert considered best practice?"
- "What would happen in real silicon if you used blocking assignments in a sequential always block?"
- "How would you verify that your reset properly initializes every register in the design?"

### 2.16 Connections to Computer Architecture / RTL
- Pipeline registers between CPU stages are exactly this: sequential elements storing intermediate results every cycle.
- Register files (the CPU's fast internal storage) are arrays of sequential elements with combinational read/write decode logic wrapped around them.
- The concept of "one clock edge = one unit of forward progress" underlies pipelining, hazard detection, and throughput analysis in architecture.

---
## 3. Finite State Machines (FSMs) ★★★★★

### 3.1 Beginner → Interview Explanation
An FSM is a sequential circuit with a finite set of states, where the next state is a function of the current state (and possibly inputs), and outputs are either a function of state only (**Moore**) or of state and inputs (**Mealy**).
- Interview one-liner: "Moore = safer/slower (output changes only on clock edge), Mealy = faster/more reactive (output can change combinationally with input mid-cycle) but glitchier."

### 3.2 Why It Matters in FPGA/ASIC/RTL Design
- FSMs are the **most commonly asked whiteboard RTL design question** in FPGA/RTL interviews — control logic for nearly every protocol (SPI, UART, AXI handshakes, arbitration) is an FSM.
- Interviewers use FSM questions to simultaneously test: state encoding choices, reset behavior, blocking/nonblocking discipline across two always blocks, and latch-avoidance in the next-state logic.

### 3.3 Practical Hardware Examples
- UART/SPI/I2C protocol controllers.
- Traffic light controller (classic textbook example, still asked).
- Bus arbiter (grant logic across multiple requesters).
- Vending machine controller (classic teaching example for Moore vs Mealy).

### 3.4 Real Semiconductor-Project Usage
- Handshake control logic for AXI/AHB bus transactions.
- Power-management sequencers (power-up/power-down state sequencing).
- DMA engine control paths.
- Protocol-layer control state machines in networking/communication IP.

### 3.5 Common Fresher Mistakes
- Using a single always block for both state register and next-state/output logic, mixing blocking/nonblocking styles.
- Incomplete `case` in next-state logic → latch inference in what should be purely combinational logic.
- Forgetting a proper reset/idle state, leaving unreachable or undefined states.
- Confusing Mealy outputs (combinational, can glitch) with Moore outputs (registered, clean) and using Mealy where a clean, glitch-free output was required.
- Not handling all encoded states in a `case` (e.g., using 3 bits for 5 states but not handling the 3 unused/invalid encodings) — a good design should default unused states back to a safe/idle state for robustness.

### 3.6–3.7 Interview Questions with Answers

**Q1 (Easy).** Moore vs Mealy — core difference?
*A:* Moore: output = f(state) only → output changes synchronously with the clock, glitch-free, but may take one extra cycle to react to an input. Mealy: output = f(state, input) → output can react immediately (same cycle) as an input changes, but combinationally, so it can glitch if the input is glitchy.

**Q2 (Easy).** What is the standard 3-always-block FSM coding style, and why use it?
*A:* (1) sequential always block for the state register (`state <= next_state` on clock edge), (2) combinational always block for next-state logic (`next_state = f(state, inputs)`), (3) combinational (or registered, for Moore) always block for output logic. Separating them keeps each block simple, avoids blocking/nonblocking mixing bugs, and makes the design easy to verify and re-time.

**Q3 (Medium).** Binary vs Gray vs one-hot state encoding — when to use each?
*A:* Binary: fewest flip-flops (log2(N) bits), but more complex next-state decode logic — good when flip-flops are scarce (ASIC-area constrained). One-hot: one flip-flop per state, very simple/fast decode logic (good for FPGAs, which have abundant flip-flops and want short combinational paths) but uses more flops. Gray: only one bit changes between adjacent states — useful for CDC-crossing state signals to minimize glitch/metastability risk during transitions.

**Q4 (Medium).** Design a Moore FSM for a simple traffic light (Red→Green→Yellow→Red) with a timer.
*A:* States: `RED, GREEN, YELLOW`. Each state has an associated fixed-duration counter; on `counter == max`, transition to next state and reset counter. Output (light color) depends only on state → Moore machine, glitch-free light output.

**Q5 (Hard).** How would you make sure your FSM can't get "stuck" in an unreachable/invalid state (e.g., due to an SEU or bad reset)?
*A:* Add a `default:` branch in the next-state `case` that forces a transition back to a known-safe state (usually IDLE/RESET state). This is especially critical for one-hot encodings, where illegal multi-hot states can otherwise occur.

**Q6 (Hard).** Convert a Mealy FSM to an equivalent Moore FSM — what's the general cost?
*A:* Generally requires adding extra states to "absorb" input-dependent output transitions into state-dependent ones, which can add one cycle of latency and increase the state count. Interview answer: draw a small handshake example (e.g., `req`/`ack`) and show the extra state needed.

### 3.8 Numerical Problems
1. A one-hot FSM has 6 states. How many flip-flops does one-hot encoding need vs binary encoding? *(Answer: one-hot = 6 flops; binary = ceil(log2(6)) = 3 flops.)*
2. An FSM's longest combinational next-state path is 3.5 ns; register setup time is 0.3 ns; clock-to-Q delay is 0.4 ns. What's the max clock frequency (ignore clock skew/margin)? *(Answer: Tclk_min = 0.4 + 3.5 + 0.3 = 4.2 ns → Fmax ≈ 238 MHz.)*

### 3.9 Timing Diagrams
```
clk        __|‾|__|‾|__|‾|__|‾|__
state      IDLE  X  REQ  X  WAIT X  DONE
                  ^        ^        ^
             updates only at posedge (Moore output tracks 'state' cleanly)
mealy_out  ──────╲        ╲────╱────  (can wiggle mid-cycle if input changes)
```

### 3.10 Verilog Implementation (3-always-block style, one-hot, Moore)
```verilog
module fsm_example (
    input  clk, rst_n, start, done_in,
    output reg busy
);
  localparam IDLE = 2'd0, RUN = 2'd1, DONE = 2'd2;
  reg [1:0] state, next_state;

  // (1) state register
  always @(posedge clk or negedge rst_n)
    if (!rst_n) state <= IDLE;
    else        state <= next_state;

  // (2) next-state logic (pure combinational)
  always @(*) begin
    next_state = state; // default: hold current state, prevents latch
    case (state)
      IDLE: if (start)   next_state = RUN;
      RUN:  if (done_in) next_state = DONE;
      DONE:               next_state = IDLE;
      default:             next_state = IDLE;
    endcase
  end

  // (3) Moore output logic
  always @(*) begin
    busy = (state == RUN);
  end
endmodule
```

### 3.11 FPGA Implementation Considerations
- One-hot encoding is generally preferred on FPGAs because flip-flops are cheap/abundant and it minimizes combinational logic depth (faster Fmax) — many synthesis tools (including Vivado) will auto re-encode a binary FSM to one-hot unless told otherwise.
- Check the synthesis tool's FSM extraction report to confirm it recognized your `case` structure as an FSM (enables state-reachability optimizations and safe-state insertion).

### 3.12 Timing-Related Issues
- Next-state combinational logic depth directly sets FSM critical path — deeply nested `if`/`case` conditions can become the bottleneck.
- Mealy outputs sitting on the critical path (combinational fan-out to other logic) can hurt timing closure more than Moore outputs, since they add an extra combinational hop after the state decode.

### 3.13 Synthesis Considerations
- Always include a `default` in both the next-state case and, if used, the output case — this both prevents latch inference and gives the synthesis tool a legal state to fall back into (important for `safe FSM` / SEU-hardening synthesis attributes some tools support).
- Confirm via the synthesis report that no latches were inferred in the next-state or output logic.

### 3.14 Verification/Debugging Considerations
- Cover every state and every legal transition in your testbench (state-transition coverage) — this is a standard interview follow-up ("how do you verify an FSM?").
- Explicitly test illegal/unreachable state recovery, especially for one-hot designs.
- Waveform debugging: if `state` looks correct but `busy`/output looks wrong for a cycle, check whether you accidentally wrote Mealy-style combinational output logic when you intended Moore.

### 3.15 Common Follow-up Questions
- "How would you verify all states are reachable and all transitions are covered?"
- "What synthesis attribute would you use to force one-hot vs binary encoding, and why would you override the tool's default?"
- "If your FSM's output needs to react in the same cycle as an input, can you still use Moore-style safely?"

### 3.16 Connections to Computer Architecture / RTL
- CPU control units (in a non-pipelined/microcoded design) are literally FSMs sequencing fetch/decode/execute.
- Cache controllers, bus protocol engines, and DMA sequencers are all FSMs at heart.
- FSM design directly exercises the "next-state combinational logic feeding a state register" pattern that is the architectural template for the entire control path of a processor.

---
## 4. Counters ★★★★★

### 4.1 Beginner → Interview Explanation
A counter is a special-case FSM whose state *is* the count value, incrementing/decrementing/wrapping each clock cycle (optionally gated by enable/load).
- Interview one-liner: "a counter is just a register plus an adder feeding back into itself, with synchronous design strongly preferred over asynchronous (ripple) counters."

### 4.2 Why It Matters in FPGA/ASIC/RTL Design
- Counters are the second most common coding exercise after FSMs/MUXes — timers, baud-rate generators, address generators, and clock dividers are all built from counters.
- Tests whether you default to **synchronous design** (all flops clocked by the same clock) vs asynchronous/ripple design, which is a red flag for FPGA-focused roles.

### 4.3 Practical Hardware Examples
- Baud-rate generator for UART.
- Frequency divider (clock-enable generation, NOT literal clock division on FPGA fabric — see below).
- Address generator for a memory scan/test pattern.
- Watchdog timers, debounce timers.
- Johnson/ring counters for simple sequencing (e.g., LED chasers, low-speed state sequencers).

### 4.4 Real Semiconductor-Project Usage
- Programmable timers/watchdogs in SoC peripheral IP.
- PWM generators (counter + comparator).
- Address sequencing in built-in self-test (BIST) logic.
- Mod-N counters generating periodic enable pulses for slower sub-blocks (clock-enable, not literal clock gating, on FPGAs).

### 4.5 Common Fresher Mistakes
- Building an **asynchronous ripple counter** on an FPGA (chaining flip-flop clock inputs) — causes huge timing/glitch problems; always prefer synchronous counters.
- Forgetting to handle the wraparound/terminal-count case (`if (count == MAX) count <= 0; else count <= count+1;`).
- Using the counter's output bit directly as a clock for another block ("clock gating by hand") instead of generating a clock-enable pulse.
- Off-by-one errors in mod-N counter boundaries.
- Not resetting the counter properly, leading to `X` propagation in simulation.

### 4.6–4.7 Interview Questions with Answers

**Q1 (Easy).** Design a 4-bit synchronous up-counter with synchronous reset.
```verilog
always @(posedge clk) begin
  if (rst) count <= 4'd0;
  else     count <= count + 1'b1;   // wraps naturally on overflow
end
```

**Q2 (Easy).** What's the difference between a synchronous and an asynchronous (ripple) counter?
*A:* Synchronous: all flip-flops share the same clock, next value computed combinationally from current count — clean, predictable timing. Asynchronous/ripple: each flip-flop's clock is driven by the previous stage's output — count "ripples" through with cumulative delay, causing glitches and making static timing analysis very difficult. FPGA/ASIC synchronous design essentially never uses ripple counters in practice.

**Q3 (Medium).** Design a mod-10 (decade) counter.
```verilog
always @(posedge clk) begin
  if (rst) count <= 4'd0;
  else if (count == 4'd9) count <= 4'd0;
  else count <= count + 1'b1;
end
```

**Q4 (Medium).** How do you generate a 1 MHz enable pulse from a 100 MHz clock using a counter?
*A:* Free-running mod-100 counter; assert a one-cycle-wide `enable` pulse when `count == 99` (then wrap to 0). This is a "clock-enable," not an actual divided clock — the FPGA fabric still runs at 100 MHz; downstream logic samples data only when `enable` is high.
```verilog
always @(posedge clk) begin
  if (rst) begin count <= 0; en <= 0; end
  else if (count == 99) begin count <= 0; en <= 1; end
  else begin count <= count + 1; en <= 0; end
end
```

**Q5 (Hard).** Design a Johnson counter and explain one practical use.
*A:* An n-bit shift register where the complement of the output is fed back to the input, producing 2n unique states with only one bit changing per transition (like Gray code) — useful for low-glitch state sequencing (e.g., simple LED chasers or low-speed one-hot-like sequencers) since only a single bit toggles each cycle, minimizing decode glitches.

**Q6 (Hard).** How would you build an up/down counter with load capability?
```verilog
always @(posedge clk) begin
  if (rst)        count <= 0;
  else if (load)  count <= load_val;
  else if (up_dn) count <= count + 1;
  else            count <= count - 1;
end
```

### 4.8 Numerical Problems
1. How many bits are needed for a counter that counts from 0 to 200? *(Answer: ceil(log2(201)) = 8 bits.)*
2. At 50 MHz, how many clock cycles does a mod-50,000,000 counter take to produce a 1 Hz enable pulse? *(Answer: exactly 50,000,000 cycles = 1 second.)*
3. A ripple counter has 4 stages, each flip-flop with 2 ns clock-to-Q delay. What is the worst-case delay before the MSB is valid after a clock edge? *(Answer: 4 × 2 ns = 8 ns cumulative ripple delay — illustrates exactly why ripple counters are avoided in synchronous FPGA design.)*

### 4.9 Timing Diagrams
```
clk    __|‾|__|‾|__|‾|__|‾|__
count   0    1    2    3    0     (synchronous: all bits update together, same edge)

ripple counter (for contrast):
clk      __|‾|________________
bit0     ____|‾|________________     (toggles on clk edge)
bit1     ________|‾|____________     (toggles on bit0's falling edge -> delayed!)
                 ^ cumulative delay grows with each stage
```

### 4.10 Verilog Implementation
```verilog
module up_down_counter #(parameter W = 8, MAXV = 255) (
    input               clk, rst, load, up_dn, en,
    input      [W-1:0]  load_val,
    output reg [W-1:0]  count
);
  always @(posedge clk) begin
    if (rst)            count <= 0;
    else if (load)       count <= load_val;
    else if (en && up_dn) count <= (count == MAXV) ? 0 : count + 1;
    else if (en)          count <= (count == 0)    ? MAXV : count - 1;
  end
endmodule
```

### 4.11 FPGA Implementation Considerations
- Synchronous counters map efficiently onto FPGA **carry chains** — the incrementer is essentially an adder, so wide counters (32/64-bit) still close timing well because they use the dedicated carry-chain hardware, not generic LUT logic.
- Never derive a literal secondary clock from a counter bit and route it through general fabric to another flip-flop's clock pin — use a clock-enable + the primary clock, or a proper Clock Management Tile / MMCM/PLL for real frequency division.

### 4.12 Timing-Related Issues
- Wide counters (comparators like `count == MAX`) can have a long combinational compare path if not registered properly — usually fine since it's part of the carry chain, but worth checking in STA reports for very wide counters.
- Enable/load muxing in front of the adder adds a small combinational delay before the carry chain — keep the enable/load logic simple to avoid growing the critical path.

### 4.13 Synthesis Considerations
- Synthesis tools will typically infer a dedicated carry-chain adder automatically from `count + 1` — verify via utilization/timing reports rather than assuming.
- Confirm reset behavior (sync vs async) matches the rest of the design's global reset strategy for consistency.

### 4.14 Verification/Debugging Considerations
- Verify wraparound behavior at the boundary (MAX → 0) explicitly — a very common off-by-one bug site.
- For enable-pulse generators, verify the pulse is exactly one clock cycle wide (a common bug is a pulse that's accidentally 2 cycles wide due to combinational vs registered enable logic).

### 4.15 Common Follow-up Questions
- "Why would you never use an asynchronous ripple counter in an FPGA design?"
- "How would you divide a 100 MHz clock down to exactly 33.3 MHz using counters — is that even possible cleanly?" *(Answer: not with a simple integer divider producing a 50% duty cycle; needs a fractional/PLL-based approach, or accept duty-cycle distortion.)*
- "How do you handle a counter whose enable and reset arrive from different clock domains?"

### 4.16 Connections to Computer Architecture / RTL
- The Program Counter (PC) in a CPU is literally a specialized counter (increment each cycle, with load capability for branches/jumps).
- Pipeline stage counters / cycle counters for performance monitoring reuse the same synchronous up-counter pattern.
- Address generation logic for sequential memory access (e.g., instruction fetch, burst transfers) is built directly from counter RTL.

---
## 5. Flip-Flops ★★★★★

### 5.1 Beginner → Interview Explanation
A flip-flop is the fundamental **edge-triggered** memory element: it samples its input(s) at a clock edge and holds the value stable until the next edge.
- Interview one-liner: "a flip-flop is a 1-bit synchronous memory cell; everything sequential (registers, counters, FSMs, pipelines) is built by wiring flip-flops together with combinational logic."
- Know the family: **SR, D, JK, T** latches/flip-flops conceptually (Mano's book), but in modern RTL you almost always code the **D flip-flop** — synthesis tools map it to whatever the target library actually has.

### 5.2 Why It Matters in FPGA/ASIC/RTL Design
- Flip-flops are the atomic unit interviewers use to test your grasp of setup/hold, metastability, and clock domain crossing (CDC) — arguably the deepest and most differentiating topic in an FPGA interview.
- Getting flip-flop fundamentals right (D-FF only, synchronous design, correct reset style) signals "RTL-ready" vs "still thinking academically."

### 5.3 Practical Hardware Examples
- 2-flop synchronizer for CDC of a single control/status bit.
- Register file storage cells.
- Pipeline stage boundary registers.
- Metastability-hardened input synchronizers on FPGA I/O pins.

### 5.4 Real Semiconductor-Project Usage
- Every register in RTL (control registers, data registers, pipeline registers) is built from D flip-flops.
- CDC synchronizers (2-flop, or handshake-based for multi-bit buses) are mandatory wherever two clock domains meet — a huge fraction of real bugs in silicon come from missed/incorrect CDC synchronization.
- Reset synchronizers (async assert, sync deassert) use a small chain of flip-flops.

### 5.5 Common Fresher Mistakes
- Using a **latch** (level-sensitive) where a flip-flop (edge-sensitive) was intended — usually from incomplete combinational always-block coding, not an explicit choice.
- Directly using a signal from another clock domain without a synchronizer — the single most common real-silicon bug category for freshers.
- Not understanding metastability — believing "it usually works in simulation" is proof of correctness (simulation cannot model metastability realistically).
- Forgetting that JK/T flip-flops are academic teaching tools — writing RTL as if the synthesis tool needs an explicit JK primitive (it doesn't; you always describe behavior with D-FF-style always blocks).

### 5.6–5.7 Interview Questions with Answers

**Q1 (Easy).** SR, D, JK, T flip-flop — one-line characteristic equation for each?
*A:* SR: `Q+ = S + R'Q` (S=R=1 forbidden). D: `Q+ = D`. JK: `Q+ = JQ' + K'Q` (toggles when J=K=1). T: `Q+ = T⊕Q` (toggles when T=1).

**Q2 (Easy).** Why is the D flip-flop the standard building block in RTL, not JK/T?
*A:* D-FF has the simplest, most direct behavioral description (`q <= d`), maps cleanly and efficiently to standard-cell/FPGA flip-flop primitives, and every other flip-flop type (T, JK) can be built from a D-FF plus combinational logic if ever needed — so HDLs and tool libraries standardize on D.

**Q3 (Medium).** What is metastability, and why can't you "just check it in simulation"?
*A:* Metastability occurs when a flip-flop's input changes too close to the clock edge (violates setup/hold), causing the output to settle to an unpredictable, possibly intermediate voltage for an indeterminate time before resolving to 0 or 1. Simulation models clean digital 0/1 logic with fixed delays — it doesn't naturally model the analog, probabilistic resolution behavior of a real metastable flop, so a design can "pass" simulation while still being unsafe in silicon.

**Q4 (Medium).** Design a 2-flop synchronizer for crossing a single-bit signal into a new clock domain, and explain why 2 flops (not 1).
```verilog
always @(posedge clk_dst or negedge rst_n) begin
  if (!rst_n) {sync2, sync1} <= 2'b00;
  else        {sync2, sync1} <= {sync1, async_in};
end
```
*A:* The first flop may go metastable (input arrived asynchronously), but it's given a full clock period to resolve before the second flop samples it — reducing the probability of propagating metastability downstream to an acceptably low (MTBF-driven) level. A single flop gives no resolution time; multi-bit buses need handshaking or gray-coding, not a plain 2-flop sync (bits could be sampled inconsistently).

**Q5 (Hard).** Why can't you use a 2-flop synchronizer directly on a multi-bit bus?
*A:* Different bits may resolve/settle at different times if multiple bits change simultaneously (bus is not guaranteed to change atomically as seen by the destination clock), causing bit-level corruption ("bus glitch") even if each individual bit is metastability-safe. Solutions: Gray-code the bus (only 1 bit changes at a time) for counters, or use a full req/ack handshake or async FIFO for general multi-bit data.

**Q6 (Hard).** What is the difference between setup time, hold time, and how do they relate to flip-flop design margin?
*A:* Setup time = minimum time D must be stable **before** the clock edge; hold time = minimum time D must remain stable **after** the clock edge. Violating either can cause metastability or incorrect capture. (Deep dive continues in the Timing section.)

### 5.8 Numerical Problems
1. A flip-flop has setup time 0.2 ns and hold time 0.1 ns. If clk-to-Q delay is 0.3 ns and the next stage's combinational delay is 1.5 ns, what's the minimum clock period (ignore skew/margin)? *(Answer: Tclk ≥ Tcq + Tcomb + Tsetup = 0.3+1.5+0.2 = 2.0 ns → Fmax = 500 MHz.)*
2. A 2-flop synchronizer's MTBF is dominated by the metastability resolution time constant τ = 200 ps and a per-cycle metastability window of 10 ps at 100 MHz with a signal toggling 1 MHz. Interview answer: you're not expected to compute exact MTBF numbers from scratch, but should be able to state the MTBF formula conceptually: `MTBF = e^(t_resolve/τ) / (T0 × f_clk × f_data)` and explain that more synchronizer stages / longer resolution time exponentially improves MTBF.

### 5.9 Timing Diagrams
```
Setup/Hold window around clock edge:
                     |<-Tsu->|<-Th->|
d      ─────XXXXXXXXXX=======XXXXXXX───   (must be stable across [Tsu before, Th after] edge)
clk    __________________|‾|______________
q      ──────────────────────X========    (captured value appears after Tcq)
```

### 5.10 Verilog Implementation
```verilog
// Canonical D flip-flop with synchronous enable and async active-low reset
module dff_en (
    input      clk, rst_n, en,
    input      d,
    output reg q
);
  always @(posedge clk or negedge rst_n) begin
    if (!rst_n)   q <= 1'b0;
    else if (en)  q <= d;
  end
endmodule
```

### 5.11 FPGA Implementation Considerations
- FPGA fabric flip-flops natively support: clock, synchronous enable, and one reset/set pin (async or sync, configurable) — this maps 1:1 to well-written RTL with a single `if (rst)` / `if (en)` pattern; don't try to build complex custom flip-flop behavior with extra combinational logic wrapped around a basic D-FF when the fabric primitive already supports it (wastes LUTs and can hurt timing).
- Cross-domain 2-flop synchronizers should be placed with a synthesis "ASYNC_REG" (Vivado) or equivalent attribute so the tool keeps the two flops physically close and doesn't optimize/retime across them.

### 5.12 Timing-Related Issues
- Setup/hold violations are the core timing failure mode at every flip-flop boundary — STA checks every register-to-register path against the clock period.
- Clock skew (different arrival times of the clock edge at different flip-flops) eats into setup margin (if source flop's clock is delayed relative to destination) or can even cause hold violations (if destination's clock arrives earlier than expected).

### 5.13 Synthesis Considerations
- Synthesis will pick the actual flip-flop cell (with/without reset, with/without enable) based on your RTL description — the more the RTL structurally matches an available primitive (e.g., D-FF with sync reset + enable), the more efficient the mapping.
- Avoid "gated clock" patterns (`assign gated_clk = clk & en;` then clocking a flop with `gated_clk`) — use the flip-flop's native enable pin via synchronous-enable RTL style instead, since manual clock gating creates glitch and timing-closure headaches on FPGAs (it's a normal, tool-managed technique in ASIC low-power flows, but should be done via proper clock-gating cells, not ad-hoc logic).

### 5.14 Verification/Debugging Considerations
- CDC-specific static checks (e.g., a CDC linting tool) are standard in real projects — mention this as awareness even if you haven't used one personally.
- In simulation, an uninitialized flip-flop showing persistent `X` is almost always a missing/incorrect reset connection.

### 5.15 Common Follow-up Questions
- "How would you verify that a CDC synchronizer is correctly implemented, beyond just functional simulation?"
- "What's the difference between metastability and a functional bug, from a debug perspective?"
- "If your design has 50 clock-domain-crossing signals, how would you approach synchronizing all of them efficiently and safely?"

### 5.16 Connections to Computer Architecture / RTL
- Every architectural register (PC, register file entries, pipeline latches) is a D flip-flop (or bank of them) under the hood.
- CDC synchronization is central to any SoC with multiple clock domains (CPU core clock vs peripheral bus clock vs I/O clock) — architecture-level clock domain planning depends directly on flip-flop-level synchronizer design.

---
## 6. Timing ★★★★★

### 6.1 Beginner → Interview Explanation
Timing is the study of **whether signals arrive where they need to be, when they need to be there**, relative to the clock. It's the bridge between "my RTL is functionally correct in simulation" and "my design actually works in real silicon at the target frequency."
- Interview one-liner: "Timing closure means every register-to-register path satisfies setup and hold constraints across all corners, at the target clock frequency."

### 6.2 Why It Matters in FPGA/ASIC/RTL Design
- Timing is where interviewers separate candidates who can write correct-looking RTL from candidates who understand **why a design that simulates perfectly can still fail in hardware**.
- Nearly every senior/lead RTL engineer's daily work involves timing closure — asking about it tests whether you're thinking like a real engineer, not just a student.

### 6.3 Practical Hardware Examples
- Static Timing Analysis (STA) reports from Vivado/Synopsys PrimeTime showing worst negative slack (WNS).
- Multicycle path constraints for slow control signals that don't need to close in 1 cycle.
- False path constraints for signals that are functionally never sampled together (e.g., across independent CDC-safe paths already synchronized elsewhere).

### 6.4 Real Semiconductor-Project Usage
- Every tapeout/FPGA bitstream generation flow ends with a timing closure signoff step.
- Pipeline depth decisions (how many pipeline stages to add) are driven directly by timing analysis of combinational path delays.
- Clock domain crossing verification (CDC tools + STA false-path/multicycle constraints) is a mandatory sign-off step in real projects.

### 6.5 Common Fresher Mistakes
- Treating "simulation passed" as proof the design will meet timing — timing is a completely separate analysis (STA), not something simulation checks.
- Not understanding the difference between setup violation (path too slow) and hold violation (path too fast / clock skew issue) — freshers often can't explain why a hold violation *cannot* be fixed by slowing the clock.
- Ignoring clock skew and clock uncertainty margins when reasoning about Fmax.
- Assuming combinational logic depth doesn't matter "if it still gives the right answer in simulation" — depth directly determines Fmax.
- Not knowing what slack is or how to read a basic STA report.

### 6.6–6.7 Interview Questions with Answers

**Q1 (Easy).** Define setup time and hold time.
*A:* Setup time (Tsu): minimum time the data input must be stable **before** the active clock edge. Hold time (Th): minimum time the data input must remain stable **after** the active clock edge. Violating either risks metastability/incorrect capture.

**Q2 (Easy).** What is "slack," and what does negative slack mean?
*A:* Slack = required time − actual arrival time for a timing path. Positive slack = path meets timing with margin to spare; negative slack = the path is too slow (setup) or too fast (hold) for the given constraint — a timing violation that must be fixed before signoff.

**Q3 (Medium).** Derive the maximum clock frequency formula for a simple register-to-register path.
*A:* `Tclk_min = Tcq (clock-to-Q of launching flop) + Tcomb (combinational path delay) + Tsu (setup time of capturing flop) + margin (skew/uncertainty)`. `Fmax = 1 / Tclk_min`.

**Q4 (Medium).** Why can't a hold violation be fixed by lowering the clock frequency?
*A:* Hold violations are about data arriving **too fast relative to the same edge** — they depend only on clock-to-Q delay, combinational delay, hold time, and clock skew, none of which involve the clock *period*. Slowing the clock changes the setup margin (more time available) but does nothing to the hold check, since hold is checked against the *same* edge, not the *next* one. Hold violations must be fixed by adding delay (buffers) in the data path or reducing/aligning clock skew.

**Q5 (Medium).** What is clock skew, and how does it affect setup vs hold checks differently?
*A:* Clock skew = difference in clock arrival time between the launching and capturing flip-flops (due to routing/buffer delay differences). Positive skew (capturing flop's clock arrives later than launching flop's) **helps** setup (more time for data to arrive) but **hurts** hold (data could still be present, but the sampling edge itself has moved later, actually — walk through the exact polarity in the interview using a diagram; the key point to state is that skew helps one check and hurts the other, so it can't be freely used to fix both).

**Q6 (Hard).** What's a multicycle path, and why would you constrain one?
*A:* A path that is functionally guaranteed (by design intent) to only need to settle over N clock cycles instead of 1 — e.g., a slow configuration register updated far less often than the clock rate. Constraining it as a multicycle path (via `set_multicycle_path`) relaxes the timing requirement on that path so the tool doesn't waste effort/area trying to force a single-cycle closure on logic that doesn't need it.

**Q7 (Hard).** What is a false path, and give an example.
*A:* A path the tool would otherwise analyze for timing but that can never actually be functionally exercised in a single clock cycle (e.g., two mutually-exclusive mode-select paths, or a path already properly synchronized via a CDC handshake elsewhere) — constrained with `set_false_path` so STA doesn't over-constrain the design trying to close a path that will never actually be latched in that window.

**Q8 (Hard).** Walk through how you would debug a design that fails setup timing by 200 ps on one path.
*A:* Identify the failing path in the STA report (launching flop → combinational logic → capturing flop); check combinational logic depth/complexity on that path (can it be simplified or pipelined?); check if it's on a non-critical function that could tolerate a multicycle constraint; check placement/routing congestion if on FPGA (physical delay contributing); as a last resort, consider re-pipelining (adding a register) to break the path into two shorter cycles.

### 6.8 Numerical Problems
1. Tcq = 0.5 ns, Tcomb = 3.2 ns, Tsu = 0.3 ns, clock uncertainty = 0.2 ns. Find Fmax. *(Answer: Tclk_min = 0.5+3.2+0.3+0.2 = 4.2 ns → Fmax ≈ 238 MHz.)*
2. A design targets 400 MHz (Tclk = 2.5 ns). Tcq = 0.4 ns, Tsu = 0.2 ns. What is the maximum allowed combinational delay budget? *(Answer: 2.5 − 0.4 − 0.2 = 1.9 ns, minus any margin for skew/uncertainty.)*
3. Hold check: Tcq_min = 0.3 ns, Tcomb_min = 0.1 ns, Th = 0.25 ns. Is there a hold violation (assume zero skew)? *(Answer: available = Tcq_min + Tcomb_min = 0.4 ns ≥ Th = 0.25 ns → hold is met, with 0.15 ns margin.)*

### 6.9 Timing Diagrams
```
Setup check (path must be FAST ENOUGH to arrive before next edge):
clk_launch  __|‾|__________________|‾|___
clk_capture __________|‾|__________________|‾|___
data        ────X== new data ==X──────────────
                 |<------ Tclk period ------>|
                 |<-Tcq->|<--Tcomb-->|<-Tsu->|
                                     must land here before edge

Hold check (path must not be TOO fast, else new data overtakes old edge's capture):
data changes almost immediately after launch edge:
clk_launch  __|‾|___
data        ────────X==new==X (arrives very fast, within Th window of same edge -> violation risk)
```

### 6.10 Verilog Implementation (illustrating pipelining to fix a timing violation)
```verilog
// BEFORE: one long combinational path (may violate setup at high frequency)
always @(posedge clk) result <= (a*b) + (c*d) + (e*f);

// AFTER: pipelined into two stages, shortening the critical path
always @(posedge clk) begin
  p1 <= a*b;
  p2 <= c*d;
  p3 <= e*f;
end
always @(posedge clk) result <= p1 + p2 + p3;
```

### 6.11 FPGA Implementation Considerations
- FPGA timing closure is heavily influenced by **placement and routing**, not just logic depth — the same RTL can meet or miss timing depending on floorplanning/congestion, which is why post-place-and-route STA (not just post-synthesis estimates) is the real signoff step.
- Use timing constraints (`.xdc` for Vivado) to correctly declare clock periods, false paths, and multicycle paths — under-constraining gives falsely optimistic timing closure; over-constraining wastes tool effort and area/power.

### 6.12 Timing-Related Issues (this topic's core content is already the deep dive — summary of the big 4)
1. **Setup violation** — path too slow → pipeline, reduce logic depth, or reduce frequency.
2. **Hold violation** — path too fast / bad skew → add delay buffers, cannot fix by changing frequency.
3. **Clock skew** — physical clock arrival mismatch → managed via clock tree synthesis / careful floorplanning.
4. **Clock domain crossing** — signal crossing unrelated clocks → needs synchronizers, not raw STA closure (STA typically treats these as false paths once properly synchronized).

### 6.13 Synthesis Considerations
- Synthesis-stage timing estimates are optimistic (wire delay is estimated, not final) — real signoff happens post place-and-route.
- Overly complex combinational RTL (deep nested ternary/case chains) can produce logic the synthesis tool can't optimize well — restructuring RTL (e.g., balancing a wide adder tree) can directly improve timing results even with functionally identical behavior.

### 6.14 Verification/Debugging Considerations
- STA (static) is the primary sign-off tool for timing — distinct from functional simulation (dynamic verification), and interviewers love asking you to articulate this distinction clearly.
- CDC verification tools flag missing/incorrect synchronizers statically, complementing STA (which typically excludes true asynchronous paths via false-path constraints).

### 6.15 Common Follow-up Questions
- "If your design meets timing in simulation but fails STA, what does that tell you, and what do you check first?"
- "Explain, without slowing the clock, two independent ways to fix a hold violation."
- "How would you decide whether a path should be constrained as multicycle vs simply re-pipelined?"

### 6.16 Connections to Computer Architecture / RTL
- Pipelining — the central performance technique in computer architecture — exists specifically *because* of timing: breaking a long combinational chain (e.g., a full ALU + memory access) into pipeline stages so each stage fits inside one clock period, directly raising Fmax and throughput.
- Clock frequency vs IPC (instructions per cycle) tradeoffs in CPU design are fundamentally a timing-closure conversation: deeper pipelines (shorter combinational stages) can hit higher clock speeds but add more hazard/stall complexity.

---
## 7. Final Prep Kit

### 7.1 One-Page Cheat Sheet

| Concept | Core Rule |
|---|---|
| Combinational | `assign` or `always @(*)` with **full** case/if coverage; blocking (`=`) only |
| Sequential | `always @(posedge clk)`; nonblocking (`<=`) only |
| Latch avoidance | Always add `default:` and pre-assign outputs at top of combinational blocks |
| FSM style | 3 always blocks: state reg (seq) / next-state (comb) / output (comb or reg) |
| Encoding | One-hot for FPGA speed; binary for area; Gray for CDC-safe counters |
| Reset | Prefer async-assert + sync-deassert; be consistent across the design |
| Counter | Always synchronous; never ripple; use carry chain via `+1` |
| Flip-flop | D-FF is the only RTL primitive you code; JK/T are academic only |
| CDC | Never cross domains raw; use 2-flop sync (1-bit) or handshake/FIFO (multi-bit/Gray for counters) |
| Setup violation | Path too slow → pipeline / reduce logic / lower Fmax |
| Hold violation | Path too fast / skew → add delay; **cannot** fix by lowering clock |
| Fmax formula | `Fmax = 1 / (Tcq + Tcomb + Tsu + margin)` |
| Mealy vs Moore | Mealy = fast but glitchy (comb output); Moore = clean but 1-cycle-lag (reg output) |
| STA vs Simulation | STA = static, checks all paths vs constraints; Simulation = dynamic, checks functional correctness only |
| Interview answer shape | Logic → RTL → Synthesis → Timing → Verification, every time |

### 7.2 50 Most Important Interview Questions
1. What defines combinational vs sequential logic?
2. Why does an incomplete `case` infer a latch?
3. Blocking vs nonblocking — when to use each and why?
4. Design a 2:1 / 4:1 MUX in Verilog.
5. Design a full adder; give sum/carry equations.
6. What is a static hazard and how do you remove it?
7. Design a priority encoder.
8. What is a race condition in RTL simulation?
9. Difference between a latch and a flip-flop.
10. Synchronous vs asynchronous reset — tradeoffs?
11. Why must a 2-flop swap use nonblocking assignments?
12. What is metastability and why can't simulation catch it?
13. Design a 2-flop CDC synchronizer for a single bit.
14. Why can't a 2-flop synchronizer be used directly on a multi-bit bus?
15. Define setup time and hold time.
16. Derive the Fmax formula for a register-to-register path.
17. Why can't a hold violation be fixed by lowering clock frequency?
18. What is clock skew, and how does it affect setup vs hold differently?
19. What is a multicycle path? Give an example.
20. What is a false path? Give an example.
21. Moore vs Mealy — core tradeoffs?
22. Explain the 3-always-block FSM coding style and why it's preferred.
23. Binary vs Gray vs one-hot state encoding — when to use each?
24. How do you prevent an FSM from getting stuck in an illegal state?
25. Design a Moore FSM for a traffic light controller.
26. Design a mod-10 counter.
27. Why avoid asynchronous ripple counters on FPGAs?
28. How do you generate a 1 MHz enable pulse from 100 MHz using a counter?
29. Design an up/down counter with load capability.
30. What is a Johnson counter, and one practical use?
31. SR, D, JK, T flip-flop — characteristic equations?
32. Why is D-FF the standard RTL building block, not JK/T?
33. What is the difference between STA and functional simulation?
34. Walk through debugging a design that fails setup timing by 200 ps.
35. How would you verify a CDC synchronizer beyond functional simulation?
36. How do FPGA LUTs implement combinational logic?
37. What is a carry chain, and why does it matter for adders/counters?
38. Why should you avoid manual clock gating on FPGAs?
39. What does the ASYNC_REG attribute do (Vivado), and why use it?
40. What is glitch/hazard, and does it affect STA timing closure?
41. How do you verify an FSM has full state and transition coverage?
42. Explain "safe FSM" / default-state recovery for SEU-hardening.
43. What is the difference between synthesis-time and post-P&R timing estimates?
44. How would you pipeline a long combinational adder chain to fix timing?
45. Why is blocking assignment correct for combinational logic but wrong for sequential logic?
46. What causes X-propagation in simulation, and how do you debug it?
47. Explain reset recovery/removal timing for asynchronous resets.
48. How do you decide whether one-hot or binary FSM encoding is better for a given design?
49. What's the relationship between pipelining (architecture) and timing closure (RTL)?
50. Walk through your general framework for approaching any new RTL design question in an interview.

### 7.3 20 Common Fresher Mistakes
1. Incomplete `case`/`if` → unintended latch inference.
2. Missing `default:` branch in combinational logic.
3. Mixing blocking and nonblocking assignments in the same always block.
4. Using blocking (`=`) in sequential (clocked) always blocks.
5. Multiple always blocks driving the same signal.
6. Using `always @(a or b)` instead of `always @(*)`.
7. Building asynchronous ripple counters instead of synchronous ones.
8. Manually gating the clock instead of using synchronous enable.
9. Crossing clock domains without a synchronizer.
10. Using a plain 2-flop synchronizer on a multi-bit bus.
11. Believing "simulation passed" proves timing closure.
12. Not knowing the difference between setup and hold violations.
13. Trying to fix a hold violation by lowering clock frequency.
14. Ignoring clock skew/uncertainty when estimating Fmax.
15. Not handling FSM's unused/illegal state encodings.
16. Confusing Mealy (comb, glitchy) output with Moore (reg, clean) output requirements.
17. Off-by-one errors in mod-N counter wraparound logic.
18. Forgetting to reset registers, causing persistent X in simulation.
19. Not stating the "Logic → RTL → Synthesis → Timing → Verification" framework explicitly in interview answers.
20. Over-focusing on academic flip-flop types (JK/T/SR) instead of practical D-FF-based RTL coding fluency.

### 7.4 3-Day Revision Plan

**Day 1 — Foundations (Combinational + Sequential + Flip-Flops)**
- Morning: Re-derive K-map/SOP/POS by hand for 2–3 practice functions; redo the MUX/adder/priority-encoder Verilog from scratch without looking.
- Afternoon: Sequential logic — write D-FF, enable-D-FF, and a 2-flop CDC synchronizer from memory; explain setup/hold + metastability out loud to yourself or a friend.
- Evening: HDLBits — combinational + sequential circuit problem sets (aim for 15–20 problems).

**Day 2 — FSMs + Counters**
- Morning: Design 2 FSMs from scratch (e.g., traffic light + simple protocol handshake) using the 3-always-block style; deliberately test yourself on Moore vs Mealy tradeoffs.
- Afternoon: Write synchronous up/down counter, mod-N counter, and an enable-pulse generator; work through the numerical problems in Section 4.8 again without looking at answers.
- Evening: HDLBits — FSM + counter/shift-register problem sets.

**Day 3 — Timing + Mock Interview**
- Morning: Redo the Fmax/setup/hold numerical problems (Sections 5.8 and 6.8) cold; make sure you can explain, unprompted, why hold violations can't be fixed by slowing the clock.
- Afternoon: Read through the 50 questions list (Section 7.2) and answer each in under 60 seconds out loud.
- Evening: Do the mock interview below end-to-end, timed, without notes; review weak spots against the topic sections above.

### 7.5 Mock Technical Interview (Mettlesemi / AMD / Qualcomm / Intel / NVIDIA / TI style)

**Round structure:** 45 minutes — Warm-up (5 min) → RTL design problem (20 min) → Timing/CDC deep dive (10 min) → Rapid-fire concepts (10 min)

---
**[Warm-up]**
> "Walk me through, at a high level, how you'd go from a spec to working RTL to a timing-closed FPGA bitstream."
*(Expected answer shape: Logic → RTL coding → simulation/functional verification → synthesis → place & route → STA/timing closure → bitstream.)*

**[RTL Design Problem]**
> "Design a synchronous FSM-based traffic light controller: RED (10 cycles) → GREEN (8 cycles) → YELLOW (2 cycles) → back to RED. Use Verilog. Walk me through your coding style choices."
- Expect the candidate to: pick a 3-always-block structure, choose Moore encoding (clean light output), add an internal counter for cycle-timing, include a `default` state, and explain reset behavior.
- Common interviewer follow-ups: *"What if I wanted a pedestrian-request input that can interrupt the cycle — how does your FSM change?"* / *"What encoding would you pick for FPGA vs ASIC, and why?"*

**[Timing / CDC Deep Dive]**
> "Suppose your traffic controller's `pedestrian_request` button input comes from a different, unrelated clock domain. How do you bring it in safely, and what could go wrong if you didn't?"
- Expect: 2-flop synchronizer explanation, metastability risk if skipped, and awareness that a button press is naturally a single asynchronous bit → 2-flop sync is sufficient (unlike multi-bit data, which needs handshake/FIFO).

> "If your synthesized design shows a setup violation of 300 ps on the counter's compare logic feeding the next-state decode, what are your options?"
- Expect: check combinational path depth, consider pipelining the compare, check if it can be a multicycle path (it's usually not, since it's checked every cycle), and mention that lowering clock frequency is a valid last resort but not a "design fix."

**[Rapid-Fire Concepts]**
- "Setup vs hold — one sentence each." 
- "Why D-FF and not JK-FF in RTL?"
- "Moore or Mealy for a glitch-sensitive output signal — why?"
- "One-hot or binary for a 20-state FPGA FSM — why?"
- "Can you fix a hold violation by lowering the clock speed? Why or why not?"

**Self-grading rubric:**
- ✅ Strong: Answers using the Logic→RTL→Synthesis→Timing→Verification framework unprompted, includes reset/latch-avoidance discipline by default, and proactively flags CDC risk without being asked.
- ⚠️ Needs work: Gets functional RTL correct but forgets reset/default handling, or can't clearly separate setup vs hold reasoning.
- ❌ Weak: Cannot explain why hold violations aren't fixed by frequency, or writes RTL with blocking assignments in sequential always blocks.

---
*End of notes.*
