cmake_minimum_required(VERSION 3.15)

project(LULESH CXX)

set(CMAKE_CXX_COMPILER "clang++")
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_EXTENSIONS OFF)

option(WITH_MPI    "Build LULESH with MPI"          FALSE)
option(WITH_OPENMP "Build LULESH with OpenMP"       TRUE)
option(WITH_SILO   "Build LULESH with silo support" FALSE)

# Compile flags
set(BUILD_FLAGS "-fopenmp -g -O3 -march=native")
set(OMP_INC $ENV{OPENMP_INC})
set(OMP_LIB $ENV{OPENMP_LIB})


if (WITH_MPI)
  find_package(MPI REQUIRED)
  include_directories(${MPI_C_INCLUDE_PATH} ${MPI_CXX_INCLUDE_PATH})
  add_definitions("-DUSE_MPI=1")
  list(APPEND LULESH_EXTERNAL_LIBS ${MPI_C_LIBRARIES} ${MPI_CXX_LIBRARIES})
else()
  add_definitions("-DUSE_MPI=0")
endif()

if (WITH_OPENMP)
  list(APPEND LULESH_EXTERNAL_LIBS ${OMP_LIB}/libomp.so)
  set(INCLUDE_DIRECTORIES ${INCLUDE_DIRECTORIES} ${OMP_INC})
  include_directories(${INCLUDE_DIRECTORIES})
endif()

if (WITH_SILO)
  find_path(SILO_INCLUDE_DIR silo.h
    HINTS ${SILO_DIR}/include)
  find_library(SILO_LIBRARY
    NAMES siloh5
    HINTS ${SILO_DIR}/lib)

  include(FindPackageHandleStandardArgs)
  find_package_handle_standard_args(SILO DEFAULT_MSG
    SILO_LIBRARY
    SILO_INCLUDE_DIR)

  if (SILO_FOUND)
    add_definitions("-DVIZ_MESH")
    include_directories(${SILO_INCLUDE_DIR})
    #   Note: silo needs to be built as a dynamic lib, otherwise 
    # there are additional dependencies (hdf5) which we don't know.
    # This would be fixed by silo providing a CMake package.
    list(APPEND LULESH_EXTERNAL_LIBS ${SILO_LIBRARY})
  endif()
endif()


set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${BUILD_FLAGS}")

set(LULESH_SOURCES
  lulesh-comm.cc
  lulesh-init.cc
  lulesh-util.cc
  lulesh-viz.cc
  lulesh.cc)

set(LULESH_EXEC lulesh2.0)

add_executable(${LULESH_EXEC} ${LULESH_SOURCES})
target_link_libraries(${LULESH_EXEC} PRIVATE ${LULESH_EXTERNAL_LIBS})