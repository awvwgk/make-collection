#!/bin/env make
# 
# April 2018 (v 2.3) created by Sebastian Ehlert
# 2.0: recreated from scratch                                          (1803)
# 2.1: refined the generation of dependencies, since it crashed if not
#      done in the main directory                                      (1803)
# 2.2: modified the clean command, to be less severe                   (1804)
# 2.3: added support for GNU compilers, some minor improvements        (1804)
#
##############################################################################
## this is a generic minimal sublevel Makefile                              ##
##############################################################################

###------------------------------------------------------------------------###
# specifiy where you are in the source tree
MAINDIR := ..
# here resides the build system
MAKEDIR := $(MAINDIR)/MAKE

# set the source by hand
SRCDIR  := .

##############################################################################
## please edit this Files only:
include $(MAKEDIR)/NAME.objs
include $(MAKEDIR)/Makeconfig
## here you can give optional configurations independent from Makeconfig
-include $(MAKEDIR)/NAME.config
##############################################################################

include $(MAKEDIR)/Makerules

