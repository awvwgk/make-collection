#!/bin/env make
# 
# created by awvwgk in March 2018 (v 2.0)
# this Makefile is aimed at FORTRAN code!
#
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

# choose your compiler here (you can choose LINUXI)
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

###------------------------------------------------------------------------###
# to use the Intel-Compiler please set MKLROOT somewhere
ifeq ($(OSTYPE),LINUXI)
FC := ifort
CC := icc

# compile flags
FCFLAGS := -O -axAVX -qopenmp
FCFLAGS += -module $(MODDIR)
FCFLAGS += -I$(INCDIR)                       \
           -I$(MKLROOT)/include/intel64/lp64 \
           -I$(MKLROOT)/include
CCFLAGS := -O -DLINUX
CCFLAGS += -D_Float128=__float128
# check for debug build
ifeq ($(DEBUG_BUILD),yes)
FCFLAGS += -g -traceback
CCFLAGS += -g -DEBUG
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
# set the targets
.PHONY: all lib prog setup depend clean veryclean

# this is the first, so it will be made by default
all: setup lib prog

# we can also enforce it by
.DEFAULT_GOAL=all

###------------------------------------------------------------------------###
# generate the name of the program and the library
LIBNAME  := $(LIBDIR)/lib$(NAME).a
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
LIBS  := $(addprefix -l,$(NAME) $(LIBS))
# we merge all files for convinience
OBJS:= $(OBJSP) $(OBJS3) $(OBJS2) $(OBJS1) $(OBJS0)
# everything should also dependent on the local Makefile
ifeq ($(MAKEFILE_DEP),yes)
   MAKEDEP := Makefile
endif # DEP
ifeq ($(DEP_MAKEFILE_DEP),yes)
   DEPMAKEDEP := Makefile
endif # DEP

###------------------------------------------------------------------------###
# set up the targets for all
lib: $(LIBNAME)
prog: $(PROGNAME)

ifeq ($(USE_DEPENDENCIES),yes)
depend: $(DEPS)
endif # DEP

###------------------------------------------------------------------------###
# define how to actually generate the library and the program
$(LIBNAME): $(OBJS3) $(OBJS2) $(OBJS1) $(OBJS0)
	@echo "creating $@"                     && \
	$(AR) rv $@ $^                          && \
	echo "finished building $@ successfully"

# build everything together
$(PROGNAME): $(LIBNAME) $(OBJSP)
	@echo "linking $@"                      && \
	$(FL) $(OBJSP) $(LIBS) $(FLFLAGS) -o $@ && \
	echo "finished linking $@ successfully"

###------------------------------------------------------------------------###
# first deactivate some implicit rules
%.o : %.mod
%.f : %.F

# now the rules to actually build the object files
$(OUTDIR)/%.o: %.f $(MAKEDEP)
	@echo "making $@ from $<" && \
	$(FC) $(FCFLAGS) -c $< -o $@

$(OUTDIR)/%.o: %.f90 $(DEPDIR)/%.d $(MAKEDEP)
	@echo "making $@ from $<" && \
	$(FC) $(FCFLAGS) -c $< -o $@

$(OUTDIR)/%.o: %.c $(MAKEDEP)
	@echo "making $@ from $<" && \
	$(CC) $(CCFLAGS) -c $< -o $@


ifeq ($(USE_DEPENDENCIES),yes)
# this is a crutial part of this Makefile!
-include $(DEPS)

# now for the dependencies,
# we use a script hacked together in ruby (v 2.5.0) for this job
$(DEPDIR)/%.d: %.f90 $(DEPMAKEDEP)
	@echo "generating dependencies for $<"                        && \
	deps=`ruby $(MAINDIR)/depend.rb $<`                           && \
	echo "\$$(OUTDIR)/$(patsubst $(DEPDIR)/%.d,%.o,./$@)$$deps" | tee $@

$(DEPDIR)/%.d: %.f $(DEPMAKEDEP)
	@echo "generating dependencies for $<"                        && \
	deps=`ruby $(MAINDIR)/depend.rb $<`                           && \
	echo "\$$(OUTDIR)/$(patsubst $(DEPDIR)/%.d,%.o,./$@)$$deps" | tee $@

# deactivate dependency generation for C-code
$(DEPDIR)/%.d: %.c %.h $(DEPMAKEDEP)
	@echo "generating no dependencies for $<"                     && \
	echo "\$$(OUTDIR)/$(patsubst $(DEPDIR)/%.d,%.o,./$@):" | tee $@

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
clean:
	$(RM) $(OUTDIR)/*.o $(MODDIR)/*.mod $(PROGNAME) $(LIBNAME)

# in case you use the dependency build you can get rid of those files to
veryclean: clean
	$(RM) $(DEPDIR)/*.d

