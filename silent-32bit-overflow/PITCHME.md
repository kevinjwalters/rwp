## Finite precision maths and overflow
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
  * Raspberry Pi have been 64 bit since the model `2B ver 1.2`.
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
## C variables size

Type          | Size (bits) | From | To |
---           | ---         | ---  | ---
unsigned char | 8  | 0 | 255
short         | 16 | -32768 | 32768  
int           | 32 | -2147483648 | 2147483647
unsigned int  | 32 | 0 | 4294967295 |
long          | 64 | 0 | 18446744073709551615

[C99](https://en.wikipedia.org/wiki/C99) standard introduced some new variables to give a known size via C `typedef` feature,
e.g `int32_t` and `uint64_t`.

---
## Large integers

* A signed 32 bit integer can stored a value just over 2 billion.
* Are "large" integers only used in scientific applications?
* Would a simple program ever encounter this?
* Time is one area:
  * CPU clock speed increases and requirements for accuracy have driven requirements.
  * NTP, GPS/GLONASS and better hardware clocks and o/s support aid accuracy.
  * Time values often have been *changed* from seconds -> ms -> us -> ns.
  * Library changesr porting to new platform may require use of different resolution timers.
  * Changes must be very careful to ensure all maths is at appropriate precision.
* File length:
  * Storage has grown over last two decades
  * Multi-TB disks common even at home.
  * O/s now supports > 2TB files so `int` *cannot* be used for file size.

Note:
* Accuracy vs precision/resolution.
* GPS/GLONASS extremely accurate but very weak signal and not authenticated for civilian use.
* Are GPS receiver/clock perfect? Firmware bug free?
* NTP essential to allow time comparison between hosts/devices.
* Commercial software solutions offer superior time sync to ntp.
+++
## Integer overflow examples
### Types used for all examples

```c
typedef int Duration;
typedef uint32_t OtherDuration;
typedef uint64_t ReplacementDuration;
#define ReplacementDurationFormat "%lu"
```

@[1](Common code from early days of C.)
@[2](Using the C99 to get a known size `unsigned` type.)
@[3](64 bit version of OtherDuration - often OtherDuration would simply be enlarged to 64 bit.)
@[4](Helpful to have the `printf` type for the data type to ensure they match.)

---
## Integer overflow examples
### Example demo1() code

```c
int i = 0;
int timer_us = 0;

for (i=0; i < 5; i++) {
    timer_us += 500*1000;  /* add half a second */
    printf("timer_us=%d\n", timer_us);
} 
```

@[1-2](Two (signed) `int`s initialised.)
@[4-7](Five iterations adding half a second in microseconds. `printf()` uses C's primitive approach to handling a variable number of arguments.)

Note:
 * 500*1000 is one way to make this easier to read and less likely to get wrong number of zeros.
 * Compiler will optimise any maths which can be calculated at compile time.

---
## Integer overflow examples
### Example demo1() output (gcc 4.8.5 Centos 7)

```
timer_us=500000
timer_us=1000000
timer_us=1500000
timer_us=2000000
timer_us=2500000
```

All is well, final value is two and half million microseconds.

+++
## Integer overflow examples
### Example demo2() code

```c
    int i = 0;
    Duration timer_ns = 0;

    for (i=0; i < 5; i++) {
        timer_ns += 500000000;  /* add half a second */
        printf("timer_us=%d timer_us=%ld\n", timer_ns, timer_ns);
    } 
```

@[1-2](`Duration` is now used to hold a nano-second based value - can we remember what size variable is?)
@[4-7](A difficult to read and therefore error-prone large value is added five times. Comment expresses programmer's intention.)

---
## Integer overflow examples
### Example demo2() output (gcc 4.8.5 Centos 7)

```
timer_us=500000000 timer_us=500000000
timer_us=1000000000 timer_us=1000000000
timer_us=1500000000 timer_us=1500000000
timer_us=2000000000 timer_us=2000000000
timer_us=-1794967296 timer_us=2500000000
```

@[1-4](Looks good.)
@[5](A problem: `%d` is correct but prints it as a negative value due to overflow of two's signed complement value, `%ld` is *dangerously* wrong on LP64 - it prints correctly due to *luck* of how compiled code passes and reads the 64 bit value.)

---
## Integer overflow examples
### Example demo2() output (cl 19.14.26431 Windows 10)

```
timer_us=500000000 timer_us=500000000
timer_us=1000000000 timer_us=1000000000
timer_us=1500000000 timer_us=1500000000
timer_us=2000000000 timer_us=2000000000
timer_us=-1794967296 timer_us=-1794967296
```

@[1-4](Looks good.)
@[5](A problem: overflowed value is printed twice but it's the same code! Microsoft uses LLP64 so `long` is 32 bits. Visual Studio does parse `printf()` string and warn about mismatches with args.)
+++
## Integers in other Languages 

* Perl - single number type which on overflow will promote to unsigned integer or double - integer size *varies* between 32 bit and 64 bit interpreter.
* Python 2 - `int` which auto promotes to infinite precision `long` since v2.2 ([PEP 237](https://www.python.org/dev/peps/pep-0237/))
* Python 3 - infinite precision (cf perl's [Math::BigInt](https://perldoc.perl.org/Math/BigInt.html))
* MicroPython - uses 31 bit `smallint` as 1 bit is re-purposed.

Note:
* Terminology on each line refers to types in each language which may differ to C.
* MicroPython is a smaller version of Python 3 for [BBC micro:bit](http://microbit.org/)
* CircuitPython is Adafruit's fork of MicroPython for their microcontroller boards.

+++
## Calculation durations

```c
t1 = timenow();
dosomething();
t2 = timenow();
duration = t2 - t1;
```

* `timenow()` returns current time - resolution and accuracy can be varied for discussion purposes.
* Will duration always be non-zero?
* Will duration always be positive?
* This leads to another topic.

Note:

+++
## Other examples

* Epoch times:
  * Unix `gettimeofday()` used `int`, now `time_t` but can still be 32 or 64 bit - signed gives range from 1970 to ~2038 - [Y2038 problem](https://en.wikipedia.org/wiki/Year_2038_problem)
* 100Hz timer - signed int max is ~248 days - [B787 bug](https://www.engadget.com/2015/05/01/boeing-787-dreamliner-software-bug/)
* Ada throws exceptions for conversions - [Ariane 5 failure](http://sunnyday.mit.edu/accidents/Ariane5accidentreport.html) with a value exceeding 16 bit range.

+++
## Conclusion

* Surprising variety in basic type across languages.
* More care required in C/C++ than some other lanugages for integer maths.
* Understanding variable size is important particularly if values are likely to exceed.
* Enlarging variables on existing code - care with:
  * maths,
  * constant values which may need `L` or `LL` suffixing.
* Microsoft LLP64 vs rest of world LP64 - care with `long` for portable code.
* Cygwin on Windows uses LP64.

+++
## Further reading

* [LP64 and LLP64](https://en.wikipedia.org/wiki/64-bit_computing#64-bit_data_models)
* [leap smear](https://developers.google.com/time/smear) and [leap seconds](https://en.wikipedia.org/wiki/Leap_second)

