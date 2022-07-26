# Copyright (c) 2022 Linaro Limited
#
# SPDX-License-Identifier: BSD-3-Clause

cmake_minimum_required(VERSION 3.13.1)

# set(CRYPTO_PROVIDER "MbedTLS")
# add_subdirectory(t_cose)

project(ietf114_hacking
    DESCRIPTION "Example of message encoding with t_cose and friends"
    LANGUAGES C
    VERSION 1.0.0)

#set(CMAKE_C_FLAGS "-Wall -O3 -pedantic")

######################################################################
# Set up Mbed TLS
option(USE_PKCS11_HELPER_LIBRARY "Build mbed TLS with the pkcs11-helper library." OFF)
option(ENABLE_ZLIB_SUPPORT "Build mbed TLS with zlib library." OFF)
option(ENABLE_PROGRAMS "Build mbed TLS programs." OFF)
option(ENABLE_TESTING "Build mbed TLS tests." OFF)

set(CMAKE_INSTALL_NAME_DIR ${CMAKE_INSTALL_FULL_LIBDIR}/lib)
include_directories(external/mbedtls/include)

# Use custom config file for Mbed TLS
add_definitions(-DMBEDTLS_CONFIG_FILE="${CMAKE_SOURCE_DIR}/mbedtls_config.h")

option(USE_STATIC_MBEDTLS_LIBRARY "Build mbed TLS static library." on)
option(USE_SHARED_MBEDTLS_LIBRARY "Build mbed TLS shared library." off)

if (CMAKE_SYSTEM MATCHES "OpenBSD")
        option(LINK_WITH_PTHREAD "Explicitly link mbed TLS library to pthread." on)
endif()

set(MBEDTLS_LIBRARIES "mbedtls") #  "mbedx509" "mbedcrypto"

add_subdirectory(external/mbedtls mbedtls)

######################################################################
# Setup QCBOR

add_subdirectory(external/QCBOR qcbor)
set(QCBOR_LIBRARY "qcbor")
set(CRYPTO_PROVIDER "MbedTLS")
set(MBEDTLS True)
include_directories(external/QCBOR/inc)

######################################################################
# Setup t_cose

# add_subdirectory(external/t_cose t_cose)
set(T_COSE_LIBRARY "t_cose")
# include_directories(external/QCBOR/inc)
include_directories(external/t_cose/inc)
include_directories(external/t_cose/src)

add_definitions(-DT_COSE_USE_PSA_CRYPTO)

# Make our own library for t_cose.
add_library(t_cose
    external/t_cose/src/t_cose_aes_kw.c
    external/t_cose/src/t_cose_encrypt_dec.c
    external/t_cose/src/t_cose_encrypt_enc.c
    external/t_cose/src/t_cose_hpke.c
    external/t_cose/src/t_cose_parameters.c
    external/t_cose/src/t_cose_sign1_sign.c
    external/t_cose/src/t_cose_sign1_verify.c
    external/t_cose/src/t_cose_util.c
    external/t_cose/crypto_adapters/t_cose_psa_crypto.c)

######################################################################
add_executable(encrypter
    src/main.c
    src/pdump.c
    )
target_link_libraries(encrypter mbedtls t_cose qcbor)
