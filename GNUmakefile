.PHONY: all
all:
	autoreconf -fvi

.PHONY: distclean
distclean:
	rm -rf *-toolchain.mk config.log config.status

.PHONY: maintainer-clean
maintainer-clean: distclean
	rm -rf autom4te.cache configure *'~'
