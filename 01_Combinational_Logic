# 01 - Combinational Logic
# Part 1 - Fundamentals, Boolean Algebra & Logic Gates

> **Primary Reference:** Morris Mano, *Digital Design with an Introduction to Verilog HDL (5th Edition)*, Chapters 2–4, which introduce Boolean algebra, logic gates, minimization, and combinational circuit design. :contentReference[oaicite:0]{index=0}

---

# 1. What is Combinational Logic?

## Definition

A **combinational circuit** is a digital circuit whose outputs depend **only on the current inputs**.

There is **no memory**.

There are **no previous states**.

Mathematically,

Output = f(Current Inputs)

---

## Example

Suppose

A = 1

B = 0

Output = A AND B

Output = 0

Now change

A = 1

B = 1

Output immediately becomes

1

Nothing is stored.

No clock is required.

---

## Block Diagram

```

A ──┐
├── Logic Circuit ─── Y
B ──┘

```

---

## Characteristics

| Feature | Combinational Logic |
|----------|---------------------|
| Memory | ❌ No |
| Clock | ❌ Not Required |
| Previous State | ❌ Ignored |
| Output Depends On | Present Inputs Only |
| Speed | Very Fast |
| Examples | MUX, Decoder, Adder |

---

# Interview Definition

**Question**

What is a combinational circuit?

**Answer**

A combinational circuit is a digital circuit whose outputs depend only on the present combination of input variables and not on previous inputs or stored information.

---

# Real FPGA Examples

Combinational logic is used everywhere.

Examples include

• ALU

• Address Decoder

• Instruction Decoder

• Multiplexer

• Comparator

• Arithmetic Unit

• Branch Logic

• Priority Encoder

• Data Selector

---

# Why FPGA Designers Care

Inside an FPGA,

every combinational circuit is eventually implemented using

• LUTs

• Routing wires

There are no actual AND or OR gates inside modern FPGAs.

Everything is converted into LUT truth tables.

This is one of the most common interview questions.

---

# Example

Suppose

Y = A·B + C

The synthesis tool

Verilog

↓

Boolean Equation

↓

Truth Table

↓

LUT Programming

↓

Hardware

You never place AND gates manually.

Vivado or Quartus does that automatically.

---

# 2. Boolean Algebra

---

## What is Boolean Algebra?

Boolean Algebra is the mathematics of digital logic.

Normal Mathematics

Variables

x = 5

Operations

+

−

×

÷

Infinite values

----------------------------

Boolean Mathematics

Variables

A

B

C

Only two values

0

1

Operations

AND

OR

NOT

---

# Why Boolean Algebra?

Without Boolean Algebra

No Digital Circuits

No CPUs

No FPGA

No ASIC

No Computer

No Mobile Processor

It is literally the language of digital hardware.

---

# Boolean Variables

A Boolean variable can only take

A = 0

or

A = 1

Nothing else.

---

# Physical Meaning

Boolean Value      Voltage

0                  0V

1                  1.8V

or

3.3V

or

5V

depending upon technology.

Interview Tip:

A logic '1' is **not always 5V**. It depends on the technology (TTL, CMOS, FPGA I/O standards, etc.).

---

# Logic Levels

Example

3.3V FPGA

0V -------- Logic 0

3.3V ------ Logic 1

Anything in between

depends upon

noise margin

technology

input threshold

---

# Boolean Operators

Three basic operators exist.

AND

OR

NOT

Every digital circuit is built from these.

---

# 3. AND Operation

Symbol

```

A ----\
AND ---- Y
B ----/

```

Equation

Y = A·B

Truth Table

| A | B | Y |
|---|---|---|
|0|0|0|
|0|1|0|
|1|0|0|
|1|1|1|

Meaning

Output becomes HIGH

ONLY IF

every input is HIGH.

---

## Real Hardware Example

Door Lock

Two switches

Door opens only if

Switch A ON

AND

Switch B ON

---

## Verilog

```verilog
assign y = a & b;
```

---

## FPGA Implementation

The synthesis tool creates a LUT entry implementing the AND truth table.

---

# Interview Question

Why don't FPGA designers instantiate AND gates?

Answer

Because synthesis automatically converts Boolean equations into LUT contents.

---

# 4. OR Operation

Equation

Y = A + B

Truth Table

|A|B|Y|
|---|---|---|
|0|0|0|
|0|1|1|
|1|0|1|
|1|1|1|

Meaning

If any input is HIGH,

output becomes HIGH.

---

## Verilog

```verilog
assign y = a | b;
```

---

## Example

Fire Alarm

Smoke Sensor

OR

Temperature Sensor

Either one

Alarm ON

---

# 5. NOT Operation

Equation

Y = A'

or

Y = ~A

Truth Table

|A|Y|
|---|---|
|0|1|
|1|0|

---

## Verilog

```verilog
assign y = ~a;
```

---

# Example

Automatic Street Light

Day

Input = 1

Light OFF

Night

Input = 0

Light ON

Uses inverter.

---

# 6. Universal Gates

Very Important Interview Topic

Question

Which gates are called Universal Gates?

Answer

NAND

NOR

---

Why?

Because

Every Boolean function

can be implemented

using ONLY NAND

or

ONLY NOR gates.

---

Example

NOT using NAND

```

A ----|
NAND |---- Y
A ----|

```

Y = A'

---

# Why Industry Uses NAND

Reasons

Smaller Area

Higher Speed

Lower Cost

Better CMOS Implementation

Lower Transistor Count

Interviewers love this question.

---

# 7. Boolean Laws

These are asked almost every FPGA interview.

---

## Identity Law

A + 0 = A

A · 1 = A

---

## Null Law

A + 1 = 1

A · 0 = 0

---

## Idempotent Law

A + A = A

A · A = A

---

## Complement Law

A + A' = 1

A · A' = 0

---

## Involution Law

(A')' = A

---

## Commutative Law

A+B=B+A

AB=BA

---

## Associative Law

(A+B)+C=A+(B+C)

(AB)C=A(BC)

---

## Distributive Law

A(B+C)=AB+AC

A+BC=(A+B)(A+C)

Very Important

---

# 8. DeMorgan's Theorem ⭐⭐⭐⭐⭐

One of the most frequently asked interview questions.

Theorems

(A·B)' = A' + B'

(A+B)' = A'B'

---

## Memory Trick

Break

Change Operator

Complement Inputs

Example

(A+B+C)'

↓

A'B'C'

---

# Why It Matters

Without DeMorgan

You cannot simplify hardware.

You cannot optimize FPGA resources.

You cannot understand NAND/NOR implementations.

---

# Verilog Example

```verilog
assign y = ~(a & b);
```

Equivalent

```verilog
assign y = (~a) | (~b);
```

---

# FPGA Optimization

Vivado automatically applies

Boolean simplification

Constant propagation

Common subexpression elimination

DeMorgan transformations

Logic sharing

Resource optimization

You usually write readable RTL; the synthesis tool optimizes it.

---

# Synthesis Considerations

✔ Avoid redundant Boolean expressions.

✔ Write clear combinational logic.

✔ Let synthesis optimize.

✔ Do not manually over-optimize RTL unless necessary.

---

# Verification Checklist

Before simulation

□ Check truth table

□ Check all input combinations

□ Verify active-high/active-low assumptions

□ Check for X and Z propagation in simulation

---

# Common Interview Questions

### Easy

What is combinational logic?

Difference between combinational and sequential circuits?

Why is memory absent?

---

### Medium

Explain DeMorgan's theorem.

Why are NAND and NOR universal gates?

How is an AND gate implemented inside an FPGA?

---

### Advanced

Why doesn't synthesis preserve your Boolean equation exactly?

How do LUTs replace gates?

Can two logically equivalent expressions produce different timing after synthesis? Why?

What is logic optimization?

---

# Key Takeaways

- Combinational circuits depend only on present inputs.
- Boolean algebra is the mathematical foundation of digital logic.
- AND, OR, and NOT are the three fundamental operations.
- NAND and NOR are universal gates capable of implementing any Boolean function.
- DeMorgan's theorem is essential for logic simplification and FPGA optimization.
- Modern FPGAs implement combinational logic using LUTs rather than discrete gates.
- Clean Verilog allows synthesis tools to generate efficient hardware automatically.

---
