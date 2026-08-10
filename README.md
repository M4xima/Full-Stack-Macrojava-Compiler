# Full-Stack MacroJava Compiler

This repository contains a full-stack compiler that translates **MacroJava** (a subset of Java extended with C-style macros) all the way down to **MIPS Assembly**. 

The project was built through a series of six distinct parsers, gradually lowering the high-level code through various intermediate representations (IR) until it reaches machine-level instructions ready to be executed on a SPIM emulator.

---

## 🏗️ Compiler Architecture

The compilation process is divided into 6 distinct stages (parsers). Each stage lowers the abstraction level of the code.

| Stage | Parser | Input | Output | Tools Used | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **P1** | Lexer & Parser | MacroJava | MiniJava | Flex & Bison (C++) | Resolves C-style macros and translates the code into standard MiniJava. |
| **P2** | Type Checker | MiniJava | MiniJava | JavaCC & JTB | Validates semantics, performs type checking, and executes basic code optimizations. |
| **P3** | IR Generator | MiniJava | MiniIR | JavaCC & JTB | Converts high-level OOP concepts into a simplified Intermediate Representation (MiniIR). |
| **P4** | IR Simplifier | MiniIR | MicroIR | JavaCC & JTB | Further simplifies expressions and restricts statement nesting. |
| **P5** | Register Allocator| MicroIR | MiniRA | JavaCC & JTB | Maps unlimited temporary variables to 24 physical MIPS registers and handles stack spilling. |
| **P6** | Code Generator | MiniRA | MIPS | JavaCC & JTB | Generates the final MIPS assembly code compatible with the SPIM emulator. |

---

## 🛠️ Tools & Technologies

*   **Flex (Lexical Analyzer) & Bison (Parser Generator):** Used in C++ to build the first macro-resolution stage.
*   **JavaCC (Java Compiler Compiler):** A parser generator used for stages P2 through P6.
*   **JTB (Java Tree Builder):** A syntax tree builder used alongside JavaCC to automatically generate visitors for traversing the abstract syntax tree.
*   **SPIM Emulator:** Used to run and test the final generated MIPS assembly code.

---

## 📜 Language Specifications

The compiler translates code through several strict subsets of Java and Intermediate Representations.

### 1. MacroJava & MiniJava
*   **Macros (MacroJava only):** Supports C-style macros. After macro expansion, the code becomes MiniJava.
*   **Restrictions:** Method overloading is *not* allowed. Nested comments are *not* supported.
*   **Built-ins:** `System.out.println(...)` can only print integers.
*   **Booleans:** The expression `e1 & e2` is of type boolean, requiring both operands to be booleans.
*   **Types:** Supports two boxed types: `Integer` and `Boolean`.
*   **Lambdas:** Only supports specific lambdas derived from Function interfaces.

### 2. MiniIR (MiniJava Intermediate Representation)
MiniIR exposes the underlying runtime model of the program:
*   **Memory:** Integers are 4 bytes. The `new` operator is replaced with the `HALLOCATE` system call.
*   **Object Model:** Field and method references now require explicit address lookups.
*   **Procedures:** Represented as `ProcLabel [args] Stmt`. Variables are referenced as unbounded temporaries (e.g., `TEMP 0`, `TEMP 1`).
*   **Heap Operations:** Uses `HSTORE` (store to address with offset) and `HLOAD` (load from address with offset).
*   **Control Flow:** Uses `CJUMP` (conditional jump) where conditions must evaluate to `1` (true) or `0` (false).

### 3. MicroIR
A stricter subset of MiniIR:
*   `StmtExp` is no longer a valid expression.
*   Complex expressions are flattened. `Exp` is replaced with `SimpleExp` or `Temp` in most contexts, forcing operations to use temporary variables.

### 4. MiniRA (Register Allocation)
MiniRA transitions from infinite temporaries to actual hardware constraints, targeting a MIPS-like architecture:
*   **Registers:** Replaces unbounded temps with 24 global MIPS registers (`s0-s7`, `t0-t9`, `a0-a3`, `v0-v1`).
*   **Stack Management:** Uses `ALOAD` and `ASTORE` for stack manipulation. Spilled variables are referenced via `SPILLEDARG i` (0-indexed).
*   **Procedure Calls:** Method calls are strictly statements. Arguments > 4 are saved to the stack using `PASSARG i` (1-indexed).
*   **Procedure Headers:** Follows the format `ProcLabel [A] [B] [C]`:
    *   `[A]`: Number of arguments the procedure takes.
    *   `[B]`: Total stack slots required (args, spilled temps, saved registers).
    *   `[C]`: Maximum number of arguments used in any function call *within* this procedure.

### 5. MIPS Assembly
The final output is executable MIPS Assembly designed for the SPIM emulator.
*   **Register Conventions:**
    *   `$a0 - $a3`: Arguments
    *   `$v0`: Return values and syscall codes
    *   `$v1`: Temporary register for transferring data to/from the stack
    *   `$t0 - $t9`: Caller-saved temporaries
    *   `$s0 - $s7`: Callee-saved temporaries
    *   `$sp`: Stack pointer / `$ra`: Return address
*   **System Calls:** Uses SPIM syscalls for functionality (e.g., `1` for print int, `9` for memory allocation/HALLOCATE, `11` for print char/newline).

---

## 📚 References & Resources

This project was developed based on specifications from **CS3310 - Language Translators Lab** (IIT Madras).

*   **Course Details & Subsets Specification:** [CS3300 Page](https://www.cse.iitm.ac.in/~krishna/cs3300/subsets.html)
*   **MIPS Instruction Reference:** [SPIM Ref](https://www.cse.iitm.ac.in/~krishna/cs3300/spim_ref.html#instructions)
