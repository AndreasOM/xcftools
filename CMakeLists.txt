## :TODO: this is work in progress
## it works on one of my (osx 10.12.1) systems
## it my break your system

cmake_minimum_required (VERSION 2.8)
project (xcftools)
set(APPNAME "xcftools")

include_directories("${PROJECT_SOURCE_DIR}")
include_directories("${PROJECT_BINARY_DIR}")

# :TODO: actually find perl
set(PERL "/usr/bin/perl")

# :TODO: actually find locale dir
set(LOCALE_DIR "/usr/local/share/locale")

list(APPEND ENUM_SOURCES
	"${PROJECT_SOURCE_DIR}/gimp/base-enums.h"
	"${PROJECT_SOURCE_DIR}/gimp/gimpbaseenums.h"
	"${PROJECT_SOURCE_DIR}/gimp/xcf-private.h"
)

add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/enums.h
  COMMAND ${PERL} ${PROJECT_SOURCE_DIR}/mkenumsh.pl ${ENUM_SOURCES} > ${CMAKE_CURRENT_BINARY_DIR}/enums.h
  DEPENDS mkenumsh.pl
)

add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/enums.c
  COMMAND ${PERL} ${PROJECT_SOURCE_DIR}/mkenumsc.pl enums.h > ${CMAKE_CURRENT_BINARY_DIR}/enums.c
  DEPENDS mkenumsc.pl enums.h
)


add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/table.c
  COMMAND ${PERL} ${PROJECT_SOURCE_DIR}/mktablec.pl > ${CMAKE_CURRENT_BINARY_DIR}/table.c
  DEPENDS mktablec.pl config.h
)

configure_file (
  "${PROJECT_SOURCE_DIR}/nlsini.c.in"
  "${PROJECT_BINARY_DIR}/nlsini.c"
  )

# :TODO: create custom rule to create %.oi
add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/xcfinfo.oi
  COMMAND ${PERL} ${PROJECT_SOURCE_DIR}/mkopti.pl xcfinfo
  DEPENDS mkopti.pl options.i config.h
)

add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/xcf2pnm.oi
  COMMAND ${PERL} ${PROJECT_SOURCE_DIR}/mkopti.pl xcf2pnm
  DEPENDS mkopti.pl options.i config.h
)

add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/xcf2png.oi
  COMMAND ${PERL} ${PROJECT_SOURCE_DIR}/mkopti.pl xcf2png
  DEPENDS mkopti.pl options.i config.h
)

# libpng
find_package(PNG REQUIRED)
include_directories(${PNG_INCLUDE_DIR})

# :TODO: I'm sure not binaries need all sources, but I'm too lazy to track
# and the linker should strip dead code anyway
list(APPEND SHARED_SOURCES
	"enums.c"
	"io-unix.c"
	"xcf-general.c"
	"utils.c"
	"nlsini.c"
	"pixels.c"
	"flatten.c"
	"flatspec.c"
	"scaletab.c"
	"table.c"
	"palette.c"
)

add_executable(xcf2png xcf2png.c xcf2png.oi ${SHARED_SOURCES})
target_link_libraries(xcf2png ${PNG_LIBRARY})

add_executable(xcf2pnm xcf2pnm.c xcf2pnm.oi ${SHARED_SOURCES})
target_link_libraries(xcf2pnm ${PNG_LIBRARY})

add_executable(xcfinfo xcfinfo.c xcfinfo.oi ${SHARED_SOURCES})
target_link_libraries(xcfinfo ${PNG_LIBRARY})
