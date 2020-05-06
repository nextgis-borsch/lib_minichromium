################################################################################
# Copyright (C) 2020, NextGIS <info@nextgis.com>
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
project(mini_chromium)

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

set(VERSION 1.0.0)

#add_library(${LIB_NAME} STATIC)
function(mc_append_sources)
    list(TRANSFORM ARGN PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/base/")
    target_sources(${LIB_NAME} PRIVATE ${ARGN})
endfunction()


if(BUILD_SHARED_LIBS)
    #add_library(${LIB_NAME} SHARED ${TARGET_OBJECTS} ${PUBLIC_HDRS} ${RESOURCE_FILES})
	add_library(${LIB_NAME} SHARED)
    set_target_properties(${LIB_NAME} PROPERTIES VERSION ${VERSION})
    if(MSVC)
        #set_target_properties(${LIB_NAME} PROPERTIES SUFFIX "-${MAJOR_VER}_${MINOR_VER}.dll")
    endif()
else()
    #add_library(${LIB_NAME} STATIC ${TARGET_OBJECTS} ${PUBLIC_HDRS})
	add_library(${LIB_NAME} STATIC)
    set_target_properties(${LIB_NAME} PROPERTIES VERSION ${VERSION})
endif()

mc_append_sources(
    ../build/build_config.h
    atomicops.h
    atomicops_internals_atomicword_compat.h
    atomicops_internals_portable.h
    auto_reset.h
    bit_cast.h
    compiler_specific.h
    debug/alias.cc
    debug/alias.h
    files/file_path.cc
    files/file_path.h
    files/file_util.h
    files/scoped_file.cc
    files/scoped_file.h
    format_macros.h
    logging.cc
    logging.h
    macros.h
    memory/free_deleter.h
    memory/scoped_policy.h
    metrics/histogram_functions.h
    metrics/histogram_macros.h
    metrics/persistent_histogram_allocator.h
    numerics/checked_math.h
    numerics/checked_math_impl.h
    numerics/clamped_math.h
    numerics/clamped_math_impl.h
    numerics/safe_conversions.h
    numerics/safe_conversions_arm_impl.h
    numerics/safe_conversions_impl.h
    numerics/safe_math.h
    numerics/safe_math_arm_impl.h
    numerics/safe_math_clang_gcc_impl.h
    numerics/safe_math_shared_impl.h
    process/memory.cc
    process/memory.h
    process/process_metrics.h
    rand_util.cc
    rand_util.h
    scoped_clear_last_error.h
    scoped_generic.h
    stl_util.h
    strings/string16.cc
    strings/string16.h
    strings/string_number_conversions.cc
    strings/string_number_conversions.h
    strings/string_piece.h
    strings/string_util.h
    strings/stringprintf.cc
    strings/stringprintf.h
    strings/sys_string_conversions.h
    strings/utf_string_conversion_utils.h
    strings/utf_string_conversions.cc
    strings/utf_string_conversions.h
    synchronization/condition_variable.h
    synchronization/lock.cc
    synchronization/lock.h
    synchronization/lock_impl.h
    sys_byteorder.h
    template_util.h
    third_party/icu/icu_utf.cc
    third_party/icu/icu_utf.h
    threading/thread_local_storage.cc
    threading/thread_local_storage.h
)

if(NOT MINGW)
    mc_append_sources(
        strings/utf_string_conversion_utils.cc
    )
else()
    mc_append_sources(
        ../../utf_string_conversion_utils.mingw.cc
    )
endif()

if(APPLE)
    mc_append_sources(
        mac/close_nocancel.cc
        mac/foundation_util.h
        mac/foundation_util.mm
        mac/mach_logging.cc
        mac/mach_logging.h
        mac/scoped_cftyperef.h
        mac/scoped_ioobject.h
        mac/scoped_launch_data.h
        mac/scoped_mach_port.cc
        mac/scoped_mach_port.h
        mac/scoped_mach_vm.cc
        mac/scoped_mach_vm.h
        mac/scoped_nsautorelease_pool.h
        mac/scoped_nsautorelease_pool.mm
        mac/scoped_nsobject.h
        mac/scoped_typeref.h
        strings/sys_string_conversions_mac.mm
    )
endif()

if(WIN32)
    mc_append_sources(
        process/process_metrics_win.cc
        scoped_clear_last_error_win.cc
        strings/string_util_win.cc
        strings/string_util_win.h
        synchronization/lock_impl_win.cc
        threading/thread_local_storage_win.cc
    )
else()
    mc_append_sources(
        files/file_util_posix.cc
        posix/eintr_wrapper.h
        posix/safe_strerror.cc
        posix/safe_strerror.h
        process/process_metrics_posix.cc
        strings/string_util_posix.h
        synchronization/condition_variable_posix.cc
        synchronization/lock_impl_posix.cc
        threading/thread_local_storage_posix.cc
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

#target_include_directories(${LIB_NAME} PUBLIC
#    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>"
#    $<INSTALL_INTERFACE:include/crashpad/mini_chromium>
#)
#target_link_libraries(mini_chromium
#    PRIVATE
#        $<BUILD_INTERFACE:crashpad_interface>
#)

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
    elseif(MINGW)
        target_compile_options(${LIB_NAME} PRIVATE 
            "-municode"
            "-Wno-format"
            "-Wno-unknown-pragmas"
        )
    endif()
endif()

#add_library(crashpad::mini_chromium ALIAS mini_chromium)



# Install
if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
    install(TARGETS ${TARGETS}
        EXPORT ${PACKAGE_UPPER_NAME}Targets
        RUNTIME DESTINATION ${INSTALL_BIN_DIR} # at least for dlls
        ARCHIVE DESTINATION ${INSTALL_LIB_DIR}
        LIBRARY DESTINATION ${INSTALL_LIB_DIR}
        INCLUDES DESTINATION ${INSTALL_INC_DIR}
        FRAMEWORK DESTINATION ${INSTALL_LIB_DIR})
endif()

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

# Create the ZLIBConfig.cmake file
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

include(util)

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

if(BUILD_TESTINTG)
    enable_testing()
    add_test(sqlite_app ${PROJECT_NAME} -version)
endif()
