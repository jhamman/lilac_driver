#===============================================================================
# Common Makefile: a framework for building all CESM components and more
#
# Command-line variables
#   MODEL=<model>  ~ a standard macro definition, often found in the included 
#                    MACFILE, used to trigger special compilation flags
# Supported compilers
# ibm, bgl, bgp, pgi, intel, pathscale, gnu, nag
#===============================================================================

# Set up special characters
null  :=
comma := ,
EXEC_NAME = clmdrv.exe
ifndef BLDDIR
  BLDDIR := $(CURDIR)
endif

# Load dependency search path.
dirs := . $(shell cat Filepath)
cpp_dirs := $(dirs)

# Expand any tildes in directory names. Change spaces to colons.
VPATH := $(foreach dir,$(cpp_dirs),$(wildcard $(dir))) 
VPATH := $(subst $(space),:,$(VPATH))               

RM    := rm
CP    := cp

exec: $(EXEC_NAME)  $(BLDDIR)/Depends
complib: $(COMPLIB)  $(BLDDIR)/Depends

ifndef MOD_SUFFIX
   MOD_SUFFIX := mod
endif

#===============================================================================
# set CPP options (must use this before any flags or cflags settings)
#===============================================================================

CPPDEFS := $(USER_CPPDEFS) -D$(OS) 

include $(BLDDIR)/Macros

FPPDEFS := $(CPPDEFS)

ifndef AR
   AR := ar
endif

ifdef NETCDF_PATH
  ifndef INC_NETCDF
    INC_NETCDF:=$(NETCDF_PATH)/include
  endif
  ifndef LIB_NETCDF
    LIB_NETCDF:=$(NETCDF_PATH)/lib
  endif
endif
ifdef PNETCDF_PATH
  ifndef $(INC_PNETCDF)
    INC_PNETCDF:=$(PNETCDF_PATH)/include
  endif
  ifndef LIB_PNETCDF
    LIB_PNETCDF:=$(PNETCDF_PATH)/lib
  endif
endif

#===============================================================================
# Set config args for pio and mct to blank and then enable serial 
#===============================================================================
ifndef CONFIG_ARGS
  CONFIG_ARGS := 
endif
ifeq ($(MODEL),pio)
  CONFIG_ARGS+= --enable-timing
  ifeq ($DEBUG,TRUE)
     CONFIG_ARGS+= --enable-debug
  endif
endif

#===============================================================================
# User-specified INCLDIR
#===============================================================================

INCLDIR := -I. 

#===============================================================================
# Compiler commands
#===============================================================================

CC  := $(MPICC)
FC  := $(MPIFC)
LD  := $(MPIFC)
ifdef MPI_PATH
  INC_MPI := $(MPI_PATH)/include
  LIB_MPI := $(MPI_PATH)/lib
endif

#===============================================================================
# Set include paths (needed after override for any model specific builds below)
#===============================================================================
INCLDIR += -I$(BLDDIR)/include 

ifdef INC_NETCDF
  INCLDIR += -I$(INC_NETCDF)
endif
ifdef MOD_NETCDF
  INCLDIR += -I$(MOD_NETCDF)
endif
ifdef INC_MPI
  INCLDIR += -I$(INC_MPI)
endif 
ifdef INC_PNETCDF
  INCLDIR += -I$(INC_PNETCDF)
endif

ifndef MCT_LIBDIR
  MCT_LIBDIR=$(BLDDIR)
endif

ifndef PIO_LIBDIR
  PIO_LIBDIR=$(BLDDIR)
endif

ifndef GPTL_LIBDIR
  GPTL_LIBDIR=$(BLDDIR)
endif

#INCLDIR +=	-I$(BLDDIR)/include -I$(CIMEROOT)/share/csm_share/shr \
#		-I$(CIMEROOT)/share/csm_share/include
#
# Use the MCT dir for the cache for all configure calls because it is the first one 
#
CFLAGS+=$(CPPDEFS)

CONFIG_ARGS +=  CC="$(SCC)" FC="$(SFC)" MPICC="$(MPICC)" \
                MPIFC="$(MPIFC)" FCFLAGS="$(FFLAGS) $(FREEFLAGS) $(INCLDIR)" \
                CPPDEFS="$(CPPDEFS)" CFLAGS="$(CFLAGS) -I.. $(INCLDIR)" \
                NETCDF_PATH=$(NETCDF_PATH) LDFLAGS="$(LDFLAGS)" 

FFLAGS += $(FPPDEFS)
FFLAGS_NOOPT += $(FPPDEFS)

# System libraries (netcdf, mpi, pnetcdf, esmf, trilinos, etc.) 
ifndef SLIBS
   SLIBS := -L$(LIB_NETCDF) -lnetcdf 
endif
ifdef LIB_PNETCDF
   SLIBS += -L$(LIB_PNETCDF) -lpnetcdf
endif
ifdef LAPACK_LIBDIR
   SLIBS += -L$(LAPACK_LIBDIR) -llapack -lblas
endif
ifdef LIB_MPI
   ifndef MPI_LIB_NAME
      SLIBS += -L$(LIB_MPI) -lmpi
   else  
      SLIBS += -L$(LIB_MPI) -l$(MPI_LIB_NAME)
   endif
endif

#------------------------------------------------------------------------------
# Drive configure scripts for support libraries (mct)
#------------------------------------------------------------------------------


$(MCT_LIBDIR)/Makefile.conf: 
	@echo "SHAREDLIBROOT |$(SHAREDLIBROOT)| BLDDIR |$(BLDDIR)|"; \
	$(CONFIG_SHELL) $(MCT_DIR)/configure $(CONFIG_ARGS) --srcdir $(MCT_DIR)

ifneq ("$(wildcard $(PIO_DIR)/src)", "")
# This is a pio2 library
  PIOLIB = $(PIO_LIBDIR)/libpiof.a $(PIO_LIBDIR)/libpioc.a
  PIOLIBNAME = -lpiof -lpioc
  PIO_SRC_DIR = $(PIO_DIR)
else
# This is a pio1 library
  PIOLIB = $(PIO_LIBDIR)/libpio.a
  PIOLIBNAME = -lpio
  ifneq ("$(wildcard $(PIO_DIR)/pio)", "")
    PIO_SRC_DIR = $(PIO_DIR)
  else
    PIO_SRC_DIR = $(PIO_DIR)/pio
  endif
endif

GPTLLIB = $(BLDDIR)/libgptl.a

ULIBS += -L. -lclm -lshare $(PIOLIBNAME) -lgptl -lnetcdf -mkl=cluster

#------------------------------------------------------------------------------
# Drive cmake script for cism and pio
#------------------------------------------------------------------------------

ifndef CMAKE_OPTS
  CMAKE_OPTS := 
endif
# note that the fortran flags include neither the FREEFLAGS nor the
# FIXEDFLAGS, so that both free & fixed code can be built (cmake
# doesn't seem to be able to differentiate between free & fixed
# fortran flags)
CMAKE_OPTS += -D CMAKE_Fortran_FLAGS:STRING="$(FFLAGS) $(INCLDIR)" \
              -D CMAKE_C_FLAGS:STRING="$(CFLAGS) $(INCLDIR)" \
              -D CMAKE_VERBOSE_MAKEFILE:BOOL=ON \
              -D NETCDF_DIR:STRING=$(NETCDF_PATH) \
              -D USER_CMAKE_MODULE_DIR:STRING=$(CIMEROOT)/externals/CMake

ifdef PNETCDF_PATH
	CMAKE_OPTS += -D PNETCDF_DIR:STRING="$(PNETCDF_PATH)" 
else
        CMAKE_OPTS += -D WITH_PNETCDF:LOGICAL=FALSE
endif
ifdef PIO_FILESYSTEM_HINTS
	CMAKE_OPTS += -D PIO_FILESYSTEM_HINTS:STRING="$(PIO_FILESYSTEM_HINTS)"
endif

# This captures the many cism-specific options to cmake
CMAKE_OPTS += $(USER_CMAKE_OPTS)

# CMake doesn't seem to like it when you define compilers via -D
# CMAKE_C_COMPILER, etc., when you rerun cmake with an existing
# cache. So doing this via environment variables instead.
ifndef CMAKE_ENV_VARS
  CMAKE_ENV_VARS := 
endif
CMAKE_ENV_VARS += CC=$(CC) \
                  FC=$(FC) \
                  LDFLAGS="$(LDFLAGS)" 

$(PIO_LIBDIR)/Makefile: 
	cd $(PIO_LIBDIR); \
	$(CMAKE_ENV_VARS) cmake $(CMAKE_OPTS) $(PIO_SRC_DIR)

#-------------------------------------------------------------------------------
# Build & include dependency files
#-------------------------------------------------------------------------------

touch_filepath: 
	touch $(BLDDIR)/Filepath

# Get list of files and build dependency file for all .o files
#   using perl scripts mkSrcfiles and mkDepends
# if a source is of form .F90.in strip the .in before creating the list of objects
SOURCES := $(shell cat Srcfiles)
BASENAMES := $(basename $(basename $(SOURCES)))
OBJS    := $(addsuffix .o, $(BASENAMES))
INCS    := $(foreach dir,$(cpp_dirs),-I$(dir)) 

CURDIR := $(shell pwd)

$(BLDDIR)/Depends: $(BLDDIR)/Srcfiles $(BLDDIR)/Deppath
	$(BLDDIR)/mkDepends $(USER_MKDEPENDS_OPTS) Deppath Srcfiles > $@

$(BLDDIR)/Deppath: $(BLDDIR)/Filepath
	$(CP) -f $(BLDDIR)/Filepath $@

$(BLDDIR)/Srcfiles: $(BLDDIR)/Filepath
	$(BLDDIR)/mkSrcfiles 

$(BLDDIR)/Filepath:
	@echo "$(VPATH)" > $@


#-------------------------------------------------------------------------------
# echo file names, paths, compile flags, etc. used during build
#-------------------------------------------------------------------------------

db_files:
	@echo " "
	@echo "* MACFILE := $(MACFILE)"
	@echo "* VPATH   := $(VPATH)"
	@echo "* INCS    := $(INCS)"
	@echo "* OBJS    := $(OBJS)"
db_flags:
	@echo " "
	@echo "* cc      := $(CC)  $(CFLAGS) $(INCS) $(INCLDIR)"
	@echo "* .F.o    := $(FC)  $(FFLAGS) $(FIXEDFLAGS) $(INCS) $(INCLDIR)"
	@echo "* .F90.o  := $(FC)  $(FFLAGS) $(FREEFLAGS) $(INCS) $(INCLDIR)"

#-------------------------------------------------------------------------------
# Rules used for the tests run by "configure -test"
#-------------------------------------------------------------------------------

test_fc: test_fc.o
	$(LD) -o $@ test_fc.o $(LDFLAGS)
test_nc: test_nc.o
	$(LD) -o $@ test_nc.o -L$(LIB_NETCDF) -lnetcdf $(LDFLAGS)
test_mpi: test_mpi.o
	$(LD) -o $@ test_mpi.o $(LDFLAGS)
test_esmf: test_esmf.o
	$(LD) -o $@ test_esmf.o $(LDFLAGS)

# libcsm_share.a is in ULIBDEP, but -lcsm_share is in ULIBS rather than CLIBS,
# so this needs to be added after creating CLIBS above
CSMSHARELIB = $(BLDDIR)/$(COMP_INTERFACE)/$(ESMFDIR)/$(NINST_VALUE)/csm_share/libcsm_share.a
ULIBDEP += $(CSMSHARELIB)

#-------------------------------------------------------------------------------
# build rules: 
#-------------------------------------------------------------------------------

.SUFFIXES:
.SUFFIXES: .F90 .F .f90 .f .c .cpp .o .in

$(PIOLIB) : $(MPISERIAL) $(GPTLLIB)

$(CSMSHARELIB):  $(PIOLIB) $(GPTLLIB) 
ifneq ($(MODEL),csm_share)
  $(OBJS):  $(CSMSHARELIB)
endif

$(EXEC_NAME): $(OBJS) $(ULIBDEP) $(CSMSHARELIB) $(PIOLIB) $(GPTLLIB)
	$(LD) -o $(EXEC_NAME) $(OBJS) $(ULIBS) $(LDFLAGS)

$(COMPLIB): $(OBJS) 
	$(AR) -r $(COMPLIB) $(OBJS)

.c.o:
	$(CC) -c $(INCLDIR) $(INCS) $(CFLAGS)  $<
.F.o:
	$(FC) -c $(INCLDIR) $(INCS) $(FFLAGS) $(FIXEDFLAGS) $<
.f.o:
	$(FC) -c $(INCLDIR) $(INCS) $(FFLAGS) $(FIXEDFLAGS) $<
.f90.o:
	$(FC) -c $(INCLDIR) $(INCS) $(FFLAGS) $(FREEFLAGS)  $<
.F90.o:
	$(FC) -c $(INCLDIR) $(INCS) $(FFLAGS) $(FREEFLAGS) $(CONTIGUOUS_FLAG) $<

%.F90: %.F90.in
	$(BLDDIR)/genf90.pl $< > $@


mostlyclean:
	$(RM) -f *.f *.f90 

clean: mostlyclean
	$(RM) -f *.d *.$(MOD_SUFFIX) $(OBJS) Srcfiles Filepath Deppath Depends

realclean: clean
	$(RM) -f $(EXEC_NAME)

# the if-tests prevent DEPS files from being created when they're not needed
ifneq ($(MAKECMDGOALS), db_files)
ifneq ($(MAKECMDGOALS), db_flags)
ifneq ($(MAKECMDGOALS), mostlyclean)
ifneq ($(MAKECMDGOALS), clean)
ifneq ($(MAKECMDGOALS), realclean)
    -include $(BLDDIR)/Depends $(BLDDIR)/Depends.$(COMPILER) $(BLDDIR)/Depends.$(MACH)
endif
endif
endif
endif
endif
