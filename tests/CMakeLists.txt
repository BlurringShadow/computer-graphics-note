#
# Project details
#
project(${CMAKE_PROJECT_NAME}Tests LANGUAGES CXX)

verbose_message("Adding tests under ${CMAKE_PROJECT_NAME}...")

find_package(Catch2 CONFIG REQUIRED)
include(Catch)

#
# Set the sources for the unit tests and add the executable(s)
#
set(src src/bresenham_draw_line_test.cpp src/cohen_sutherland_clipping_test.cpp
        src/midpoint_clipping_test.cpp src/liang_barsky_clipping_test.cpp)

config_lib(${PROJECT_NAME}Lib INTERFACE)

target_link_libraries(${PROJECT_NAME}Lib INTERFACE ${CMAKE_PROJECT_NAME}
                                                   Catch2::Catch2WithMain)

set_target_properties(${PROJECT_NAME}Lib PROPERTIES COMPILE_WARNING_AS_ERROR ON)

target_compile_options(
  ${PROJECT_NAME}Lib
  INTERFACE $<$<CXX_COMPILER_ID:MSVC>:/W4>
            $<$<CXX_COMPILER_ID:Clang>:-ftemplate-backtrace-limit=0
            -ftime-trace -fdiagnostics-show-template-tree>)

config_exe(${PROJECT_NAME} EXE_SRC ${src})

target_enable_clang_sanitizer(${PROJECT_NAME} PUBLIC SANITIZER address
                              undefined)

target_link_libraries(${PROJECT_NAME} PRIVATE ${PROJECT_NAME}Lib)

#
# Configure clang-tidy
#
option(ENABLE_CLANG_TIDY "Enable clang-tidy" OFF)

if(${ENABLE_CLANG_TIDY})
  message(STATUS "clang-tidy is enabled")

  target_enable_clang_tidy(${PROJECT_NAME})
endif()

catch_discover_tests(${PROJECT_NAME})

#
# Configure code coverage
#
set(CODE_COVERAGE_EXT CACHE STRING "Enable code coverage")

if(CODE_COVERAGE_EXT)
  set(PROFILE_FILE "${PROJECT_NAME}.profraw")

  add_custom_command(
    OUTPUT ${PROFILE_FILE}
    DEPENDS ${PROJECT_NAME}
    COMMAND ${CMAKE_COMMAND} -E rm -f ${PROFILE_FILE}
    COMMAND ${CMAKE_COMMAND} -E env LLVM_PROFILE_FILE=${PROFILE_FILE}
            $<TARGET_FILE:${PROJECT_NAME}> || exit 0 # Run first to generate
                                                     # profraw file
  )

  target_coverage(
    ${PROJECT_NAME}
    FORMAT
    ${CODE_COVERAGE_EXT}
    DEPENDS
    ${PROFILE_FILE}
    PROFILE_FILE
    ${PROFILE_FILE})
endif()
