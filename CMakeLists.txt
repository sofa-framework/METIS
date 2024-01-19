cmake_minimum_required(VERSION 3.22)
project(metis C)


set(GKRAND ON CACHE BOOL "enable GKRAND support?" FORCE)

include(SubmoduleInit.cmake)
get_filename_component(GKLIB_PATH "GKlib" ABSOLUTE)
include(GKlib/GKlibSystem.cmake)

if("x${CMAKE_CXX_COMPILER_ID}" STREQUAL "xGNU" OR "x${CMAKE_CXX_COMPILER_ID}" STREQUAL "xClang")
    add_compile_options(-fPIC)
endif()

## Set precision used in metis and generate corresponding header file
option(USE_64BIT_IDX "Use long int for ids" OFF)
option(USE_64BIT_REAL "Use double precision" OFF)


set(IDXTYPE  32)
set(REALTYPE 32)

if(USE_64BIT_IDX)
    set(IDXTYPE 64)
endif()
if(USE_64BIT_REAL)
    set(REALTYPE  64)
endif()
configure_file(include/metis.h.in include/metis.h @ONLY)

## Find sources.
file(GLOB METIS_SRC libmetis/*.c)
add_library(${PROJECT_NAME} SHARED ${GKlib_sources} ${METIS_SRC})
target_include_directories(${PROJECT_NAME} PRIVATE GKlib)
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/include) #Include the generated metis file



## Artifacts from old cmake used in SOFA
if(UNIX)
    target_link_libraries(${PROJECT_NAME} m)
endif()

if(WIN32)
    # remove warnings about deprecation (CRT,etc)
    target_compile_options(${PROJECT_NAME} PRIVATE "/wd4996")
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

## Install configuration
set_target_properties(${PROJECT_NAME} PROPERTIES
        FRAMEWORK TRUE
        PUBLIC_HEADER ${CMAKE_CURRENT_BINARY_DIR}/include/metis.h)


include(CMakePackageConfigHelpers)
configure_package_config_file(MetisConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/MetisConfig.cmake
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake)


install(TARGETS ${PROJECT_NAME})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/MetisConfig.cmake
        DESTINATION cmake)
