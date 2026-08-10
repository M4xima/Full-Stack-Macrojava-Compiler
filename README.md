# Full-Stack-Macrojava-Compiler
This repository has the all the parsers I wrote to create a Full Stack Macrojava compiler, in total which converts a Macrojava code to MIPS assembly.
##MacroJava
Macrojava is a subset of Java extended with C style macros. Overloading is not allowed in MacroJava. The MacroJava statement System.out.println( ... ); can only print integers. The MacroJava expression e1 & e2 is of type boolean, and both e1 and e2 must be of type boolean. In addition, Macrojava supports two boxed types: Integer and Boolean. MacroJava supports both inline as well as C style comments, but does not support nested comments. MacroJava supports only a specific type of lambdas: from Function interaces.
##Mips Assembly(https://www.cse.iitm.ac.in/~krishna/cs3300/spim_ref.html#instructions)
Mips Assembly is made up assembly which almost looks like RISCV but with some changes to support less operations and with limited features
There are series of steps that are done for this compiler to be success which takes it from a high level Macro java to low level MIPS Assembly.
There are 6 parsers created in this project.This following link give you specifications of all the Java Subsets used - https://www.cse.iitm.ac.in/~krishna/cs3300/subsets.html
#Here is what each file indicates in this project 
parser P1 - converts Macrojava to MiniJava
parser P2 - Type checker for MiniJava (also code optimizations are done)
parser P3 - converts MiniJava to MiniIR
parser P4 - converts MiniIR to MicroIR
parser P5 - converts MicroIR to miniRA
parser P6 - converts miniRA to MIPS Assembly

For P1 I have used flex(lexer) and Bison(parser) (in C++), whereas for the remaining parsers I have used JTB(Java Tool Builder) which is tree building tool used with JavaCC parser buildeing tool.
For more details check the code and visit this(https://www.cse.iitm.ac.in/~krishna/cs3300/)
