
# Fancy Rules

## Implicit Rules

Make loves c compilation. And every time it expresses its love, things get confusing. Perhaps the most confusing part of Make is the magic/automatic rules that are made. Make calls these "implicit" rules. I don't personally agree with this design decision, and i dont recommend using them, but they're often used and are thus useful to know. Here's a list of implicit rules:
- Compiling a C prohram: n.o is made automatically from n.c with a command of the form \$(CC) -c $(CPPFLAGS) \$(CFLAGS) \$^ -o $@
- Compiling a C++ program: n.o is made automatically from n.cc or n.cpp with a command of the form /$(CXX) -c /$(CPPFLAGS) /$(CXXFLAGS) /$^ -o $@
- Linking a single object file: n is made automatically from n.o by running the command /$(CC) /$(LDFLAGS) /$^ /$(LOADLIBES) /$(LDLIBS) -o $@ 

The important variables used by implicit rules are:

- CC: Program for compiling C programs default cc
- CXX: Program fro compiling C++ programs default g++
- CFLAGS: Extra flags to give to the C compiler
- CXXFLAGS: Extra flags to give to the C++ compiler
- CPPFLAGS: Extra to give to the C preprocessor
- LDFLAGS: Extra flags to give to compilers when they are supposed in invoke the linker.

Let's see how we can now buid a C program without ever explicitly telling Make how to do the compilation:

    CC = gcc # Flag for implicit rules
    CFLAGS = -g # Flag for implicit rules. Turn on debug info

    # Implicit rule #1: blah is built via the C linker implicit rule
    # Implicit rule #2: blah.o is built via the C compilation implicit rule, because blah.c exists
    blah: blah.o

    blah.c:
        echo "int main() { return 0; }" > blah.c

    clean:
        rm -f blah*

**Result:**

    echo "int main() { return 0; }" > blah.c
    gcc  -g    -c -o blah.o blah.c
    gcc    blah.o   -o blah

## Static Pattern Rules
Static pattern rules are another way to write less in a Makefiles. Here's their syntax:
    targets...: target-pattern: prereq-patterns ...
        command
The essence is that the given target is matched by the target-pattern (via a % wildcard). Whatever was matched is called the stem. The stem is then substituted into the prereq-pattern, to generate the target's prereqs.

A typical use case is to compile .c files into .o files. Here's the manual way:

    objects = foo.o bar.o all.o
    objects = foo.o bar.o all.o
    all: $(objects)
        $(CC) $^ -o all

    foo.o: foo.c
        $(CC) -c foo.c -o foo.o

    bar.o: bar.c
        $(CC) -c bar.c -o bar.o

    all.o: all.c
        $(CC) -c all.c -o all.o

    all.c:
        echo "int main() { return 0; }" > all.c

    # Note: all.c does not use this rule because Make prioritizes more specific matches when there is more than one match.
    %.c:
        touch $@

    clean:
        rm -f *.c *.o all

**Result**

    touch foo.c
    cc -c foo.c -o foo.o
    touch bar.c
    cc -c bar.c -o bar.o
    echo "int main() { return 0; }" > all.c
    cc -c all.c -o all.o
    cc foo.o bar.o all.o -o all

Here's the more efficient way, using a static pattern rule:

    objects = foo.o bar.o all.o
    all: $(objects)
        $(CC) $^ -o all

    # Syntax - targets ...: target-pattern: prereq-patterns ...
    # In the case of the first target, foo.o, the target-pattern matches foo.o and sets the "stem" to be "foo".
    # It then replaces the '%' in prereq-patterns with that stem
    $(objects): %.o: %.c
        $(CC) -c $^ -o $@

    all.c:
        echo "int main() { return 0; }" > all.c

    # Note: all.c does not use this rule because Make prioritizes more specific matches when there is more than one match.
    %.c:
        touch $@

    clean:
        rm -f *.c *.o all

**Result:**

    touch foo.c
    cc -c foo.c -o foo.o
    touch bar.c
    cc -c bar.c -o bar.o
    echo "int main() { return 0; }" > all.c
    cc -c all.c -o all.o
    cc foo.o bar.o all.o -o all

## Static Pattern Rules and Filter
While I introduce the filter function later on, it's common to use in static pattern rules, so i'll mention that here. The filter function can be used in Static pattern rules to match the correct files. In this example, I made up the .raw and .result extensions. 

    obj_files = foo.result bar.o lose.o
    src_files = foo.raw bar.c lose.c

    all: $(obj_files)
    # Note: PHONY is important here. Without it, implicit rules will try to build the executable "all", since the prereqs are ".o" files.
    .PHONY: all 

    # Ex 1: .o files depend on .c files. Though we don't actually make the .o file.
    $(filter %.o,$(obj_files)): %.o: %.c
        echo "target: $@ prereq: $<"

    # Ex 2: .result files depend on .raw files. Though we don't actually make the .result file.
    $(filter %.result,$(obj_files)): %.result: %.raw
        echo "target: $@ prereq: $<" 

    %.c %.raw:
        touch $@

    clean:
        rm -f $(src_files)

**Result**

    echo "target: foo.result prereq: foo.raw" 
    target: foo.result prereq: foo.raw
    echo "target: bar.o prereq: bar.c"
    target: bar.o prereq: bar.c
    echo "target: lose.o prereq: lose.c"
    target: lose.o prereq: lose.c

**Explain**

    Ex2: This filters any .result file in obj_files and makes it depend on the corresponding .raw file. 
    Ex1: This filters any .o file in obj_files and makes it depend on the corresponding .c file.


The echo command is simply for logging purposes, showing which target and prerequisite are beging processed.

## Pattern Rules
Pattern rules are often used but quite confusing. You can look at them as two ways:
    - A way to define your own implicit rules
    - A simpler form of static pattern rules
Let's start with an example first:

    # Define a pattern rule that compiles every .c file into a .o file
    %.o : %.c
            $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@

Pattern rules contain a '%' in the target. This '%' matches any nonempty string, and the other characters match themselves. '%' in a prerequiste of a pattern rule stands for the same stem that was matched by the '%' in the target.

Here's another example:
    # Define a pattern rule that has no pattern in the prerequisites.
    # This just creates empty .c files when needed.
    %.c:
    touch $@

## Double-Colon Rules
Double-Colon Rules are rarely used, but allow multiple rules to be defined for the same target. If these were single colons, a warning would be printed and only the second set of commands would run.

    all: blah
    blah::
        echo "hello"
    blah:: 
        echo "hello again"

**Result**

    echo "hello"
    hello
    echo "hello again"
    hello again
