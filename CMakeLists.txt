IF(NOT JPEGLS_NAMESPACE)
  SET(JPEGLS_NAMESPACE "JPEGLS")
  SET(JPEGLS_STANDALONE 1)
ENDIF(NOT JPEGLS_NAMESPACE)
# In all cases:
STRING(TOLOWER ${JPEGLS_NAMESPACE} JPEGLS_LIBRARY_NAME)

PROJECT(${JPEGLS_NAMESPACE} C)
CMAKE_MINIMUM_REQUIRED(VERSION 2.2)

#-----------------------------------------------------------------------------
# JPEG-LS version number
SET(JPEGLS_VERSION_MAJOR 2)
SET(JPEGLS_VERSION_MINOR 3)
SET(JPEGLS_VERSION_BUILD 0)
SET(JPEGLS_VERSION 
      "${JPEGLS_VERSION_MAJOR}.${JPEGLS_VERSION_MINOR}.${JPEGLS_VERSION_BUILD}")

# This setting of SOVERSION assumes that any API change
# will increment either the minor or major version number of JPEGLS.
SET(JPEGLS_LIBRARY_PROPERTIES 
  VERSION   "${JPEGLS_VERSION_MAJOR}.${JPEGLS_VERSION_MINOR}.${JPEGLS_VERSION_BUILD}"
  SOVERSION "${JPEGLS_VERSION_MAJOR}.${JPEGLS_VERSION_MINOR}"
)

#-----------------------------------------------------------------------------
# Build shared lib by default
OPTION(BUILD_SHARED_LIBS "Build JPEG-LS with shared libraries." OFF)

#-----------------------------------------------------------------------------
SET (EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/bin
  CACHE PATH "Single output directory for building all executables.")
SET (LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/bin
  CACHE PATH "Single output directory for building all libraries.")
MARK_AS_ADVANCED(LIBRARY_OUTPUT_PATH EXECUTABLE_OUTPUT_PATH)


#-----------------------------------------------------------------------------
# Test if os defines a lenght for sockaddr
# only doing the test when we have sys/socket
IF("HAVE_C_INLINE" MATCHES "^HAVE_C_INLINE$")
  STRING(ASCII 35 POUND)
  FILE(WRITE ${CMAKE_CURRENT_BINARY_DIR}/jpeglsTestInline.c
    "inline int foo() { return 0; }\n
    int main() { return foo(); }")
  MESSAGE(STATUS "Checking if C compiler has inline keyword")
  TRY_COMPILE(HAVE_C_INLINE
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}/jpeglsTestInline.c
    OUTPUT_VARIABLE OUTPUT)
  IF(HAVE_C_INLINE)
    MESSAGE(STATUS "Checking if C compiler has inline keyword -- yes")
    SET(HAVE_C_INLINE 1 CACHE INTERNAL "Support if C compiler has inline keyword")
    WRITE_FILE(${CMAKE_BINARY_DIR}/CMakeOutput.log
      "Determining if C compiler has inline keyword"
      "passed with the following output:\n"
      "${OUTPUT}\n" APPEND)
  ELSE(HAVE_C_INLINE)
    MESSAGE(STATUS "Checking if C compiler has inline keyword -- no")
    SET(HAVE_C_INLINE 0 CACHE INTERNAL "Support if C compiler has inline keyword")
    WRITE_FILE(${CMAKE_BINARY_DIR}/CMakeError.log
      "Determining if C compiler has inline keyword"
      "failed with the following output:\n"
      "${OUTPUT}\n" APPEND)
  ENDIF(HAVE_C_INLINE)
ENDIF("HAVE_C_INLINE" MATCHES "^HAVE_C_INLINE$")

INCLUDE (${CMAKE_ROOT}/Modules/CheckIncludeFile.cmake)
CHECK_INCLUDE_FILE("unistd.h"    HAVE_UNISTD_H)
CONFIGURE_FILE(
  "${CMAKE_CURRENT_SOURCE_DIR}/jpegls_config.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/jpegls_config.h"
  @ONLY IMMEDIATE)
SET(MANGLE_PREFIX ${JPEGLS_LIBRARY_NAME})
CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/jpegls_mangle.h.in
               ${CMAKE_CURRENT_BINARY_DIR}/jpegls_mangle.h
               @ONLY IMMEDIATE)

# Include dir for jpegls_config.h
INCLUDE_DIRECTORIES(BEFORE
  ${CMAKE_CURRENT_BINARY_DIR}
  )
SUBDIRS(
  src
  )

