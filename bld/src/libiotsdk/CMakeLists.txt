# Copyright (c) Microsoft. All rights reserved.
# Licensed under the MIT license. See LICENSE file in the project root for full license information.

#
# Build libiotsdk - we build with custom cmake file since we only need 
# a subset and replace other parts in the public shared sdk, e.g. logging.
#

#
# Source
#
    set(source_c_files
        ${_SHARED_ROOT}/adapters/agenttime.c
        ${_SHARED_ROOT}/src/base64.c
        ${_SHARED_ROOT}/src/buffer.c
        ${_SHARED_ROOT}/src/consolelogger.c
        ${_SHARED_ROOT}/src/constbuffer.c
        ${_SHARED_ROOT}/src/crt_abstractions.c
        ${_SHARED_ROOT}/src/constmap.c
        ${_SHARED_ROOT}/src/doublylinkedlist.c
        ${_SHARED_ROOT}/src/gballoc.c
        ${_SHARED_ROOT}/src/hmac.c
        ${_SHARED_ROOT}/src/hmacsha256.c
        ${_SHARED_ROOT}/src/xio.c
        ${_SHARED_ROOT}/src/singlylinkedlist.c
        ${_SHARED_ROOT}/src/map.c
        ${_SHARED_ROOT}/src/optionhandler.c
        ${_SHARED_ROOT}/src/sastoken.c
        ${_SHARED_ROOT}/src/sha1.c
        ${_SHARED_ROOT}/src/sha224.c
        ${_SHARED_ROOT}/src/sha384-512.c
        ${_SHARED_ROOT}/src/strings.c
        ${_SHARED_ROOT}/src/string_tokenizer.c
        ${_SHARED_ROOT}/src/urlencode.c
        ${_SHARED_ROOT}/src/usha.c
        ${_SHARED_ROOT}/src/vector.c
        ${_SHARED_ROOT}/src/httpapiex.c
        ${_SHARED_ROOT}/src/httpapiexsas.c
        ${_SHARED_ROOT}/src/httpheaders.c
        ${_SHARED_ROOT}/src/xlogging.c
    )
if(WIN32)
    set(source_c_files ${source_c_files}
        ${_SHARED_ROOT}/adapters/condition_win32.c
        ${_SHARED_ROOT}/adapters/threadapi_win32.c
        ${_SHARED_ROOT}/adapters/lock_win32.c
        ${_SHARED_ROOT}/adapters/httpapi_winhttp.c
    )
    if(${use_openssl})
        set(source_c_files ${source_c_files}
            ${_SHARED_ROOT}/src/tlsio_openssl.c
            ${_SHARED_ROOT}/src/x509_openssl.c
        )
    else()
        set(source_c_files ${source_c_files}
            ${_SHARED_ROOT}/src/tlsio_schannel.c
            ${_SHARED_ROOT}/src/x509_schannel.c
        )
    endif()
else()
    set(source_c_files ${source_c_files}
        ${_SHARED_ROOT}/adapters/httpapi_curl.c
        ${_SHARED_ROOT}/adapters/condition_pthreads.c
        ${_SHARED_ROOT}/adapters/threadapi_pthreads.c
        ${_SHARED_ROOT}/adapters/lock_pthreads.c
    )
    if(${use_wolfssl})
        set(source_c_files ${source_c_files}
            ${_SHARED_ROOT}/src/tlsio_wolfssl.c
        )
    else()
        set(use_openssl 1)
        set(source_c_files ${source_c_files}
            ${_SHARED_ROOT}/src/tlsio_openssl.c
            ${_SHARED_ROOT}/src/x509_openssl.c
        )
    endif()
endif()

#
# Object Library target
#
    add_library(libiotsdk OBJECT ${source_c_files})
    set_property(TARGET libiotsdk PROPERTY POSITION_INDEPENDENT_CODE ON)

if(WIN32)
    target_include_directories(libiotsdk PUBLIC ${_IOTSDK_INC})
else()
    set_target_properties(libiotsdk PROPERTIES PREFIX "")
    target_include_directories(libiotsdk 
        PUBLIC ${_IOTSDK_INC}
        PRIVATE ${CURL_INCLUDE_DIRS}
    )
endif()
if(${use_openssl})
    include(FindOpenSSL)
if(${OPENSSL_FOUND})
    target_include_directories(libiotsdk PRIVATE "${OPENSSL_INCLUDE_DIR}")
else()
    message(FATAL_ERROR, "No OpenSSL found, please install!")
endif()
elseif(${use_wolfssl})
    target_include_directories(libiotsdk PRIVATE $ENV(WOLFSSL_INCLUDE_DIR))
endif()
