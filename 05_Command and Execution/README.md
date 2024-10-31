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
If you want a string to have a dollar sign, you can use $$. This is hown to use a shell variable in bash or sh.
Note the differences between Makefile variables and Shell variables in this next sample.

    make_var = I am a make variable
    all:
    # Same as running "sh_var='I am a shell variable'; echo $sh_var" in the shell
        sh_var='I am a shell variable'; echo $$sh_var

    # Same as running "echo I am a make variable" in the shell
        echo $(make_var)