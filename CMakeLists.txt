cmake_minimum_required(VERSION 2.8.12)

project(nanopb C)

set(nanopb_VERSION_STRING nanopb-0.4.4)
set(nanopb_SOVERSION 0)

string(REPLACE "nanopb-" "" nanopb_VERSION ${nanopb_VERSION_STRING})

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
option(BUILD_STATIC_LIBS "Build static libraries" ON)

option(nanopb_BUILD_RUNTIME "Build the headers and libraries needed at runtime" ON)
option(nanopb_BUILD_GENERATOR "Build the protoc plugin for code generation" ON)
option(nanopb_MSVC_STATIC_RUNTIME "Link static runtime libraries" ON)

if(NOT DEFINED nanopb_PROTOC_PATH)
    set(nanopb_PROTOC_PATH "protoc")
endif()

if(NOT DEFINED CMAKE_DEBUG_POSTFIX)
    set(CMAKE_DEBUG_POSTFIX "d")
endif()

include(GNUInstallDirs)

if(MSVC AND nanopb_MSVC_STATIC_RUNTIME)
    foreach(flag_var
            CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
            CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO)
        if(${flag_var} MATCHES "/MD")
            string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
        endif(${flag_var} MATCHES "/MD")
    endforeach(flag_var)
endif()

if(NOT DEFINED CMAKE_INSTALL_CMAKEDIR)
    set(CMAKE_INSTALL_CMAKEDIR "${CMAKE_INSTALL_LIBDIR}/cmake/nanopb")
endif()

if(nanopb_BUILD_RUNTIME)
    if(BUILD_SHARED_LIBS)
        add_library(protobuf-nanopb SHARED
                pb.h
                pb_common.h
                pb_common.c
                pb_encode.h
                pb_encode.c
                pb_decode.h
                pb_decode.c)
        set_target_properties(protobuf-nanopb PROPERTIES
                SOVERSION ${nanopb_SOVERSION})
        install(TARGETS protobuf-nanopb EXPORT nanopb-targets
                ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
                LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
                RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
        target_include_directories(protobuf-nanopb INTERFACE
                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
                )
    endif()

    if(BUILD_STATIC_LIBS)
        add_library(protobuf-nanopb-static STATIC
                pb.h
                pb_common.h
                pb_common.c
                pb_encode.h
                pb_encode.c
                pb_decode.h
                pb_decode.c)
        set_target_properties(protobuf-nanopb-static PROPERTIES
                OUTPUT_NAME protobuf-nanopb)
        install(TARGETS protobuf-nanopb-static EXPORT nanopb-targets
                ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
        target_include_directories(protobuf-nanopb-static INTERFACE
                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
                )
    endif()

    configure_file(extra/nanopb-config-version.cmake.in
            nanopb-config-version.cmake @ONLY)

    install(EXPORT nanopb-targets
            DESTINATION ${CMAKE_INSTALL_CMAKEDIR}
            NAMESPACE nanopb::)

    install(FILES extra/nanopb-config.cmake
            ${CMAKE_CURRENT_BINARY_DIR}/nanopb-config-version.cmake
            DESTINATION ${CMAKE_INSTALL_CMAKEDIR})

    install(FILES pb.h pb_common.h pb_encode.h pb_decode.h
            DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
endif()
