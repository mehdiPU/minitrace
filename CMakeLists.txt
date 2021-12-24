cmake_minimum_required(VERSION 3.10)
project(minitrace)

option(OP_MINITRACE "Enable minitrace" ON)

option(OP_MTR_CUSTOM_BUFFER_SIZE "Set events buffer to a custom size" ON)
if(NOT DEFINED CUSTOM_MINITRACE_BUFFER_SIZE)
  set(CUSTOM_MINITRACE_BUFFER_SIZE 2000)
endif()

add_library(${PROJECT_NAME} STATIC minitrace.c)

set(POS_INDEP_MTR ${PROJECT_NAME}_pos_indep)
add_library(${POS_INDEP_MTR} STATIC minitrace.c)
set_property(TARGET ${POS_INDEP_MTR} PROPERTY POSITION_INDEPENDENT_CODE ON)

set(DISABLED_MTR ${PROJECT_NAME}_disabled)
add_library(${DISABLED_MTR} INTERFACE minitrace.h)
target_include_directories(${DISABLED_MTR} INTERFACE ${CMAKE_CURRENT_LIST_DIR})

set(BUILT_TARGETS ${PROJECT_NAME} ${POS_INDEP_MTR})

foreach(BUILT_TARGET IN LISTS BUILT_TARGETS)

  target_include_directories(${BUILT_TARGET} PUBLIC ${CMAKE_CURRENT_LIST_DIR})

  if(OP_MINITRACE)
    target_compile_definitions(${BUILT_TARGET} PUBLIC MTR_ENABLED)

    if(OP_MTR_CUSTOM_BUFFER_SIZE)
      target_compile_definitions(${BUILT_TARGET}
	PRIVATE INTERNAL_MINITRACE_BUFFER_SIZE=${CUSTOM_MINITRACE_BUFFER_SIZE}
      )
    endif()
  endif()
endforeach()

option(MTR_BUILD_TEST "Build test apps" OFF)
if(MTR_BUILD_TEST)
    add_executable(minitrace_test minitrace_test.cpp)
    target_link_libraries(minitrace_test ${PROJECT_NAME})

    find_package(Threads)
    add_executable(minitrace_test_mt minitrace_test_mt.cpp)
    target_link_libraries(minitrace_test_mt ${PROJECT_NAME} Threads::Threads)
endif()
