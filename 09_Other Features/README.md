# Other Features
## Include Makefiles

The include directive tells make read one or more other makefiles. It's a line in the makefile that looks like this:

    include filenames...

This is paricularly useful when you use compiler flags like -M that create Makefiles based on the source. For example, if some c files includes a header, that header will be added to a Makefile that's written by gcc. I talk about this more in the Makefile cookbook

## The vpath Directive
Use vpath to specify where some set of prerequisites exist. The format is vpath `<pattern> <directories, space/colon separated> <pattern>` can have a `%, which matches any zero or more characters. You can also do this globallyish with the variable VPATH.

    vpath %.h ../headers ../other-directory

    # Note: vpath allows blah.h to be found even though blah.h is never in the current directory
    some_binary: ../headers blah.h
        touch some_binary

    ../headers:
        mkdir ../headers

    # We call the target blah.h instead of ../headers/blah.h, because that's the prereq that some_binary is looking for
    # Typically, blah.h would already exist and you wouldn't need this.
    blah.h:
        touch ../headers/blah.h

    clean:
        rm -rf ../headers
        rm -f some_binary

**Result**

    mkdir ../headers
    touch ../headers/blah.h
    touch some_binary

## Multiline
The backslash("\") character gives us the ability to use multiple lines when the commands are too long

    some_file: 
        @echo This line is too long, so \
            it is broken up into multiple lines

## .phony
Adding .PHONY to a target will prevent Make from confusing the phony target with a file name. In this example, if the file clean is created, make clean will still be run. Technically, I should have used it in every example with all or clean, but I wanted to keep the examples clean. Additionaly, "phony" targets typically have names that are rarely file names, and in pratice many people skip this.

## .delete_on_error
The make tool will stop running a rule (and will propogate back to prerequisites) if a command returns a nonzero exit status. DELETE_ON_ERROR will delete the target of a rule if the rule fails in this manner. This will happen for all targets, not just the one it is before like PHONY. It's a good idea to always use this, even though make does not for historical reasons.

    .DELETE_ON_ERROR:
    all: one two

    one:
        touch one ## touch one
        false ## delete one and do not execute two

    two:
        touch two
        false