cmake_minimum_required (VERSION 3.2.0)
project (decomp)

set(CMAKE_CXX_STANDARD 11)
set(TARGET_NAME ${PROJECT_NAME})

option(${PROJECT_NAME}_USE_CONAN "Use conan" OFF)
option(${PROJECT_NAME}_BUILD_TESTS "Build tests" OFF)

if (${PROJECT_NAME}_BUILD_TESTS AND NOT ${PROJECT_NAME}_USE_CONAN )
  message(FATAL_ERROR "Cannot build tests without conan")
endif()

if (${PROJECT_NAME}_USE_CONAN)
  find_file( Conan_BUILD_INFO conanbuildinfo.cmake ${CMAKE_CURRENT_BINARY_DIR} )
  if( NOT EXISTS ${Conan_BUILD_INFO} )
    execute_process(COMMAND conan install -g cmake ${CMAKE_CURRENT_SOURCE_DIR} )
    find_file( Conan_BUILD_INFO conanbuildinfo.cmake ${CMAKE_CURRENT_BINARY_DIR} )
    if(NOT EXISTS ${Conan_BUILD_INFO})
      MESSAGE( FATAL_ERROR "Unable to build conan info file")
    endif()
  endif()
  include(${Conan_BUILD_INFO})
  message(${Conan_BUILD_INFO})

  conan_basic_setup(TARGETS)
endif()

set(HEADERS
  source/decomp/convex_decomposition.hpp
  source/decomp/triangulation.hpp
  source/decomp/operations.hpp
  source/decomp/output.hpp)

set( SOURCES
        ${CMAKE_CURRENT_SOURCE_DIR}/source/decomp/convex_decomposition.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/source/decomp/triangulation.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/source/decomp/operations.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/source/decomp/output.cpp
        )

# expose the source list to any calling parent (hacky)
set( decomp_SOURCES ${SOURCES} PARENT_SCOPE )

# Build the main library
add_library(${TARGET_NAME}
  ${HEADERS}
  ${SOURCES}
)

target_include_directories(${TARGET_NAME}
  INTERFACE source)
