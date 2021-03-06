# VERSION numbers
POSTGIS_VERSION=2.1.1
POSTGIS_MAJOR_VERSION=2.1
POSTGRES_VERSION=9.3.4
POSTGRES_MAJOR_VERSION=9.3
GDAL_VERSION=1.10.0
GEOS_VERSION=3.4.2
JPEG_VERSION=8d
LIBEDIT_VERSION=20130611-3.1
LIBJASPER_VERSION=1.900.1
LIBUUID_VERSION=1.6.2
LIBXML2_VERSION=2.9.1
OPENSSL_VERSION=1.0.1f
PROJ_VERSION=4.8.0
DATUMGRID_VERSION=1.5
V8_VERSION=3.18
JSONC_VERSION=0.11
LIBTIFF_VERSION=4.0.3

#path configuration
PREFIX=/Applications/Postgres.app/Contents/Versions/$(POSTGRES_MAJOR_VERSION)
PATH=$(PREFIX)/bin:/bin:/usr/bin:/opt/local/bin

#compiler options
MACOSX_DEPLOYMENT_TARGET=10.7
CFLAGS:=$(CFLAGS) -mmacosx-version-min=10.7
CXXFLAGS:=$(CFLAGS) -mmacosx-version-min=10.7

export CFLAGS CXXFLAGS MACOSX_DEPLOYMENT_TARGET

# commands used for downloading and extracting sources
CURL=/usr/bin/curl -L10 --silent --show-error --remote-name
TAR=/usr/bin/tar xzf

all: postgresql postgis plv8
clean: clean-postgresql clean-libuuid clean-openssl clean-libxml2 clean-libgdal clean-libproj clean-postgis clean-plv8 clean-libjpeg clean-libjasper clean-libgeos clean-libtiff

#########################
###### PostgreSQL #######
#########################

postgresql: $(PREFIX)/bin/psql

$(PREFIX)/bin/psql: postgresql-$(POSTGRES_VERSION)/GNUmakefile
	make -C "postgresql-$(POSTGRES_VERSION)" world
	make -C "postgresql-$(POSTGRES_VERSION)" install-world
	
# setting PATH is to make sure we find the right xml2-config
# the --with-includes and --with-libraries options are necessary so
# that postgres will be compiled and linked against our own versions
# of libraries like openssl, instead of system provided versions
postgresql-$(POSTGRES_VERSION)/GNUmakefile: $(PREFIX)/lib/libssl.dylib $(PREFIX)/lib/libxml2.dylib ${PREFIX}/include/uuid.h postgresql-$(POSTGRES_VERSION)/configure 
	cd "postgresql-$(POSTGRES_VERSION)" && export PATH="$(PREFIX)/bin:$$PATH" && ./configure --prefix="$(PREFIX)" --with-includes="$(PREFIX)/include" --with-libraries="$(PREFIX)/lib" --enable-thread-safety --with-openssl --with-gssapi --with-bonjour --with-krb5 --with-libxml --with-libxslt --with-perl --with-python --with-ossp-uuid --with-readline

# after extracting PostgreSQL sources, we must patch uuid-ossp so it builds on OSX
postgresql-$(POSTGRES_VERSION)/configure: postgresql-$(POSTGRES_VERSION).tar.bz2
	$(TAR) "postgresql-$(POSTGRES_VERSION).tar.bz2"
	echo '#define _XOPEN_SOURCE' >"postgresql-$(POSTGRES_VERSION)/contrib/uuid-ossp/uuid-ossp.c.patched"
	cat "postgresql-$(POSTGRES_VERSION)/contrib/uuid-ossp/uuid-ossp.c" >>"postgresql-$(POSTGRES_VERSION)/contrib/uuid-ossp/uuid-ossp.c.patched"
	mv "postgresql-$(POSTGRES_VERSION)/contrib/uuid-ossp/uuid-ossp.c.patched" "postgresql-$(POSTGRES_VERSION)/contrib/uuid-ossp/uuid-ossp.c"
	touch $@

postgresql-$(POSTGRES_VERSION).tar.bz2:	
	$(CURL) "http://ftp.postgresql.org/pub/source/v$(POSTGRES_VERSION)/postgresql-$(POSTGRES_VERSION).tar.bz2"
	
clean-postgresql:
	rm -Rf postgresql-$(POSTGRES_VERSION)

#########################
####### libuuid #########
#########################

uuid-${LIBUUID_VERSION}.tar.gz:
	$(CURL) "http://www.mirrorservice.org/sites/ftp.ossp.org/pkg/lib/uuid/uuid-${LIBUUID_VERSION}.tar.gz"

uuid-${LIBUUID_VERSION}: uuid-${LIBUUID_VERSION}.tar.gz
	$(TAR) "uuid-${LIBUUID_VERSION}.tar.gz"
	touch "uuid-${LIBUUID_VERSION}"

${PREFIX}/include/uuid.h: uuid-${LIBUUID_VERSION}
	cd uuid-${LIBUUID_VERSION} && ./configure --prefix="${PREFIX}" --disable-debug
	make -C uuid-${LIBUUID_VERSION}
	make -C uuid-${LIBUUID_VERSION} install
	touch ${PREFIX}/include/uuid.h
	
libuuid: ${PREFIX}/include/uuid.h

clean-libuuid:
	rm -Rf "uuid-${LIBUUID_VERSION}"
	
	
#########################
####### OpenSSL #########
#########################

openssl: $(PREFIX)/lib/libssl.dylib

$(PREFIX)/lib/libssl.dylib: openssl-${OPENSSL_VERSION}/Makefile
	make -C openssl-${OPENSSL_VERSION}
	make -C openssl-${OPENSSL_VERSION} install_sw

openssl-${OPENSSL_VERSION}/Makefile: openssl-${OPENSSL_VERSION}/Configure
	cd openssl-${OPENSSL_VERSION} && ./Configure --prefix="${PREFIX}" darwin64-x86_64-cc zlib no-asm no-krb5 shared

openssl-${OPENSSL_VERSION}/Configure: openssl-${OPENSSL_VERSION}.tar.gz
	$(TAR) openssl-${OPENSSL_VERSION}.tar.gz
	touch $@

openssl-${OPENSSL_VERSION}.tar.gz:
	$(CURL) "http://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz"
	
clean-openssl:
	rm -Rf "openssl-${OPENSSL_VERSION}"

#########################
######## LibXML2 ########
#########################

libxml2: $(PREFIX)/lib/libxml2.dylib

$(PREFIX)/lib/libxml2.dylib: libxml2-${LIBXML2_VERSION}/Makefile
	make -C libxml2-${LIBXML2_VERSION} install-exec
	make -C libxml2-${LIBXML2_VERSION}/include install
	touch $(PREFIX)/lib/libxml2.dylib

libxml2-${LIBXML2_VERSION}/Makefile: libxml2-${LIBXML2_VERSION}/configure
	cd libxml2-${LIBXML2_VERSION} && ./configure --prefix="$(PREFIX)" --disable-dependency-tracking

libxml2-${LIBXML2_VERSION}/configure: libxml2-${LIBXML2_VERSION}.tar.gz
	$(TAR) libxml2-${LIBXML2_VERSION}.tar.gz
	touch $@
	
libxml2-${LIBXML2_VERSION}.tar.gz:
	$(CURL) "ftp://xmlsoft.org/libxml2/libxml2-${LIBXML2_VERSION}.tar.gz"
	
clean-libxml2:
	rm -Rf "libxml2-$(LIBXML2_VERSION)"

#########################
####### LibEdit #########
#########################

libedit: $(PREFIX)/lib/libedit.dylib

$(PREFIX)/lib/libedit.dylib: libedit-$(LIBEDIT_VERSION)/Makefile
	make -C "libedit-$(LIBEDIT_VERSION)" install

libedit-$(LIBEDIT_VERSION)/Makefile: libedit-$(LIBEDIT_VERSION)/configure
	cd libedit-$(LIBEDIT_VERSION) && ./configure --prefix="$(PREFIX)"

libedit-$(LIBEDIT_VERSION)/configure: libedit-$(LIBEDIT_VERSION).tar.gz
	$(TAR) "libedit-${LIBEDIT_VERSION}.tar.gz"
	touch $@

libedit-$(LIBEDIT_VERSION).tar.gz:
	$(CURL) "http://www.thrysoee.dk/editline/libedit-$(LIBEDIT_VERSION).tar.gz"

clean-libedit:
	rm -Rf "libedit-$(LIBEDIT_VERSION)"


#########################
####### LibJasper #######
#########################

libjasper: $(PREFIX)/lib/libjasper.dylib

$(PREFIX)/lib/libjasper.dylib: jasper-${LIBJASPER_VERSION}.uuid/Makefile
	make -C jasper-${LIBJASPER_VERSION}.uuid install

jasper-${LIBJASPER_VERSION}.uuid/Makefile: jasper-${LIBJASPER_VERSION}.uuid/configure
	cd jasper-${LIBJASPER_VERSION}.uuid && ./configure --prefix="$(PREFIX)" --disable-debug --disable-dependency-tracking --enable-shared --enable-dynamic


jasper-${LIBJASPER_VERSION}.uuid/configure: jasper-$(LIBJASPER_VERSION).uuid.tar.gz
	$(TAR) "jasper-$(LIBJASPER_VERSION).uuid.tar.gz"
	touch $@

jasper-$(LIBJASPER_VERSION).uuid.tar.gz:
	$(CURL) "http://download.osgeo.org/gdal/jasper-$(LIBJASPER_VERSION).uuid.tar.gz"

clean-libjasper:
	rm -Rf jasper-${LIBJASPER_VERSION}.uuid


#########################
#######  libjpeg  #######
#########################

libjpeg: $(PREFIX)/lib/libjpeg.dylib

$(PREFIX)/lib/libjpeg.dylib: jpeg-$(JPEG_VERSION)/Makefile
	make -C jpeg-$(JPEG_VERSION) install

jpeg-$(JPEG_VERSION)/Makefile: jpeg-$(JPEG_VERSION)/configure
	cd jpeg-$(JPEG_VERSION) && ./configure --prefix="$(PREFIX)" --disable-dependency-tracking

jpeg-$(JPEG_VERSION)/configure: jpegsrc.v$(JPEG_VERSION).tar.gz
	$(TAR) jpegsrc.v$(JPEG_VERSION).tar.gz
	touch $@

jpegsrc.v$(JPEG_VERSION).tar.gz:
	$(CURL) "http://www.ijg.org/files/jpegsrc.v$(JPEG_VERSION).tar.gz"

clean-libjpeg:
	rm -Rf jpeg-$(JPEG_VERSION)

#########################
#######  libgeos  #######
#########################

# Depends on postgresql

libgeos: $(PREFIX)/lib/libgeos.dylib

$(PREFIX)/lib/libgeos.dylib: geos-${GEOS_VERSION}/Makefile
	make -C geos-${GEOS_VERSION} install

geos-${GEOS_VERSION}/Makefile: geos-${GEOS_VERSION}/configure $(PREFIX)/bin/psql
	cd geos-${GEOS_VERSION} && ./configure --prefix="$(PREFIX)"

#geos needs to be patched
geos-${GEOS_VERSION}/configure: geos-${GEOS_VERSION}.tar.bz2
	$(TAR) geos-${GEOS_VERSION}.tar.bz2
	/usr/bin/sed -i -e 's/@CFLAGS@$$/@CFLAGS@ -O1/g' geos-${GEOS_VERSION}/src/geom/Makefile.in
	/usr/bin/sed -i -e 's/@CXXFLAGS@$$/@CXXFLAGS@ -O1/g' geos-${GEOS_VERSION}/src/geom/Makefile.in
	touch $@

geos-${GEOS_VERSION}.tar.bz2:
	$(CURL) "http://download.osgeo.org/geos/geos-${GEOS_VERSION}.tar.bz2"

clean-libgeos:
	rm -Rf geos-${GEOS_VERSION}

#########################
#######  libgdal  #######
#########################

# Depends on libproj, libjasper, postgresql, libjpeg, libtiff

libgdal: $(PREFIX)/lib/libgdal.dylib

$(PREFIX)/lib/libgdal.dylib: gdal-$(GDAL_VERSION)/GNUMakefile
	make -C gdal-$(GDAL_VERSION) install

gdal-$(GDAL_VERSION)/GNUMakefile: gdal-$(GDAL_VERSION)/autogen.sh $(PREFIX)/lib/libproj.dylib $(PREFIX)/lib/libjasper.dylib  $(PREFIX)/lib/libjpeg.dylib $(PREFIX)/bin/psql $(PREFIX)/lib/libtiff.dylib
	cd gdal-$(GDAL_VERSION) && ./autogen.sh && ./configure --prefix="$(PREFIX)" --with-pg="$(PREFIX)/bin/pg_config"  --with-jpeg="$(PREFIX)" --with-jasper="$(PREFIX)"
	touch $@

gdal-$(GDAL_VERSION)/autogen.sh: gdal-${GDAL_VERSION}.tar.gz
	$(TAR) gdal-${GDAL_VERSION}.tar.gz
	touch $@	

gdal-${GDAL_VERSION}.tar.gz:
	$(CURL) "http://download.osgeo.org/gdal/${GDAL_VERSION}/gdal-${GDAL_VERSION}.tar.gz"

clean-libgdal:
	rm -Rf gdal-$(GDAL_VERSION)


#########################
###### proj4 ############
#########################

# Depends on postgresql

libproj: $(PREFIX)/lib/libproj.dylib

$(PREFIX)/lib/libproj.dylib: proj-$(PROJ_VERSION)/Makefile
	make -C proj-$(PROJ_VERSION) install

proj-$(PROJ_VERSION)/Makefile: proj-$(PROJ_VERSION)/configure $(PREFIX)/bin/psql
	cd proj-$(PROJ_VERSION) && ./configure --prefix="$(PREFIX)"

proj-${PROJ_VERSION}/configure: proj-${PROJ_VERSION}.tar.gz proj-datumgrid-$(DATUMGRID_VERSION).zip
	$(TAR) "proj-${PROJ_VERSION}.tar.gz"
	unzip -o "proj-datumgrid-${DATUMGRID_VERSION}.zip" -d "./proj-${PROJ_VERSION}/nad"
	touch $@

proj-${PROJ_VERSION}.tar.gz:
	$(CURL) "http://download.osgeo.org/proj/proj-${PROJ_VERSION}.tar.gz"

proj-datumgrid-${DATUMGRID_VERSION}.zip:
	$(CURL) "http://download.osgeo.org/proj/proj-datumgrid-${DATUMGRID_VERSION}.zip"
	
clean-libproj:
	rm -Rf proj-${PROJ_VERSION}



#########################
###### JSON-c ###########
#########################

$(PREFIX)/include/json/json_object_iterator.h: json-c-$(JSONC_VERSION)/Makefile
	make -C json-c-$(JSONC_VERSION) install
	cp json-c-$(JSONC_VERSION)/json_object_iterator.h "$(PREFIX)/include/json/json_object_iterator.h"


json-c-$(JSONC_VERSION)/Makefile: json-c-$(JSONC_VERSION)/configure
	cd json-c-$(JSONC_VERSION) && ./configure --prefix="$(PREFIX)"

json-c-$(JSONC_VERSION)/configure: json-c-$(JSONC_VERSION).tar.gz
	$(TAR) json-c-$(JSONC_VERSION).tar.gz
	touch $@

json-c-$(JSONC_VERSION).tar.gz:
	$(CURL) "https://s3.amazonaws.com/json-c_releases/releases/json-c-$(JSONC_VERSION).tar.gz"


#########################
####### libtiff #########
#########################

libtiff: $(PREFIX)/lib/libtiff.dylib

$(PREFIX)/lib/libtiff.dylib: tiff-$(LIBTIFF_VERSION)/Makefile
	make -C tiff-$(LIBTIFF_VERSION) install

tiff-$(LIBTIFF_VERSION)/Makefile: tiff-$(LIBTIFF_VERSION)/configure
	cd tiff-$(LIBTIFF_VERSION) && ./configure --prefix="$(PREFIX)" --disable-dependency-tracking

tiff-$(LIBTIFF_VERSION)/configure: tiff-$(LIBTIFF_VERSION).tar.gz
	$(TAR) tiff-$(LIBTIFF_VERSION).tar.gz
	touch $@

tiff-$(LIBTIFF_VERSION).tar.gz:
	$(CURL) ftp://ftp.remotesensing.org/pub/libtiff/tiff-4.0.3.tar.gz

clean-libtiff:
	rm -Rf tiff-$(LIBTIFF_VERSION)

#########################
###### PostGIS ##########
#########################

#depends on libgdal, libgeos, libjpeg, postgresql, jsonc

postgis: $(PREFIX)/lib/postgresql/postgis-$(POSTGIS_MAJOR_VERSION).so $(PREFIX)/lib/liblwgeom.dylib

$(PREFIX)/lib/liblwgeom.dylib: postgis-${POSTGIS_VERSION}/GNUMakefile $(PREFIX)/lib/postgresql/postgis-$(POSTGIS_MAJOR_VERSION).so
	make -C postgis-${POSTGIS_VERSION}/liblwgeom
	make -C postgis-${POSTGIS_VERSION}/liblwgeom install

$(PREFIX)/lib/postgresql/postgis-$(POSTGIS_MAJOR_VERSION).so: postgis-${POSTGIS_VERSION}/GNUMakefile
	make -C postgis-${POSTGIS_VERSION}
	make -C postgis-${POSTGIS_VERSION} install

postgis-${POSTGIS_VERSION}/GNUMakefile: postgis-${POSTGIS_VERSION}/autogen.sh $(PREFIX)/lib/libgdal.dylib $(PREFIX)/lib/libgeos.dylib $(PREFIX)/lib/libjpeg.dylib $(PREFIX)/bin/psql $(PREFIX)/include/json/json_object_iterator.h
	cd postgis-${POSTGIS_VERSION} && ./autogen.sh && ./configure --prefix="$(PREFIX)" --with-pgconfig="$(PREFIX)/bin/pg_config" --with-geosconfig="$(PREFIX)/bin/geos-config" --with-projdir="$(PREFIX)" --with-gdaldir="$(PREFIX)" --with-jsondir="$(PREFIX)"
	/usr/bin/sed -i -e 's#/usr/local$$#"$(PREFIX)"#g' postgis-${POSTGIS_VERSION}/liblwgeom/Makefile
	touch $@

postgis-${POSTGIS_VERSION}/autogen.sh: postgis-${POSTGIS_VERSION}.tar.gz
	$(TAR) $<
	touch $@

postgis-${POSTGIS_VERSION}.tar.gz:
	$(CURL) "http://download.osgeo.org/postgis/source/postgis-${POSTGIS_VERSION}.tar.gz"

clean-postgis:
	rm -Rf postgis-${POSTGIS_VERSION}


##########################
######## PLV8 ############
##########################

plv8: $(PREFIX)/lib/postgresql/plv8.so

$(PREFIX)/lib/postgresql/plv8.so: plv8js/Makefile $(PREFIX)/lib/libv8.dylib
	export PGHOME="$(PREFIX)"; export CC=clang; export CXX=clang++; export V8_SRCDIR="$(CURDIR)/v8-${V8_VERSION}"; make -C plv8js install 
	/usr/bin/install_name_tool -change "/usr/local/lib/libv8.dylib" "$(PREFIX)/lib/libv8.dylib" "$(PREFIX)/lib/postgresql/plv8.so"


plv8js/Makefile:
	/usr/bin/git clone "https://code.google.com/p/plv8js/" -q
	touch $@

$(PREFIX)/lib/libv8.dylib: v8-${V8_VERSION}/out/x64.release/libv8.dylib
	cp v8-${V8_VERSION}/out/x64.release/libv8.dylib $(PREFIX)/lib/libv8.dylib

v8-${V8_VERSION}/out/x64.release/libv8.dylib: v8-${V8_VERSION}/Makefile
	export CC=clang; export CXX=clang++; export GYP_DEFINES="clang=1"; make -C v8-${V8_VERSION} dependencies
	export CC=clang; export CXX=clang++; export GYP_DEFINES="clang=1"; make -C v8-${V8_VERSION} library=shared x64.release

v8-${V8_VERSION}/Makefile:
	svn export --force "http://v8.googlecode.com/svn/branches/$(V8_VERSION)/" "v8-$(V8_VERSION)"
	touch $@

clean-plv8:
	rm -Rf plv8js v8-${V8_VERSION}
