cmake_minimum_required(VERSION 3.18)
project(refcat2 C)

# Set build type to "RelWithDebInfo" per default, but allow user-specification.
if(NOT CMAKE_BUILD_TYPE) # built-in variable
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Build type" FORCE)
endif()
message(STATUS "Build type set to ${CMAKE_BUILD_TYPE} [cache variable]")



# Prepare system-wide install paths
include(GNUInstallDirs)
set(REFCAT2_BIN refcat)
set(REFCAT2_MAN refcat.man)
set(REFCAT2_DATADIR refcat)
# Set build tree output directories to mimick install tree output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR})
if(NOT EXISTS ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_MANDIR}/${REFCAT2_MAN})
  file(COPY ${CMAKE_SOURCE_DIR}/${REFCAT2_MAN}
      DESTINATION ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_MANDIR}
      USE_SOURCE_PERMISSIONS)
endif()
configure_file(${CMAKE_SOURCE_DIR}/sysconfig.h.in ${CMAKE_SOURCE_DIR}/sysconfig.h @ONLY)


# Fetch refcat2 data, if absent
set(WWW https://archive.stsci.edu/hlsp/atlas-refcat2)
set(MASTER_SITE https://archive.stsci.edu/hlsps/atlas-refcat2/orig)
unset(REFCAT2_ARCHIVES)
list(APPEND REFCAT2_ARCHIVES
    hlsp_atlas-refcat2_atlas_ccd_00-m-16_multi_v1_cat.tbz
    hlsp_atlas-refcat2_atlas_ccd_16-m-17_multi_v1_cat.tbz
    hlsp_atlas-refcat2_atlas_ccd_17-m-18_multi_v1_cat.tbz
    hlsp_atlas-refcat2_atlas_ccd_18-m-19_multi_v1_cat.tbz
    hlsp_atlas-refcat2_atlas_ccd_19-m-20_multi_v1_cat.tbz
)
unset(REFCAT2_HASHES)
list(APPEND REFCAT2_HASHES
    d725aaf7cd45715949a28e0efb929f51cf57abc1
    2ac4ea6a7d68b3685ef3c260ee1dd96b859c8948
    8e62a5be118c55982c3702983542e2c413713f3f
    946e56d729015509444a879d523e7f3432d5ae1b
    a7e7c64c72d9e2c8bb0794a9d2f3fe8e3b583a19
)
foreach(file hash IN ZIP_LISTS REFCAT2_ARCHIVES REFCAT2_HASHES)
  if(NOT EXISTS ${CMAKE_BINARY_DIR}/${file})
    message(STATUS "${file} does not exist in ${CMAKE_BINARY_DIR}.")
    message(STATUS "Downloading from ${WWW}/${file}...")
    message(STATUS "Note: File has a size of 5-17GB. If you have the data already at hand, interrupt the current process, place the data in ${CMAKE_BINARY_DIR} and re-run cmake.")
    list(APPEND CMAKE_MESSAGE_INDENT "  ")
    file(DOWNLOAD ${MASTER_SITE}/${file} ${CMAKE_BINARY_DIR}/${file}
        EXPECTED_HASH SHA1=${hash}
        SHOW_PROGRESS)
    list(POP_BACK CMAKE_MESSAGE_INDENT)
  endif()
endforeach()



# Extract refcat2 data, if not yet extracted
unset(REFCAT2_DIRS)
list(APPEND REFCAT2_DIRS
    00_m_16
    16_m_17
    17_m_18
    18_m_19
    19_m_20
)
foreach(dir archive IN ZIP_LISTS REFCAT2_DIRS REFCAT2_ARCHIVES)
  if(NOT EXISTS ${CMAKE_BINARY_DIR}/${dir})
    message(STATUS "${dir} does not exist in ${CMAKE_BINARY_DIR}.")
    message(STATUS "Extracting ${archive} to ${dir} in ${CMAKE_BINARY_DIR}...")
    file(ARCHIVE_EXTRACT INPUT ${CMAKE_BINARY_DIR}/${archive} 
        DESTINATION ${CMAKE_BINARY_DIR})
  endif()
endforeach()
# TODO: fetch and extract can be combined into one and be carried out in parallel with FetchContent?



# Target executable: 'refcat'
add_executable(refcat refcat.c sysconfig.h)
target_link_libraries(refcat m)
set_target_properties(refcat PROPERTIES RUNTIME_OUTPUT_NAME refcat)
get_target_property(REFCAT2_BIN refcat RUNTIME_OUTPUT_NAME)



# Compose "-dir ..." argument for CSV-to-binary compilation
set(REFCAT2_CSV_TO_BIN_ARG "-dir ")
foreach(dir IN LISTS REFCAT2_DIRS)
  string(PREPEND dir ${CMAKE_BINARY_DIR}/)
  string(APPEND REFCAT2_CSV_TO_BIN_ARG "${dir},")
endforeach()
# remove trailing ',' in ${REFCAT2_CSV_TO_BIN_ARG}
string(LENGTH ${REFCAT2_CSV_TO_BIN_ARG} _N)
math(EXPR _N "${_N} - 1")
string(SUBSTRING ${REFCAT2_CSV_TO_BIN_ARG} 0 ${_N} REFCAT2_CSV_TO_BIN_ARG)



# Convert all csv files to binary format
add_custom_target(csv_to_bin ALL
    /usr/local/bin/bash -c "mkdir -p ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_DATADIR}/${REFCAT2_DATADIR} && ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${REFCAT2_BIN} 0 0 ${REFCAT2_CSV_TO_BIN_ARG} -CSV_to_binary ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_DATADIR}/${REFCAT2_DATADIR}"
    DEPENDS refcat
    BYPRODUCTS ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_DATADIR}/${REFCAT2_DATADIR}/*
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    VERBATIM USES_TERMINAL)



# Define install targets
install(TARGETS refcat 
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} 
    COMPONENT bin)
install(DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_DATADIR}/${REFCAT2_DATADIR}
    DESTINATION ${CMAKE_INSTALL_DATADIR}
    COMPONENT share)
install(FILES ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_MANDIR}/${REFCAT2_MAN}
    DESTINATION ${CMAKE_INSTALL_MANDIR}
    COMPONENT man)
