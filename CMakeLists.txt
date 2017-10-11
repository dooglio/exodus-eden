##===============================================================================
## Copyright (c) 2017 Exodus
## 
## All Rights Reserved.
## 
## See LICENSE for licensing.
##===============================================================================
cmake_minimum_required( VERSION 3.5 )

project( exodus-eden )

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

configure_file( ${CMAKE_SOURCE_DIR}/exodus-eden.desktop.in ${CMAKE_BINARY_DIR}/exodus-eden.desktop )
configure_file( ${CMAKE_SOURCE_DIR}/debian/postinst.in     ${CMAKE_SOURCE_DIR}/debian/postinst     )
configure_file( ${CMAKE_SOURCE_DIR}/debian/postrm.in       ${CMAKE_SOURCE_DIR}/debian/postrm       )

install(
    PROGRAMS ${CMAKE_SOURCE_DIR}/bin/exodus-installer.sh
    DESTINATION share/exodus
    RENAME exodus-installer-eden
)

install(
    FILES ${CMAKE_SOURCE_DIR}/LICENSE
    DESTINATION share/doc/exodus-eden
)

install(
    FILES ${CMAKE_BINARY_DIR}/exodus-eden.desktop
    DESTINATION share/applications
)

install(
    FILES ${CMAKE_SOURCE_DIR}/exodus-eden.png
    DESTINATION share/icons
)

## vim: ts=4 sw=4 et
