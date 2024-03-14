cmake_minimum_required(VERSION 3.20)
project(CepGen-website)

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR} ${CMAKE_SOURCE_DIR}/cmake)

find_package(Doxygen REQUIRED)
find_package(Sphinx REQUIRED)

set(CEPGEN_VERSION "${VERSION}") #FIXME
string(TIMESTAMP CURRENT_YEAR "%Y")

# Doxygen build

set(DOXYGEN_INPUT_DIR "$ENV{CEPGEN_PATH}/src \\
                       $ENV{CEPGEN_PATH}/CepGen \\
                       $ENV{CEPGEN_PATH}/CepGenAddOns \\
                       $ENV{CEPGEN_PATH}/CepGenProcesses \\
                       $ENV{CEPGEN_PATH}/python")
set(DOXYGEN_OUTPUT_DIR ${CMAKE_CURRENT_BINARY_DIR}/doxygen)
set(DOXYGEN_INDEX_FILE ${DOXYGEN_OUTPUT_DIR}/xml/index.xml)
set(DOXYGEN_XML YES)
set(DOXYGEN_HTML NO)
set(DOXYFILE_IN ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in)
set(DOXYFILE_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

configure_file(${DOXYFILE_IN} ${DOXYFILE_OUT} @ONLY)

# Doxygen won't create this for us
file(MAKE_DIRECTORY ${DOXYGEN_OUTPUT_DIR})
message(STATUS "Doxygen build started")
add_custom_command(OUTPUT ${DOXYGEN_INDEX_FILE}
                   COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYFILE_OUT}
                   MAIN_DEPENDENCY ${DOXYFILE_OUT} ${DOXYFILE_IN}
                   COMMENT "Generating documentation with Doxygen"
                   VERBATIM)
add_custom_target(Doxygen DEPENDS ${DOXYGEN_INDEX_FILE})

# Sphinx build

set(SPHINX_SOURCE ${CMAKE_CURRENT_SOURCE_DIR})
set(SPHINX_BUILD ${CMAKE_CURRENT_BINARY_DIR}/output_html)
set(SPHINX_INDEX_FILE ${SPHINX_BUILD}/index.html)
set(SPHINX_IN ${CMAKE_CURRENT_SOURCE_DIR}/conf.in.py)
set(SPHINX_OUT ${SPHINX_SOURCE}/conf.py)

configure_file(${SPHINX_IN} ${SPHINX_OUT} @ONLY)

message(STATUS "Sphinx found in ${SPHINX_EXECUTABLE}")
add_custom_target(Sphinx
                  COMMAND ${SPHINX_EXECUTABLE} -b html
                  -Dbreathe_projects.CepGen=${DOXYGEN_OUTPUT_DIR}/xml  # tell Breathe where to find the Doxygen output
                  ${SPHINX_SOURCE} ${SPHINX_BUILD}
                  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
                  DEPENDS ${SPHINX_SOURCE}/index.md ${DOXYGEN_INDEX_FILE}
                  #MAIN_DEPENDENCY ${SPHINX_OUT}
                  COMMENT "Generating documentation with Sphinx")