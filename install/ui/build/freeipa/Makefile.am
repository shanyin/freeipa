NULL =

appdir = $(IPA_DATA_DIR)/ui/js/freeipa
app_DATA =				\
	app.js 				\
	core.js 			\
	$(NULL)

EXTRA_DIST =                            \
        $(app_DATA)                     \
        $(NULL)

MAINTAINERCLEANFILES =                  \
        *~                              \
        Makefile.in

core := $(wildcard ../../src/freeipa/*.js)
base := $(wildcard ../../src/freeipa/_base/*.js)
widgets := $(wildcard ../../src/freeipa/widgets/*.js)
nav := $(wildcard ../../src/freeipa/navigation/*.js)

app.js: $(core) $(base) $(widgets) $(nav)
	../../util/make-ui.sh

core.js: app.js
