# MIT License
#
# Copyright (c) 2023-2026 The OneLuaPro project authors.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy of
# this software and associated documentation files (the "Software"), to deal in 
# the Software without restriction, including without limitation the rights to 
# use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
# the Software, and to permit persons to whom the Software is furnished to do so,
# subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all 
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
#
# ------------------------------------------------------------------------------
# CMake for argparse
# ------------------------------------------------------------------------------
# Setup with Visual Studio 17 2022 generator for x64
# ------------------------------------------------------------------------------
# Visual Studio is a mutlti-configuration generator
# https://stackoverflow.com/questions/24460486/
#
# Basic instructions for out-of-source build
# ------------------------------------------
# mkdir build && cd build
# cmake .. -G "Visual Studio 17 2022" -A x64
# cmake --build . --config Release
# cmake --install . --config Release
# 
# Available architectures (-A ...) are: Win32, x64, ARM, ARM64
#
# Note: As argparse does not comprise any files to be compiled and linked (just
# files to be installed properly) the mention of architecture or config is
# somewhat arbitrary.

# ------------------------------------------------------------------------------
# General definitions
cmake_minimum_required(VERSION 3.23 FATAL_ERROR)

# ------------------------------------------------------------------------------
# find liblua installation and version info
if(NOT LUA_HINTS)
  if(WIN32)
    set(LUA_HINTS "c:/Apps")
  endif()
endif()
find_package(liblua REQUIRED CONFIG HINTS ${LUA_HINTS})
if(liblua_FOUND)
  message(STATUS "liblua version        : ${liblua_VERSION}")
  message(STATUS "liblua install prefix : ${LIBLUA_INSTALLDIR}")
  message(STATUS "liblua include dir    : ${LIBLUA_INCLUDEDIR}")
  message(STATUS "liblua lib dir        : ${LIBLUA_LIBDIR}")
else()
  message(FATAL_ERROR "Unable to find liblua version ${liblua_VERSION}.")
endif()
# Note: liblua_VERSION is set by find_package() directly. LIBLUA_INSTALLDIR,
# LIBLUA_INCLUDEDIR and LIBLUA_LIBDIR are set by libluaConfigVersion.cmake in
# <prefix>/share/cmake/liblua.

# ------------------------------------------------------------------------------
# Installation prefix directory - automatically set from find_package()
# Needs to be defined before project definition statement - for whatever reason
if(NOT CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX ${LIBLUA_INSTALLDIR})
endif()

# ------------------------------------------------------------------------------
# Project defintion
project(argparse C)

# ------------------------------------------------------------------------------
# Other settings
set(CMAKE_VERBOSE_MAKEFILE OFF)
# argparse has nothing to build, only some files to install
# set(CMAKE_SKIP_INSTALL_ALL_DEPENDENCY TRUE)

# ------------------------------------------------------------------------------
# Setup GNU-alike installatin directories
include (GNUInstallDirs)
set(INSTALL_DOCDIR
  ${CMAKE_INSTALL_DOCDIR} CACHE PATH "Installation directory for documentation")
set(INSTALL_DATAROOTDIR
  ${CMAKE_INSTALL_DATAROOTDIR} CACHE PATH "Installation directory for data")
# Lua-specific installation dirs
set(INSTALL_TOP_LDIR
  ${INSTALL_DATAROOTDIR}/lua/${liblua_VERSION_MAJOR}.${liblua_VERSION_MINOR})

# ------------------------------------------------------------------------------
# Documentation Generation (Pandoc)
find_program(PANDOC_EXECUTABLE pandoc DOC "Path to pandoc executable")
if(PANDOC_EXECUTABLE)
  # Collect all RST files from docsrc directory
  # Note: This will sort them alphabetically!
  file(GLOB ARGPARSE_RST_SOURCES 
    RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}/docsrc" 
    "${CMAKE_CURRENT_SOURCE_DIR}/docsrc/*.rst")

  # Ensure index.rst is ALWAYS first:
  list(REMOVE_ITEM ARGPARSE_RST_SOURCES "index.rst")
  set(ARGPARSE_RST_SOURCES "index.rst" ${ARGPARSE_RST_SOURCES})

  # Define build and output paths
  set(DOC_BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}/doc")
  set(DOC_HTML_OUTPUT "${DOC_BUILD_DIR}/index.html")
  # Add command to merge and convert RST files to HTML
  add_custom_command(
    OUTPUT "${DOC_HTML_OUTPUT}"
    COMMAND ${CMAKE_COMMAND} -E make_directory "${DOC_BUILD_DIR}"
    COMMAND ${PANDOC_EXECUTABLE} ${ARGPARSE_RST_SOURCES}
    -f rst -t html5 -s
    --toc --toc-depth=2
    --embed-resources
    --css=assets/oneluapro.css
    --metadata title=argparse
    -o "${DOC_HTML_OUTPUT}"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/docsrc"
    COMMENT "Merging and converting argparse RST documents to HTML"
    VERBATIM
  )
  # Create a target that is built by default
  add_custom_target(argparse_doc ALL DEPENDS "${DOC_HTML_OUTPUT}")
  # Install the generated HTML documentation
  install(FILES "${DOC_HTML_OUTPUT}" DESTINATION "${INSTALL_DOCDIR}")
else()
  message(FATAL_ERROR "Pandoc not found. Documentation will not be generated.")
endif()

# ------------------------------------------------------------------------------
# Install lua-file
install(FILES src/argparse.lua DESTINATION ${INSTALL_TOP_LDIR})
