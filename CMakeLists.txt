cmake_minimum_required(VERSION 3.10.2)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_VERBOSE_MAKEFILE TRUE)


# Use ccache is available, has to be before "project()"
find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
  # Support Unix Makefiles and Ninja
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

project(test_swig)

# Xcode/Ninja generators undefined MAKE
if(CMAKE_GENERATOR MATCHES "Make")
  set(MAKE "$(MAKE)")
else()
  set(MAKE make)
endif()

# Add Color Output if Using Ninja
macro(AddCXXFlagIfSupported flag test)
  CHECK_CXX_COMPILER_FLAG(${flag} ${test})
  if(${${test}})
    message("adding ${flag}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${flag}")
  endif()
endmacro()

if("Ninja" STREQUAL ${CMAKE_GENERATOR})
  # Clang
  if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    AddCXXFlagIfSupported(-fcolor-diagnostics COMPILER_SUPPORTS_fcolor-diagnostics)
  endif()

  # g++
  if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    # For some reason it doesn't say its supported, but it works...
    # AddCXXFlagIfSupported(-fdiagnostics-color COMPILER_SUPPORTS_fdiagnostics-color)
    message(STATUS "Ninja: Forcing -fdiagnostics-color=always")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color=always")
  endif()
endif()


#######################################################################
#                              C O N A N                              #
#######################################################################

set(CMAKE_CONAN_EXPECTED_HASH 709180234748692a642f9e5c4d80d328)
set(CMAKE_CONAN_VERSION "v0.14")

if(EXISTS "${CMAKE_BINARY_DIR}/conan.cmake")
  file(MD5 "${CMAKE_BINARY_DIR}/conan.cmake" CMAKE_CONAN_HASH)
endif()
if(NOT EXISTS "${CMAKE_BINARY_DIR}/conan.cmake" OR NOT "${CMAKE_CONAN_HASH}" MATCHES "${CMAKE_CONAN_EXPECTED_HASH}")
  # Put it in CMAKE_BINARY_DIR so we don't end up with two when building OpenStudioApplication
  message(STATUS "openstudio: Downloading conan.cmake ${CMAKE_CONAN_VERSION} from https://github.com/conan-io/cmake-conan")
  file(DOWNLOAD "https://github.com/conan-io/cmake-conan/raw/${CMAKE_CONAN_VERSION}/conan.cmake"
     "${CMAKE_BINARY_DIR}/conan.cmake")
else()
  message(STATUS "openstudio: using existing conan.cmake")
endif()

include(${CMAKE_BINARY_DIR}/conan.cmake)

conan_check(VERSION 1.0.0 REQUIRED)

message(STATUS "openstudio: RUNNING CONAN")

conan_add_remote(NAME bincrafters
  URL https://api.bintray.com/conan/bincrafters/public-conan)

  conan_cmake_run(REQUIRES
    OpenSSL/1.1.0g@conan/stable
    websocketpp/0.8.1@bincrafters/stable
    zlib/1.2.11@conan/stable
    BASIC_SETUP CMAKE_TARGETS NO_OUTPUT_DIRS
    OPTIONS ${CONAN_OPTIONS}
    BUILD ${CONAN_BUILD}
    # Passes `-u, --update`    to conan install: Check updates exist from upstream remotes
    # That and build=outdated should ensure we track the right
    UPDATE
  )

message("CONAN_LIBS=${CONAN_LIBS}")

# Search first in the binary dir, where conan will install finders, then
# search for modules in the root dir to override cmake ones
# Start with ROOT, then PROJECT_BINARY_DIR
list(APPEND CMAKE_MODULE_PATH "${CMAKE_BINARY_DIR} ${PROJECT_BINARY_DIR}" "${PROJECT_SOURCE_DIR}/CMake")
include_directories("${CMAKE_CURRENT_BINARY_DIR}")

#######################################################################
#                               T E S T                               #
#######################################################################

add_executable(test
  main.cpp
)

