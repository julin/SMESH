# SMESH cmake build file
project(SMESH C CXX)
cmake_minimum_required(VERSION 2.8)

# --------------------------------------------------------------------------- #
# OPTIONS
# --------------------------------------------------------------------------- #
option(ENABLE_NETGEN "Enable NETGEN" ON)
option(ENABLE_BLSURF "Enable BLSURF" OFF)
set(CMAKE_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}/install" CACHE PATH "Installation directory.")


# --------------------------------------------------------------------------- #
# SETTINGS
# --------------------------------------------------------------------------- #
set(SMESH_VERSION_MAJOR 8)
set(SMESH_VERSION_MINOR 3)
set(SMESH_VERSION_PATCH 0)
set(SMESH_VERSION_TWEAK 0)

# Build shared libraries
set(BUILD_SHARED_LIBS ON)

# Includes
include_directories(inc)

# --------------------------------------------------------------------------- #
# OpenCASCADE
# --------------------------------------------------------------------------- #
if(NOT DEFINED OpenCASCADE_INCLUDE_DIR OR NOT DEFINED OpenCASCADE_LIBRARY_DIR)
    message(STATUS "Searching for OpenCASCADE...")
    find_package(OpenCASCADE 7.2.0 REQUIRED)
endif()

if(NOT EXISTS ${OpenCASCADE_INCLUDE_DIR})
    message(FATAL_ERROR "Failed to find OpenCASCADE include directory.")
endif()

if(NOT EXISTS ${OpenCASCADE_LIBRARY_DIR})
    message(FATAL_ERROR "Failed to find OpenCASCADE library directory.")
endif()
                             
message(STATUS "OpenCASCADE include directory: ${OpenCASCADE_INCLUDE_DIR}")
message(STATUS "OpenCASCADE library directory: ${OpenCASCADE_LIBRARY_DIR}")                                                                                
include_directories(${OpenCASCADE_INCLUDE_DIR})
link_directories(${OpenCASCADE_LIBRARY_DIR})


# --------------------------------------------------------------------------- #
# VTK
# --------------------------------------------------------------------------- #
if(NOT DEFINED VTK_INCLUDE_DIRS OR NOT DEFINED VTK_LIBRARIES)
    message(STATUS "Searching for VTK...")
    find_package(VTK REQUIRED)
endif()
    
if(NOT DEFINED VTK_INCLUDE_DIRS)
    message(FATAL_ERROR "Failed to define VTK include directories.")
endif()

if(NOT DEFINED VTK_LIBRARIES)
    message(FATAL_ERROR "Failed to define VTK libraries.")
endif()
    
message(STATUS "VTK include directories: ${VTK_INCLUDE_DIRS}")
include_directories(${VTK_INCLUDE_DIRS})


# --------------------------------------------------------------------------- #
# BOOST
# --------------------------------------------------------------------------- #
if(NOT DEFINED Boost_INCLUDE_DIR)
    message(STATUS "Searching for Boost...")
    find_package(Boost REQUIRED)
endif()

if(NOT EXISTS ${Boost_INCLUDE_DIR})
    message(FATAL_ERROR "Failed to find Boost include directory.")
endif()

message(STATUS "Boost include directory: ${Boost_INCLUDE_DIR}")
include_directories(${Boost_INCLUDE_DIR})


# --------------------------------------------------------------------------- #
# PTHREAD
# --------------------------------------------------------------------------- #
set(PTHREAD_INCLUDE_DIRS "" CACHE PATH "pthread include directory.")
if(NOT EXISTS ${PTHREAD_INCLUDE_DIRS})
    message(FATAL_ERROR "pthread include directory is required.")
endif()
message(STATUS "pthread include directory: ${PTHREAD_INCLUDE_DIRS}")
include_directories(${PTHREAD_INCLUDE_DIRS})

set(PTHREAD_LIB_DIRS "" CACHE PATH "pthread library directory.")
if(NOT EXISTS ${PTHREAD_LIB_DIRS})
    message(FATAL_ERROR "pthread library directory is required.")
endif()
message(STATUS "pthread library directory: ${PTHREAD_LIB_DIRS}")
link_directories(${pthread_LIB_DIRS})


# --------------------------------------------------------------------------- #
# NETGEN
# --------------------------------------------------------------------------- #
if(ENABLE_NETGEN)
    if(NOT DEFINED NETGEN_INCLUDE_DIR OR NOT DEFINED NETGEN_LIBRARY_DIR)
        message(STATUS "Searching for Netgen...")
        find_package(Netgen REQUIRED)
    endif()
    
    if(NOT EXISTS ${NETGEN_INCLUDE_DIR})
        message(FATAL_ERROR "Failed to find Netgen include directory.")
    endif()
    
    if(NOT EXISTS ${NETGEN_LIBRARY_DIR})
        message(FATAL_ERROR "Failed to find Netgen library directory.")
    endif()
    
    message(STATUS "Netgen include directory: ${NETGEN_INCLUDE_DIR}")
    message(STATUS "Netgen library directory: ${NETGEN_LIBRARY_DIR}")
    
    link_directories(${NETGEN_LIBRARY_DIR})
    include_directories(${NETGEN_INCLUDE_DIR})
    include_directories(${NETGEN_INCLUDE_DIR}/occ)
    include_directories(${NETGEN_INCLUDE_DIR}/meshing)
    include_directories(${NETGEN_INCLUDE_DIR}/general)
    include_directories(${NETGEN_INCLUDE_DIR}/linalg)
    include_directories(${NETGEN_INCLUDE_DIR}/csg)
    include_directories(${NETGEN_INCLUDE_DIR}/geom2d)
    include_directories(${NETGEN_INCLUDE_DIR}/gprim)
    include_directories(${NETGEN_INCLUDE_DIR}/include)
    include_directories(${NETGEN_INCLUDE_DIR}/stlgeom)
    
    # Set NETGEN_VERSION
    file(STRINGS ${NETGEN_INCLUDE_DIR}/mydefs.hpp NETGEN_VERSION
        REGEX "#define PACKAGE_VERSION.*"
        )
    
    if(NETGEN_VERSION)
        string(REGEX MATCHALL "[0-9]+" NETGEN_VERSION ${NETGEN_VERSION})
        list(LENGTH NETGEN_VERSION NETGEN_VERSION_COUNT)
        list(GET NETGEN_VERSION 0 NETGEN_VERSION_MAJOR)
        if(NETGEN_VERSION_COUNT GREATER 1)
            list(GET NETGEN_VERSION 1 NETGEN_VERSION_MINOR)
        else()
            set(NETGEN_VERSION_MINOR 0)
        endif()        
    else()
        # Assume version 6.2.
        message(STATUS "WARNING: Assuming Netgen 6.2.")
        set(NETGEN_VERSION_MAJOR 6)
        set(NETGEN_VERSION_MINOR 2)
    endif()

    MATH(EXPR NETGEN_VERSION "(${NETGEN_VERSION_MAJOR} << 16) + (${NETGEN_VERSION_MINOR} << 8)")
    message(STATUS "Using Netgen version ${NETGEN_VERSION_MAJOR}.${NETGEN_VERSION_MINOR}, calculated: ${NETGEN_VERSION}")
    
endif()


# --------------------------------------------------------------------------- #
# BLSURF
# --------------------------------------------------------------------------- #
if(ENABLE_BLSURF)
    set(MESHGEMS_INCLUDE "" CACHE PATH "MeshGems include directory.")
    if(NOT EXISTS ${MESHGEMS_INCLUDE})
        message(FATAL_ERROR "MeshGems include directory is required.")
    endif()
    
    set(MESHGEMS_LIB "" CACHE FILE "MeshGems library.")
    if(NOT EXISTS ${MESHGEMS_LIB})
        message(FATAL_ERROR "MeshGems library is required.")
    endif()
    
    set(MESHGEMS_CADSURF_LIB "" CACHE FILE "MeshGems MG-CADSurf library.")
    if(NOT EXISTS ${MESHGEMS_CADSURF_LIB})
        message(FATAL_ERROR "MeshGems MG-CADSurf library is required.")
    endif()
    
endif()


# --------------------------------------------------------------------------- #
# SMESH
# --------------------------------------------------------------------------- #
# Build settings according to the platform
if(UNIX)
    # Same settings are used for both MacOSX and Unix/Linux
    add_definitions(-DHAVE_CONFIG_H -DHAVE_LIMITS_H -DCSFDB -DLIN -DOCC_CONVERT_SIGNALS)
else(UNIX)
  if(WIN32)
    if(MSVC)
        add_definitions(-DWNT -DWIN32 -D_WINDOWS -DCSFDB -DUSE_CLOCK -DMSDOS -DNO_ONEXIT -DNO_My_ctype -DNO_ISATTY -DNO_FPINIT /wd4290 /wd4251 /wd4018 /wd4800 /wd4996 /wd4244 /wd4805 /wd4806 /wd4275 /wd4005 /wd4099 /wd4101 /wd4146 /wd4267 /wd4390 /wd4503 /wd4436)
    else(MSVC)
        add_definitions(-DWNT -DWIN32 -D_WINDOWS -DCSFDB)
    endif(MSVC)
  else(WIN32)
    message(FATAL_ERROR "Unknown platform")
  endif(WIN32)
endif(UNIX)

# Basics
file(GLOB Basics_SRC src/Basics/*.cxx)
add_library(SMESHBasics ${Basics_SRC})
target_link_libraries(SMESHBasics pthreads)
if(WIN32)
    set_target_properties(SMESHBasics PROPERTIES COMPILE_FLAGS "-DBASICS_EXPORTS")
endif()

# Controls
file(GLOB Controls_SRC src/Controls/*.cxx)
add_library(SMESHControls ${Controls_SRC})
target_link_libraries(SMESHControls SMDS SMESHDS SMESHUtils TKernel TKBRep TKG3d TKTopAlgo TKGeomBase TKGeomAlgo ${VTK_LIBRARIES})
if(WIN32)
    set_target_properties(SMESHControls PROPERTIES COMPILE_FLAGS "-DSMESHCONTROLS_EXPORTS")
endif()

# Driver
file(GLOB Driver_SRC src/Driver/*.cxx)
add_library(Driver ${Driver_SRC})
target_link_libraries(Driver SMESHMisc SMESHDS SMESHUtils TKernel TKTopAlgo)
if(WIN32)
    set_target_properties(Driver PROPERTIES COMPILE_FLAGS "-DMESHDRIVER_EXPORTS")
endif()

# DriverDAT
file(GLOB DriverDAT_SRC src/DriverDAT/*.cxx)
add_library(DriverDAT ${DriverDAT_SRC})
target_link_libraries(DriverDAT SMESHMisc SMESHTrace SMESHBasics Driver SMDS)
if(WIN32)
    set_target_properties(DriverDAT PROPERTIES COMPILE_FLAGS "-DMESHDRIVERDAT_EXPORTS")
endif()

# DriverGMF
file(GLOB DriverGMF_SRC src/DriverGMF/*.cxx src/DriverGMF/libmesh5.c)
add_library(DriverGMF ${DriverGMF_SRC})
target_link_libraries(DriverGMF SMESHBasics SMESHUtils Driver)
if(WIN32)
    set_target_properties(DriverGMF PROPERTIES COMPILE_FLAGS "-DMESHDriverGMF_EXPORTS")
endif()

# DriverSTL
file(GLOB DriverSTL_SRC src/DriverSTL/*.cxx)
add_library(DriverSTL ${DriverSTL_SRC})
target_link_libraries(DriverSTL Driver SMDS SMESHUtils TKernel TKSTL TKTopAlgo TKMesh)
if(WIN32)
    set_target_properties(DriverSTL PROPERTIES COMPILE_FLAGS "-DMESHDRIVERSTL_EXPORTS")
endif()

# DriverUNV
file(GLOB DriverUNV_SRC src/DriverUNV/*.cxx)
add_library(DriverUNV ${DriverUNV_SRC})
target_link_libraries(DriverUNV SMDS Driver SMESHMisc SMESHTrace SMESHBasics)
if(WIN32)
    set_target_properties(DriverUNV PROPERTIES COMPILE_FLAGS "-DMESHDRIVERUNV_EXPORTS")
endif()

# MEFISTO2
file(GLOB MEFISTO2_SRC src/MEFISTO2/aptrte.cxx src/MEFISTO2/trte.c)
add_library(MEFISTO2 ${MEFISTO2_SRC})
set_target_properties(MEFISTO2 PROPERTIES COMPILE_FLAGS "-DMEFISTO2D_EXPORTS")

# Misc
file(GLOB Misc_SRC src/Misc/*.cxx)
add_library(SMESHMisc ${Misc_SRC})
target_link_libraries(SMESHMisc SMESHTrace TKBRep TKShHealing TKMesh TKGeomAlgo TKTopAlgo TKG2d TKG3d TKV3d TKGeomBase TKBO)
if(WIN32)
    set_target_properties(SMESHMisc PROPERTIES COMPILE_FLAGS "-DUTILS_EXPORTS")
endif()

# NETGENPlugin
if(ENABLE_NETGEN)
    file(GLOB NETGENPlugin_SRC src/NETGENPlugin/*.cxx)
    add_library(NETGENPlugin ${NETGENPlugin_SRC})
    target_link_libraries(NETGENPlugin StdMeshers nglib)
    if(WIN32)
        set_target_properties(NETGENPlugin PROPERTIES COMPILE_FLAGS "-DNETGENPLUGIN_EXPORTS -DNO_PARALLEL_THREADS -DOCCGEOMETRY -DNETGEN_VERSION=${NETGEN_VERSION}")
    else()
        set_target_properties(NETGENPlugin PROPERTIES COMPILE_FLAGS "${NETGEN_CXX_FLAGS}")
    endif()    
endif()

# SMDS
file(GLOB SMDS_SRC src/SMDS/*.cxx)
add_library(SMDS ${SMDS_SRC})
target_link_libraries(SMDS SMESHMisc SMESHTrace ${VTK_LIBRARIES})
if(WIN32)
    set_target_properties(SMDS PROPERTIES COMPILE_FLAGS "-DSMDS_EXPORTS")
endif()

# SMESH
file(GLOB SMESH_SRC src/SMESH/*.cxx)
add_library(SMESH ${SMESH_SRC})
target_link_libraries(SMESH SMESHControls SMESHMisc SMESHDS DriverDAT DriverSTL DriverUNV DriverGMF TKShHealing TKPrim TKG2d TKCDF TKMeshVS)
if(WIN32)
    set_target_properties(SMESH PROPERTIES COMPILE_FLAGS "-DSMESHimpl_EXPORTS")
endif()

# SMESHDS
file(GLOB SMESHDS_SRC src/SMESHDS/*.cxx)
add_library(SMESHDS ${SMESHDS_SRC})
target_link_libraries(SMESHDS SMDS SMESHTrace TKBRep)
if(WIN32)
    set_target_properties(SMESHDS PROPERTIES COMPILE_FLAGS "-DSMESHDS_EXPORTS")
endif()

# StdMeshers
file(GLOB StdMeshers_SRC src/StdMeshers/*.cxx)
add_library(StdMeshers ${StdMeshers_SRC})
target_link_libraries(StdMeshers SMESH MEFISTO2 TKOffset)
if(WIN32)
    set_target_properties(StdMeshers PROPERTIES COMPILE_FLAGS "-DSTDMESHERS_EXPORTS")
else()
    set_target_properties(StdMeshers PROPERTIES COMPILE_FLAGS "${StdMeshers_CFLAGS}")
endif()

# Trace
file(GLOB Trace_SRC src/Trace/*.cxx)
add_library(SMESHTrace ${Trace_SRC})
target_link_libraries(SMESHTrace pthreads SMESHBasics)
if(WIN32)
    set_target_properties(SMESHTrace PROPERTIES COMPILE_FLAGS "-DSALOMELOCALTRACE_EXPORTS")
endif()

# Utils
file(GLOB Utils_SRC src/Utils/*.cxx)
add_library(SMESHUtils ${Utils_SRC})
target_link_libraries(SMESHUtils SMDS TKShHealing TKPrim TKernel TKBRep TKG2d TKG3d TKGeomBase TKGeomAlgo TKTopAlgo TKMesh)
if(WIN32)
    set_target_properties(SMESHUtils PROPERTIES COMPILE_FLAGS "-DSMESHUtils_EXPORTS")
endif()

# BLSURFPlugin
if(ENABLE_BLSURF)
    include_directories(${MESHGEMS_INCLUDE})
    file(GLOB BLSURFPlugin_SRC src/BLSURFPlugin/*.cxx)
    add_library(BLSURFPlugin ${BLSURFPlugin_SRC})
    target_link_libraries(BLSURFPlugin TKBRep TKG2d TKG3d TKGeomBase TKGeomAlgo TKShHealing SMDS SMESHDS SMESH StdMeshers ${MESHGEMS_CADSURF_LIB} ${MESHGEMS_LIB})
    if(WIN32)
        set_target_properties(BLSURFPlugin PROPERTIES COMPILE_FLAGS "-DBLSURFPLUGIN_EXPORTS")
    endif()
endif()

# Install
set(SMESH_LIBRARIES Driver DriverDAT DriverGMF DriverSTL DriverUNV MEFISTO2 SMDS SMESH SMESHBasics SMESHControls SMESHDS SMESHMisc SMESHTrace SMESHUtils StdMeshers)

if(ENABLE_NETGEN)
    set(SMESH_LIBRARIES ${SMESH_LIBRARIES} NETGENPlugin)
endif()

if(ENABLE_BLSURF)
    set(SMESH_LIBRARIES ${SMESH_LIBRARIES} BLSURFPlugin)
endif()

install(TARGETS ${SMESH_LIBRARIES}
        ARCHIVE DESTINATION "lib"
        RUNTIME DESTINATION "bin")

install(DIRECTORY ${CMAKE_SOURCE_DIR}/inc/ DESTINATION "include/smesh")

# Configuration file
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/SMESHConfig.cmake.in
	           ${CMAKE_CURRENT_BINARY_DIR}/SMESHConfig.cmake @ONLY)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/SMESHConfig.cmake DESTINATION cmake)
