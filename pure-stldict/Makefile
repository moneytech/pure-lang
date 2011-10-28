
modname = stldict
version = 0.0

# platform-specific setup

DLL         = $(shell pkg-config pure --variable DLL)
PIC         = $(shell pkg-config pure --variable PIC)
shared      = $(shell pkg-config pure --variable shared)

libdir      = $(shell pkg-config pure --variable libdir)
installdir  = $(addprefix $(DESTDIR), $(libdir)/pure)

pure-stlvec = ../pure-stlvec

MOD_CXXFLAGS = $(PIC) $(shell pkg-config pure --cflags) -I$(pure-stlvec) $(CFLAGS) $(CPPFLAGS)
MOD_LDFLAGS = $(shell pkg-config pure --libs) $(LDFLAGS)

CFLAGS = -g -O2

# Basic rules to build the module, clean, install and uninstall.

all: $(modname)$(DLL)

$(modname)$(DLL): $(modname).cpp $(modname).hpp
	$(CXX) $(shared) -o $@ $(MOD_CXXFLAGS) $(filter %.cpp,$^)  $(MOD_LDFLAGS) $(LIBS)

clean:
	rm -f $(modname)$(DLL)

install: $(modname).pure $(modname)$(DLL)
	test -d $(installdir) || mkdir -p $(installdir)
	cp $^ $(installdir)

uninstall:
	rm -f $(installdir)/$(modname).pure $(installdir)/$(modname)$(DLL)

# Roll a distribution tarball.

DISTFILES = README Makefile $(modname).cpp $(modname).hpp *.pure
#SEDFILES = README

dist = pure-$(modname)-$(version)

date = $(shell date "+%B %-d, %Y")
datesubst = sed -e "s?@version@?$(version)?g" -e "s?|today|?$(date)?g" < $(1) > $(2)

dist: $(DISTFILES)
	rm -rf $(dist)
	mkdir $(dist)
	for x in $(DISTFILES); do ln -sf $$PWD/$$x $(dist)/$$x; done
	for x in $(SEDFILES); do rm -f $(dist)/$$x; $(call datesubst,$$PWD/$$x,$(dist)/$$x); done
	rm -f $(dist).tar.gz
	tar cfzh $(dist).tar.gz $(dist)
	rm -rf $(dist)

distcheck: dist
	tar xfz $(dist).tar.gz
	cd $(dist) && make && make install DESTDIR=./BUILD
	rm -rf $(dist)