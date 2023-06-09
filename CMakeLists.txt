cmake_minimum_required(VERSION 3.24)

#
# Project details
#
project(
  graphics
  VERSION 0.0.1
  LANGUAGES CXX)

#
# Model project dependencies
#
find_package(stdsharp CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)

include(stdsharp/Utils)

config_lib(${PROJECT_NAME} INTERFACE STD 23)

add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

target_compile_options(
  ${PROJECT_NAME} INTERFACE $<$<CXX_COMPILER_ID:MSVC>: /external:anglebrackets>
                            $<$<CXX_COMPILER_ID:Clang>: -fexperimental-library>)

target_link_options(${PROJECT_NAME} INTERFACE $<$<CXX_COMPILER_ID:Clang>:
                    -lc++experimental>)

target_link_libraries(${PROJECT_NAME} INTERFACE stdsharp::stdsharp glm::glm)

verbose_message("Successfully added all dependencies and linked against them.")

include(CTest)
add_subdirectory(tests)
