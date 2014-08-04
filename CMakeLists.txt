# Copyright (C) 2012, 2013  iCub Facility, Istituto Italiano di Tecnologia
# Author: Daniele E. Domenichelli <daniele.domenichelli@iit.it>
# CopyPolicy: Released under the terms of the LGPLv2.1 or later, see LGPL.TXT

cmake_minimum_required(VERSION 2.8.8)
project(tinyxml CXX)

set(TINYXML_MAJOR_VERSION 2)
set(TINYXML_MINOR_VERSION 6)
set(TINYXML_PATCH_VERSION 2)
set(TINYXML_VERSION ${TINYXML_MAJOR_VERSION}.${TINYXML_MINOR_VERSION}.${TINYXML_PATCH_VERSION})

set(TINYXML_BUILD_LIB_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(TINYXML_BUILD_BIN_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(TINYXML_BUILD_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(TINYXML_BUILD_LIB_DIR ${CMAKE_CURRENT_BINARY_DIR})

# Offer the user the choice of overriding the installation directories
set(TINYXML_INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries")
set(TINYXML_INSTALL_BIN_DIR bin CACHE PATH "Installation directory for executables")
set(TINYXML_INSTALL_INCLUDE_DIR include CACHE PATH "Installation directory for header files")
if(WIN32 AND NOT CYGWIN)
    set(DEF_TINYXML_INSTALL_CMAKE_DIR CMake)
else()
    set(DEF_TINYXML_INSTALL_CMAKE_DIR lib/cmake/TinyXML)
endif()
set(TINYXML_INSTALL_CMAKE_DIR ${DEF_TINYXML_INSTALL_CMAKE_DIR} CACHE PATH "Installation directory for CMake files")
mark_as_advanced(TINYXML_INSTALL_LIB_DIR TINYXML_INSTALL_BIN_DIR TINYXML_INSTALL_INCLUDE_DIR TINYXML_INSTALL_CMAKE_DIR)

# Make relative paths absolute (needed later on)
foreach(p LIB BIN INCLUDE CMAKE)
    set(var TINYXML_BUILD_${p}_DIR)
    if(NOT IS_ABSOLUTE "${${var}}")
      set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
    endif()
    set(var TINYXML_INSTALL_${p}_DIR)
    if(NOT IS_ABSOLUTE "${${var}}")
      set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
    endif()
endforeach()




set(tinyxml_SRCS
    tinystr.cpp
    tinyxml.cpp
    tinyxmlerror.cpp
    tinyxmlparser.cpp
)

set(tinyxml_HDRS
    tinystr.h
    tinyxml.h
)




add_definitions("-DTIXML_USE_STL")

#In Windows (Visual Studio) it is necessary to specify the postfix
#of the debug library name. Furthermore no symbols are exported by 
#TinyXML, so it is necessary to compile it as a static library.
IF(MSVC)
    SET(CMAKE_DEBUG_POSTFIX "d")
    SET(LIB_TYPE STATIC)
ELSE(MSVC)
    SET(LIB_TYPE SHARED)
ENDIF(MSVC)

add_library(tinyxml ${LIB_TYPE} ${tinyxml_SRCS})

set_target_properties(tinyxml PROPERTIES VERSION ${TINYXML_VERSION}
                                         SOVERSION ${TINYXML_VERSION}
                                         PUBLIC_HEADER "${tinyxml_HDRS}")

target_include_directories(tinyxml PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
                                          "$<INSTALL_INTERFACE:${TINYXML_INSTALL_INCLUDE_DIR}>")

install(TARGETS tinyxml
        EXPORT TinyXMLTargets
        RUNTIME DESTINATION "${TINYXML_INSTALL_BIN_DIR}" COMPONENT bin
        LIBRARY DESTINATION "${TINYXML_INSTALL_LIB_DIR}" COMPONENT shlib
		ARCHIVE DESTINATION "${TINYXML_INSTALL_LIB_DIR}" COMPONENT lib
        PUBLIC_HEADER DESTINATION "${TINYXML_INSTALL_INCLUDE_DIR}" COMPONENT dev)
set(TINYXML_TARGETS tinyxml)

include(CMakePackageConfigHelpers)


# TinyXMLConfigVersion.cmake file (same for build tree and intall)
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/TinyXMLConfigVersion.cmake
                                 VERSION ${TINYXML_VERSION}
                                 COMPATIBILITY AnyNewerVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/TinyXMLConfigVersion.cmake
        DESTINATION ${TINYXML_INSTALL_CMAKE_DIR})


# TinyXMLConfig.cmake (build tree)
set(PACKAGE_TINYXML_LIB_DIR "${TINYXML_BUILD_LIB_DIR}")
set(PACKAGE_TINYXML_BIN_DIR "${TINYXML_BUILD_BIN_DIR}")
set(PACKAGE_TINYXML_INCLUDE_DIR "${TINYXML_BUILD_INCLUDE_DIR}")
set(PACKAGE_TINYXML_CMAKE_DIR "${TINYXML_BUILD_CMAKE_DIR}")
configure_package_config_file(TinyXMLConfig.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/TinyXMLConfig.cmake
                              INSTALL_DESTINATION ${CMAKE_CURRENT_BINARY_DIR}
                              PATH_VARS TINYXML_BUILD_LIB_DIR
                                        TINYXML_BUILD_BIN_DIR
                                        TINYXML_BUILD_INCLUDE_DIR
                                        TINYXML_BUILD_CMAKE_DIR)

# TinyXMLConfig.cmake (installed)
set(PACKAGE_TINYXML_LIB_DIR "${TINYXML_INSTALL_LIB_DIR}")
set(PACKAGE_TINYXML_BIN_DIR "${TINYXML_INSTALL_BIN_DIR}")
set(PACKAGE_TINYXML_INCLUDE_DIR "${TINYXML_INSTALL_INCLUDE_DIR}")
set(PACKAGE_TINYXML_CMAKE_DIR "${TINYXML_INSTALL_CMAKE_DIR}")
configure_package_config_file(TinyXMLConfig.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/TinyXMLConfigInstall.cmake
                              INSTALL_DESTINATION ${TINYXML_INSTALL_CMAKE_DIR}
                              PATH_VARS TINYXML_INSTALL_LIB_DIR
                                        TINYXML_INSTALL_BIN_DIR
                                        TINYXML_INSTALL_INCLUDE_DIR
                                        TINYXML_INSTALL_CMAKE_DIR)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/TinyXMLConfigInstall.cmake
        DESTINATION ${TINYXML_INSTALL_CMAKE_DIR}
        RENAME TinyXMLConfig.cmake)


# TinyXMLTargets.cmake (build tree)
export(TARGETS ${TINYXML_TARGETS}
       NAMESPACE "TinyXML::"
       FILE ${CMAKE_CURRENT_BINARY_DIR}/TinyXMLTargets.cmake)

# TinyXMLTargets.cmake (installed)
install(EXPORT TinyXMLTargets
        NAMESPACE "TinyXML::"
        DESTINATION ${TINYXML_INSTALL_CMAKE_DIR})
