#!/bin/env make
# 
# April 2018 (v 2.3) created by Sebastian Ehlert
# 2.0: recreated from scratch                                          (1803)
# 2.1: refined the generation of dependencies, since it crashed if not
#      done in the main directory                                      (1803)
# 2.2: modified the clean command, to be less severe                   (1804)
# 2.3: added support for GNU compilers, some minor improvements        (1804)
#
###------------------------------------------------------------------------###
# a nice trick to print all 'important' targets this Makefile can produce
.PHONY: help

# would be made by default if there is no .DEFAULT_GOAL set
help:
	@echo "This Makefile kindly provides to you:"                     && \
	$(MAKE) --print-data-base --question |                               \
	awk '/^[^.%][-A-Za-z0-9]*:/ {print substr($$1, 1, length($$1)-1)}' | \
	sort | pr --omit-pagination --width=80 --columns=4

##############################################################################
## put all you sources here, if you do not know how this Makefile works,    ##
## only edit here:                                                          ##
##############################################################################

# the program you want to have, eventually...
NAME := 

###------------------------------------------------------------------------###
# some options first
BUILD_IN_BIN := no
DEBUG_BUILD  := yes

# beware! this means if you modify this file everything gets updated
# if you only edit this section, keeping this false might be a good idea
MAKEFILE_DEP := no

# use the dependency build, if so your source will be scanned for
# include and use statements and the results are included in this Makefile
#
# the command make depend becomes available to generate the dependency files
#
# you need the depend.rb script und a copy of ruby for this job
USE_DEPENDENCIES := yes
# also make the dependency files Makefile dependent
DEP_MAKEFILE_DEP := no

# sometimes it is not desirable to create a library, or even a binary
# use LIB_ONLY to create only the library and PROG_ONLY to ommit the library
BUILD_MODE  := default

# for big libraries it is sometimes faster to recreate the library then
# replacing some members, but this matters only for *really big* libraries
REBUILD_LIB := no

###------------------------------------------------------------------------###
# put all sources needed for the build here
# the source holding your main-program should go in OBJSP
# The numbering is abitrary, but it is garanteed that
# OBJS3 is always build before OBJS2 and so on.
OBJSP := 
OBJS0 := 
OBJS1 := 
OBJS2 := 
# to be save, put your modules here
OBJS3 := 

# put all include files here (currently not used)
INC  := 

# libraries, do not put the program name itself here
LIBS := mkl_blas95_lp64\
        mkl_lapack95_lp64\
        pthread\
        m

# choose your compiler here (you can choose LINUXI or LINUXG)
OSTYPE := LINUXI

###------------------------------------------------------------------------###
# specifiy where you are in the source tree
MAINDIR := .
# here resides the build system, only needed for larger projects
MAKEDIR := $(MAINDIR)/MAKE
# put all files to appropiate places
EXEDIR  := $(MAINDIR)/exe
LIBDIR  := $(MAINDIR)/lib
SRCDIR  := $(MAINDIR)/src
INCDIR  := $(MAINDIR)/include
MODDIR  := $(MAINDIR)/modules
# you can use a local build directory to separate the build
# from the main program
OUTDIR  := $(MAINDIR)/build
DEPDIR  := $(MAINDIR)/.depend

##############################################################################
## and stop editing here, ;)                                                ##
##############################################################################

###------------------------------------------------------------------------###
# set the path for make to search in (normally the SRCDIR is enough):
vpath % $(SRCDIR)
# look for modules expecially in the directory where the modules are put
vpath %.mod $(MODDIR)

###------------------------------------------------------------------------###
# to use the Intel-Compiler please set MKLROOT somewhere
ifeq ($(OSTYPE),LINUXI)
FC := ifort
CC := icc
CXX := icc

# compile flags
FCFLAGS := -O -axAVX -qopenmp
FCFLAGS += -module $(MODDIR)
FCFLAGS += -I$(INCDIR)                       \
           -I$(MKLROOT)/include/intel64/lp64 \
           -I$(MKLROOT)/include
CCFLAGS := -O -DLINUX
CCFLAGS += -D_Float128=__float128
CXXFLAGS := -O
CXXFLAGS += -D_Float128=__float128
# check for debug build
ifeq ($(DEBUG_BUILD),yes)
FCFLAGS += -g -traceback
CCFLAGS += -g -DEBUG
CXXFLAGS += -g -DEBUG
endif # DEBUG

# linker
FL := ifort
# general flags for linking, this is build incremental to get an overview
FLFLAGS := -static -fopenmp
# add path to libraries
FLFLAGS += -L$(MKLROOT)/lib/intel64 \
           -L$(LIBDIR)
# add path to modules
FLFLAGS += -module $(MODDIR)
# add path to include files
FLFLAGS += -I$(MKLROOT)/include/intel64/lp64 \
           -I$(MKLROOT)/include
# add special linker options for the MKL library
FLFLAGS += -Wl,--start-group     \
              -lmkl_intel_lp64   \
              -lmkl_core         \
              -lmkl_intel_thread \
           -Wl,--end-group

endif # LINUXI

###------------------------------------------------------------------------###
# GNU compilers using Intel's MKL
ifeq ($(OSTYPE),LINUXG)
FC := gfortran
CC := gcc
CXX := g++

# compile flags
CCFLAGS := -O -DLINUX
CXXFLAGS := -O -DLINUX

FCFLAGS := -O 
# add path to modules
FCFLAGS += -J$(MODDIR)
# add path to include files
FCFLAGS += -I$(INCDIR)
# dunno
FCFLAGS += -m64
# you don't want this
#FCFLAGS += -Wall
# check for debug build
ifeq ($(DEBUG_BUILD),yes)
FCFLAGS += -g
CCFLAGS += -g
CXXFLAGS += -g
endif # DEBUG


# linker
FL := gfortran

# general flags for linking, this is build incremental to get an overview
FLFLAGS := -static-libgfortran -fopenmp
# add path to libraries
FLFLAGS += -L$(MKLROOT)/lib/intel64 \
           -L$(LIBDIR)
# add special linker options for the MKL library
FLFLAGS += -Wl,--start-group     \
              -lmkl_gf_lp64      \
              -lmkl_intel_thread \
              -lmkl_core         \
           -Wl,--end-group
# dunno
LIBS += iomp5 dl
endif # LINUXG

###------------------------------------------------------------------------###
# set the targets
.PHONY: all library program setup depend

# this is the first, so it will be made by default
all: setup
ifneq ($(BUILD_MODE),PROG_ONLY)
all: library
endif # BUILD_MODE
ifneq ($(BUILD_MODE),LIB_ONLY)
all: program
endif # BUILD_MODE

# we can also enforce it by
.DEFAULT_GOAL=all

###------------------------------------------------------------------------###
# generate the name of the program and the library
ifneq ($(BUILD_MODE),PROG_ONLY)
   LIBNAME := $(LIBDIR)/lib$(NAME).a
endif # BUILD_MODE
PROGNAME := $(EXEDIR)/$(NAME)
ifeq ($(BUILD_IN_BIN),yes)
   PROGNAME += $(HOME)/bin/$(NAME)
endif # BIN
# set up the dependencies
ifeq ($(USE_DEPENDENCIES),yes)
   DEPS := $(patsubst %.o,$(DEPDIR)/%.d,\
      $(OBJSP) $(OBJS3) $(OBJS2)  $(OBJS1) $(OBJS0))
endif # DEP
# put all objects in the build directory
OBJS3 := $(addprefix $(OUTDIR)/,$(OBJS3))
OBJS2 := $(addprefix $(OUTDIR)/,$(OBJS2))
OBJS1 := $(addprefix $(OUTDIR)/,$(OBJS1))
OBJS0 := $(addprefix $(OUTDIR)/,$(OBJS0))
OBJSP := $(addprefix $(OUTDIR)/,$(OBJSP))
# libraries
ifneq ($(BUILD_MODE),PROG_ONLY)
    LIBS := $(NAME) $(LIBS)
endif # BUILD_MODE
LIBS  := $(addprefix -l,$(LIBS))
# we merge all files for convinience
OBJS := $(OBJSP)
OBJS += $(OBJS3)
OBJS += $(OBJS2)
OBJS += $(OBJS1)
OBJS += $(OBJS0)
ifeq ($(BUILD_MODE),PROG_ONLY)
   OBJSP := $(OBJS)
endif # BUILD_MODE
# everything should also dependent on the local Makefile
ifeq ($(MAKEFILE_DEP),yes)
   MAKEDEP := $(filter-out %.d, $(MAKEFILE_LIST))
endif # DEP
ifeq ($(DEP_MAKEFILE_DEP),yes)
   DEPMAKEDEP := $(filter-out %.d, $(MAKEFILE_LIST))
endif # DEP

###------------------------------------------------------------------------###
# set up the targets for all
library: $(LIBNAME)
program: $(PROGNAME)

ifeq ($(USE_DEPENDENCIES),yes)
depend: $(DEPS)
endif # DEP

###------------------------------------------------------------------------###
# define how to actually generate the library and the program
ifeq ($(REBUILD_LIB),yes)
$(LIBNAME): $(OBJS3) $(OBJS2) $(OBJS1) $(OBJS0)
	@echo "recreating $@"                   && \
	$(RM) $@                                && \
	$(AR) $(ARFLAGS) $@ $^                  && \
	echo "finished building $@ successfully"

else
$(LIBNAME): $(OBJS3) $(OBJS2) $(OBJS1) $(OBJS0)
	@echo "creating $@"                     && \
	$(AR) $(ARFLAGS) $@ $?                  && \
	echo "finished building $@ successfully"

endif


# build everything together
$(PROGNAME): $(LIBNAME) $(OBJSP)
	@echo "linking $@"                      && \
	$(FL) $(OBJSP) $(LIBS) $(FLFLAGS) -o $@ && \
	echo "finished linking $@ successfully"

###------------------------------------------------------------------------###
# first deactivate some implicit rules
%.o : %.mod
%.f : %.F

%.mod: %.o

# now the rules to actually build the object files
$(OUTDIR)/%.o: %.f $(MAKEDEP)
	@echo "making $@ from $<" && \
	$(FC) $(FCFLAGS) -c $< -o $@

$(OUTDIR)/%.o: %.f90 $(MAKEDEP)
	@echo "making $@ from $<" && \
	$(FC) $(FCFLAGS) -c $< -o $@

$(OUTDIR)/%.o: %.c $(MAKEDEP)
	@echo "making $@ from $<" && \
	$(CC) $(CCFLAGS) -c $< -o $@

$(OUTDIR)/%.o: %.cc $(MAKEDEP)
	@echo "making $@ from $<" && \
	$(CXX) $(CXXFLAGS) -c $< -o $@


ifeq ($(USE_DEPENDENCIES),yes)
# this is a crutial part of this Makefile!
-include $(DEPS)

$(OUTDIR)/%.o: $(DEPDIR)/%.d

# now for the dependencies,
# we use a script hacked together in ruby (v 2.5.0) for this job
$(DEPDIR)/%.d: %.f90 $(DEPMAKEDEP)
	@echo "generating dependencies for $<"    && \
	deps=`ruby $(MAKEDIR)/depend.rb $<`       && \
	echo "\$$(OUTDIR)/$*.o:$$deps" > $@

$(DEPDIR)/%.d: %.f $(DEPMAKEDEP)
	@echo "generating dependencies for $<"    && \
	deps=`ruby $(MAKEDIR)/depend.rb $<`       && \
	echo "\$$(OUTDIR)/$*.o:$$deps" > $@

# no hacks needed for C/C++ dependencies
$(DEPDIR)/%.d: %.c $(DEPMAKEDEP)
	@echo "generating dependencies for $<" && \
	$(CC) $(CPPFLAGS) -MM $< > $@

$(DEPDIR)/%.d: %.cc $(DEPMAKEDEP)
	@echo "generating dependencies for $<" && \
	$(CXX) $(CPPFLAGS) -MM $< > $@

endif # DEP

###------------------------------------------------------------------------###
# set up all directories
setup: $(EXEDIR) $(LIBDIR) $(MODDIR) $(OUTDIR) $(DEPDIR)

$(EXEDIR):
	@echo "creating directory for executables"  && \
	mkdir -p $@

$(LIBDIR):
	@echo "creating directory for libraries"    && \
	mkdir -p $@

$(MODDIR):
	@echo "creating directory for modules"      && \
	mkdir -p $@

$(OUTDIR):
	@echo "creating directory for object files" && \
	mkdir -p $@

$(DEPDIR):
	@echo "creating directory for dependencies" && \
	mkdir -p $@

###------------------------------------------------------------------------###
# clean up, at the right places
.PHONY: clean distclean veryclean

clean:
	$(RM) $(filter %.o,$(OBJS))

distclean:
	$(RM) $(OUTDIR)/*.o $(MODDIR)/*.mod $(PROGNAME) $(LIBNAME)

# in case you use the dependency build you can get rid of those files to
veryclean: distclean
	$(RM) $(DEPDIR)/*.d
