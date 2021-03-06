PROJECT (librepo C)
CMAKE_MINIMUM_REQUIRED (VERSION 2.6)

OPTION (ENABLE_TESTS "Build test?" ON)
OPTION (ENABLE_DOCS "Build docs?" ON)

INCLUDE (${CMAKE_SOURCE_DIR}/VERSION.cmake)
SET (VERSION "${LIBREPO_MAJOR}.${LIBREPO_MINOR}.${LIBREPO_PATCH}")

SET (CMAKE_C_FLAGS          "${CMAKE_C_FLAGS} -std=c99 -Wall")
SET (CMAKE_C_FLAGS_DEBUG    "${CMAKE_C_FLAGS} -ggdb -O0 -Wall -Wextra -DDEBUG")

IF(CMAKE_BUILD_TYPE AND (CMAKE_BUILD_TYPE STREQUAL "DEBUG" OR CMAKE_BUILD_TYPE STREQUAL "debug"))
    MESSAGE("Build type is set do DEBUG!")
ENDIF()

# Set librepo log domain

add_definitions( -DG_LOG_DOMAIN="librepo" )

# Add path with own cmake modules

INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR})
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")


# Find necessare libraries

FIND_PACKAGE(PkgConfig)
PKG_CHECK_MODULES(GLIB2 glib-2.0 REQUIRED)
PKG_SEARCH_MODULE(LIBCRYPTO REQUIRED libcrypto openssl)
FIND_PACKAGE(EXPAT REQUIRED)
FIND_PACKAGE(CURL REQUIRED)
FIND_PACKAGE(Gpgme REQUIRED)
FIND_PACKAGE(Xattr REQUIRED)

INCLUDE_DIRECTORIES(${GLIB2_INCLUDE_DIRS})

# Enable large file support

ADD_DEFINITIONS(-D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE -D_LARGEFILE64_SOURCE)

# Check libraries

IF (NOT EXPAT_FOUND)
    MESSAGE(FATAL_ERROR "No Expat library installed")
ENDIF (NOT EXPAT_FOUND)

IF (NOT CURL_FOUND)
    MESSAGE(FATAL_ERROR "No CURL library installed")
ENDIF (NOT CURL_FOUND)


# Add include dirs

INCLUDE_DIRECTORIES(${EXPAT_INCLUDE_DIRS})
INCLUDE_DIRECTORIES(${CURL_INCLUDE_DIR})
#INCLUDE_DIRECTORIES(${CHECK_INCLUDE_DIR})

IF (NOT LIB_INSTALL_DIR)
  SET (LIB_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/usr/lib")
ENDIF (NOT LIB_INSTALL_DIR)

IF (NOT INCLUDE_INSTALL_DIR)
  SET (INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/usr/include")
ENDIF (NOT INCLUDE_INSTALL_DIR)

# Python stuff

if (NOT PYTHON_DESIRED)
    set (PYTHON_DESIRED "2")
endif()

# Subdirs

ADD_SUBDIRECTORY (librepo)

IF (ENABLE_TESTS)
  FIND_LIBRARY(CHECK_LIBRARY NAMES check)
  ENABLE_TESTING()
  ADD_SUBDIRECTORY (tests)
ENDIF (ENABLE_TESTS)

IF (ENABLE_DOCS)
  ADD_SUBDIRECTORY (doc)
ENDIF (ENABLE_DOCS)
