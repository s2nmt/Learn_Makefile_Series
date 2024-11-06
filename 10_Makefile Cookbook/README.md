# Makefile Cookbook

Let's go through a really juicy Make example that works well for medium sized projects.

The neat thing about this makefile is it automatically determines dependencies for you. All you have to do is put your C/C++ files in the src/folder.

    # Compiler to be used
    CC = gcc

    # Compiler flags
    CFLAGS = -Wall

    # Target executable name
    TARGET = test

    # Directory paths
    SRC_DIR = src
    BUILD_DIR = build

    # Object files
    OBJS = $(BUILD_DIR)/test.o

    # Rule to compile the program
    all: $(BUILD_DIR)/$(TARGET)

    $(BUILD_DIR)/$(TARGET): $(OBJS) | $(BUILD_DIR)
        $(CC) $(CFLAGS) -o $(BUILD_DIR)/$(TARGET) $(OBJS)

    $(BUILD_DIR)/test.o: $(SRC_DIR)/test.c | $(BUILD_DIR)
        $(CC) $(CFLAGS) -c $(SRC_DIR)/test.c -o $(BUILD_DIR)/test.o

    # Create the build directory if it doesn't exist
    $(BUILD_DIR):
        mkdir -p $(BUILD_DIR)

    # Clean up generated files
    clean:
        rm -rf $(BUILD_DIR)
