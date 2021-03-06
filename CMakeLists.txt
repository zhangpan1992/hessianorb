# $Id$
cmake_minimum_required(VERSION 2.8)

project(hessianorb)
include(macros.cmake)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY
        ${CMAKE_BINARY_DIR}/lib
        CACHE PATH "Single directory for all static libraries")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY
        ${CMAKE_BINARY_DIR}/bin
        CACHE PATH "Single directory for all executables")

enable_testing()

# testing framework
include_directories(testing/include)
add_subdirectory(testing/src)

# common Ant scripts
copy_file(default.properties ${PROJECT_BINARY_DIR}/default.properties)
copy_file(target.xml ${PROJECT_BINARY_DIR}/target.xml)

# code generator
set(HESSIANC_EXE ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/hessianc)
add_subdirectory(codegen)

# runtime library
include_directories(runtime/include)
add_subdirectory(runtime/src)
add_subdirectory(runtime/test)

# sample application
add_subdirectory(sample)

# Get version from tag.
find_package(Subversion)
if(Subversion_FOUND)
    execute_process(
            COMMAND svnversion -c ${CMAKE_SOURCE_DIR}
            OUTPUT_VARIABLE SVN_REVISION_RANGE
            OUTPUT_STRIP_TRAILING_WHITESPACE)
    string(REGEX REPLACE "^[0-9]+:" "" BUILD_NUMBER ${SVN_REVISION_RANGE})
else()
    set(BUILD_NUMBER UNKNOWN)
endif()

if(Subversion_FOUND AND NOT (BUILD_NUMBER STREQUAL "exported"))
    Subversion_WC_INFO(${CMAKE_SOURCE_DIR} SVN)
    string(REGEX REPLACE ".*/([^/]*)$" "\\1" CPACK_PACKAGE_VERSION ${SVN_WC_URL})
    message(STATUS "Version ${CPACK_PACKAGE_VERSION}")
    message(STATUS "Build number ${BUILD_NUMBER}")
else()
    set(CPACK_PACKAGE_VERSION UNKNOWN)
endif()

# Define target "package_source" to create source distribution.
set(CPACK_SOURCE_PACKAGE_FILE_NAME ${CMAKE_PROJECT_NAME}-${CPACK_PACKAGE_VERSION}.${BUILD_NUMBER})
set(CPACK_SOURCE_IGNORE_FILES "/\\\\.svn/;/build/")
if(WIN32)
    set(CPACK_GENERATOR ZIP)
    set(CPACK_SOURCE_GENERATOR ZIP)
else()
    set(CPACK_GENERATOR TGZ)
    set(CPACK_SOURCE_GENERATOR TGZ)
endif()
include(CPack)
