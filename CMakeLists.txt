# ATLAS - Kick Designer, Tuner & Layering Plugin
# Targets: Windows (MSVC) and Linux. macOS is intentionally not supported.
cmake_minimum_required(VERSION 3.22)
project(Atlas VERSION 0.1.0 LANGUAGES CXX)

if(APPLE)
    message(FATAL_ERROR "ATLAS does not support macOS. Build on Windows or Linux.")
endif()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# --- Framework-independent DSP core (header-only) -------------------------------------------
add_library(atlas_dsp INTERFACE)
target_include_directories(atlas_dsp INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/src)
target_compile_features(atlas_dsp INTERFACE cxx_std_20)

# --- Automated tests ------------------------------------------------------------------------
option(ATLAS_BUILD_TESTS "Build DSP unit tests" ON)
if(ATLAS_BUILD_TESTS)
    enable_testing()
    add_executable(atlas_tests tests/main.cpp tests/test_pitch.cpp tests/test_realtime.cpp)
    target_link_libraries(atlas_tests PRIVATE atlas_dsp)
    if(MSVC)
        target_compile_options(atlas_tests PRIVATE /W4 /permissive-)
    else()
        target_compile_options(atlas_tests PRIVATE -Wall -Wextra -Wpedantic)
    endif()
    add_test(NAME atlas_dsp_tests COMMAND atlas_tests)
    install(TARGETS atlas_tests RUNTIME DESTINATION bin COMPONENT Runtime)
endif()

# --- Installer (CPack) ----------------------------------------------------------------------
# Windows: NSIS setup .exe (Start-menu entry, uninstaller, PATH-free) + portable ZIP.
# Linux:   TGZ archive.
# Build it with:  cmake --build build --config Release --target package
install(DIRECTORY src/ DESTINATION include/atlas COMPONENT Development FILES_MATCHING PATTERN "*.h")
install(FILES docs/STATUS.md DESTINATION share/doc/atlas COMPONENT Runtime)

set(CPACK_PACKAGE_NAME "ATLAS")
set(CPACK_PACKAGE_VENDOR "ATLAS")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "ATLAS - Kick Designer, Tuner & Layering Plugin")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "ATLAS")
set(CPACK_PACKAGE_FILE_NAME "ATLAS-${PROJECT_VERSION}-${CMAKE_SYSTEM_NAME}-setup")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/docs/STATUS.md")
set(CPACK_THREADS 0)
if(WIN32)
    set(CPACK_GENERATOR "NSIS;ZIP")
    set(CPACK_NSIS_DISPLAY_NAME "ATLAS ${PROJECT_VERSION}")
    set(CPACK_NSIS_PACKAGE_NAME "ATLAS")
    set(CPACK_NSIS_MODIFY_PATH OFF)
    set(CPACK_NSIS_ENABLE_UNINSTALL_BEFORE_INSTALL ON)
    set(CPACK_NSIS_MUI_FINISHPAGE_RUN "")
    if(ATLAS_BUILD_TESTS)
        set(CPACK_PACKAGE_EXECUTABLES "atlas_tests" "ATLAS DSP Tests")
    endif()
else()
    set(CPACK_GENERATOR "TGZ")
endif()
include(CPack)

# --- JUCE plugin (added in milestone 3) -----------------------------------------------------
# option(ATLAS_BUILD_PLUGIN "Build the VST3 / Standalone plugin (needs JUCE)" OFF)
