cmake_minimum_required(VERSION 3.0)
project(isapi)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(cmake/utils.cmake)

find_package(PCRE REQUIRED COMPONENTS pcre)

set(jk_native_dir        "${CMAKE_CURRENT_SOURCE_DIR}/../..")
set(jk_native_common_dir "${jk_native_dir}/common")
set(jk_native_iis_dir    "${jk_native_dir}/iis")

set(target_name "${PROJECT_NAME}")
set(output_name "isapi_redirect")

set(cxx_compile_definitions )
set(cxx_compile_options )
set(cxx_headers )
set(cxx_sources )
set(resource_headers )
set(resource_sources )
set(def_file )
set(link_options )
set(libraries )

list(APPEND cxx_compile_definitions
    "PCRE_STATIC"
    "JK_ISAPI"
    "ISAPI_EXPORTS")

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    list(APPEND cxx_compile_definitions
        "_AMD64_=1"
        "WIN64"
        "_WIN64")
endif()

if(MSVC)
    list(APPEND cxx_compile_options
        "/Zi"
        "/EHsc"
        "/Oy-")
endif()

list(APPEND cxx_headers
    "${jk_native_common_dir}/ap_snprintf.h"
    "${jk_native_common_dir}/jk_ajp12_worker.h"
    "${jk_native_common_dir}/jk_ajp13.h"
    "${jk_native_common_dir}/jk_ajp13_worker.h"
    "${jk_native_common_dir}/jk_ajp14.h"
    "${jk_native_common_dir}/jk_ajp14_worker.h"
    "${jk_native_common_dir}/jk_ajp_common.h"
    "${jk_native_common_dir}/jk_connect.h"
    "${jk_native_common_dir}/jk_context.h"
    "${jk_native_common_dir}/jk_global.h"
    "${jk_native_common_dir}/jk_lb_worker.h"
    "${jk_native_common_dir}/jk_logger.h"
    "${jk_native_common_dir}/jk_map.h"
    "${jk_native_common_dir}/jk_md5.h"
    "${jk_native_common_dir}/jk_msg_buff.h"
    "${jk_native_common_dir}/jk_mt.h"
    "${jk_native_common_dir}/jk_pool.h"
    "${jk_native_common_dir}/jk_service.h"
    "${jk_native_common_dir}/jk_shm.h"
    "${jk_native_common_dir}/jk_sockbuf.h"
    "${jk_native_common_dir}/jk_status.h"
    "${jk_native_common_dir}/jk_uri_worker_map.h"
    "${jk_native_common_dir}/jk_url.h"
    "${jk_native_common_dir}/jk_util.h"
    "${jk_native_common_dir}/jk_version.h"
    "${jk_native_common_dir}/jk_worker.h"
    "${jk_native_common_dir}/jk_worker_list.h")

list(APPEND cxx_sources
    "${jk_native_common_dir}/jk_ajp12_worker.c"
    "${jk_native_common_dir}/jk_ajp13.c"
    "${jk_native_common_dir}/jk_ajp13_worker.c"
    "${jk_native_common_dir}/jk_ajp14.c"
    "${jk_native_common_dir}/jk_ajp14_worker.c"
    "${jk_native_common_dir}/jk_ajp_common.c"
    "${jk_native_common_dir}/jk_connect.c"
    "${jk_native_common_dir}/jk_context.c"
    "${jk_native_common_dir}/jk_lb_worker.c"
    "${jk_native_common_dir}/jk_map.c"
    "${jk_native_common_dir}/jk_md5.c"
    "${jk_native_common_dir}/jk_msg_buff.c"
    "${jk_native_common_dir}/jk_nwmain.c"
    "${jk_native_common_dir}/jk_pool.c"
    "${jk_native_common_dir}/jk_shm.c"
    "${jk_native_common_dir}/jk_sockbuf.c"
    "${jk_native_common_dir}/jk_status.c"
    "${jk_native_common_dir}/jk_uri_worker_map.c"
    "${jk_native_common_dir}/jk_url.c"
    "${jk_native_common_dir}/jk_util.c"
    "${jk_native_common_dir}/jk_worker.c"
    "${jk_native_iis_dir}/jk_isapi_plugin.c")

list(APPEND resource_sources
    "${jk_native_common_dir}/jk.rc")

set(def_file "${jk_native_iis_dir}/isapi.def")

set(resource_files
    ${resource_headers}
    ${resource_sources}
    "${def_file}")

if(MSVC)
    list(APPEND link_options
        "/DEBUG")
endif()

list(APPEND libraries
    "kernel32"
    "user32"
    "advapi32"
    "ws2_32"
    "mswsock"
    "strsafe"
    PCRE::pcre)

add_library(${target_name} SHARED
    ${cxx_headers}
    ${cxx_sources}
    ${resource_headers}
    ${resource_sources}
    "${def_file}")
target_compile_options(${target_name}
    PRIVATE
    ${cxx_compile_options})
target_compile_definitions(${target_name}
    PRIVATE
    ${cxx_compile_definitions})
target_include_directories(${target_name}
    PRIVATE
    ${jk_native_common_dir})

set(link_option_str "")
if(NOT MSVC)
    # This should work for most of CMake generators except Visual Studio
    jk_list_to_string("${link_options}" "," link_option_str)
endif()

target_link_libraries(${target_name}
    PRIVATE
    "${link_option_str}"
    ${libraries})

if(MSVC)
    foreach(link_option IN LISTS link_options)
        # This is the only way I found to pass linker options using Visual Studio CMake generator
        set_property(TARGET ${target_name} APPEND_STRING PROPERTY LINK_FLAGS " ${link_option}")
    endforeach()
endif()

set_target_properties(${target_name} PROPERTIES OUTPUT_NAME "${output_name}")

if(NOT jk_no_cmake_dir_source_group)
    # Group files according to file path
    jk_dir_source_group("Header Files" "${jk_native_dir}" "${cxx_headers}")
    jk_dir_source_group("Source Files" "${jk_native_dir}" "${cxx_sources}")
    jk_dir_source_group("Resource Files" "${jk_native_dir}" "${resource_files}")
endif()
