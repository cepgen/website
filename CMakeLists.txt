cmake_minimum_required(VERSION 3.20)
project(CepGen-website)

set(CEPGEN_PATH $ENV{CEPGEN_PATH})
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR} ${CMAKE_SOURCE_DIR}/cmake)

include($ENV{CEPGEN_PATH}/cmake/CepGenVersion.cmake)

find_package(Doxygen REQUIRED)
find_package(Sphinx REQUIRED)

set(CEPGEN_VERSION "${VERSION}")
string(TIMESTAMP CURRENT_YEAR "%Y")

set(SPHINX_OUTPUT_DIR ${CMAKE_CURRENT_BINARY_DIR}/output)
set(DOXYGEN_OUTPUT_DIR ${SPHINX_OUTPUT_DIR}/docs)

# Doxygen build

file(GLOB_RECURSE cepgen_sources ${CEPGEN_PATH}/CepGen*/*.cpp ${CEPGEN_PATH}/src/*.cc ${CEPGEN_PATH}/python/*.py)

set(DOXYGEN_INPUT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/Doxygen_index.md \\
                       $ENV{CEPGEN_PATH}/src \\
                       $ENV{CEPGEN_PATH}/CepGen \\
                       $ENV{CEPGEN_PATH}/CepGenAddOns \\
                       $ENV{CEPGEN_PATH}/CepGenProcesses \\
                       $ENV{CEPGEN_PATH}/python")
set(DOXYGEN_XML_DIR ${CMAKE_CURRENT_BINARY_DIR}/xml)
set(DOXYGEN_INDEX_FILE ${DOXYGEN_XML_DIR}/index.xml)
set(DOXYGEN_XML YES)
set(DOXYGEN_HTML YES)
set(DOXYFILE_IN ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in)
set(DOXYFILE_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

configure_file(${DOXYFILE_IN} ${DOXYFILE_OUT} @ONLY)

# Doxygen won't create this for us
file(MAKE_DIRECTORY ${DOXYGEN_OUTPUT_DIR})
message(STATUS "Doxygen build started")
add_custom_command(OUTPUT ${DOXYGEN_INDEX_FILE} ${DOXYGEN_OUTPUT_DIR}/index.html
                   COMMAND Doxygen::doxygen ${DOXYFILE_OUT}
                   MAIN_DEPENDENCY ${DOXYFILE_OUT} ${DOXYFILE_IN}
                   DEPENDS ${cepgen_sources}
                   COMMENT "Generating documentation with Doxygen")
add_custom_target(Doxygen DEPENDS ${DOXYGEN_INDEX_FILE} ${DOXYGEN_OUTPUT_DIR}/index.html)

# Sphinx build

set(SPHINX_SOURCE ${CMAKE_CURRENT_SOURCE_DIR})
set(SPHINX_INDEX_FILE ${SPHINX_OUTPUT_DIR}/index.html)
set(SPHINX_OUT ${CMAKE_CURRENT_SOURCE_DIR}/conf.py)

configure_file(${SPHINX_SOURCE}/conf.in.py ${SPHINX_OUT} @ONLY)

message(STATUS "Sphinx found in ${SPHINX_EXECUTABLE}")
add_custom_target(Sphinx ALL
                  COMMAND ${SPHINX_EXECUTABLE} -b html
                  -Dbreathe_projects.CepGen=${DOXYGEN_XML_DIR}  # tell Breathe where to find the Doxygen output
                  ${SPHINX_SOURCE} ${SPHINX_OUTPUT_DIR}
                  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
                  DEPENDS ${SPHINX_SOURCE}/index.md ${DOXYGEN_INDEX_FILE}
                  COMMENT "Generating documentation with Sphinx")
