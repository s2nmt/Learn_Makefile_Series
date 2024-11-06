# Conditional part of Makefiles

## Conditional if/else

    foo = ok

    all:
    ifeq ($(foo), ok)
        echo "foo equals ok"
    else
        echo "nope"
    endif

**Result**

    echo "foo equals ok"
    foo equals ok

## Check if a variable is empty

    nullstring =
    foo = $(nullstring) # end of line; there is a space here

    all:
    ifeq ($(strip $(foo)),)
        echo "foo is empty after being stripped"
    endif
    ifeq ($(nullstring),)
        echo "nullstring doesn't even have spaces"
    endif

**Result:**

    echo "foo is empty after being stripped"
    foo is empty after being stripped
    echo "nullstring doesn't even have spaces"
    nullstring doesn't even have spaces

# Check if a variable if empty

ifdef does not expand variables references, it just sees if something is defined at all

    bar =
    foo = $(bar)

    all:
    ifdef foo
        echo "foo is defined"
    endif
    ifndef bar
        echo "but bar is not"
    endif

**Result**

    echo "foo is defined"
    foo is defined
    echo "but bar is not"
    but bar is no

## $(MAKEFLAGS)
    This example shows you how to test make flags with findstring and MAKEFLAGS. Run this example with make -i to see it print out the echo statement.

    all:
        # Search for the "-i" flag. MAKEFLAGS is just a list of single characters
        # , one per flag. So look for "i" in this case.
        ifneq (,$(findstring i, $(MAKEFLAGS)))
            echo "i was passed to MAKEFLAGS"
        endif