# Declare project
PROJECT(DCMTK)

# Minimum CMake version required
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

# Check the build system
INCLUDE(CMake/dcmtkPrepare.cmake NO_POLICY_SCOPE)

#-----------------------------------------------------------------------------
# General project settings to configure DCMTK build process
#-----------------------------------------------------------------------------

# Modules to be built
SET(DCMTK_LIBS ofstd oflog dcmdata dcmimgle
  dcmimage dcmjpeg dcmjpls dcmtls dcmnet dcmsr
  dcmsign dcmwlm dcmqrdb dcmpstat dcmrt)

SET(DCMTK_MODULES config doxygen ofstd oflog dcmdata dcmimgle
  dcmimage dcmjpeg dcmjpls dcmtls dcmnet dcmsr
  dcmsign dcmwlm dcmqrdb dcmpstat dcmrt)

set(DCMTK_LIBRARIES ${DCMTK_LIBS} ijg8 ijg12 ijg16 charls dcmdsig)

#-----------------------------------------------------------------------------
# Include directories
#-----------------------------------------------------------------------------
set(DCMTK_MODULES_W_INCLUDE
  config
  ofstd
  oflog
  dcmdata
  dcmimgle
  dcmimage
  dcmjpeg
  dcmjpeg
  dcmjpeg
  dcmjpeg
  dcmjpls
  dcmnet
  dcmpstat
  dcmqrdb
  dcmsign
  dcmsr
  dcmtls
  dcmwlm
)

SET(DCMTK_INCLUDE_DIR)
foreach(inc ${DCMTK_MODULES_W_INCLUDE})
  if(inc STREQUAL config)
    list(APPEND DCMTK_INCLUDE_DIR ${DCMTK_BINARY_DIR}/config/include)
  else()
    list(APPEND DCMTK_INCLUDE_DIR ${DCMTK_SOURCE_DIR}/${inc}/include)
  endif()
endforeach()

INCLUDE_DIRECTORIES(${DCMTK_INCLUDE_DIR})

# set paths for install -- empty initially
# Offer the user the choice of overriding the installation directories
set(INSTALL_BIN_DIR CACHE PATH "Installation directory for libraries")
set(INSTALL_LIB_DIR CACHE PATH "Installation directory for libraries")
set(INSTALL_INCLUDE_DIR CACHE PATH "Installation directory for include")
# set suffix for CMake files used for packaging
if(WIN32 AND NOT CYGWIN)
  set(INSTALL_CMAKE_DIR cmake)
else()
  set(INSTALL_CMAKE_DIR lib/cmake/dcmtk)
endif()

# Make relative paths absolute (needed later)
foreach(p LIB BIN INCLUDE CMAKE)
  set(var INSTALL_${p}_DIR)
  if(NOT IS_ABSOLUTE "${${var}}")
    set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
  endif()
endforeach()

#-----------------------------------------------------------------------------
# Prepare osconfig.h
#-----------------------------------------------------------------------------

# add the osconfig.h.in file
CONFIGURE_FILE(${DCMTK_SOURCE_DIR}/CMake/osconfig.h.in
  ${DCMTK_BINARY_DIR}/config/include/dcmtk/config/osconfig.h)

#-----------------------------------------------------------------------------
# Start actual compilation tasks
#-----------------------------------------------------------------------------

# Recurse into subdirectories
FOREACH(module ${DCMTK_MODULES})
  ADD_SUBDIRECTORY(${module})
ENDFOREACH(module)
INCLUDE(CMake/dcmtkAfterModules.cmake NO_POLICY_SCOPE)

#-----------------------------------------------------------------------------
# exports
#-----------------------------------------------------------------------------
if(BUILD_SINGLE_SHARED_LIBRARY)
  set(EXPORT_LIBS dcmtk)
else()
  set(EXPORT_LIBS ${DCMTK_LIBRARIES})
endif()

export(TARGETS ${EXPORT_LIBS}
  FILE ${PROJECT_BINARY_DIR}/DCMTKLibraryDepends.cmake)

export(PACKAGE DCMTK)

#
# determine where include is relative to the CMake dir in
# in installed tree
file(RELATIVE_PATH CONF_REL_INCLUDE_DIR "${INSTALL_CMAKE_DIR}"
  "${INSTALL_INCLUDE_DIR}")

#-----------------------------------------------------------------------------
# Write current configuration to file (for external projects)
#-----------------------------------------------------------------------------
set(DCMTKCONFIG_INCLUDE_DIR_CODE
  "include(\"\${DCMTK_CMAKE_DIR}/DCMTKBuildTreeSettings.cmake\")")
configure_file(${DCMTK_SOURCE_DIR}/CMake/DCMTKConfig.cmake.in
  ${DCMTK_BINARY_DIR}/DCMTKConfig.cmake @ONLY)

set(DCMTKCONFIG_INCLUDE_DIR_CODE
"SET(DCMTK_INCLUDE_DIRS)
foreach(inc \${DCMTK_INCLUDE_DIR_LIST})
  list(APPEND DCMTK_INCLUDE_DIRS \${DCMTK_CMAKE_DIR}/${CONF_REL_INCLUDE_DIR}include/dcmtk/\${inc})
endforeach()
list(APPEND DCMTK_INCLUDE_DIRS \${DCMTK_CMAKE_DIR}/${CONF_REL_INCLUDE_DIR}include)"
)
configure_file(${DCMTK_SOURCE_DIR}/CMake/DCMTKConfig.cmake.in
  CMakeFiles/DCMTKConfig.cmake @ONLY)

#-----------------------------------------------------------------------------
# Export version # checked by find_package
#-----------------------------------------------------------------------------
configure_file(${DCMTK_SOURCE_DIR}/CMake/DCMTKConfigVersion.cmake.in
  "${PROJECT_BINARY_DIR}/DCMTKConfigVersion.cmake"
  @ONLY)

#
# make a cmake file -- in this case, all that needs defining
# is double-conversion_INCLUDE_DIRS
configure_file(${DCMTK_SOURCE_DIR}/CMake/DCMTKBuildTreeSettings.cmake.in
  "${PROJECT_BINARY_DIR}/DCMTKBuildTreeSettings.cmake"
  @ONLY)

#-----------------------------------------------------------------------------
# Installation tasks
#-----------------------------------------------------------------------------

# Install global headers
INSTALL(FILES ${DCMTK_BINARY_DIR}/config/include/dcmtk/config/osconfig.h
        DESTINATION ${DCMTK_INSTALL_INCDIR}/dcmtk/config
        COMPONENT include)

# Install DCMTK's general documentation files
INSTALL(FILES ANNOUNCE.${DCMTK_PACKAGE_VERSION_NUMBER} CHANGES.${DCMTK_PACKAGE_VERSION_NUMBER} COPYRIGHT FAQ HISTORY VERSION
        DESTINATION ${DCMTK_INSTALL_DOCDIR}
        COMPONENT doc)

# Install DCMTK's CMake configuration file
INSTALL(FILES ${DCMTK_BINARY_DIR}/CMakeFiles/DCMTKConfig.cmake
  ${DCMTK_BINARY_DIR}/DCMTKConfigVersion.cmake
  DESTINATION ${INSTALL_CMAKE_DIR}
  COMPONENT dev)

install(EXPORT DCMTKLibraryDepends DESTINATION
      "${INSTALL_CMAKE_DIR}" COMPONENT dev)
