cmake_minimum_required(VERSION 3.14...3.20)

# ---- Project ----

# Note: update this to your new project's name and version
project(
  kseq
  VERSION 0.1
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

include(../../cmake/CPM.cmake)

# PackageProject.cmake will be used to make our target installable
CPMAddPackage(
  NAME PackageProject.cmake
  GITHUB_REPOSITORY TheLartians/PackageProject.cmake
  VERSION 1.5.0
)


# ##################################################################################################

# ---- Add source files ----

# ---- Create library ----

add_library(kseq INTERFACE)

set_target_properties(kseq PROPERTIES CXX_STANDARD 17)

target_include_directories(
  kseq INTERFACE $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
                 $<INSTALL_INTERFACE:include/${PROJECT_NAME}>
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
  INCLUDE_DESTINATION include
  VERSION_HEADER "${VERSION_HEADER_LOCATION}"
  COMPATIBILITY SameMajorVersion
  NO_VERSION_SUFFIX
)
