cmake_minimum_required(VERSION 2.8)

# automagically detect if we should cross-compile
if(DEFINED ENV{TOOLCHAIN})
    set(CMAKE_C_COMPILER	$ENV{TOOLCHAIN}gcc)
    set(CMAKE_CXX_COMPILER	$ENV{TOOLCHAIN}g++)
    set(CMAKE_AR	"$ENV{TOOLCHAIN}ar" CACHE FILEPATH "CW archiver" FORCE)
endif()

project(libiec61850)
ENABLE_TESTING()

set(LIB_VERSION_MAJOR "1")
set(LIB_VERSION_MINOR "2")
set(LIB_VERSION_PATCH "0")
set(LIB_VERSION "${LIB_VERSION_MAJOR}.${LIB_VERSION_MINOR}.${LIB_VERSION_PATCH}")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/third_party/cmake/modules/")

# feature checks
include(CheckLibraryExists)
check_library_exists(rt clock_gettime "time.h" CONFIG_SYSTEM_HAS_CLOCK_GETTIME)

# check if we are on a little or a big endian
include (TestBigEndian)
test_big_endian(PLATFORM_IS_BIGENDIAN)

set(CONFIG_MMS_MAXIMUM_PDU_SIZE "65000" CACHE STRING "Configure the maximum size of an MMS PDU (default 65000)"   )
set(CONFIG_MAXIMUM_TCP_CLIENT_CONNECTIONS 5 CACHE STRING "Configure the maximum number of clients allowed to connect to the server")

option(BUILD_EXAMPLES "Build the examples" ON)
option(BUILD_PYTHON_BINDINGS "Build Python bindings" OFF)

option(CONFIG_MMS_SINGLE_THREADED "Compile for single threaded version" ON)
option(CONFIG_MMS_THREADLESS_STACK "Optimize stack for threadless operation (warning: single- or multi-threaded server will not work!)" OFF)
option(CONFIG_ACTIVATE_TCP_KEEPALIVE "Activate TCP keepalive" ON)
option(CONFIG_INCLUDE_GOOSE_SUPPORT "Build with GOOSE support" ON)

# choose the library features which shall be included
option(CONFIG_IEC61850_CONTROL_SERVICE "Build with support for IEC 61850 control features" ON)
option(CONFIG_IEC61850_REPORT_SERVICE "Build with support for IEC 61850 reporting services" ON)
option(CONFIG_IEC61850_LOG_SERVICE "Build with support for IEC 61850 logging services" ON)
option(CONFIG_IEC61850_SETTING_GROUPS "Build with support for IEC 61850 setting group services" ON)

set(CONFIG_REPORTING_DEFAULT_REPORT_BUFFER_SIZE "8000" CACHE STRING "Default buffer size for buffered reports in byte" )

# advanced options
option(DEBUG "Enable debugging mode (include assertions)" OFF)
option(DEBUG_SOCKET "Enable printf debugging for socket layer" ${DEBUG})
option(DEBUG_COTP "Enable COTP printf debugging" ${DEBUG})
option(DEBUG_ISO_SERVER "Enable ISO SERVER printf debugging" ${DEBUG})
option(DEBUG_ISO_CLIENT "Enable ISO CLIENT printf debugging" ${DEBUG})
option(DEBUG_IED_SERVER "Enable IED SERVER printf debugging" ${DEBUG})
option(DEBUG_IED_CLIENT "Enable IED CLIENT printf debugging" ${DEBUG})
option(DEBUG_MMS_SERVER "Enable MMS SERVER printf debugging" ${DEBUG})
option(DEBUG_MMS_CLIENT "Enable MMS CLIENT printf debugging" ${DEBUG})
option(DEBUG_GOOSE_SUBSCRIBER "Enable GOOSE subscriber printf debugging" ${DEBUG})
option(DEBUG_GOOSE_PUBLISHER "Enable GOOSE publisher printf debugging" ${DEBUG})
option(DEBUG_SV_SUBSCRIBER "Enable Sampled Values subscriber debugging" ${DEBUG})
option(DEBUG_SV_PUBLISHER "Enable Sampled Values publisher debugging" ${DEBUG})
option(DEBUG_HAL_ETHERNET "Enable Ethernet HAL printf debugging" ${DEBUG})

#mark_as_advanced(
#    DEBUG_SOCKET
#    DEBUG_COTP
#    DEBUG_ISO_SERVER
#    DEBUG_ISO_CLIENT
#    DEBUG_IED_SERVER
#    DEBUG_IED_CLIENT
#    DEBUG_MMS_SERVER
#    DEBUG_MMS_CLIENT
#    DEBUG_GOOSE_SUBSCRIBER
#    DEBUG_GOOSE_PUBLISHER
#    DEBUG_SV_SUBSCRIBER
#    DEBUG_SV_PUBLISHER
#    DEBUG_HAL_ETHERNET
#)

include_directories(
    ${CMAKE_CURRENT_BINARY_DIR}/config
    ${CMAKE_CURRENT_LIST_DIR}/src/common/inc
    ${CMAKE_CURRENT_LIST_DIR}/src/goose
    ${CMAKE_CURRENT_LIST_DIR}/src/sampled_values
    ${CMAKE_CURRENT_LIST_DIR}/src/hal/inc
    ${CMAKE_CURRENT_LIST_DIR}/src/iec61850/inc
    ${CMAKE_CURRENT_LIST_DIR}/src/iec61850/inc_private
    ${CMAKE_CURRENT_LIST_DIR}/src/mms/inc
    ${CMAKE_CURRENT_LIST_DIR}/src/mms/inc_private
    ${CMAKE_CURRENT_LIST_DIR}/src/mms/iso_mms/asn1c
    ${CMAKE_CURRENT_LIST_DIR}/src/logging
    ${CMAKE_CURRENT_LIST_DIR}/src/tls
)

set(API_HEADERS
    src/hal/inc/hal_time.h
    src/hal/inc/hal_thread.h
    src/hal/inc/hal_filesystem.h
    src/hal/inc/hal_ethernet.h
    src/hal/inc/platform_endian.h
    src/common/inc/libiec61850_common_api.h
    src/common/inc/libiec61850_platform_includes.h
    src/common/inc/linked_list.h
    src/common/inc/byte_buffer.h
    src/common/inc/lib_memory.h
    src/common/inc/string_utilities.h
    src/iec61850/inc/iec61850_client.h
    src/iec61850/inc/iec61850_common.h
    src/iec61850/inc/iec61850_server.h
    src/iec61850/inc/iec61850_model.h
    src/iec61850/inc/iec61850_cdc.h
    src/iec61850/inc/iec61850_dynamic_model.h
    src/iec61850/inc/iec61850_config_file_parser.h
    src/mms/inc/mms_value.h
    src/mms/inc/mms_common.h
    src/mms/inc/mms_types.h
    src/mms/inc/mms_device_model.h
    src/mms/inc/mms_server.h
    src/mms/inc/mms_named_variable_list.h
    src/mms/inc/mms_type_spec.h
    src/mms/inc/mms_client_connection.h
    src/mms/inc/iso_connection_parameters.h
    src/mms/inc/iso_server.h
    src/mms/inc/ber_integer.h
    src/mms/inc/asn1_ber_primitive_value.h
    src/goose/goose_subscriber.h
    src/goose/goose_receiver.h
    src/goose/goose_publisher.h
    src/sampled_values/sv_subscriber.h
    src/sampled_values/sv_publisher.h
    src/logging/logging_api.h
    src/tls/tls_api.h
    ${CMAKE_CURRENT_BINARY_DIR}/config/stack_config.h
)

if(MSVC)
    include_directories(
        ${CMAKE_CURRENT_LIST_DIR}/src/vs
    )
endif(MSVC)

if(EXISTS ${CMAKE_CURRENT_LIST_DIR}/third_party/mbedtls/mbedtls-2.6.0)
set(WITH_MBEDTLS 1)
endif(EXISTS ${CMAKE_CURRENT_LIST_DIR}/third_party/mbedtls/mbedtls-2.6.0)

if(WITH_MBEDTLS)
include_directories(
    ${CMAKE_CURRENT_LIST_DIR}/src/tls/mbedtls
    ${CMAKE_CURRENT_LIST_DIR}/third_party/mbedtls/mbedtls-2.6.0/include
)

file(GLOB tls_SRCS ${CMAKE_CURRENT_LIST_DIR}/third_party/mbedtls/mbedtls-2.6.0/library/*.c)

add_definitions(-DCONFIG_MMS_SUPPORT_TLS=1)
add_definitions(-DMBEDTLS_CONFIG_FILE="mbedtls_config.h")

endif(WITH_MBEDTLS)

# write the detected stuff to this file
configure_file(
    ${CMAKE_CURRENT_LIST_DIR}/config/stack_config.h.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/config/stack_config.h
)

if(BUILD_EXAMPLES)
    add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/examples)
endif(BUILD_EXAMPLES)

add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/src)

install(FILES ${API_HEADERS} DESTINATION include/libiec61850 COMPONENT Development)

if(BUILD_PYTHON_BINDINGS)
    add_subdirectory(${CMAKE_CURRENT_LIST_DIR}/pyiec61850)
endif(BUILD_PYTHON_BINDINGS)

set(CPACK_PACKAGE_DESCRIPTION "IEC 61850 MMS/GOOSE client and server library")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "IEC 61850 MMS/GOOSE client and server library")
set(CPACK_PACKAGE_VENDOR "MZ Automation GmbH")
set(CPACK_PACKAGE_CONTACT "info@libiec61850.com")
set(CPACK_PACKAGE_VERSION_MAJOR "${LIB_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${LIB_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${LIB_VERSION_PATCH}")
set(CPACK_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}_${LIB_VERSION_MAJOR}.${LIB_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}_${CMAKE_SYSTEM_PROCESSOR}")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}_${LIB_VERSION_MAJOR}.${LIB_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")

set(CPACK_COMPONENTS_ALL Libraries Development Applications)
#set(CPACK_PACKAGE_INSTALL_DIRECTORY "${CMAKE_PROJECT_NAME}")

if(EXISTS "${CMAKE_ROOT}/Modules/CPack.cmake")
    include(InstallRequiredSystemLibraries)

    include(CPack)
endif(EXISTS "${CMAKE_ROOT}/Modules/CPack.cmake")
