cmake_minimum_required(VERSION 2.8)

set(UGET_VERSION "2.2.3")

if(CMAKE_VERSION VERSION_LESS "3.0")
    project(uget2 C CXX)
    set(PROJECT_VERSION ${UGET_VERSION})
else()
    cmake_policy(SET CMP0042 NEW)
    cmake_policy(SET CMP0048 NEW)
    project(uget2 VERSION ${UGET_VERSION} LANGUAGES C CXX)
endif()

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include(CheckFunctionExists)
include(CheckSymbolExists)
include(CheckLibraryExists)
include(CheckIncludeFile)

#
# Package information
#
set(PACKAGE "${PROJECT_NAME}")
set(VERSION "${PROJECT_VERSION}")
set(PACKAGE_BUGREPORT "tuduongquyet@gmail.com")
set(PACKAGE_URL "https://ugetdm.com")
set(PACKAGE_NAME "${PACKAGE}")
set(PACKAGE_TARNAME "${PACKAGE}")
set(PACKAGE_VERSION "${VERSION}")
set(PACKAGE_STRING "${PACKAGE} ${VERSION}")

#
# Options
#
option(USE_GNUTLS "Use GnuTLS as SSL backend" AUTO)
option(USE_OPENSSL "Use OpenSSL as SSL backend" ON)
option(WITH_GSTREAMER "Use GStreamer" ON)
option(WITH_LIBNOTIFY "Use libnotify" ON)
option(WITH_APPINDICATOR "Use appindicator" ON)
option(WITH_LIBPWMD "Use libpwmd" OFF)
option(ENABLE_RSS_NOTIFY "Enable RSS Notify" ON)
option(ENABLE_UNIX_DOMAIN_SOCKET "Enable Unix Domain Socket" ON)

# Use OpenSSL if GnuTLS is not selected or not found
if(USE_GNUTLS)
    set(USE_OPENSSL OFF)
else()
    set(USE_OPENSSL ON)
endif()

# Check for required libraries
check_include_file("dlfcn.h" HAVE_DLFCN_H)
check_include_file("inttypes.h" HAVE_INTTYPES_H)
check_include_file("memory.h" HAVE_MEMORY_H)
check_include_file("stdint.h" HAVE_STDINT_H)

if(NOT HAVE_STDINT_H)
    if(WIN32)
        include_directories(Windows/msvc)
    else()
        message(FATAL_ERROR "stdint.h is required")
    endif()
endif()

check_include_file("ctype.h" HAVE_CTYPE_H)
check_include_file("stdlib.h" HAVE_STDLIB_H)

if(HAVE_CTYPE_H AND HAVE_STDLIB_H)
    set(STDC_HEADERS 1)
endif()

check_include_file("strings.h" HAVE_STRINGS_H)
check_include_file("string.h" HAVE_STRING_H)
check_include_file("sys/stat.h" HAVE_SYS_STAT_H)
check_include_file("sys/types.h" HAVE_SYS_TYPES_H)
check_include_file("unistd.h" HAVE_UNISTD_H)

check_function_exists(posix_fallocate HAVE_POSIX_FALLOCATE)
check_function_exists(ftruncate HAVE_FTRUNCATE)
check_function_exists(CFPreferencesCopyAppValue HAVE_CFPREFERENCESCOPYAPPVALUE)
check_function_exists(CFLocaleCopyCurrent HAVE_CFLOCALECOPYCURRENT)
check_function_exists(gettext HAVE_GETTEXT)
check_function_exists(dcgettext HAVE_DCGETTEXT)
check_symbol_exists(bind_textdomain_codeset "libintl.h" HAVE_BIND_TEXTDOMAIN_CODESET)

if(HAVE_GETTEXT)
    set(ENABLE_NLS 1)
endif()

check_include_file(locale.h HAVE_LOCALE_H)
check_include_file(locale.h HAVE_LC_MESSAGES)
check_library_exists(pthread pthread_create "" HAVE_LIBPTHREAD)

# Use the package PkgConfig to detect GTK+ headers/library files
find_package(PkgConfig REQUIRED)

# Threads
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
list(APPEND UGET_REQUIRED_LIBRARIES ${CMAKE_THREAD_LIBS_INIT})

# Glib
pkg_check_modules(GLIB2 glib-2.0)

if(GLIB2_FOUND)
    set(HAVE_GLIB 1)
endif()

# GTK+
pkg_check_modules(GTK3 REQUIRED gtk+-3.0)

include_directories(${GTK3_INCLUDE_DIRS})
link_directories(${GTK3_LIBRARY_DIRS})
add_definitions(${GTK3_CFLAGS_OTHER})
list(APPEND UGET_REQUIRED_LIBRARIES ${GTK3_LIBRARIES})

# LFS
find_program(GETCONF getconf)

if(GETCONF)
    execute_process(COMMAND ${GETCONF} LFS_CFLAGS OUTPUT_VARIABLE LFS_CFLAGS OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process(COMMAND ${GETCONF} LFS_LDFLAGS OUTPUT_VARIABLE LFS_LDFLAGS OUTPUT_STRIP_TRAILING_WHITESPACE)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${LFS_CFLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${LFS_LDFLAGS}")
endif()

# CURL
find_package(CURL 7.19.1 REQUIRED)

if(CURL_FOUND)
    message(STATUS "Using CURL ${CURL_VERSION} ${CURL_INCLUDE_DIRS} ${CURL_LIBRARIES}")
else(CURL_FOUND)
    message(STATUS "NOT Using CURL ${CURL_VERSION} ${CURL_INCLUDE_DIRS} ${CURL_LIBRARIES}")
endif(CURL_FOUND)

include_directories(${CURL_INCLUDE_DIRS})
link_directories(${CURL_LIBRARY_DIRS})
list(APPEND UGET_REQUIRED_LIBRARIES ${CURL_LIBRARIES})

# GnuTLS
if(USE_GNUTLS)
    find_package(GnuTLS)

    if(NOT GNUTLS_FOUND)
        message(FATAL_ERROR "GnuTLS was selected but not found")
    endif(NOT GNUTLS_FOUND)

    find_program(LIBGCRYPT_CONFIG libgcrypt-config)

    if(NOT LIBGCRYPT_CONFIG)
        if(USE_GNUTLS STREQUAL "YES")
            message(FATAL_ERROR "Unable to find libgcrypt-config, please install libgcrypt")
        endif()
    else()
        execute_process(COMMAND ${LIBGCRYPT_CONFIG} --cflags OUTPUT_VARIABLE LIBGCRYPT_CFLAGS OUTPUT_STRIP_TRAILING_WHITESPACE)
        execute_process(COMMAND ${LIBGCRYPT_CONFIG} --libs OUTPUT_VARIABLE LIBGCRYPT_LIBS OUTPUT_STRIP_TRAILING_WHITESPACE)

        set(USE_GNUTLS 1)
        set(SSL_INCLUDE_DIRS ${GNUTLS_INCLUDE_DIRS})
        set(SSL_LIBRARIES ${GNUTLS_LIBRARIES})
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${LIBGCRYPT_CFLAGS}")
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${LIBGCRYPT_LIBS}")
    endif()
endif()

# OpenSSL
if(USE_OPENSSL)
    find_package(OpenSSL)

    # Depend on OpenSSL via imported targets if supported by the running
    # version of CMake.  This allows our dependents to get our dependencies
    # transitively.
    if(NOT CMAKE_VERSION VERSION_LESS 3.4)
        list(APPEND UGET_REQUIRED_LIBRARIES OpenSSL::SSL OpenSSL::Crypto)
    else()
        list(APPEND UGET_REQUIRED_LIBRARIES ${OPENSSL_LIBRARIES})
        include_directories(${OPENSSL_INCLUDE_DIR})
    endif()

    set(USE_OPENSSL 1)
    set(SSL_INCLUDE_DIRS ${OPENSSL_INCLUDE_DIRS})
    set(SSL_LIBRARIES ${OPENSSL_LIBRARIES})
endif(USE_OPENSSL)

include_directories(${SSL_INCLUDE_DIRS})
link_directories(${SSL_LIBRARY_DIRS})
list(APPEND UGET_REQUIRED_LIBRARIES ${SSL_LIBRARIES})

# libnotify
if(WITH_LIBNOTIFY)
    pkg_check_modules(LIBNOTIFY libnotify)

    if(LIBNOTIFY_FOUND)
        set(HAVE_LIBNOTIFY 1)
        include_directories(${LIBNOTIFY_INCLUDE_DIRS})
        link_directories(${LIBNOTIFY_LIBRARY_DIRS})
        list(APPEND UGET_REQUIRED_LIBRARIES ${LIBNOTIFY_LIBRARIES})
    else()
        set(HAVE_LIBNOTIFY 0)
    endif()
endif()

# appindicator
if(WITH_APPINDICATOR)
    pkg_check_modules(APPINDICATOR appindicator3-0.1)

    if(APPINDICATOR_FOUND)
        set(HAVE_APP_INDICATOR 1)
        include_directories(${APPINDICATOR_INCLUDE_DIRS})
        link_directories(${APPINDICATOR_LIBRARY_DIRS})
        list(APPEND UGET_REQUIRED_LIBRARIES ${APPINDICATOR_LIBRARIES})
    else()
        set(HAVE_APP_INDICATOR 0)
    endif()
endif()

# GStreamer
if(WITH_GSTREAMER)
    pkg_check_modules(GSTREAMER gstreamer-1.0)

    if(GSTREAMER_FOUND)
        set(HAVE_GSTREAMER 1)
        include_directories(${GSTREAMER_INCLUDE_DIRS})
        link_directories(${GSTREAMER_LIBRARY_DIRS})
        list(APPEND UGET_REQUIRED_LIBRARIES ${GSTREAMER_LIBRARIES})
    endif()
endif()

# libpwmd
if(WITH_LIBPWMD)
    pkg_check_modules(LIBPWMD libpwmd)

    if(LIBPWMD_FOUND)
        set(HAVE_LIBPWMD 1)
        include_directories(${LIBPWMD_INCLUDE_DIRS})
        link_directories(${LIBPWMD_LIBRARY_DIRS})
        list(APPEND UGET_REQUIRED_LIBRARIES ${LIBPWMD_LIBRARIES})
    else()
        set(HAVE_LIBPWMD 0)
    endif()
endif()

# Enable RSS Notify
if(ENABLE_RSS_NOTIFY)
    set(HAVE_RSS_NOTIFY 1)
else()
    set(HAVE_RSS_NOTIFY 0)
endif()

# Enable Unix Domain Socket
if(ENABLE_UNIX_DOMAIN_SOCKET)
    set(HAVE_UNIX_DOMAIN_SOCKET 1)
else()
    set(HAVE_UNIX_DOMAIN_SOCKET 0)
endif()

# gettext
find_package(Gettext REQUIRED)

if(NOT GETTEXT_MSGMERGE_EXECUTABLE)
    message(FATAL_ERROR "Please install msgmerge binary")
endif(NOT GETTEXT_MSGMERGE_EXECUTABLE)

if(NOT GETTEXT_MSGFMT_EXECUTABLE)
    message(FATAL_ERROR "Please install msgmerge binary")
endif(NOT GETTEXT_MSGFMT_EXECUTABLE)

if(APPLE)
    pkg_check_modules(GTKMACINTEGRATION3 REQUIRED gtk-mac-integration-gtk3)
    include_directories(${GTKMACINTEGRATION3_INCLUDE_DIRS})
    link_directories(${GTKMACINTEGRATION3_LIBRARY_DIRS})
    list(APPEND UGET_REQUIRED_LIBRARIES ${GTKMACINTEGRATION3_LIBRARIES})
endif(APPLE)

# Config.h
add_definitions(-DHAVE_CONFIG_H)
set(CONFIG_H ${CMAKE_CURRENT_SOURCE_DIR}/config.h)
configure_file(config.h.cmakein ${CONFIG_H})

include_directories(${CMAKE_SOURCE_DIR})

# uglib
add_subdirectory(uglib)

# uget
add_subdirectory(uget)

# uget-gtk
add_subdirectory(ui-gtk)

# uget-gtk-1to2
add_subdirectory(ui-gtk-1to2)
