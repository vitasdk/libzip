# TODO:
# 2. installation not tested.
# 3. Problems generating html pages.
# 4. create usable libtool .la file
# 5. runtest not created

CMAKE_MINIMUM_REQUIRED(VERSION 3.16)

PROJECT(libzip C)

INCLUDE(CheckFunctionExists)
INCLUDE(CheckIncludeFiles)
INCLUDE(CheckSymbolExists)
INCLUDE(CheckTypeSize)
INCLUDE(CheckCSourceRuns)
INCLUDE(CheckCSourceCompiles)
INCLUDE(CheckStructHasMember)
INCLUDE(TestBigEndian)

SET(PACKAGE "libzip")
SET(PACKAGE_NAME ${PACKAGE})
SET(PACKAGE_VERSION_MAJOR "1")
SET(PACKAGE_VERSION_MINOR "2")
SET(PACKAGE_VERSION_PATCH "0a")
#SET(VERSION "${PACKAGE_VERSION_MAJOR}.${PACKAGE_VERSION_MINOR}")
SET(VERSION "${PACKAGE_VERSION_MAJOR}.${PACKAGE_VERSION_MINOR}.${PACKAGE_VERSION_PATCH}")
SET(PACKAGE_VERSION ${VERSION})
SET(PACKAGE_STRING "${PACKAGE_NAME} ${PACKAGE_VERSION}")

# Checks

CHECK_FUNCTION_EXISTS(_close HAVE__CLOSE)
CHECK_FUNCTION_EXISTS(_dup HAVE__DUP)
CHECK_FUNCTION_EXISTS(_fdopen HAVE__FDOPEN)
CHECK_FUNCTION_EXISTS(_fileno HAVE__FILENO)
CHECK_FUNCTION_EXISTS(_open HAVE__OPEN)
CHECK_FUNCTION_EXISTS(_setmode HAVE__SETMODE)
CHECK_FUNCTION_EXISTS(_snprintf HAVE__SNPRINTF)
CHECK_FUNCTION_EXISTS(_strdup HAVE__STRDUP)
CHECK_FUNCTION_EXISTS(_stricmp HAVE__STRICMP)
CHECK_FUNCTION_EXISTS(_strtoi64 HAVE__STRTOI64)
CHECK_FUNCTION_EXISTS(_strtoui64 HAVE__STRTOUI64)
CHECK_FUNCTION_EXISTS(explict_bzero HAVE_EXPLICIT_BZERO)
CHECK_FUNCTION_EXISTS(explicit_memset HAVE_EXPLICIT_MEMSET)
CHECK_FUNCTION_EXISTS(fileno HAVE_FILENO)
CHECK_FUNCTION_EXISTS(fseeko HAVE_FSEEKO)
CHECK_FUNCTION_EXISTS(ftello HAVE_FTELLO)
CHECK_FUNCTION_EXISTS(getprogname HAVE_GETPROGNAME)
CHECK_FUNCTION_EXISTS(open HAVE_OPEN)
CHECK_FUNCTION_EXISTS(mkstemp HAVE_MKSTEMP)
CHECK_FUNCTION_EXISTS(setmode HAVE_SETMODE)
CHECK_FUNCTION_EXISTS(snprintf HAVE_SNPRINTF)
CHECK_FUNCTION_EXISTS(strcasecmp HAVE_STRCASECMP)
CHECK_FUNCTION_EXISTS(strdup HAVE_STRDUP)
CHECK_FUNCTION_EXISTS(stricmp HAVE_STRICMP)
CHECK_FUNCTION_EXISTS(strtoll HAVE_STRTOLL)
CHECK_FUNCTION_EXISTS(strtoull HAVE_STRTOULL)

CHECK_INCLUDE_FILES(fts.h HAVE_FTS_H)
CHECK_INCLUDE_FILES(stdbool.h HAVE_STDBOOL_H)
CHECK_INCLUDE_FILES(strings.h HAVE_STRINGS_H)
CHECK_INCLUDE_FILES(unistd.h HAVE_UNISTD_H)

CHECK_INCLUDE_FILES(inttypes.h HAVE_INTTYPES_H_LIBZIP)
CHECK_INCLUDE_FILES(stdint.h HAVE_STDINT_H_LIBZIP)
CHECK_INCLUDE_FILES(sys/types.h HAVE_SYS_TYPES_H_LIBZIP)

# TODO: fix test
# this test does not find __progname even when it exists
#CHECK_SYMBOL_EXISTS(__progname stdlib.h HAVE___PROGNAME)

CHECK_TYPE_SIZE(__int8 __INT8_LIBZIP)
CHECK_TYPE_SIZE(int8_t INT8_T_LIBZIP)
CHECK_TYPE_SIZE(uint8_t UINT8_T_LIBZIP)
CHECK_TYPE_SIZE(__int16 __INT16_LIBZIP)
CHECK_TYPE_SIZE(int16_t INT16_T_LIBZIP)
CHECK_TYPE_SIZE(uint16_t UINT16_T_LIBZIP)
CHECK_TYPE_SIZE(__int32 __INT32_LIBZIP)
CHECK_TYPE_SIZE(int32_t INT32_T_LIBZIP)
CHECK_TYPE_SIZE(uint32_t UINT32_T_LIBZIP)
CHECK_TYPE_SIZE(__int64 __INT64_LIBZIP)
CHECK_TYPE_SIZE(int64_t INT64_T_LIBZIP)
CHECK_TYPE_SIZE(uint64_t UINT64_T_LIBZIP)
CHECK_TYPE_SIZE("short" SHORT_LIBZIP)
CHECK_TYPE_SIZE("int" INT_LIBZIP)
CHECK_TYPE_SIZE("long" LONG_LIBZIP)
CHECK_TYPE_SIZE("long long" LONG_LONG_LIBZIP)
CHECK_TYPE_SIZE("off_t" SIZEOF_OFF_T)
CHECK_TYPE_SIZE("size_t" SIZE_T_LIBZIP)
CHECK_TYPE_SIZE("ssize_t" SSIZE_T_LIBZIP)

TEST_BIG_ENDIAN(WORDS_BIGENDIAN)

FIND_PACKAGE(ZLIB REQUIRED)
INCLUDE_DIRECTORIES(${ZLIB_INCLUDE_DIR})
set(CMAKE_REQUIRED_INCLUDES ${ZLIB_INCLUDE_DIR})
IF(ZLIB_VERSION_STRING VERSION_LESS "1.1.2")
  MESSAGE(FATAL_ERROR "-- ZLIB version too old, please install at least v1.1.2")
ENDIF(ZLIB_VERSION_STRING VERSION_LESS "1.1.2")

IF(MSVC)
ADD_DEFINITIONS("-D_CRT_SECURE_NO_WARNINGS")
ENDIF(MSVC)

ADD_DEFINITIONS("-DHAVE_CONFIG_H")

# Targets
ADD_SUBDIRECTORY(lib)
#ADD_SUBDIRECTORY(man)
#ADD_SUBDIRECTORY(src)
#ADD_SUBDIRECTORY(regress)
#ADD_SUBDIRECTORY(examples)

# pkgconfig file
SET(prefix ${CMAKE_INSTALL_PREFIX})
SET(exec_prefix \${prefix})
SET(bindir \${exec_prefix}/bin)
SET(libdir \${exec_prefix}/lib)
SET(includedir \${prefix}/include)
IF(CMAKE_SYSTEM_NAME MATCHES BSD)
  SET(PKG_CONFIG_RPATH "-Wl,-R\${libdir}")
ENDIF(CMAKE_SYSTEM_NAME MATCHES BSD)
get_target_property(LIBS_PRIVATE zip LINK_LIBRARIES)
foreach(LIB ${LIBS_PRIVATE})
  if(LIB MATCHES "^/")
    get_filename_component(LIB ${LIB} NAME_WE)
    string(REGEX REPLACE "^lib" "" LIB ${LIB})
  endif()
  set(LIBS "${LIBS} -l${LIB}")
endforeach()
CONFIGURE_FILE(libzip.pc.in libzip.pc @ONLY)
INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/libzip.pc DESTINATION lib/pkgconfig)

# write out config file
CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/cmake-config.h.in
  ${CMAKE_CURRENT_BINARY_DIR}/config.h)
CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/cmake-zipconf.h.in
  ${CMAKE_CURRENT_BINARY_DIR}/zipconf.h)

# installation

INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/zipconf.h DESTINATION lib/libzip/include)
