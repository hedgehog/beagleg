# (c) 2013 h.zeller@acm.org
# This is free software. License: GNU 3.0

# Stuff based on the environment. This assumes stuff to be compiled on the
# beaglebone and the am335x_pru_package checked out.
# https://github.com/beagleboard/am335x_pru_package

# Change here for which hardware you are compiling. See hardware/ directory.
# Currently supported BUMPS, CRAMPS, and VGEN5
BEAGLEG_HARDWARE_TARGET?=BUMPS

# In case you cross compile this on a different architecture, uncomment this
# and set the prefix. Or simply set the environment variable.
#CROSS_COMPILE?=arm-arago-linux-gnueabi-

# Tuning options for ARM CPU. Unset this in an environment variable if compiled
# on a different system.
ARM_COMPILE_FLAGS?=-mtune=cortex-a8 -march=armv7-a

# Location of am335x package https://github.com/beagleboard/am335x_pru_package
# We check this out in a local git submodule.
AM335_BASE=../am335x_pru_package
PASM=$(AM335_BASE)/pru_sw/utils/pasm
LIBDIR_APP_LOADER?=$(AM335_BASE)/pru_sw/app_loader/lib
INCDIR_APP_LOADER?=$(AM335_BASE)/pru_sw/app_loader/include
CAPE_INCLUDE=../hardware/$(BEAGLEG_HARDWARE_TARGET)

CXX=g++
CFLAGS+= -Wall -I$(INCDIR_APP_LOADER) -I$(CAPE_INCLUDE) -D_XOPEN_SOURCE=500 -O3  $(ARM_COMPILE_FLAGS)
CXXFLAGS+=$(CFLAGS)

LDFLAGS+=-lpthread -lm
PRUSS_LIBS=-Wl,-rpath=$(LIBDIR_APP_LOADER) -L$(LIBDIR_APP_LOADER) -lprussdrv

# Assembled binary from *.p file.
PRU_BIN=motor-interface-pru_bin.h

GCODE_OBJECTS=gcode-parser.o gcode-machine-control.o determine-print-stats.o \
              generic-gpio.o arc-gen.o pwm-timer.o logging.o
OBJECTS=motor-operations.o sim-firmware.o pru-motion-queue.o $(GCODE_OBJECTS)
MAIN_OBJECTS=machine-control.o gcode-print-stats.o

TARGETS=../machine-control ../gcode-print-stats
UNITTEST_BINARIES=gcode-machine-control_test gcode-parser_test

all : $(TARGETS) $(UNITTEST_BINARIES)

../gcode-print-stats: gcode-print-stats.o $(GCODE_OBJECTS)
	$(CROSS_COMPILE)$(CXX) $(CFLAGS) -o $@ $^ $(LDFLAGS)

../machine-control: machine-control.o $(OBJECTS)
	$(CROSS_COMPILE)$(CXX) $(CFLAGS) -o $@ $^ $(PRUSS_LIBS) $(LDFLAGS)

test: $(UNITTEST_BINARIES)
	for test_bin in $(UNITTEST_BINARIES) ; do ./$$test_bin ; done

pru-motion-queue.o : motor-interface-constants.h $(PRU_BIN)
motor-operations.o : motor-interface-constants.h
sim-firmware.o : motor-interface-constants.h

# test dependencies.
gcode-machine-control_test.cc: gcode-machine-control.cc

$(PRU_BIN) : motor-interface-constants.h \
             $(CAPE_INCLUDE)/beagleg-pin-mapping.h \
	     $(CAPE_INCLUDE)/pru-io-routines.hp

%_test: %_test.cc $(OBJECTS)
	$(CROSS_COMPILE)$(CXX) $(CFLAGS) -o $@ $< $(OBJECTS) $(PRUSS_LIBS) $(LDFLAGS)

# Using old < c++11 version as not all embedded devices have recent compilers. Sigh.
%.o: %.cc
	$(CROSS_COMPILE)$(CXX) $(CXXFLAGS) -std=c++98 -c -o $@ $<

%_bin.h : %.p $(PASM)
	$(PASM) -I$(CAPE_INCLUDE) -V3 -c $<

$(PASM):
	make -C $(AM335_BASE)

clean:
	rm -rf $(TARGETS) $(MAIN_OBJECTS) $(OBJECTS) $(PRU_BIN) $(UNITTEST_BINARIES)