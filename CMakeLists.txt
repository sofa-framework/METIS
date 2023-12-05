cmake_minimum_required(VERSION 3.22)
project(METIS C)

set(GKRAND ON CACHE BOOL "enable GKRAND support?" FORCE)

include(GKlib/GKlibSystem.cmake)

# Find sources.
file(GLOB METIS_SRC libmetis/*.c)
add_library(metis SHARED ${GKlib_sources} ${METIS_SRC})
target_include_directories(metis PRIVATE GKlib)
target_include_directories(metis PUBLIC include)
