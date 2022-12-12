# Thanks to Job Vranish (https://spin.atomicobject.com/2016/08/26/makefile-c-projects/)
TARGET_EXEC := libhirt

BUILD_DIR := ./build
SRC_DIRS := .

# Find all the C and C++ files we want to compile
# Note the single quotes around the * expressions. Make will incorrectly expand these otherwise.
SRCS := $(basename $(shell find $(SRC_DIRS) -name '*.cpp' -or -name '*.c' -or -name '*.s'))

# String substitution for every C/C++ file.
# As an example, hello.cpp turns into ./build/hello.cpp.o
OBJSARM64 := $(SRCS:%=$(BUILD_DIR)/%.arm64.o)
OBJSX86 := $(SRCS:%=$(BUILD_DIR)/%.x86_64.o)

# String substitution (suffix version without %).
# As an example, ./build/hello.cpp.o turns into ./build/hello.cpp.d
DEPS := $(OBJSARM64:.o=.d) $(OBJSX86:.o.d)

# Every folder in ./src will need to be passed to GCC so that it can find header files
INC_DIRS := $(shell find $(SRC_DIRS) -type d)
# Add a prefix to INC_DIRS. So moduleA would become -ImoduleA. GCC understands this -I flag
INC_FLAGS := $(addprefix -I,$(INC_DIRS))

# The -MMD and -MP flags together generate Makefiles for us!
# These files will have .d instead of .o as the output.
CPPFLAGS := $(INC_FLAGS) -MMD -MP -std=c++11 -O3 -ffast-math

$(BUILD_DIR)/$(TARGET_EXEC).a : $(BUILD_DIR)/$(TARGET_EXEC).arm64.a $(BUILD_DIR)/$(TARGET_EXEC).x86_64.a
	lipo -create -arch arm64 $(BUILD_DIR)/$(TARGET_EXEC).arm64.a -arch x86_64 $(BUILD_DIR)/$(TARGET_EXEC).x86_64.a -output $@

$(BUILD_DIR)/$(TARGET_EXEC).arm64.a: $(OBJSARM64)
	ar rcs $@ $(OBJSARM64)

$(BUILD_DIR)/$(TARGET_EXEC).x86_64.a: $(OBJSX86)
	ar rcs $@ $(OBJSX86)

# Build step for C source
$(BUILD_DIR)/%.arm64.o: %.c
	mkdir -p $(dir $@)
	$(CC) -arch arm64 $(CPPFLAGS) $(CFLAGS) -c $< -o $@

# Build step for C++ source
$(BUILD_DIR)/%.arm64.o: %.cpp
	mkdir -p $(dir $@)
	$(CXX) -arch arm64 $(CPPFLAGS) $(CXXFLAGS) -c $< -o $@

$(BUILD_DIR)/%.x86_64.o: %.c
	mkdir -p $(dir $@)
	$(CC) -arch x86_64 $(CPPFLAGS) $(CFLAGS) -c $< -o $@

# Build step for C++ source
$(BUILD_DIR)/%.x86_64.o: %.cpp
	mkdir -p $(dir $@)
	$(CXX) -arch x86_64 $(CPPFLAGS) $(CXXFLAGS) -c $< -o $@


.PHONY: clean
clean:
	rm -r $(BUILD_DIR)

# Include the .d makefiles. The - at the front suppresses the errors of missing
# Makefiles. Initially, all the .d files will be missing, and we don't want those
# errors to show up.
-include $(DEPS)
