cmake_minimum_required(VERSION 3.10)
project(minitrace)

add_library(${PROJECT_NAME} STATIC minitrace.c)
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

set(POS_INDEP_MTR ${PROJECT_NAME}_pos_indep)
add_library(${POS_INDEP_MTR} STATIC minitrace.c)

set_property(TARGET ${POS_INDEP_MTR} PROPERTY POSITION_INDEPENDENT_CODE ON)

message("mtr libs ${PROJECT_NAME} ${POS_INDEP_MTR}")

set(BUILT_TARGETS ${PROJECT_NAME} ${POS_INDEP_MTR})

option(MTR_ENABLED "Enable minitrace" ON)
option(MTR_CUSTOM_BUFFER_SIZE "Set the buffer size to a custom value" OFF)

foreach(BUILT_TARGET IN LISTS BUILT_TARGETS)

  target_include_directories(${BUILT_TARGET} PUBLIC ${CMAKE_CURRENT_LIST_DIR})

  if(MTR_ENABLED)
    target_compile_definitions(${BUILT_TARGET} PUBLIC MTR_ENABLED)

    if(MTR_CUSTOM_BUFFER_SIZE)
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
