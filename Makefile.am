#
# Copyright (C) 2010-2017 Red Hat, Inc.  All rights reserved.
#
# Author: Fabio M. Di Nitto <fabbione@kronosnet.org>
#
# This software licensed under GPL-2.0+, LGPL-2.0+
#

MAINTAINERCLEANFILES	= Makefile.in aclocal.m4 configure depcomp \
			  config.guess config.sub missing install-sh \
			  ltmain.sh compile config.h.in config.h.in~ \
			  autoscan.log configure.scan test-driver

include $(top_srcdir)/build-aux/check.mk

AUTOMAKE_OPTIONS	= foreign

AM_DISTCHECK_CONFIGURE_FLAGS = --enable-kronosnetd

ACLOCAL_AMFLAGS		= -I m4

SPEC			= $(PACKAGE_NAME).spec

DEBCHANGELOG		= debian/changelog
DEBPOSTINST		= debian/kronosnetd.postinst

TARGZFILE		= $(PACKAGE_NAME)-$(VERSION).tar.gz

EXTRA_DIST		= build-aux autogen.sh .version \
			  debian $(DEBCHANGELOG) $(DEBPOSTINST) \
			  $(SPEC) $(SPEC).in

SUBDIRS			= init libtap libknet kronosnetd

if BUILD_DOCS
SUBDIRS			+= docs
endif

if BUILD_POC
SUBDIRS			+= poc-code
endif

dist_doc_DATA		= \
			  COPYING.applications \
			  COPYING.libraries \
			  COPYRIGHT \
			  README.licence \
			  README \
			  TODO

all-local: $(SPEC) $(DEBCHANGELOG) $(DEBPOSTINST)

clean-local:
	rm -f $(SPEC)
	rm -f $(DEBPOSTINST)

distclean-local:
	rm -f $(PACKAGE_NAME)-*.tar.* $(PACKAGE_NAME)-*.sha256* tag-*

maintainer-clean-local:
	make -f debian/rules clean || true
	rm -rf m4 $(DEBCHANGELOG) $(DEBPOSTINST) debian/patches

## make rpm/srpm section.

$(SPEC): $(SPEC).in .version config.status
	rm -f $@-t $@
	date="$(shell LC_ALL=C date "+%a %b %d %Y")" && \
	if [ -f .tarball-version ]; then \
		gitver="$(shell cat .tarball-version)" && \
		rpmver=$$gitver && \
		alphatag="" && \
		dirty="" && \
		numcomm="0"; \
	else \
		gitver="$(shell GIT_DIR=$(abs_srcdir)/.git git describe --abbrev=4 --match='v*' HEAD 2>/dev/null)" && \
		rpmver=`echo $$gitver | sed -e "s/^v//" -e "s/-.*//g"` && \
		alphatag=`echo $$gitver | sed -e "s/.*-//" -e "s/^g//"` && \
		vtag=`echo $$gitver | sed -e "s/-.*//g"` && \
		numcomm=`GIT_DIR=$(abs_srcdir)/.git git rev-list $$vtag..HEAD | wc -l` && \
		GIT_DIR=$(abs_srcdir)/.git git update-index --refresh > /dev/null 2>&1 || true && \
		dirty=`GIT_DIR=$(abs_srcdir)/.git git diff-index --name-only HEAD 2>/dev/null`; \
	fi && \
	if [ -n "$$dirty" ]; then dirty="dirty"; else dirty=""; fi && \
	if [ "$$numcomm" = "0" ]; then \
		sed \
			-e "s#@version@#$$rpmver#g" \
			-e "s#%glo.*alpha.*##g" \
			-e "s#%glo.*numcomm.*##g" \
			-e "s#@dirty@#$$dirty#g" \
			-e "s#@date@#$$date#g" \
		$< > $@-t; \
	else \
		sed \
			-e "s#@version@#$$rpmver#g" \
			-e "s#@alphatag@#$$alphatag#g" \
			-e "s#@numcomm@#$$numcomm#g" \
			-e "s#@dirty@#$$dirty#g" \
			-e "s#@date@#$$date#g" \
		$< > $@-t; \
	fi; \
	if [ -z "$$dirty" ]; then sed -i -e "s#%glo.*dirty.*##g" $@-t; fi
if BUILDSCTP
	sed -i -e "s#@sctp@#bcond_without#g" $@-t
else
	sed -i -e "s#@sctp@#bcond_with#g" $@-t
endif
if BUILDCRYPTONSS
	sed -i -e "s#@nss@#bcond_without#g" $@-t
else
	sed -i -e "s#@nss@#bcond_with#g" $@-t
endif
if BUILDCRYPTOOPENSSL
	sed -i -e "s#@openssl@#bcond_without#g" $@-t
else
	sed -i -e "s#@openssl@#bcond_with#g" $@-t
endif
if BUILDCOMPZLIB
	sed -i -e "s#@zlib@#bcond_without#g" $@-t
else
	sed -i -e "s#@zlib@#bcond_with#g" $@-t
endif
if BUILDCOMPLZ4
	sed -i -e "s#@lz4@#bcond_without#g" $@-t
else
	sed -i -e "s#@lz4@#bcond_with#g" $@-t
endif
if BUILDCOMPLZO2
	sed -i -e "s#@lzo2@#bcond_without#g" $@-t
else
	sed -i -e "s#@lzo2@#bcond_with#g" $@-t
endif
if BUILDCOMPLZMA
	sed -i -e "s#@lzma@#bcond_without#g" $@-t
else
	sed -i -e "s#@lzma@#bcond_with#g" $@-t
endif
if BUILDCOMPBZIP2
	sed -i -e "s#@bzip2@#bcond_without#g" $@-t
else
	sed -i -e "s#@bzip2@#bcond_with#g" $@-t
endif
if BUILD_KRONOSNETD
	sed -i -e "s#@kronosnetd@#bcond_without#g" $@-t
else
	sed -i -e "s#@kronosnetd@#bcond_with#g" $@-t
endif
if BUILD_LIBTAP
	sed -i -e "s#@libtap@#bcond_without#g" $@-t
else
	sed -i -e "s#@libtap@#bcond_with#g" $@-t
endif
	sed -i -e "s#@defaultadmgroup@#$(DEFAULTADMGROUP)#g" $@-t
	chmod a-w $@-t
	mv $@-t $@

$(TARGZFILE):
	$(MAKE) dist

RPMBUILDOPTS	= --define "_sourcedir $(abs_builddir)" \
		  --define "_specdir $(abs_builddir)" \
		  --define "_builddir $(abs_builddir)" \
		  --define "_srcrpmdir $(abs_builddir)" \
		  --define "_rpmdir $(abs_builddir)"

srpm: clean
	$(MAKE) $(SPEC) $(TARGZFILE)
	rpmbuild $(RPMBUILDOPTS) --nodeps -bs $(SPEC)

rpm: clean
	$(MAKE) $(SPEC) $(TARGZFILE)
	rpmbuild $(RPMBUILDOPTS) -ba $(SPEC)

## make deb

$(DEBPOSTINST): $(DEBPOSTINST).in config.h
	mkdir -p debian
	rm -f $@-t $@
	sed -e "s#@defaultadmgroup@#$(DEFAULTADMGROUP)#g" $< > $@-t
	chmod a-w $@-t
	mv $@-t $@

$(DEBCHANGELOG): $(DEBCHANGELOG).in .version
	mkdir -p debian
	rm -f $@-t $@
	date="$(shell LC_ALL=C date -R)" && \
	if [ -f .tarball-version ]; then \
		gitver="$(shell cat .tarball-version)" && \
		debver=$$gitver && \
		alphatag="" && \
		dirty="" && \
		numcomm="0"; \
	else \
		gitver="$(shell GIT_DIR=$(abs_srcdir)/.git git describe --abbrev=4 --match='v*' HEAD 2>/dev/null)" && \
		debver=`echo $$gitver | sed -e "s/^v//" -e "s/-.*//g"` && \
		alphatag=`echo $$gitver | sed -e "s/.*-//" -e "s/^g//"` && \
		vtag=`echo $$gitver | sed -e "s/-.*//g"` && \
		numcomm=`GIT_DIR=$(abs_srcdir)/.git git rev-list $$vtag..HEAD | wc -l` && \
		GIT_DIR=$(abs_srcdir)/.git git update-index --refresh > /dev/null 2>&1 || true && \
		dirty=`GIT_DIR=$(abs_srcdir)/.git git diff-index --name-only HEAD 2>/dev/null`; \
	fi && \
	if [ -n "$$dirty" ]; then dirty="-dirty"; else dirty=""; fi && \
	if [ "$$numcomm" = "0" ]; then \
		sed \
			-e "s#@version@#$$debver#g" \
			-e "s#-@alphatag@##g" \
			-e "s#.@numcomm@##g" \
			-e "s#@dirty@#$$dirty#g" \
			-e "s#@date@#$$date#g" \
		$< > $@-t; \
	else \
		sed \
			-e "s#@version@#$$debver#g" \
			-e "s#@alphatag@#$$alphatag#g" \
			-e "s#@numcomm@#$$numcomm#g" \
			-e "s#@dirty@#$$dirty#g" \
			-e "s#@date@#$$date#g" \
		$< > $@-t; \
	fi;
	chmod a-w $@-t
	mv $@-t $@

deb-clean:
	make -f debian/rules clean
	./configure

debs: deb-clean dist
	if [ -f .tarball-version ]; then \
		gitver="$(shell cat .tarball-version)"; \
	else \
		gitver="$(shell git describe --abbrev=4 --match='v*' HEAD 2>/dev/null)" && \
		gitver=`echo $$gitver | sed -e "s/^v//" -e "s/-g/-/" -e "s/-/./"`; \
	fi && \
	cp $(TARGZFILE) ../$(PACKAGE_NAME)_$$gitver.orig.tar.gz
	dpkg-buildpackage -uc -us

# release/versioning
BUILT_SOURCES	= .version
.version:
	echo $(VERSION) > $@-t && mv $@-t $@

dist-hook: gen-ChangeLog
	echo $(VERSION) > $(distdir)/.tarball-version

gen_start_date = 2000-01-01
.PHONY: gen-ChangeLog
gen-ChangeLog:
	if test -d .git; then						\
		LC_ALL=C $(top_srcdir)/build-aux/gitlog-to-changelog		\
			--since=$(gen_start_date) > $(distdir)/cl-t;	\
		rm -f $(distdir)/ChangeLog;				\
		mv $(distdir)/cl-t $(distdir)/ChangeLog;		\
	fi
