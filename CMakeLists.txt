##===============================================================================
## Copyright (c) 2017 Exodus
## 
## All Rights Reserved.
## 
## See LICENSE for licensing.
##===============================================================================
cmake_minimum_required( VERSION 3.5 )

project( exodus )

find_program( DPKG_PARSECHANGELOG dpkg-parsechangelog )
if( ${DPKG_PARSECHANGELOG} STREQUAL ${DPKG_PARSECHANGELOG}-NOTFOUND )
    message( FATAL_ERROR "dpkg-parsechangelog not found! Is dpkg-dev installed?" )
endif()

execute_process(
    COMMAND ${DPKG_PARSECHANGELOG} -S Version
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE VERSION
    )

string( REPLACE "\n" ""  ORIG_VERSION ${VERSION}      )
string( REPLACE "."  " " VERSION      ${ORIG_VERSION} )
separate_arguments( VERSION )

list( LENGTH VERSION LEN )
if( NOT ${LEN} EQUAL 3 )
    message( FATAL_ERROR "Package name '${PACKAGE_NAME}' does not have a valid version number: '${ORIG_VERSION}'. It must have 3 parts: major.minor.patch!" )
endif()

list( GET VERSION 0 major_version  )
list( GET VERSION 1 minor_version  )
list( GET VERSION 2 _patch_version )

string( REPLACE "-"  " " temp_ver ${_patch_version} )
list( LENGTH temp_ver temp_ver_len )
if( ${temp_ver_len} EQUAL 2 )
    #
    # Throw away the debian build number if it is present (e.g. 1.2.3-4, where we throw out the '-4').
    #
    list( GET temp_ver 0 patch_version   )
    list( GET temp_ver 1 package_version )
else()
    set( patch_version ${_patch_version} )
endif()

set( EXODUS_VERSION ${major_version}.${minor_version}.${patch_version} )

set( DOWNLOAD_BASE_URL "https://exodusbin.azureedge.net/releases" CACHE STRING "Base URL for downloading zip files" )
set( PROCESSOR         "x64"                                      CACHE STRING "Processor type"                     )

set( DOWNLOAD_URL      ${DOWNLOAD_BASE_URL}/exodus-linux-${PROCESSOR}-${EXODUS_VERSION}.zip    )
set( PACKAGE_FILENAME  ${CMAKE_BINARY_DIR}/package_target.zip                                  )

if( NOT EXISTS ${PACKAGE_FILENAME} )
    file( DOWNLOAD ${DOWNLOAD_URL} ${PACKAGE_FILENAME} SHOW_PROGRESS )
endif()

set( UNPACK_DIR ${CMAKE_BINARY_DIR}/Exodus-linux-${PROCESSOR} )
if( NOT EXISTS ${UNPACK_DIR} )
    execute_process(
        COMMAND cmake -E tar xvf ${PACKAGE_FILENAME}
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    )
endif()

install(
    DIRECTORY ${UNPACK_DIR}/
    DESTINATION share/exodus
)

## vim: ts=4 sw=4 et
