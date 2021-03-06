cmake_minimum_required(VERSION 3.14...3.20)

# ---- Project ----

# Note: update this to your new project's name and version
project(
  Greeter
  VERSION 1.0
  LANGUAGES CXX
)

# ---- Include guards ----

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(
    FATAL_ERROR
      "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there."
  )
endif()

# ---- Add dependencies via CPM ----
# see https://github.com/TheLartians/CPM.cmake for more info

include(cmake/CPM.cmake)

# PackageProject.cmake will be used to make our target installable
CPMAddPackage(
  NAME PackageProject.cmake
  GITHUB_REPOSITORY TheLartians/PackageProject.cmake
  VERSION 1.5.0
)

CPMAddPackage(
  NAME fmt
  GIT_TAG 7.1.3
  GITHUB_REPOSITORY fmtlib/fmt # to get an installable target
  OPTIONS "FMT_INSTALL YES"
)

# FIXME: do it as simple as possible!
add_subdirectory(lib/kseq)

# ##################################################################################################
# CPMAddPackage(NAME kseq SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/lib/kseq)
# if(kseq_ADDED)
# TODO: add this in you lib/kseq dir as CMakeList.txt like this! this is your source, so make a
# standalone cmake project!
# message(WARNING "kseq ADDED FROM ${kseq_SOURCE_DIR}")
# endif()
# ##################################################################################################

# ---- Add source files ----

# Note: globbing sources is considered bad practice as CMake's generators may not detect new files
# automatically. Keep that in mind when changing files, or explicitly mention them here.
file(GLOB_RECURSE headers CONFIGURE_DEPENDS include/*.h)
file(GLOB_RECURSE sources CONFIGURE_DEPENDS source/*.cpp)

# ---- Create library ----

# Note: for header-only libraries change all PUBLIC flags to INTERFACE and create an interface
# target: add_library(Greeter INTERFACE)
add_library(Greeter ${headers} ${sources})

set_target_properties(Greeter PROPERTIES CXX_STANDARD 17)

# being a cross-platform target, we enforce standards conformance on MSVC
target_compile_options(Greeter PUBLIC "$<$<COMPILE_LANG_AND_ID:CXX,MSVC>:/permissive>")

# Link dependencies

target_link_libraries(Greeter PUBLIC fmt::fmt kseq::kseq) # ISSUE #86

target_include_directories(
  Greeter PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
                 $<INSTALL_INTERFACE:include/${PROJECT_NAME}-${PROJECT_VERSION}>
)

# ---- Create an installable target ----
# this allows users to install and find the library via `find_package()`.

# the location where the project's version header will be placed should match the project's regular
# header paths
string(TOLOWER ${PROJECT_NAME}/version.h VERSION_HEADER_LOCATION)

packageProject(
  NAME ${PROJECT_NAME}
  VERSION ${PROJECT_VERSION}
  NAMESPACE ${PROJECT_NAME}
  BINARY_DIR ${PROJECT_BINARY_DIR}
  INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include
  INCLUDE_DESTINATION include/${PROJECT_NAME}-${PROJECT_VERSION}
  VERSION_HEADER "${VERSION_HEADER_LOCATION}"
  COMPATIBILITY SameMajorVersion
  DEPENDENCIES "fmt 7.1.3; kseq 0.1"
)
