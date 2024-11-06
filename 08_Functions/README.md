# Functions

## First Functions

Functions are mainly just for text processing. Call functions with $(fnm arguments) or ${fn, arguments}. Make has a decent amount of builtin funtions.

    bar := ${subst not,"totally", "I am not superman"}
    all: 
        @echo $(bar)

**result**

    I am totally superman

If you want to replace spaces or commas, use variables

    comma := ,
    empty:=
    space := $(empty) $(empty)
    foo := a b c
    bar := $(subst $(space),$(comma),$(foo))

    all: 
        @echo $(bar)

**Result**

    a,b,c

Do NOT include spaces in the arguments after the first. That will be seen as part if the string.

    comma := ,
    empty:=
    space := $(empty) $(empty)
    foo := a b c
    bar := $(subst $(space), $(comma) , $(foo)) # Watch out!

    all: 
        @echo $(bar)

**result**

    # Output is ", a , b , c". Notice the spaces introduced
    , a , b , c

## String Subsitution

$(patsubst pattern,replacement,text) does the following:

"Finds whitespace-separated words in text that match pattern and replaces them with replacement. Here pattern may contain a '\%' which acts as a wildcard, matching any number of any characters within a word. If replacement also contain a '\%', the '\%' is replaced by the text that matched the '\%' is unchanged.

The substitution reference $(text:pattern=replacement) is a shorthand for this. There's another shorthand that replaces only suffixes: $(text:suffix=replacement) . No % wildcard is used here.

Note: don't add extra spaces for this shorthand. It will be seen as a search or placement term.

    foo := a.o b.o l.a c.o
    one := $(patsubst %.o,%.c,$(foo))
    # This is a shorthand for the above
    two := $(foo:%.o=%.c)
    # This is the suffix-only shorthand, and is also equivalent to the above.
    three := $(foo:.o=.c)

    all:
        echo $(one)
        echo $(two)
        echo $(three)

**Result**

    echo a.c b.c l.a c.c
    a.c b.c l.a c.c
    echo a.c b.c l.a c.c
    a.c b.c l.a c.c
    echo a.c b.c l.a c.c
    a.c b.c l.a c.c

## The foreach function

The foreach function looks like this: $(foreach var,list,text). It converts one list of words (separated by spaces) to another. var is set to each word in list, and text is expanded for each word.

This appends an exclamation after each word.

    foo := who are you
    # For each "word" in foo, output that same word with an exclamation after
    bar := $(foreach wrd,$(foo),$(wrd)!)

    all:
        # Output is "who! are! you!"
        @echo $(bar)

## The if function

if checks if the first argument is nonempty. If so, runs the second argument, otherwise runs the third.

    foo := $(if this-is-not-empty,then!,else!)
    empty :=
    bar := $(if $(empty),then!,else!)

    all:
        @echo $(foo)
        @echo $(bar)

**Result:**

    then!
    else!

## The if function
if checks if the first argument is nonempty. If so, rún the second argument, otherwise runs the third

    foo := $(if this-is-not-empty,then!,else!)
    empty :=
    bar := $(if $(empty),then!,else!)

    all:
        @echo $(foo)
        @echo $(bar)

**Result**
    then!
    else!

## The call function

Make supports creating basic functions. You "define" the function just by creating a variable, but use the parameters $(0) . $(1) , etc. You then call the function with the special call builtin function. The syntax is $(call variable,param,param). $(0) is the variable, while $(1), $(2), etc are thee params.

    sweet_new_fn = Variable Name: $(0) First: $(1) Second: $(2) Empty Variable: $(3)

    all:
        # Outputs "Variable Name: sweet_new_fn First: go Second: tigers Empty Variable:"
        @echo $(call sweet_new_fn, go, tigers)

## The shell function

shell - This calls the shell, but it replaces newlines with spaces!
    all: 
        @echo $(shell ls -la) # Very ugly because the newlines are gone!

## The filter function
The filter function is used to select certain elements from a list that match a specific pattern. For example, this will select all elements in obj_files that end with .o

    obj_files = foo.result bar.o lose.o
    filtered_files = $(filter %.o,$(obj_files))

    all:
        @echo $(filtered_files)

**Result:**

    bar.o lose.o

Filter can also be used in more complex ways:
    1. Filtering multiple patterns: You can filter multiple patterns at once. For example `$(filter %.c %.h, $(files))` will select all .c and .h files from the files list.
    2. Negation: if you want to select all elements that do not match a pattern, you can use filter-out . For example, `$(filter-out %.h, $(files))` will select all files that are not .h file.
    3. Nested filter: You can nest filter functions to apply multipe filters. For example, `$(filter %.o, $(filter-out test%, $(objects)))` will select all files that end with .o but don't start with test.