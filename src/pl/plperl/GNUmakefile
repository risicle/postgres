# Makefile for PL/Perl
# $PostgreSQL: pgsql/src/pl/plperl/GNUmakefile,v 1.37.2.1 2010/05/13 16:40:36 adunstan Exp $

subdir = src/pl/plperl
top_builddir = ../../..
include $(top_builddir)/src/Makefile.global

ifeq ($(perl_useshrplib),true)
shared_libperl = yes
endif
ifeq ($(perl_useshrplib),yes)
shared_libperl = yes
endif

# If we don't have a shared library and the platform doesn't allow it
# to work without, we have to skip it.
ifneq (,$(findstring yes, $(shared_libperl)$(allow_nonpic_in_shlib)))

ifeq ($(PORTNAME), win32)
perl_archlibexp := $(subst \,/,$(perl_archlibexp))
perl_privlibexp := $(subst \,/,$(perl_privlibexp))
perl_lib := $(basename $(notdir $(wildcard $(perl_archlibexp)/CORE/perl[5-9]*.lib)))
perl_embed_ldflags = -L$(perl_archlibexp)/CORE -l$(perl_lib)
override CPPFLAGS += -DPLPERL_HAVE_UID_GID
# Perl on win32 contains /* within comment all over the header file,
# so disable this warning.
override CFLAGS += -Wno-comment
endif

override CPPFLAGS := -I. -I$(srcdir) $(CPPFLAGS) -I$(perl_archlibexp)/CORE

rpathdir = $(perl_archlibexp)/CORE


NAME = plperl

OBJS = plperl.o spi_internal.o SPI.o

SHLIB_LINK = $(perl_embed_ldflags)

REGRESS_OPTS = --dbname=$(PL_TESTDB) --load-language=plperl  --load-language=plperlu
REGRESS = plperl plperl_trigger plperl_shared plperl_elog
# if Perl can support two interpreters in one backend, 
# test plperl-and-plperlu cases
ifneq ($(PERL),)
ifeq ($(shell $(PERL) -V:usemultiplicity), usemultiplicity='define';)
	REGRESS += plperlu_plperl
endif
endif
# where to find psql for running the tests
PSQLDIR = $(bindir)

include $(top_srcdir)/src/Makefile.shlib


all: all-lib

plperl.o: plperl_opmask.h

plperl_opmask.h: plperl_opmask.pl
	$(PERL) $< $@


SPI.c: SPI.xs
	$(PERL) $(perl_privlibexp)/ExtUtils/xsubpp -typemap $(perl_privlibexp)/ExtUtils/typemap $< >$@

install: all installdirs install-lib

installdirs: installdirs-lib

uninstall: uninstall-lib

installcheck: submake
	$(top_builddir)/src/test/regress/pg_regress --inputdir=$(srcdir) --psqldir="$(PSQLDIR)" $(REGRESS_OPTS) $(REGRESS)

.PHONY: submake
submake:
	$(MAKE) -C $(top_builddir)/src/test/regress pg_regress$(X)

clean distclean maintainer-clean: clean-lib
	rm -f SPI.c $(OBJS) plperl_opmask.h
	rm -rf results
	rm -f regression.diffs regression.out

else # can't build

all:
	@echo ""; \
	 echo "*** Cannot build PL/Perl because libperl is not a shared library."; \
	 echo "*** You might have to rebuild your Perl installation.  Refer to"; \
	 echo "*** the documentation for details."; \
	 echo ""

endif # can't build
