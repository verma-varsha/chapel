## Process this file with automake to produce Makefile.in	-*- makefile -*-
#   $Source: bitbucket.org:berkeleylab/gasnet.git/other/ammpi/Makefile.am $
# Description: Makefile for AMMPI/GASNet 
# Copyright 2004, Dan Bonachea <bonachea@cs.berkeley.edu>
# Terms of use are as specified in license.txt
# NOTE: This Makefile is used to tie in with the GASNet build infrastructure
#       the stand-alone AMMPI product doesn't currently provide an automake framework

AUTOMAKE_OPTIONS = foreign 1.4 no-dependencies

apputils_flags =

if BUILDCONFIG_DEBUG
  BUILDCONFIG_DEFINES = -DAMMPI_DEBUG=1
else
  BUILDCONFIG_DEFINES = -DAMMPI_NDEBUG=1
endif

if USE_STRICT_PROTOTYPES
  strict_proto_flags = -DGASNET_USE_STRICT_PROTOTYPES
else
  strict_proto_flags = 
endif

CC = @MPI_CC@
CFLAGS = @MPI_CFLAGS@ $(MANUAL_MPICFLAGS) $(apputils_flags) $(strict_proto_flags)
LDFLAGS = @LDFLAGS@ $(MANUAL_LDFLAGS)

PERL = @PERL@

platform_defines = @GASNET_EXTRA_DEFINES@ $(BUILDCONFIG_DEFINES) @INTTYPES_DEFINES@ -DAMMPI_DISABLE_AMTAGS $(MANUAL_DEFINES)

platform_libraries = @LIBS@ @MPI_LIBS@

VPATH = $(srcdir)
testdir = $(srcdir)/../amxtests
altincdir = $(srcdir)/..
INCLUDES = -I$(srcdir) -I. -I$(altincdir) $(platform_defines)
compile = $(CC) $(CFLAGS) $(INCLUDES)
link = $(CC) $(CFLAGS) $(LDFLAGS) $(INCLUDES) 
linkend = $(platform_libraries) $(MANUAL_LIBS)

if USE_AMMPI
BUILT_SOURCES = ammpi_portable_platform.h

lib_LIBRARIES = libammpi.a
endif
headers = 	     \
    ammpi.h          \
    ammpi_internal.h \
    ammpi_spmd.h

allheaders = $(BUILT_SOURCES) $(headers)

libammpi_a_SOURCES = \
    ammpi_ep.c       \
    ammpi_reqrep.c   \
    ammpi_spmd.c     

ammpi_portable_platform.h: $(altincdir)/gasnet_portable_platform.h
	cp -f $< $@

libammpi_a_DEPENDENCIES = $(libammpi_a_SOURCES:%.c=%.o)  

$(libammpi_a_DEPENDENCIES): $(allheaders)

EXTRA_DIST = \
        $(headers)              \
	Makefile.tests  	\
	Makefile.titanium.in	\
	license.txt

# ---------- test programs -------------
# all the test executables
testprograms =    		\
    testam       		\
    testbounce    		\
    testbulk      		\
    testlatency   		\
    testlatencyM   		\
    testping      		\
    testreduce			\
    testgetput    		\
    testreadwrite 

tests: apputils.o $(testprograms)

test%.o : $(testdir)/test%.c $(testdir)/*.h $(allheaders)
	$(compile) -DAMMPI -I$(testdir) -c $< -o $@

apputils.o : $(testdir)/apputils.c $(testdir)/apputils.h $(allheaders)
	$(compile) -DAMMPI -I$(testdir) -c $(testdir)/apputils.c -o apputils.o

test%: test%.o apputils.o libammpi.a 
	$(link) -o $@ $< apputils.o -L. -lammpi $(linkend)

tests-clean:
	rm -f $(testprograms)

AMMPI_RUNCMD='@MPIRUN_CMD@'
include Makefile.tests

%.o: %.c ammpi.h ammpi_spmd.h ammpi_internal.h
	$(compile) -DAMMPI -o $@ -c $<

if USE_AMMPI
if GNU_NM
check-exports: libammpi.a
	@echo Checking libammpi exports...
	@lib=libammpi.a ; \
	    if test x$(CHECK_EXPORTS) = x0; then                                    \
	      echo Skipped by user request ;                                        \
	      exit 0 ;                                                              \
	    fi ;                                                                    \
            @NM@ --defined-only $$lib |                                             \
	    @PERL@ -pe 's/ \.refptr\.//' |                                          \
            grep -v -e ' [\._]*ammpi_' -e ' [\._]*AMMPI_' -e ' [\._]*AM_'           \
                    -e ' [\._]*gasnett_' -e ' [\._]*gasneti_'                       \
                    -e ' [\._][\._]*debug_' -e 'get_pc_thunk'                       \
                    -e __FUNCTION__ -e __PRETTY_FUNCTION__ -e debug_info_seg |      \
            @PERL@ -n -e 'print if /^[0-9a-fA-F]+\s+[A-Z]\s+/' > .$$lib.exp;        \
          if test -s .$$lib.exp ; then                                              \
            cat .$$lib.exp ;                                                        \
            echo FAILED ;                                                           \
            failed=1 ;                                                              \
          else                                                                      \
            echo PASSED ;                                                           \
          fi ;                                                                      \
          rm -f .$$lib.exp ;                                                        \
          exit $$failed
else
check-exports: libammpi.a
	@echo check-exports test SKIPPED
endif

check: check-exports tests

clean_local_files = libammpi.a *.o core .test-results $(BUILT_SOURCES)
clean-local:
	@for f in $(testprograms); do                                \
	  cmd="rm -f '$$f$(EXESUFFIX)'"; echo "$$cmd"; eval "$$cmd" || exit $$? ; \
	 done;
else
clean_local_files =
clean-local:
	@: #empty rule
endif

CLEANFILES = $(clean_local_files)

force:

.PHONY: force tests run-tests

