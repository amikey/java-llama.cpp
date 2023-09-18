cmake_minimum_required(VERSION 3.12)

project(jllama CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED true)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

find_package(Java REQUIRED)
find_program(JAVA_EXECUTABLE NAMES java)

execute_process(
  COMMAND ${JAVA_EXECUTABLE} -cp ${CMAKE_SOURCE_DIR}/target/classes de.kherud.llama.OSInfo --os
  OUTPUT_VARIABLE OS_NAME
  OUTPUT_STRIP_TRAILING_WHITESPACE
)
execute_process(
  COMMAND ${JAVA_EXECUTABLE} -cp ${CMAKE_SOURCE_DIR}/target/classes de.kherud.llama.OSInfo --arch
  OUTPUT_VARIABLE OS_ARCH
  OUTPUT_STRIP_TRAILING_WHITESPACE
)

find_path(JNI_INCLUDE_DIR NAMES jni.h HINTS ENV JAVA_HOME PATH_SUFFIXES include)
if(NOT JNI_INCLUDE_DIR)
    message(FATAL_ERROR "Could not find jni.h")
endif()

file(GLOB_RECURSE JNI_MD_PATHS RELATIVE "${JNI_INCLUDE_DIR}" "${JNI_INCLUDE_DIR}/**/jni_md.h")
if(NOT JNI_MD_PATHS)
    message(FATAL_ERROR "Could not find jni_md.h")
endif()

foreach(PATH IN LISTS JNI_MD_PATHS)
    get_filename_component(DIR ${PATH} DIRECTORY)
    list(APPEND JNI_MD_INCLUDE_DIRS "${JNI_INCLUDE_DIR}/${DIR}")
endforeach()

include_directories(
    ${JNI_INCLUDE_DIR}
    ${JNI_MD_INCLUDE_DIRS}
    src/main/cpp
    src/main/cpp/llama.cpp
    src/main/cpp/llama.cpp/common
)

add_subdirectory(
    src/main/cpp/llama.cpp
)

add_library(jllama SHARED src/main/cpp/jllama.cpp)

target_link_libraries(jllama PRIVATE common llama ${CMAKE_THREAD_LIBS_INIT})

set_target_properties(jllama PROPERTIES
  LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/src/main/resources/de/kherud/llama/${OS_NAME}/${OS_ARCH}"
)