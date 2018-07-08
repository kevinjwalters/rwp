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

---
## Large integers
### Time

* Time is one area:
  * CPU clock speed increases and requirements for accuracy have driven requirements.
  * NTP, GPS/GLONASS and better hardware clocks and o/s support aid accuracy.
  * Time values often have been *changed* from seconds -> ms -> us -> ns.
  * Library changesr porting to new platform may require use of different resolution timers.
  * Changes must be very careful to ensure all maths is at appropriate precision.

Note:
* Accuracy vs precision/resolution.
* GPS/GLONASS extremely accurate but very weak signal and not authenticated for civilian use.
* Are GPS receiver/clock perfect? Firmware bug free?
* NTP essential to allow time comparison between hosts/devices.
* Commercial software solutions offer superior time sync to ntp.

---
## Large integers
### File size

* Disk space and partition sizes.
  * Storage has grown over last two decades
  * Multi-TB disks common even at home.
* File length:
  * Files may typically be smaller than a disk but o/s should allow large ones.
  * O/s now supports > 2GB files so `int` *cannot* be used for file size.
* Unix has various leftovers to be wary of:
  * `fseek()` on 32 bit binaries 
  * `open()` behaviour needs modifying with `_FILE_OFFSET_BITS`

Note:

+++
## Integer overflow examples
### Types used for all examples

All code is compiled to produce 64 bit binaries.

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
 * Compiler's optimiser can get calculate any constant values at compile time.

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

@[1-2](`Duration` is now used to hold a nano-second based value - can we remember what size that type is?)
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
## Integer overflow examples
### Example demo3() code

```c
    int i = 0;
    Duration timer_ns = 0;
    int six_us = 6e6;
    int six_ns = six_us * 1000; /* no compiler warning - static tools may detect this */

    timer_ns += six_ns;
    printf("timer_ns=%ld\n", timer_ns, timer_ns);
```

@[1-2](Looks reasonable.)
@[3](Still reasonable. `e` notation makes numbers easier to check.)
@[4](An overflow from multiply *not* detected by code, no warnings from compiler. Static code analysers should find this.)
@[6](Bad value is now added to `Duration` which is also too small to hold intended value.)
@[7](Another bug here, too many values passed to `printf()`, wrong but harmless.)

---
## Integer overflow examples
### Example demo3() output (gcc 4.8.5 Centos 7)

```
timer_ns=1705032704
```

* Value is clearly not 6 billion (nanoseconds)!
* Value is 6*1000*1000*1000-(2^32)
* Unit tests might pick this up.

Note:
* C has no operator for power, other languages use `^` or `**`
* Same output on Windows.

+++
## Integer overflow examples
### Example demo4() code 

```c
    int i = 0;
    ReplacementDuration timer_ns = 0;
    int six_us = 6e6;
    int64_t six_ns = six_us * 1000;

    timer_ns += six_ns;
    printf("timer_ns=%ld\n", timer_ns, timer_ns);
```

@[1-3](Looks reasonable.)
@[4](Undetected overflow and no compiler warning. Static code analysers should find this.)
@[6](`ReplacementDuration` sufficiently large to hold intended value but bad value is added.)
@[7](Another bug here, too many values passed to `printf()`, wrong but harmless.)

Note:
* Mixing 32 bit and 64 bit arithmetic here without care has broken this.

---
## Integer overflow examples
### Example demo4() output (gcc 4.8.5 Centos 7)

```
timer_ns=1705032704
```

* Value is clearly not 6 billion (nanoseconds)!
* Value is 6*1000*1000*1000-(2^32)

Note:
* Same output on Windows.

+++
## Integer overflow examples
### Example demo5() code

```c
    int i = 0;
    ReplacementDuration timer_ns = 0;
    int six_us = 6e6;
    int64_t six_ns = six_us * 1000L;

    timer_ns += six_ns;
    printf("timer_ns=%ld\n", timer_ns, timer_ns);
```

@[1-3](Reasonable.)
@[4](C is harsh here - arithmetic is performed based on operand sizes (`int` and `long`) rather than assignment size.)
@[6](Ok if the value in six_ns is ok.)
@[7](Cut and paste buglet from previous examples.)

Note:

---
## Integer overflow examples
### Example demo5() output (gcc 4.8.5 Centos 7)

```
timer_ns=6000000000
```

* Correct on Linux (LP64) but not portable and would not work if compiled to produce a 32 bit binary.
* `1000LL` rather than `1000L` would be one fix here, `(int64_t)six_us` is another.

---
## Integer overflow examples
### Example demo5() output (cl 19.14.26431 Windows 10)

```
timer_ns=1705032704
```

* Output wrong on Windows (LLP64).
* `1000LL` rather than `1000L` would be one fix here, `(int64_t)six_us` is another.

+++
## Integers in other languages 

* Perl - single number type which on overflow will promote to unsigned integer or double - integer size *varies* between 32 bit and 64 bit interpreter.
* Python 2 - `int` which auto promotes to infinite precision `long` since v2.2 ([PEP 237](https://www.python.org/dev/peps/pep-0237/))
* Python 3 - infinite precision (cf perl's [Math::BigInt](https://perldoc.perl.org/Math/BigInt.html))
* MicroPython - uses 31 bit `smallint` as 1 bit is re-purposed.
* Java - size of `int` is always 32 bit, size of `long` is always 64, both signed only and both overflow (cf [Integer](https://docs.oracle.com/javase/8/docs/api/?java/lang/Integer.html) object and [autoboxing](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html))
* bc - a basic calculator on unix - infinite precision 

Note:
* Terminology on each line refers to types in each language which may differ to C.
* Infinite precision also known as arbitrary precision. Both clearly limited by available memory!
* MicroPython is a smaller version of Python 3 for [BBC micro:bit](http://microbit.org/)
* `smallint` is an efficiency measure for small microcontroller boards.
* CircuitPython is Adafruit's fork of MicroPython for their boards.

+++
## Calculating durations

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
  * math operations,
  * constant values which may need `L` or `LL` suffixing.
  * likely to enlarge data structures
  * more complicated for library code particularly with a diverse userbase
  * likely to break ABI, e.g. a library change may force a recompile of (ALL!) code using it

---
## Further thoughts

* Think about atypical but possible values for unit tests.
* Think hard about interfaces in advance to minimise breakage and new versions.
* Error conditions may give unexpected numbers, more so if documentation isn't read!
* Exceptions may cause lack of assignment to a variable leaving a previous or random value.
* Microsoft LLP64 vs rest of world LP64 - care with `long` for portable code.
* Cygwin on Windows uses LP64.

---
## Further reading

* Code examples: [silent-32bit-overflow.c](https://github.com/kevinjwalters/rwp-examples/blob/master/examples/silent-32bit-overflow/silent-32bit-overflow.c)
* [LP64 and LLP64](https://en.wikipedia.org/wiki/64-bit_computing#64-bit_data_models)
* [Communications of the ACM: Lessons from Building Static Analysis Tools at Google](https://cacm.acm.org/magazines/2018/4/226371-lessons-from-building-static-analysis-tools-at-google/fulltext)
