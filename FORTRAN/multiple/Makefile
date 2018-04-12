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
.PHONY: all main libraries programs setup depend
.PHONY: $(PROGRAMS)

# this is the first, so it would be made by default
all: setup
all: libraries
all: programs

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
   MAKEDEP := $(filter-out %.d, $(MAKEFILE_LIST))
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
programs: $(PROGNAME)

libraries: $(LIBNAME)

###------------------------------------------------------------------------###
# now the rules to actually start the submakefiles
$(EXEDIR)/%:
	$(MAKE) -C $(patsubst $(EXEDIR)/%,%,$(MAINDIR)/$@) program

$(LIBDIR)/lib%.a:
	$(MAKE) -C $(patsubst $(LIBDIR)/lib%.a,%,$(MAINDIR)/$@) library

###------------------------------------------------------------------------###
# clean up, at the right places
.PHONY: clean distclean veryclean

clean:
	$(RM) $(OUTDIR)/*.o

distclean:
	$(RM) $(OUTDIR)/*.o $(MODDIR)/*.mod $(PROGNAME) $(LIBNAME)

# in case you use the dependency build you can get rid of those files to
veryclean: distclean
	$(RM) $(DEPDIR)/*.d

