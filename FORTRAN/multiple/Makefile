#!/bin/env make
# 
# created by awvwgk in March 2018 (v 2.0)
# this Makefile is aimed at FORTRAN code!
#
##############################################################################
## this is the toplevel Makefile controlling the build system               ##
##############################################################################

###------------------------------------------------------------------------###
# specifiy where you are in the source tree
MAINDIR := .
# here resides the build system
MAKEDIR := $(MAINDIR)/MAKE

include $(MAKEDIR)/Makeconfig

##############################################################################
## put all programs here, we use this silly syntax for you, to comment
## out parts of it more easily
PROGRAMS := NAME
PROGRAMS += 
##############################################################################
## only edit below if you know what you are doing                           ##
##############################################################################

###------------------------------------------------------------------------###
# set the targets
.PHONY: all main libs progs setup depend clean veryclean
.PHONY: $(PROGRAMS)

# this is the first, so it would be made by default
all: setup libs progs

# we can also enforce it by
.DEFAULT_GOAL=all

###------------------------------------------------------------------------###
# generate the name of the program and the library
LIBNAME  := $(patsubst %,$(LIBDIR)/lib%.a,$(PROGRAMS))
PROGNAME := $(patsubst %,$(EXEDIR)/%,$(PROGRAMS))
# this might be buggy, so don't use it
#ifeq ($(BUILD_IN_BIN),yes)
#   PROGNAME := $(patsubst %, $(HOME)/bin/$(NAME), $(PROGRAMS))
#endif # BIN
# everything should also dependent on the local Makefile
ifeq ($(MAKEFILE_DEP),yes)
   MAKEDEP := Makefile
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
# set up the targets for all
progs: $(PROGNAME)

libs: $(LIBNAME)

###------------------------------------------------------------------------###
# now the rules to actually start the submakefiles
$(EXEDIR)/%:
	$(MAKE) -C $(patsubst $(EXEDIR)/%,%,$(MAINDIR)/$@) prog

$(LIBDIR)/lib%.a:
	$(MAKE) -C $(patsubst $(LIBDIR)/lib%.a,%,$(MAINDIR)/$@) lib

###------------------------------------------------------------------------###
# clean up, at the right places
clean:
	$(RM) $(OUTDIR)/*.o $(MODDIR)/*.mod $(PROGNAME) $(LIBNAME)

# in case you use the dependency build you can get rid of those files to
veryclean: clean
	$(RM) $(DEPDIR)/*.d

