# Variables Pt. 2

## Flavors and modification

There are two flavors of variables:
    - recursive (use = ) - only looks for the variables when the commadn is used, not when it's defined.
    - simply expanded (use := ) - like normal imperative --only those defined so far get expanded

    # Recursive variable. This will print "later" below
    one = one ${later_variable}
    # Simply expanded variable. This will not print "later" below
    two := two ${later_variable}

    later_variable = later

    all: 
        echo $(one)
        echo $(two)

**Result**

    echo one later
    one later
    echo two 
    two

Simply expanded (using := ) allows you to append to a variable. Recursive definitions will give an infinite loop error.
    one = hello
    # one gets defined as a simply expanded variable (:=) and thus can handle appending
    one := ${one} there

    all: 
        echo $(one)

**Result**

    echo hello there
    hello there

?= only sets variables if they have not yet been set
    one = hello
    one ?= will not be set
    two ?= will be set

    all: 
        echo $(one)
        echo $(two)

**Result**

    echo hello
    hello
    echo will be set
    will be set

Spaces at the of a line are not stripped, but those at the start are. To make a variable with a single space, use $(nullstring)

    with_spaces = hello   # with_spaces has many spaces after "hello"
    after = $(with_spaces)there

    nullstring =
    space = $(nullstring) # Make a variable with a single space.

    all: 
        echo "$(after)"
        echo start"$(space)"end

**Result**

    echo "hello   there"
    hello   there
    echo start" "end
    start end

An undefined variable is actually an empty string!

    all: 
        # Undefined variables are just empty strings!
        echo $(nowhere)

**Results**
    # Undefined variables are just empty strings!
    echo ""

Use += to append

    foo := start
    foo += more

    all: 
        echo $(foo)

**Result**

    echo start more
    start more

String Substitution is also a really common and useful way to modify variables. Also check out Text Functions and Filename Functions.

## Command line arguments and override
You can override variables that come from the command line by using override. Here we can make with make option_one=hi

    # Overrides command line arguments
    override option_one = did_override
    # Does not override command line arguments
    option_two = not_override
    all: 
        echo $(option_one)
        echo $(option_two)

**Result**
    make option_one=hi
    did_override
    echo not_override
    not_override
    PS D:\NIC\Makefile_Series\06_Variables Pt.2> make option_two=hi
    echo did_override
    did_override
    echo hi
    hi

If i use make option_one=hi, it will still echo did_override because of the override directive, However, when i use make option_two=hi, it will echo hi.

## list of commands and define

The define directive is not a function, though it may look that way. I've seen it used so infrequently that I won't go into details, but it's mainly used for defining canned recipes and also pairs well with the eval function.

Define / endef simply creates a variable that is set to a list of commands. Note here that it's a bit different that having a semi-colon between commands, because each is run in a separate shell, as expected.

    one = export blah="I was set!"; echo $$blah

    define two
    export blah="I was set!"
    echo $$blah
    endef

    all: 
        @echo "This prints 'I was set'"
        @$(one)
        @echo "This does not print 'I was set' because each command runs in a separate shell"
        @$(two)

**Result:**

    This prints 'I was set'
    I was set!
    This does not print 'I was set' because each command runs in a separate shell

Each line inside two runs in a separate shell, as Make treats each line of a multi-line variable as a distinct command.