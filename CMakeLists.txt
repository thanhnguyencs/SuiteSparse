cmake_minimum_required(VERSION 2.8.9)

project(SuiteSparse)

set(SuiteSparse_WORLD_VERSION 1)
set(SuiteSparse_MAJOR_VERSION 0)
set(SuiteSparse_MINOR_VERSION 0)
message(STATUS "Welcome! SuiteSparse version ${SuiteSparse_COLOR_BLUE}${SuiteSparse_WORLD_VERSION}.${SuiteSparse_MAJOR_VERSION}.${SuiteSparse_MINOR_VERSION}${SuiteSparse_COLOR_RESET}")

# common settings
set(SuiteSparse_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(SuiteSparse_LIB_PREFIX SuiteSparse_)
set(SuiteSparse_CXX_FLAGS)

message(STATUS "SuiteSparse_ROOT_DIR:${SuiteSparse_COLOR_BLUE}${SuiteSparse_ROOT_DIR}${SuiteSparse_COLOR_RESET}")
message(STATUS "SuiteSparse_LIB_PREFIX:${SuiteSparse_COLOR_BLUE}${SuiteSparse_LIB_PREFIX}${SuiteSparse_COLOR_RESET}")

## Build Options ===============================================================


## General settings ============================================================

# postfix, based on type
SET(CMAKE_DEBUG_POSTFIX "_d" CACHE STRING "postfix applied to debug build of libraries")
SET(CMAKE_RELEASE_POSTFIX "" CACHE STRING "postfix applied to release build of libraries")
SET(CMAKE_RELWITHDEBINFO_POSTFIX "_rd" CACHE STRING "postfix applied to release-with-debug-information libraries")
SET(CMAKE_MINSIZEREL_POSTFIX "_s" CACHE STRING "postfix applied to minimium-size-build libraries")

# work out the postfix; required where we use OUTPUT_NAME
IF(CMAKE_BUILD_TYPE MATCHES Release)
  SET(EXE_POSTFIX)
ELSEIF(CMAKE_BUILD_TYPE MATCHES Debug)
  SET(EXE_POSTFIX ${CMAKE_DEBUG_POSTFIX})
ELSEIF(CMAKE_BUILD_TYPE MATCHES RelWithDebInfo)
  SET(EXE_POSTFIX ${CMAKE_RELWITHDEBINFO_POSTFIX})
ELSEIF(CMAKE_BUILD_TYPE MATCHES MinSizeRel)
  SET(EXE_POSTFIX ${CMAKE_MINSIZEREL_POSTFIX})
ENDIF(CMAKE_BUILD_TYPE MATCHES Release)

set(SuiteSparse_RUNTIME_OUTPUT_DIRECTORY ${SuiteSparse_ROOT_DIR}/bin CACHE PATH "Target for the binaries")
set(SuiteSparse_LIBRARY_OUTPUT_DIRECTORY ${SuiteSparse_ROOT_DIR}/lib CACHE PATH "Target for the libraries")
link_directories(${SuiteSparse_LIBRARY_OUTPUT_DIRECTORY})

# set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${SuiteSparse_LIBRARY_OUTPUT_DIRECTORY})
# set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${SuiteSparse_LIBRARY_OUTPUT_DIRECTORY})
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${SuiteSparse_RUNTIME_OUTPUT_DIRECTORY})

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${SuiteSparse_LIBRARY_OUTPUT_DIRECTORY})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${SuiteSparse_LIBRARY_OUTPUT_DIRECTORY})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${SuiteSparse_RUNTIME_OUTPUT_DIRECTORY})

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${SuiteSparse_LIBRARY_OUTPUT_DIRECTORY})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE ${SuiteSparse_LIBRARY_OUTPUT_DIRECTORY})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${SuiteSparse_RUNTIME_OUTPUT_DIRECTORY})

add_definitions(-DGL_GLEXT_PROTOTYPES=1 -DUSE_EIGEN_OPENGLSUPPORT)

# Detect OS and define macros appropriately
if(WIN32)
  add_definitions(-DWINDOWS)
  message(STATUS "Compiling on Windows")
  if(MSVC)
    message(STATUS "Compiling with MSVC")
    add_definitions(-DNOMINMAX)
    add_definitions(-D_USE_MATH_DEFINES)
    # SSE2 optimizations
    add_definitions("/arch:SSE2")
    if (BUILD_SHARED_LIBS)
      # disable warning on missing DLL interfaces
      add_definitions("/wd4251")
    endif()
  endif(MSVC)
else()
  message(FATAL_ERROR  "This only supports build on Windows")
endif(WIN32)

set(CMAKE_CXX_FLAGS ${SuiteSparse_CXX_FLAGS})

# Allow the developer to select if Dynamic or Static libraries are built
option(BUILD_SHARED_LIBS "Build Shared Libraries (preferred and required for the g2o plugin system)" OFF)
set(SuiteSparse_LIB_TYPE STATIC)
if(BUILD_SHARED_LIBS)
  set(SuiteSparse_LIB_TYPE SHARED)
endif()


## Dependencies configuration ==================================================

## External libraries (included in this repo) ==================================
message(STATUS "CMAKE_CXX_FLAGS:${CMAKE_CXX_FLAGS}")
# Metis
set(EIGEN_INCLUDE_INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/include/SuiteSparse_external)
add_subdirectory(external/metis-5.1.0)
include_directories(external/metis-5.1.0/include)

## SuiteSparse libraries ============================================================
set(SuiteSparse_LIB_NAMES  SuiteSparse_config AMD CAMD COLAMD CCOLAMD)  # core components


include_directories("${CMAKE_CURRENT_SOURCE_DIR}/SuiteSparse_config")

foreach(LIB_NAME ${SuiteSparse_LIB_NAMES})
  include_directories("${CMAKE_CURRENT_SOURCE_DIR}/${LIB_NAME}/Include")
  add_subdirectory(${LIB_NAME})
endforeach(LIB_NAME ${SuiteSparse_LIB_NAMES})

message(STATUS "CMAKE_BINARY_DIR:${CMAKE_BINARY_DIR}")

# Installation =================================================================
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/SuiteSparseConfig.cmake.in"
               "${CMAKE_CURRENT_BINARY_DIR}/SuiteSparseConfig.cmake"
               IMMEDIATE @ONLY)

#install FindSuiteSparse.cmake to common destination
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/SuiteSparseConfig.cmake
        DESTINATION ${CMAKE_INSTALL_PREFIX}/share/SuiteSparse)

# uninstallation ===============================================================
# uninstall target
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
               "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
               IMMEDIATE @ONLY)

add_custom_target(uninstall
  COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

# # Useful paths configuration for windows 
# if(WIN32)
#   configure_file("${CMAKE_CURRENT_SOURCE_DIR}/set_paths.bat.in" 
#                          "${CMAKE_CURRENT_SOURCE_DIR}/set_paths.bat")
#   configure_file("${CMAKE_CURRENT_SOURCE_DIR}/set_paths_and_run_vc.bat.in"
#                          "${CMAKE_CURRENT_SOURCE_DIR}/set_paths_and_run_vc.bat")
# endif()
