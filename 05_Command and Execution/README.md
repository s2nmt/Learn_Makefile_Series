# Commnds and execution

## Command Echoing/Silencing
Add an @ before a command to stop it from being printed
You can also run make with -s to add an @ before each line

    all:
        @echo "This make line will not be printed"
        echo "But this will"

## Command Execution
Each command is run in a new shell (or at least the effect is as such)

    all: 
        cd ..
        # The cd above does not affect this line, because each command is effectively run in a new shell
        echo `pwd`

        # This cd command affects the next because they are on the same line
        cd ..;echo `pwd`

        # Same as above
        cd ..; \
        echo `pwd`

**Result**

    cd ..
    # The cd above does not affect this line, because each command is effectively run in a new shell
    echo `pwd`
    /d/NIC/Makefile_Series/05_Command and Execution
    # This cd command affects the next because they are on the same line
    cd ..;echo `pwd`
    /d/NIC/Makefile_Series
    # Same as above
    cd ..; \
    echo `pwd`
    /d/NIC/Makefile_Series

## Default Shell

The default shell is /bin/sh. You can change this by chaning the variable SHELL:
    SHELL=/bin/bash

    cool:
        echo "Hello from bash"

**Result**

    echo "Hello from bash"
    Hello from bash

## Double dollar sign
If you want a string to have a dollar sign, you can use $$. This is how to use a shell variable in bash or sh.
Note the differences between Makefile variables and Shell variables in this next sample.

    make_var = I am a make variable
    all:
    # Same as running "sh_var='I am a shell variable'; echo $sh_var" in the shell
        sh_var='I am a shell variable'; echo $$sh_var

    # Same as running "echo I am a make variable" in the shell
        echo $(make_var)

## Error handling with -k, -i, and -
Add -k when running make to continue running even in the of errors. Helpful if you want to see all the errors of Make at once.
Add a - before a command to suppress the error
Add -i to mkae to have this happen for every command
    one:
        # This error will be printed but ignored, and make will continue to rung
        -flase
        touch one
        
**Result**

    # This error will be printed but ignored, and make will continue to run
    false
    make: [Makefile:3: one] Error 1 (ignored)
    touch one

## Interrupting or killing make
Note only: If you ctrl+c make, it will delete the newer targets it just made.

Resursive use of make
To recursively call a makefile, use the special $(MAKE) instead of make becaude it will pass the make flags for you and won't itself be affected by them.

    new_contents = "hello:\n\ttouch inside_file"
    all:
        mkdir -p subdir
        printf $(new_contents)> subdir/makefile
        cd subdir && $(MAKE)

    clean:
        rm -rf subdir

**Result:**

    mkdir -p subdir
    printf "hello:\n\ttouch inside_file"> subdir/makefile
    cd subdir && /usr/bin/make
    make[1]: Entering directory '/d/NIC/Makefile_Series/05_Command and Execution/subdir'
    touch inside_file
    make[1]: Leaving directory '/d/NIC/Makefile_Series/05_Command and Execution/subdir'

## Export, environments, and recursive make
When Make starts, it automatically creates Make variables out of all the enviroment variables that are set when it's executed.

    # Run this with "export shell_env_var='I am an environment variable'; make"
    all:
        # Print out the Shell variable
        echo $$shell_env_var

        # Print out the Make variable
        echo $(shell_env_var)

**Result:**

    # Print out the Shell variable
    echo $shell_env_var

    # Print out the Make variable
    echo 
The export directive takes a variable and sets it the environment for all shell commands in all the recipes:

    shell_env_var=Shell env var, created inside of Make
    export shell_env_var
    all:
        echo $(shell_env_var)
        echo $$shell_env_var

**Result**

    echo Shell env var, created inside of Make
    Shell env var, created inside of Make
    echo $shell_env_var
    Shell env var, created inside of Make

As such, when you run the mkae command inside of make, you can use the export directive to make it accessible to sub-make commands. In the example, cooly is exported such that the makefile in subdir can use it.

    new_contents = "hello:\n\techo \$$(cooly)"

    all:
        mkdir -p subdir
        printf $(new_contents) | sed -e 's/^ //' > subdir/makefile
        @echo "---MAKEFILE CONTENTS---"
        @cd subdir && cat makefile
        @echo "---END MAKEFILE CONTENTS---"
        cd subdir && $(MAKE)

    # Note that variables and exports. They are set/affected globally.
    cooly = "The subdirectory can see me!"
    export cooly
    # This would nullify the line above: unexport cooly

    clean:
        rm -rf subdir

**Result**

    mkdir -p subdir
    printf "hello:\n\techo \$(cooly)" | sed -e 's/^ //' > subdir/makefile
    ---MAKEFILE CONTENTS---
    hello:
            echo $(cooly)---END MAKEFILE CONTENTS---
    cd subdir && /usr/bin/make
    make[1]: Entering directory '/d/NIC/Makefile_Series/05_Command and Execution/subdir'
    echo "The subdirectory can see me!"
    The subdirectory can see me!
    make[1]: Leaving directory '/d/NIC/Makefile_Series/05_Command and Execution/subdir'

You need to export variables to have them run in the shell as well.
    one=this will only work locally
    export two=we can run subcommands with this

    all: 
        @echo $(one)
        @echo $$one
        @echo $(two)
        @echo $$two

**Result**

    this will only work locally

    we can run subcommands with this
    we can run subcommands with this

There is a blank line because $$ prints the value of a shell variable. Since it’s not exported, it outputs a blank line.

.EXPORT_ALL_VARIABLES exports all variables for you.

    .EXPORT_ALL_VARIABLES:
    new_contents = "hello:\n\techo \$$(cooly)"

    cooly = "The subdirectory can see me!"
    # This would nullify the line above: unexport cooly

    all:
        mkdir -p subdir
        printf $(new_contents) | sed -e 's/^ //' > subdir/makefile
        @echo "---MAKEFILE CONTENTS---"
        @cd subdir && cat makefile
        @echo "---END MAKEFILE CONTENTS---"
        cd subdir && $(MAKE)

    clean:
        rm -rf subdir

**Result**

    mkdir -p subdir
    printf "hello:\n\techo \$(cooly)" | sed -e 's/^ //' > subdir/makefile
    ---MAKEFILE CONTENTS---
    hello:
            echo $(cooly)---END MAKEFILE CONTENTS---
    cd subdir && /usr/bin/make
    make[1]: Entering directory '/d/NIC/Makefile_Series/05_Command and Execution/subdir'
    echo "The subdirectory can see me!"
    The subdirectory can see me!
    make[1]: Leaving directory '/d/NIC/Makefile_Series/05_Command and Execution/subdir'

## Arguments to make

There's a nice [list](http://www.gnu.org/software/make/manual/make.html#Options-Summary)
 of options that can be run from make. Check out --dry-run, --touch, --old-file.
You can have multiple targets to make, i.e. make clean run test runs the clean goal, then run, and then test.

