all: test

TESTTHAT_REPORTER ?= 'check'

REVDEP := RMariaDB RSQLite RPostgres RKazam
LOAD_TARGETS := $(patsubst %,load-%,${REVDEP})
TEST_TARGETS := $(patsubst %,test-%,${REVDEP})

LIB_DIR := $(shell Rscript -e "cat(.libPaths()[1])")

${LIB_DIR}/DBItest/DESCRIPTION: ../R/*.R ../DESCRIPTION ${LOAD_TARGETS}
	R CMD INSTALL ..

install: ${LIB_DIR}/DBItest/DESCRIPTION

RKazam RSQLite RPostgres RMySQL RMariaDB:
	# Use local checkout if available, otherwise clone from rstats-db GitHub
	if [ -d ../../$@/$@-pkg ]; then ln -s $(realpath ../../$@/$@-pkg) $@; else if [ -d ../../$@ ]; then ln -s $(realpath ../../$@) .; else git clone https://github.com/rstats-db/$@.git; Rscript -e "devtools::install_deps('$@', dependencies = TRUE)"; fi; fi

bigrquery:
	# Use local checkout if available, otherwise clone from GitHub
	if [ -d ../../$@ ]; then ln -s $(realpath ../../$@) .; else git clone https://github.com/krlmlr/$@.git -b feature/dbi-test; Rscript -e "devtools::install_deps('$@', dependencies = TRUE)"; fi

test: ${TEST_TARGETS}

safe-load-%:
	Rscript -e "devtools::load_all('$*')"

# Necessary because of https://github.com/hadley/devtools/issues/1070
load-%: %
	make --quiet -j1 safe-load-$*

test-%: install load-% %
	Rscript -e "options(crayon.enabled=TRUE); devtools::test('$*', filter = 'DBItest', reporter = ${TESTTHAT_REPORTER})"

clean:
	rm -rf ${REVDEP}
