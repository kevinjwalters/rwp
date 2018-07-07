## Finite precision maths and overflow
##### some text in the page using header
![Logo test](/images/test-logo1.jpg)

![Create Commons License BY-NC-ND/4.0](https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png)

This work is licensed under a [Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License](http://creativecommons.org/licenses/by-nc-nd/4.0/)

Note:
* --- between groups, +++ between pages in a group

---
## Arithmetic operations in a computer

* CPUs have an Arithmetic Logic Unit (ALU) for integer maths
* Operations are on a fixed word size:
  * Most processors in large devices are 64 bit now.
  * Raspberry Pi have been 64bit since the model `2B ver 1.2`.
  * Small/cheap/low-power devices may use 32 bit or even lower.
  * IBM PC (1981) used Intel 8088 with 16 bit word size.
* ALUs set carry/overflow flag for operations.
* Computer languages vary widely on how they represent numbers and when they use a fixed size representation what happens.

Note: 
* notes for speaker

---
## C language

* C sometimes described as a medium level language.
* Used to write early versions of Unix, intended for system programming.
* Very close to the hardware, compiled, close mapping to assembler.
* Many risky features (safety traded for performance):
  * Low level access to memory with pointers.
  * No bounds checking for arrays (cf buffer overflows)
  * Program is responsible for managing dynamic memory use with malloc library (often leads to leaks and vulnerabilities).
  * Some variables are not initialised.
  * No checks for integer over flows and divided by zero
* Widely used for application programming!

Note:
* Memory access constrained by operating system which will be aided by memory management features on CPU.

---
## C variables

The built-in numerical types in the C language are fixed size.
The sizes are compiler/platform dependent with the widely used 
`int` type intending to be the word size.

The large amount of code written in the 32 bit era made it difficult
and unwise to enlarge `int` to 64 bit for modern hardware so it remained
at 32 bit.

Memory pointers had to enlarge to allow access to all the memory -
a common porting task was to fix code which incorrectly assumed
`sizeof(int) == sizeof(void *)`.

The `long` type unfortunately varies across platforms.
Windows is LLP64, Unix is LP64. Cray Unicos was SILP64!

Note:
* Cray 1 64 bit 1975
* Silicon Graphics started using 64 bit MIPS circa 1991
* DEC Alpha was 64 bit circa 1991

+++
## C variables

Type          | Size (bits) | From | To |
---           | ---         | ---  | ---
unsigned char | 8  | 0 | 255
short         | 16 | -32768 | 32768  
int           | 32 | -2147483648 | 2147483647
unsigned int  | 32 | 0 | 4294967295 |
long          | 64 | 0 | 18446744073709551615

[C99](https://en.wikipedia.org/wiki/C99) standard introduced some new variables to give a known size via C `typedef` feature,
e.g `int32_t` and `uint64_t`.

