cmake_minimum_required(VERSION 2.8.7 FATAL_ERROR)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------

set(CPM_MODULE_NAME udis86)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_VARIABLE head_sha)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()
CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# udis86
#------------------------------------------------------------------------------

set(UDIS86_DIR "${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/udis86")
CPM_EnsureRepoIsCurrent(
  TARGET_DIR ${UDIS86_DIR}
  GIT_REPOSITORY "https://github.com/vmt/udis86.git"
  GIT_TAG "master"
  USE_CACHING TRUE)

set(ITAB_H "${UDIS86_DIR}/libudis86/itab.h")
set(ITAB_C "${UDIS86_DIR}/libudis86/itab.c")

if(NOT EXISTS ${ITAB_H} OR NOT EXISTS ${ITAB_C})
  add_custom_command(
    OUTPUT ${ITAB_H} ${ITAB_C}
    COMMAND python "${UDIS86_DIR}/scripts/ud_itab.py" "${UDIS86_DIR}/docs/x86/optable.xml" "${UDIS86_DIR}/libudis86")
endif()

#-----------------------------------------------------------------------
# Source
#-----------------------------------------------------------------------

set(ModuleHeaders
  "${UDIS86_DIR}/udis86.h"
  "${UDIS86_DIR}/libudis86/decode.h"
  "${UDIS86_DIR}/libudis86/itab.h"
  "${UDIS86_DIR}/libudis86/extern.h"
  "${UDIS86_DIR}/libudis86/syn.h"
  "${UDIS86_DIR}/libudis86/types.h"
  "${UDIS86_DIR}/libudis86/udint.h")

set(ModuleSources
  "${UDIS86_DIR}/libudis86/decode.c"
  "${UDIS86_DIR}/libudis86/itab.c"
  "${UDIS86_DIR}/libudis86/syn.c"
  "${UDIS86_DIR}/libudis86/syn-att.c"
  "${UDIS86_DIR}/libudis86/syn-intel.c"
  "${UDIS86_DIR}/libudis86/udis86.c")

#-----------------------------------------------------------------------
# Library setup
#-----------------------------------------------------------------------

# Build the library.
add_library(${CPM_LIB_TARGET_NAME} ${ModuleSources} ${ModuleHeaders})

CPM_ExportAdditionalIncludeDir("${UDIS86_DIR}")
