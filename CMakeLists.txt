cmake_minimum_required(VERSION 3.5)

project(libxslt_vendor)

find_package(ament_cmake REQUIRED)

set(PACKAGE_VERSION "1.0.0")

macro(build_libxslt)
  set(configure_opts)

  if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
    set(TARGET "--host=x86_64-pc-nto-qnx7.1.0")
  elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64le")
    set(TARGET "--host=aarch64-unknown-nto-qnx7.0.0")
  elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "armv7")
    set(TARGET "--host=arm-unknown-nto-qnx7.1.0eabi")
  else()
    message(FATAL_ERROR "invalid cpu CMAKE_SYSTEM_PROCESSOR:${CMAKE_SYSTEM_PROCESSOR}")
  endif()

  list(APPEND configure_opts "${TARGET}")
  list(APPEND configure_opts "--prefix=${CMAKE_INSTALL_PREFIX}")

  list(APPEND configure_opts "--with-libxml-include-prefix=${QNX_TARGET}/usr/include")
  list(APPEND configure_opts "--with-libxml-libs-prefix=${QNX_TARGET}/usr/lib")
  list(APPEND configure_opts "--without-python")
  list(APPEND configure_opts "LIBS=-lxml2")

  list(APPEND configure_opts "CFLAGS=${CFLAGS} -Wno-unused-function -Wno-unused-parameter")

  # installation of docs fails due to a known bug https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=947914
  # this does not affect the installation of the libraries
  # the following two lines removes the doc directory from the list of src directories
  find_package(Patch)
  include(ExternalProject)
  ExternalProject_Add(libxslt
    URL ftp://xmlsoft.org/libxslt/libxslt-1.1.34.tar.gz
    TIMEOUT 600
    CONFIGURE_COMMAND
    cd ${CMAKE_CURRENT_BINARY_DIR}/libxslt-prefix/src/libxslt && ./configure "${configure_opts}" &&
    ${Patch_EXECUTABLE} Makefile ${CMAKE_CURRENT_SOURCE_DIR}/libxslt.patch
    BUILD_COMMAND cd ${CMAKE_CURRENT_BINARY_DIR}/libxslt-prefix/src/libxslt && make
    INSTALL_COMMAND cd ${CMAKE_CURRENT_BINARY_DIR}/libxslt-prefix/src/libxslt && make install
  )
endmacro()

find_package(libxslt QUIET)

if(NOT libxslt_FOUND)
  build_libxslt()
endif()

ament_package()
