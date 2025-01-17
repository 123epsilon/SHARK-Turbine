# Copyright 2023 Nod Labs, Inc
#
# Licensed under the Apache License v2.0 with LLVM Exceptions.
# See https://llvm.org/LICENSE.txt for license information.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

cmake_minimum_required(VERSION 3.21...3.24)

project(SHARK_TURBINE)

include(CMakeDependentOption)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)


# TODO: Fix this once the project is slotted into place.
if(NOT IREE_ROOT_DIR)
  set(IREE_ROOT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/vendor/iree")
endif()
if(NOT TORCH_MLIR_ROOT_DIR)
  set(TORCH_MLIR_ROOT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/vendor/torch-mlir")
endif()

set(IREE_CXX_STANDARD ${CMAKE_CXX_STANDARD})

# Enable testing before any subdirectories are added.
enable_testing()

#-------------------------------------------------------------------------------
# Add self to the list of compiler plugins.
#-------------------------------------------------------------------------------

list(APPEND IREE_COMPILER_BUILTIN_PLUGIN_PATHS "${CMAKE_CURRENT_SOURCE_DIR}/compiler")

#-------------------------------------------------------------------------------
# Set up IREE toolchain with OpenXLA Nvgpu modules.
#-------------------------------------------------------------------------------

# Export additional CAPI from libIREECompiler.so.
# See compiler/src/torch-mlir/CMakeLists.txt
set(IREE_COMPILER_API_ADDL_EXPORT_OBJECTS "torch-mlir_CAPI.objects")
set(IREE_INPUT_TORCH OFF CACHE BOOL "IREE in-tree Torch input disabled" FORCE)
add_subdirectory("${IREE_ROOT_DIR}" "iree")

# Handle various global definitions that need to be set at the global
# toolchain level.
iree_setup_toolchain()

#-------------------------------------------------------------------------------
# Output .env files.
#-------------------------------------------------------------------------------

if(IREE_BUILD_PYTHON_BINDINGS)
  # Write out a .env file to make IDEs and developers happy.
  # Yes, we are writing this to the source dir. It is only for IDEs and if
  # it gets clobbered, it is fine (it is also ignored in .gitignore).
  set(_PYTHONPATH_ENV "PYTHONPATH=$<SHELL_PATH:${CMAKE_CURRENT_BINARY_DIR}/iree/compiler/bindings/python;${CMAKE_CURRENT_BINARY_DIR}/iree/runtime/bindings/python>\n")
  file(GENERATE OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/.env" CONTENT "${_PYTHONPATH_ENV}")
  file(GENERATE OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/.env" CONTENT "${_PYTHONPATH_ENV}")
  # Similarly, write out .env.bat for Windows.
  set(_PYTHONPATH_ENV_BAT "set PYTHONPATH=$<SHELL_PATH:${CMAKE_CURRENT_BINARY_DIR}/iree/compiler/bindings/python;${CMAKE_CURRENT_BINARY_DIR}/iree/runtime/bindings/python>\n")
  file(GENERATE OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/.env.bat" CONTENT "${_PYTHONPATH_ENV_BAT}")
endif()
