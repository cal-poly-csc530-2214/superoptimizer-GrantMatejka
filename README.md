Was confused on setup for this assignment so initial repo can be found here https://github.com/GrantMatejka/Aha

## Time

Spent reading first paper and glancing over second, then getting aha to compile and run.
Than proceeded to try some simple preprocessor optimization while messing around with program

### Forked for Exploration

Looking into program synthesis and forked this repo to make a few 
code changes and just play around.

Made some simple optimizations, but major wins include just getting the 
code running and analyzing the structure of the optimizer.

## To Use

Should be able to just run `make` and `./aha n` where n 1-4

To look into different functions to analyze just replace userfun with 
function of interest and NARGS with respective number of arguments

userfun needs to be a function of 1 or 2 ints that returns a 32 bit int

## Brief Usage Description

Create a header file that decribes your problem and your machine. There
are two examples provided: abs.h for a function of one variable, and
avg.h for a function of two variables. The first solves the problem of
how to compute the absolute value function on a machine that does not
have that instruction. The second finds the Dietz formula for computing
the "floor average" of two unsigned integers without causing overflow.

Modify either abs.h or avg.h to fit your problem and save the file under
a name of your choice (with file extension .h). Let us assume the file
is named "mine.h".

Make the executable file by entering, on Windows, "make mine". This
creates file "mine.exe".

Execute it by entering "mine n", where n is the number of instructions
for which you want to find a solution, generally 1, 2, 3, or 4.

The solutions found will be displayed and also placed in file
"mine.out".

See aha.pdf for a complete writeup.


## History of Improvements (Carried over from orig repo)

Changing from calculating the correct answer for each new program to
calculating them in advance and storing in a table, reduced the
execution time by about 2.7%.

An 8% improvement resulted from adding the "commutative" bit to the five
commutative operations (add, mul, and, or, xor).  Perhaps more
importantly, it reduced the printout of essentially duplicate solutions.

An improvement by a factor of 2.58 (25.3/9.8) resulted from ensuring
that the last register operand of the last instruction, when this
instruction is created, refers to the result of the immediately
preceding instruction.

Continued the above idea for other register operands, i.e., ensured that
SOME operand of the last instruction always refers to the result of the
immediately preceding instruction.  Got an improvement by a factor of
1.04 (9.8/9.4).

3/16/02:  Got a factor of 1.85 by having it simulate the program only
from the last changed instruction to the end, which means that usually
only the last instruction is simulated.  Also, changed the trial
value(s) so they "stick" at the last failed one(s).  When a trial value
is changed, which happens after a success, the whole program must be
simulated.

3/16/02:  Got a factor of 1.010 by moving the assignment to
computed_result inside the loop just ahead of where it was.  (The loop
is usually executed only once.)

3/16/02:  Got a factor of 1.020 by computing corr_res only when sticky_i
and/or sticky_j change.

3/17/02:  Tried making "numi" a constant defined with #define.  Got a 5%
improvement.  Decided not to do this.

3/19/02:  Got a factor of 1.166 by inlining "increment."

3/23/02:  Took 1614 secs (26.9 min) to search with numi = 4.

9/19/02:  Got a factor of 1.131 by requiring that immediate values be in
the order 0, -1, 1, ... and using isa.opndstart[3] to avoid certain
silly cases like ADD of 0, ADD of -1 (we do a subtract of 1), AND of 0
or -1, etc.  This was made kind of necessary because the compare ops
should have an immediate value of 0 as a possibility, whereas for most
other ops, immediate 0 would never be used.

9/22/02:  Changed shift immediate amounts to be given in an array
(shimmed), so that fewer than 31 values can be specified.  This gave no
change to execution time if all 31 are specified (1.222 secs for
absolute value problem on a basic RISC, running on my 1.8 mHz Thinkpad).
If only 4 values are specified, e.g. 1, 2, 30, and 31, the execution
dropped to 0.450 secs, a factor of 2.71 improvement.
   I don't quite understand this, because the number of evaluations of
the third instruction reduced from 14.2 million to 2.74 million, a
factor of 5.18.
   It's partly explained by the program load time.  If you run aha with
an argument of 1, it takes 0.140 secs.  Thus the time to start and end
the program is about that amount.  So the ration of actual execution
time is (1.222 - 0.140)/(0.450 - 0.140) = 3.49.  Closer to 5.18, but not
very close.

9/24/02:  Put ALL immediates (both ordinary and shift amounts) in the
registers.  This did not affect the execution time (if compiled -O2),
but it allowed deleting the operand type info in the isa table, and
simplified the code a little (by 77 lines).
   Execution time for the standard run is now 0.591 secs on my 667 mHz
machine (compiled -O2, which I guess I'll use from now on).

9/25/02:  Made Aha! measure and print its own execution time, using
clock().  I believe this is user + system time for the Aha! process,
rather than wall clock time.  Found that -O2 and -O3 make no difference;
the assembly language files search.s and check.s are identical.  Am
using -O2.  The standard job runs in from 0.520 to 0.540 seconds process
time on my 667 mHz office machine.
   The number of instruction evaluations is 62248 + 82618 + 2743328 (for
the first, second, and third instruction resp.), or 2888194 total.  This
corresponds to 122 cycles per evaluation.

9/30/02:  Before today, the program consisted of three .c files:  aha,
search, and check.  Made it all one file (aha.c), mainly because of
problems with C in defining a preset array of values and not requiring
the user to also set a variable equal to the number of values in the
array.  No change to execution time.  Build (mainly compilation) time
dropped from about 2.0 secs to 1.2 secs.  This change also permits
inlining fix_operands, but trying that did not change execution time
measurably (so it is not inlined now).

10/14/02:  Before today, the incrementing of instructions was done with
the rightmost operand varying most rapidly.  Today it was changed so
that the leftmost operand varies most rapidly.  This simplifies the
handling of commutative ops, and permits a few other minor
simplifications.
   This gave a factor of 1.05 improvement in execution time.  Quite
minor, but the program is a little simpler and I think it will simplify
more complicated optimizations that may be done, such as (somehow)
avoiding programs that have an instruction whose result is unused.
   A preliminary investigation of this shows that for a typical RISC
instruction set, 39% of three-instruction programs have an unused
result, and 70% of four-instruction programs have an unused result.
This is compared to the present program, which ensures only that the
second from last computed result does not go unused.  Thus there is hay
to be made here.
   An attempt to skip ALL these silly programs resulted in a net
increase in execution time, because it was implemented inefficiently.
It seems to be hard to devise an efficient way to do this.  Some
compromise might be practical, such as ensuring only that the second and
third from last results are not both unused.

10/15/02:  Changed the program as just mentioned, i.e., to ensure that
instruction n (the last) uses the result of instruction n-1 and, if
instruction n-1 does not use the result of instruction n-2, then the
last instruction does.  This improved execution time by a factor of 1.4
for three-instruction programs, and a factor of 1.8 for four-instruction
programs.

4/22/03: Ran Aha! on a two-input problem with n = 5 and 17 instructions
enabled. Was searching for 5-instruction programs to compute the average
of two signed integers (without overflowing). Shut it off after 144
hours (6 days). I should make it display a "progress report" for such
long jobs, such as printing out the first instruction in the list each
time a new opcode is selected for it. Otherwise, you don't know if it
somehow got into an infinite loop and you have no idea how long the run
will take.

2/25/11: Incorporated a correction to the printb routine from Greg
Parker, which makes it run on the 64-bit Mac OS X (and probably other
machines).
