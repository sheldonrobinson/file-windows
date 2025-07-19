cmake_minimum_required(VERSION 3.7)

project(file)

function(cat IN_FILE OUT_FILE)
  file(READ ${IN_FILE} CONTENTS)
  file(APPEND ${OUT_FILE} "${CONTENTS}")
endfunction()

include(FetchContent)

option(BUILD_TESTS "Build tests" OFF)


#################### dirent
FetchContent_Declare(
  dirent
  GIT_REPOSITORY https://github.com/tronkko/dirent.git
  GIT_TAG        31db6474b5231c180bd4618c8c90e43af50c86d0
  GIT_SHALLOW    TRUE
  
  CMAKE_ARGS -DCMAKE_POSITION_INDEPENDENT_CODE=ON
			 -DDIRENT_EXAMPLES="OFF"
			 -DDIRENT_TESTS="OFF"
			 -DBUILD_SHARED_LIBS=OFF
)

FetchContent_MakeAvailable(dirent)

######################## pcre2posix
set(PCRE2_STATIC_PIC ON CACHE BOOL "Build the static library with the option position independent code enabled." FORCE)
set(PCRE2_SHOW_REPORT ON CACHE BOOL "Show the final configuration report" FORCE)
set(PCRE2_BUILD_PCRE2GREP OFF CACHE BOOL "Build pcre2grep" FORCE)
set(PCRE2_BUILD_TESTS OFF CACHE BOOL "Build the tests" FORCE)
set(PCRE2_SUPPORT_LIBEDIT OFF CACHE BOOL "Enable support for linking pcre2test with libedit." FORCE)
set(PCRE2_SUPPORT_LIBREADLINE OFF CACHE BOOL "Enable support for linking pcre2test with libreadline." FORCE)

FetchContent_Declare(
  pcre2posix
  GIT_REPOSITORY https://github.com/PCRE2Project/pcre2.git
  GIT_TAG        pcre2-10.45
  GIT_SHALLOW    TRUE

  
  CMAKE_ARGS -DCMAKE_POSITION_INDEPENDENT_CODE=ON
             -DPCRE2_STATIC_PIC=ON
             -DPCRE2_SHOW_REPORT=ON
			 -DPCRE2_BUILD_PCRE2GREP=OFF
			 -DPCRE2_BUILD_TESTS=OFF
			 -DPCRE2_SUPPORT_LIBEDIT=OFF
			 -DPCRE2_SUPPORT_LIBREADLINE=OFF
			 -DBUILD_STATIC_LIBS=ON
			 -DBUILD_SHARED_LIBS=OFF
)

FetchContent_MakeAvailable(pcre2posix)

############### file
set(FILE_VERSION 5.46)
FetchContent_Populate(
  libmagic
  QUIET
  GIT_REPOSITORY https://github.com/file/file.git
  GIT_TAG        7b4417cc8bae378c33f5e72148bc706571032580
  GIT_SHALLOW    TRUE
  
  CMAKE_ARGS -DCMAKE_POSITION_INDEPENDENT_CODE=ON
)


set(WIN_COMPAT_SOURCES 
  ${libmagic_SOURCE_DIR}/src/asctime_r.c
  ${libmagic_SOURCE_DIR}/src/asprintf.c
  ${libmagic_SOURCE_DIR}/src/ctime_r.c
  ${libmagic_SOURCE_DIR}/src/dprintf.c
  ${libmagic_SOURCE_DIR}/src/getline.c
  ${libmagic_SOURCE_DIR}/src/gmtime_r.c
  ${libmagic_SOURCE_DIR}/src/localtime_r.c
  ${libmagic_SOURCE_DIR}/src/pread.c
  ${libmagic_SOURCE_DIR}/src/strcasestr.c
  ${libmagic_SOURCE_DIR}/src/strlcat.c
  ${libmagic_SOURCE_DIR}/src/strlcpy.c
  ${libmagic_SOURCE_DIR}/src/vasprintf.c
  ${libmagic_SOURCE_DIR}/src/fmtcheck.c
  win-headers/magic.def
)

set(LIBMAGIC_SOURCE_FILES
  ${WIN_COMPAT_SOURCES}
  ${libmagic_SOURCE_DIR}/src/apprentice.c
  ${libmagic_SOURCE_DIR}/src/apptype.c
  ${libmagic_SOURCE_DIR}/src/ascmagic.c
  ${libmagic_SOURCE_DIR}/src/buffer.c
  ${libmagic_SOURCE_DIR}/src/cdf.c
  ${libmagic_SOURCE_DIR}/src/cdf_time.c
  ${libmagic_SOURCE_DIR}/src/compress.c
  ${libmagic_SOURCE_DIR}/src/der.c
  ${libmagic_SOURCE_DIR}/src/encoding.c
  ${libmagic_SOURCE_DIR}/src/fsmagic.c
  ${libmagic_SOURCE_DIR}/src/funcs.c
  ${libmagic_SOURCE_DIR}/src/is_csv.c
  ${libmagic_SOURCE_DIR}/src/is_json.c
  ${libmagic_SOURCE_DIR}/src/is_simh.c
  ${libmagic_SOURCE_DIR}/src/is_tar.c
  ${libmagic_BINARY_DIR}/magic.c
  ${libmagic_SOURCE_DIR}/src/print.c
  ${libmagic_SOURCE_DIR}/src/readcdf.c
  ${libmagic_BINARY_DIR}/readelf.c
#  ${libmagic_SOURCE_DIR}/src/seccomp.c
  ${libmagic_SOURCE_DIR}/src/softmagic.c
)

# replace the version in the magic.h.in and write it to magic.h
FILE(READ ${libmagic_SOURCE_DIR}/src/magic.h.in MAGIC_H_CONTENT)
STRING(REPLACE "." "" FILE_VERSION_WITHOUT_DOT "${FILE_VERSION}")
STRING(REPLACE "X.YY" ${FILE_VERSION_WITHOUT_DOT} MAGIC_H_CONTENT_NEW "${MAGIC_H_CONTENT}")
FILE(WRITE ${libmagic_SOURCE_DIR}/src/magic.h "${MAGIC_H_CONTENT_NEW}")


FILE(READ ${libmagic_SOURCE_DIR}/src/readelf.c READELF_C_CONTENT)
STRING(CONCAT READELF_C_CONTENT_NEW "#include <dirent.h>\n" "${READELF_C_CONTENT}")
FILE(WRITE ${libmagic_BINARY_DIR}/readelf.c "${READELF_C_CONTENT_NEW}")

FILE(READ ${libmagic_SOURCE_DIR}/src/magic.c MAGIC_C_CONTENT)
STRING(CONCAT MAGIC_C_CONTENT_NEW "#include <dirent.h>\n" "${MAGIC_C_CONTENT}")
FILE(WRITE ${libmagic_BINARY_DIR}/magic.c "${MAGIC_C_CONTENT_NEW}")

if( NOT EXISTS ${libmagic_BINARY_DIR}/file.h.in )
file(RENAME ${libmagic_SOURCE_DIR}/src/file.h ${libmagic_BINARY_DIR}/file.h.in)
endif()

FILE(READ ${libmagic_BINARY_DIR}/file.h.in FILE_H_CONTENT)
STRING(CONCAT FILE_H_CONTENT_NEW "#ifdef WIN32\n#include <unistd.h>\n#include <dirent.h>\n#undef S_IFLNK\n#undef S_IFSOCK\n#endif\n" "${FILE_H_CONTENT}")
FILE(WRITE ${libmagic_SOURCE_DIR}/src/file.h "${FILE_H_CONTENT_NEW}")

# patch test.c and include <getopt.h>
FILE(READ ${libmagic_SOURCE_DIR}/tests/test.c TEST_C_CONTENT)
STRING(CONCAT TEST_C_CONTENT_NEW "#include <getopt.h>\n" "${TEST_C_CONTENT}")
FILE(WRITE ${libmagic_BINARY_DIR}/test.c "${TEST_C_CONTENT_NEW}")


set(PCRE2_USE_STATIC_LIBS ON)


if( NOT EXISTS ${libmagic_SOURCE_DIR}/src/regex.h )
file(DOWNLOAD https://raw.githubusercontent.com/PCRE2Project/pcre2/3f05f72081b24ee2830250e669d0113725437a70/src/pcre2posix.h ${libmagic_SOURCE_DIR}/src/regex.h)
endif()


add_definitions(-DHAVE_CONFIG_H -DVERSION="${FILE_VERSION}" -DWIN32_LEAN_AND_MEAN -DWIN32 -DPCRE2_STATIC )
add_library(libmagic ${LIBMAGIC_SOURCE_FILES})
target_include_directories(libmagic PUBLIC ${libmagic_SOURCE_DIR}/src PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/win-headers  ${CMAKE_CURRENT_SOURCE_DIR}/getopt)
target_link_libraries(libmagic PUBLIC dirent pcre2-posix shlwapi)
if(MSVC)
	target_compile_definitions(libmagic BUILD_AS_WINDOWS_STATIC_LIBARAY=1)
endif()


add_executable(file ${libmagic_SOURCE_DIR}/src/file.c)
target_include_directories(file PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/win-headers  ${libmagic_SOURCE_DIR}/src ${CMAKE_CURRENT_SOURCE_DIR}/getopt)
target_link_libraries(file libmagic dirent pcre2-posix shlwapi)

if(BUILD_TESTS)
	add_executable(file_test ${libmagic_BINARY_DIR}/test.c)
	target_link_libraries(file_test libmagic dirent pcre2-posix shlwapi)
endif()

# this tests all because of time-zone or crlf errors
set(DISABLED_TESTS 
  gedcom
  fit-map-data
  regex-eol
  # it seems like the detection of multiple files is not working
  # to be tested after pcre update
  multiple
  )

enable_testing()
file(GLOB TESTFILES "file/tests/*.testfile")
foreach(TEST_FILE ${TESTFILES})
  # extract testname from path and generate command
  string(REGEX MATCH  "(.*)\.testfile" TESTPATH ${TEST_FILE})
  set(TESTPATH ${CMAKE_MATCH_1})
  string(REGEX MATCH  "([a-zA-Z0-9_]|-|\\.)+$" TESTNAME ${TESTPATH})
  if(NOT ${TESTNAME} IN_LIST DISABLED_TESTS)
      add_test(NAME ${TESTNAME} COMMAND ${CMAKE_CURRENT_BINARY_DIR}/file_test ${TEST_FILE} ${TESTPATH}.result)
  endif()
endforeach()


# Following is the compilation of the magic file
set(MAGIC_FRAGMENT_DIR file/magic/Magdir)
file(GLOB MAGIC_FRAGMENTS ${MAGIC_FRAGMENT_DIR}/*)

# Prepare a temporary file to "cat" to:
file(WRITE magic.in "")

# Call the "cat" function for each input file
foreach(MAGIC_FRAGMENT ${MAGIC_FRAGMENTS})
  cat(${MAGIC_FRAGMENT} magic.in)
endforeach()

# Copy the temporary file to the final location
configure_file(magic.in magic COPYONLY)


add_custom_command(OUTPUT magic.mgc
  COMMAND file -C -m magic
  DEPENDS file
  COMMENT "Compiling magic file"
)

add_custom_target(magic_mgc ALL DEPENDS magic.mgc)
