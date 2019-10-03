cmake_minimum_required (VERSION 3.2.0)
project (decomp)

set(CMAKE_CXX_STANDARD 11)
set(TARGET_NAME ${PROJECT_NAME})

find_package(catkin REQUIRED)

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


###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if your package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
  INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/source/decomp
  LIBRARIES ${TARGET_NAME}
  CATKIN_DEPENDS 
  DEPENDS 
)



## Mark executables and/or libraries for installation
install(TARGETS ${PROJECT_NAME}
   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

## Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
   FILES_MATCHING PATTERN "*.h"
   PATTERN ".svn" EXCLUDE
 )

# Only enable tests for top-level projects
#if(${PROJECT_NAME}_BUILD_TESTS)
#  set(TEST_NAME test_${PROJECT_NAME})
#  add_executable(${TEST_NAME}
#    test/test_main.cpp
#    test/remapper.cpp
#    test/remove_holes.cpp
#    test/ear_clipping.cpp
#    test/half_edge.cpp
#    test/decomposition.cpp
#    test/winding.cpp)
#
#  target_link_libraries(${TEST_NAME}
#    PUBLIC decomp CONAN_PKG::Catch2)
#endif()
