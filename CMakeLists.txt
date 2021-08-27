################################################################################
# Copyright (C) 2020-2021, NextGIS <info@nextgis.com>
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
# OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
################################################################################

cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)
project(minichromium)

if(NOT DEFINED PACKAGE_VENDOR)
    set(PACKAGE_VENDOR NextGIS)
endif()

if(NOT DEFINED PACKAGE_BUGREPORT)
    set(PACKAGE_BUGREPORT info@nextgis.com)
endif()

# some init settings
set(CMAKE_COLOR_MAKEFILE ON)
# set path to additional CMake modules
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

if(OSX_FRAMEWORK AND (BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS))
  message(FATAL_ERROR "Only OSX_FRAMEWORK key or any or both BUILD_SHARED_LIBS
                       and BUILD_STATIC_LIBS keys are permitted")
endif()

if(OSX_FRAMEWORK)
    set(INSTALL_BIN_DIR "bin" CACHE INTERNAL "Installation directory for executables" FORCE)
    set(INSTALL_LIB_DIR "Library/Frameworks" CACHE INTERNAL "Installation directory for libraries" FORCE)
    set(INSTALL_CMAKECONF_DIR ${INSTALL_LIB_DIR}/${PROJECT_NAME}.framework/Resources/CMake CACHE INTERNAL "Installation directory for cmake config files" FORCE)
    set(INSTALL_INC_DIR ${INSTALL_LIB_DIR}/${PROJECT_NAME}.framework/Headers CACHE INTERNAL "Installation directory for headers" FORCE)
    set(SKIP_INSTALL_HEADERS ON)
    set(SKIP_INSTALL_EXECUTABLES ON)
    set(SKIP_INSTALL_FILES ON)
    set(SKIP_INSTALL_EXPORT ON)
    set(CMAKE_MACOSX_RPATH ON)
else()
    include(GNUInstallDirs)

    set(INSTALL_BIN_DIR ${CMAKE_INSTALL_BINDIR} CACHE INTERNAL "Installation directory for executables" FORCE)
    set(INSTALL_LIB_DIR ${CMAKE_INSTALL_LIBDIR} CACHE INTERNAL "Installation directory for libraries" FORCE)
    set(INSTALL_INC_DIR ${CMAKE_INSTALL_INCLUDEDIR} CACHE INTERNAL "Installation directory for headers" FORCE)
    set(INSTALL_PKGCONFIG_DIR "${INSTALL_LIB_DIR}/pkgconfig" CACHE INTERNAL "Installation directory for pkgconfig (.pc) files" FORCE)
    set(INSTALL_CMAKECONF_DIR ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}/CMake CACHE INTERNAL "Installation directory for cmake config files" FORCE)
endif()

set(PACKAGE_NAME ${PROJECT_NAME})
string(TOUPPER ${PACKAGE_NAME} PACKAGE_UPPER_NAME)

set(LIB_NAME ${PROJECT_NAME})
set(TARGETS ${LIB_NAME})

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(util)

check_version(MAJOR_VER MINOR_VER REL_VER FIX_VER)
set(VERSION ${MAJOR_VER}.${MINOR_VER}.${REL_VER})
report_version(${PROJECT_NAME} ${VERSION})

add_library(${LIB_NAME} STATIC) 

set_target_properties(${LIB_NAME} PROPERTIES VERSION ${VERSION})

target_sources(${LIB_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/build/build_config.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/atomicops.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/atomicops_internals_atomicword_compat.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/atomicops_internals_portable.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/auto_reset.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/bit_cast.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/check.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/check_op.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/compiler_specific.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/cxx17_backports.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/debug/alias.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/debug/alias.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_path.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_util.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/files/scoped_file.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/files/scoped_file.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/format_macros.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/logging.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/logging.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/macros.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/memory/free_deleter.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/memory/page_size.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/memory/scoped_policy.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/metrics/histogram_functions.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/metrics/histogram_macros.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/metrics/persistent_histogram_allocator.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/notreached.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/checked_math.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/checked_math_impl.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/clamped_math.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/clamped_math_impl.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/safe_conversions.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/safe_conversions_arm_impl.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/safe_conversions_impl.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/safe_math.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/safe_math_arm_impl.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/safe_math_clang_gcc_impl.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/numerics/safe_math_shared_impl.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/process/memory.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/process/memory.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/rand_util.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/rand_util.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/scoped_clear_last_error.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/scoped_generic.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/string_number_conversions.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/string_number_conversions.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/string_piece.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/string_util.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/stringprintf.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/stringprintf.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/sys_string_conversions.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/utf_string_conversion_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/utf_string_conversions.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/utf_string_conversions.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/condition_variable.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/lock.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/lock.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/lock_impl.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/sys_byteorder.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/template_util.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/third_party/icu/icu_utf.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/third_party/icu/icu_utf.h
    ${CMAKE_CURRENT_SOURCE_DIR}/base/threading/thread_local_storage.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/base/threading/thread_local_storage.h
)

if(APPLE)
    target_sources(${LIB_NAME} PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/close_nocancel.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/foundation_util.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/foundation_util.mm
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/mach_logging.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/mach_logging.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_cftyperef.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_ioobject.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_launch_data.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_mach_port.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_mach_port.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_mach_vm.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_mach_vm.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_nsautorelease_pool.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_nsautorelease_pool.mm
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_nsobject.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/mac/scoped_typeref.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/sys_string_conversions_mac.mm
    )
endif()

if(WIN32)
    target_sources(${LIB_NAME} PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/base/memory/page_size_win.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/scoped_clear_last_error_win.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/string_util_win.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/string_util_win.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/lock_impl_win.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/threading/thread_local_storage_win.cc
    )
else()
    target_sources(${LIB_NAME} PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/base/files/file_util_posix.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/memory/page_size_posix.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/posix/eintr_wrapper.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/posix/safe_strerror.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/posix/safe_strerror.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/strings/string_util_posix.h
        ${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/condition_variable_posix.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/synchronization/lock_impl_posix.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/base/threading/thread_local_storage_posix.cc
    )
endif()

if(APPLE)
    target_link_libraries(${LIB_NAME} PUBLIC
        "-framework ApplicationServices"
        "-framework CoreFoundation"
        "-framework Foundation"
        "-framework IOKit"
        "-framework Security"
    )
endif()

if(WIN32)
    target_link_libraries(${LIB_NAME} PRIVATE advapi32 kernel32)
    if(MSVC)
        target_compile_options(${LIB_NAME} PRIVATE "/wd4201" "/wd4996")
        target_compile_definitions(${LIB_NAME} PRIVATE
            NOMINMAX
            UNICODE
            WIN32_LEAN_AND_MEAN
            _CRT_SECURE_NO_WARNINGS
            _HAS_EXCEPTIONS=0
            _UNICODE
        )
    endif()
endif()


if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
    install(TARGETS ${TARGETS}
        EXPORT ${PACKAGE_UPPER_NAME}Targets
        RUNTIME DESTINATION ${INSTALL_BIN_DIR} # at least for dlls
        ARCHIVE DESTINATION ${INSTALL_LIB_DIR}
        LIBRARY DESTINATION ${INSTALL_LIB_DIR}
        INCLUDES DESTINATION ${INSTALL_INC_DIR}
        FRAMEWORK DESTINATION ${INSTALL_LIB_DIR}
    )

    install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/base"
        DESTINATION "${INSTALL_INC_DIR}"
        FILES_MATCHING PATTERN "*.h"
    )

    install(FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/build/build_config.h
        ${CMAKE_CURRENT_SOURCE_DIR}/build/buildflag.h
        DESTINATION "${INSTALL_INC_DIR}/build/"
    )
endif()

add_custom_target(uninstall COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

# Export package ===============================================================

# Add path to includes to build-tree export
target_include_directories(${TARGETS} PUBLIC
 $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
 $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
)

# Add all targets to the build-tree export set
export(TARGETS ${TARGETS}
   FILE ${PROJECT_BINARY_DIR}/${PACKAGE_UPPER_NAME}Targets.cmake)

if(REGISTER_PACKAGE)
   # Export the package for use from the build-tree
   # (this registers the build-tree with a global CMake-registry)
   export(PACKAGE ${PACKAGE_UPPER_NAME})
endif()

# Create the MINICHROMIUMConfig.cmake file
configure_file(cmake/PackageConfig.cmake.in
   ${PROJECT_BINARY_DIR}/${PACKAGE_UPPER_NAME}Config.cmake @ONLY)

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
   # Install the <Package>Config.cmake
   install(FILES
     ${PROJECT_BINARY_DIR}/${PACKAGE_UPPER_NAME}Config.cmake
     DESTINATION ${INSTALL_CMAKECONF_DIR} COMPONENT dev)

   # Install the export set for use with the install-tree
   install(EXPORT ${PACKAGE_UPPER_NAME}Targets DESTINATION ${INSTALL_CMAKECONF_DIR} COMPONENT dev)
endif()

# Archiving ====================================================================

set(CPACK_PACKAGE_NAME "${PACKAGE_NAME}")
set(CPACK_PACKAGE_VENDOR "${PACKAGE_VENDOR}")
set(CPACK_PACKAGE_VERSION "${VERSION}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "${PACKAGE_NAME} Installation")
set(CPACK_PACKAGE_RELOCATABLE TRUE)
set(CPACK_ARCHIVE_COMPONENT_INSTALL ON)
set(CPACK_GENERATOR "ZIP")
set(CPACK_MONOLITHIC_INSTALL ON)
set(CPACK_STRIP_FILES TRUE)

# Get cpack zip archive name
get_cpack_filename(${VERSION} PROJECT_CPACK_FILENAME)
set(CPACK_PACKAGE_FILE_NAME ${PROJECT_CPACK_FILENAME})

include(CPack)