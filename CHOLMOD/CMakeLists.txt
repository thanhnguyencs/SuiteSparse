cmake_minimum_required(VERSION 2.8)

project(SuiteSparse_CHOLMOD)
file(GLOB AMD_INC "Include/*.h")
file(GLOB AMD_SRC "Source/*.c")

# Library ======================================================================
set(LIB_TARGET CHOLMOD)
add_library(${LIB_TARGET} ${SUITE_SPARSE_LIB_TYPE} ${CHOLMOD_SRC} ${CHOLMOD_INC})
target_link_libraries(${LIB_TARGET})

# Definitions ==================================================================
if(WIN32)
  add_definitions(/D_CRT_SECURE_NO_WARNINGS)
endif(WIN32)

# Installation =================================================================
install(TARGETS ${LIB_TARGET}
  RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
  LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
  ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
)

install(FILES ${CHOLMOD_INC} DESTINATION ${CMAKE_INSTALL_PREFIX}/include/)
