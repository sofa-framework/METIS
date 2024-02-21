cmake_minimum_required(VERSION 3.22)
project(metis C)


set(GKRAND ON CACHE BOOL "enable GKRAND support?" FORCE)

include(SubmoduleInit.cmake)
get_filename_component(GKLIB_PATH "GKlib" ABSOLUTE)
include(GKlib/GKlibSystem.cmake)

if("x${CMAKE_CXX_COMPILER_ID}" STREQUAL "xGNU" OR "x${CMAKE_CXX_COMPILER_ID}" STREQUAL "xClang")
    add_compile_options(-fPIC)
endif()


if(WIN32)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
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
file(GLOB METIS_SRC libmetis/*.c libmetis/*.h)
add_library(${PROJECT_NAME} SHARED ${GKlib_sources} ${METIS_SRC})
target_include_directories(${PROJECT_NAME} PRIVATE GKlib)
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/libmetis)
target_include_directories(${PROJECT_NAME} PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>) #Include the generated metis file
target_include_directories(${PROJECT_NAME} PUBLIC $<INSTALL_INTERFACE:include>) #Include the generated metis file



## Artifacts from old cmake used in SOFA
if(UNIX)
    target_link_libraries(${PROJECT_NAME} m)
endif()

if(WIN32)
    # remove warnings about deprecation (CRT,etc)
    target_compile_options(${PROJECT_NAME} PRIVATE "/wd4996")
endif()

## Install configuration
set_target_properties(${PROJECT_NAME} PROPERTIES
        FRAMEWORK TRUE
        PUBLIC_HEADER ${CMAKE_CURRENT_BINARY_DIR}/include/metis.h)

include(CMakePackageConfigHelpers)
configure_package_config_file(metisConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/metisConfig.cmake
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

# Install the header file and export the target
install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}Targets DESTINATION ${CMAKE_INSTALL_LIBDIR} PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(EXPORT ${PROJECT_NAME}Targets FILE ${PROJECT_NAME}Targets.cmake  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/metisConfig.cmake DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})


