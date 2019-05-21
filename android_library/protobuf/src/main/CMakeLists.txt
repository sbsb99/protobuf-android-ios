# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)


# SET PROTOBUF
set(PROTOBUF_SRC_DIR ../../../../protobuf/cmake)
set(PROTOBUF_BUILD_DIR ../protobuf/${ANDROID_ABI})
file(MAKE_DIRECTORY ${PROTOBUF_BUILD_DIR})

add_subdirectory(${PROTOBUF_SRC_DIR} ${PROTOBUF_BUILD_DIR})
get_directory_property(protobuf_VERSION DIRECTORY ${PROTOBUF_SRC_DIR} DEFINITION protobuf_VERSION)


# COPY HEADERS (script from protobuf-android-native/protobuf/cmake/install.cmake)
set(OUTPUT_INCLUDE_DIRECTORY ../../../../../../output/${protobuf_VERSION}/include)
file(STRINGS ${PROTOBUF_SRC_DIR}/extract_includes.bat.in _extract_strings
        REGEX "^copy")
foreach(_extract_string ${_extract_strings})
  string(REGEX REPLACE "^.* .+ include\\\\(.+)$" "\\1"
          _header ${_extract_string})
  string(REPLACE "\\" "/" _header ${_header})
  get_filename_component(_extract_from "${protobuf_SOURCE_DIR}/../src/${_header}" ABSOLUTE)
  get_filename_component(_extract_name ${_header} NAME)
  get_filename_component(_extract_to "${CMAKE_INSTALL_INCLUDEDIR}/${_header}" PATH)
  get_filename_component(_extract_to "${OUTPUT_INCLUDE_DIRECTORY}/${_header}" PATH)
  if(NOT EXISTS ${_extract_to})
    file(MAKE_DIRECTORY ${_extract_to})
  endif()
  if(EXISTS "${_extract_from}")
    file(COPY ${_extract_from} DESTINATION ${_extract_to})
  else()
    message(AUTHOR_WARNING "The file \"${_extract_from}\" is listed in "
            "\"${protobuf_SOURCE_DIR}/cmake/extract_includes.bat.in\" "
            "but there not exists. The file will not be installed.")
  endif()
endforeach()


# BUILD dynamicLibrary for dependency
add_library( # Sets the name of the library.
        dynamicLibrary

        # Sets the library as a shared library.
        SHARED

        # Provides a relative path to your source file(s).
        cpp/native-lib.cpp)

find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)

target_link_libraries( # Specifies the target library.
        dynamicLibrary

        libprotobuf
        libprotobuf-lite

        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})


# COPY/STRIP STATIC LIBRARIES
set(OUTPUT_LIB_DIRECTORY ../../../../../../output/${protobuf_VERSION}/lib/${ANDROID_ABI})
add_custom_command(TARGET dynamicLibrary
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory ${OUTPUT_LIB_DIRECTORY})
add_custom_command(TARGET dynamicLibrary
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:libprotobuf> ${OUTPUT_LIB_DIRECTORY})
add_custom_command(TARGET dynamicLibrary
        POST_BUILD
        COMMAND ${CMAKE_STRIP} ${OUTPUT_LIB_DIRECTORY}/libprotobuf.a)
add_custom_command(TARGET dynamicLibrary
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:libprotobuf-lite> ${OUTPUT_LIB_DIRECTORY})
add_custom_command(TARGET dynamicLibrary
        POST_BUILD
        COMMAND ${CMAKE_STRIP} ${OUTPUT_LIB_DIRECTORY}/libprotobuf-lite.a)
