#
# \file CMakeLists.txt
# \brief cmake configuration file for generating IT++ library
# \author Bogdan Cristea
#
# -------------------------------------------------------------------------
#
# Copyright (C) 1995-2012  (see AUTHORS file for a list of contributors)
#
# This file is part of IT++ - a C++ library of mathematical, signal
# processing, speech processing, and communications classes and functions.
#
# IT++ is free software: you can redistribute it and/or modify it under the
# terms of the GNU General Public License as published by the Free Software
# Foundation, either version 3 of the License, or (at your option) any
# later version.
#
# IT++ is distributed in the hope that it will be useful, but WITHOUT ANY
# WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
# FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more
# details.
#
# You should have received a copy of the GNU General Public License along
# with IT++.  If not, see <http://www.gnu.org/licenses/>.
#
# -------------------------------------------------------------------------

file ( GLOB ITPP_SRCS
       "base/*.cpp"
       "base/algebra/*.cpp"
       "base/bessel/*.cpp"
       "base/math/*.cpp"
       "comm/*.cpp" 
       "fixed/*.cpp"
       "optim/*.cpp"
       "protocol/*.cpp"
       "signal/*.cpp"
       "srccode/*.cpp"
       "stat/*.cpp" )

include_directories ( ${CMAKE_BINARY_DIR}
                      ${CMAKE_SOURCE_DIR}
                      ${BLAS_INCLUDES}
                      ${LAPACK_INCLUDES}
                      ${FFT_INCLUDES}
                    )

if (CMAKE_BUILD_TYPE STREQUAL Static)
  add_library (${libitpp_target} STATIC ${ITPP_SRCS})
else()
  add_library (${libitpp_target} SHARED ${ITPP_SRCS})
endif()

target_link_libraries (${libitpp_target} ${ITPP_LIBS})

set(LIBITPP_VERSION_MAJOR 8)
set(LIBITPP_VERSION_MINOR 1)
set(LIBITPP_VERSION_PATCH 0)
set(LIBITPP_VERSION_STRING ${LIBITPP_VERSION_MAJOR}.${LIBITPP_VERSION_MINOR}.${LIBITPP_VERSION_PATCH})

set_target_properties(${libitpp_target} PROPERTIES VERSION ${LIBITPP_VERSION_STRING}
                                                   SOVERSION ${LIBITPP_VERSION_MAJOR})

#library                                                 
install(TARGETS ${libitpp_target}
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

#headers      
install(DIRECTORY ${CMAKE_SOURCE_DIR}/itpp DESTINATION include FILES_MATCHING PATTERN "*.h")
if (NOT WIN32)
  install(FILES ${CMAKE_BINARY_DIR}/itpp/config.h DESTINATION include/itpp)
else()
  install(FILES ${CMAKE_BINARY_DIR}/itpp/config_msvc.h DESTINATION include/itpp)
endif()

#extra files (MATLAB and python)
install(FILES ${CMAKE_SOURCE_DIR}/extras/itsave.m
              ${CMAKE_SOURCE_DIR}/extras/itload.m
              ${CMAKE_SOURCE_DIR}/extras/pyitpp.py
        DESTINATION share/itpp)

#html help
if (NOT NO_DOCS)
  if(DOXYGEN_FOUND)
    configure_file(${CMAKE_SOURCE_DIR}/doc/doxygen_html.cfg.cmake.in ${CMAKE_BINARY_DIR}/Doxyfile @ONLY)
    configure_file(${CMAKE_SOURCE_DIR}/doc/local/index.doc.in ${CMAKE_BINARY_DIR}/doc/local/index.doc @ONLY)
    add_custom_target(doc ALL
      ${DOXYGEN_EXECUTABLE} ${CMAKE_BINARY_DIR}/Doxyfile
      WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
      COMMENT "Generating API documentation with Doxygen" VERBATIM)
    install(DIRECTORY ${CMAKE_BINARY_DIR}/html DESTINATION share/doc/itpp)
  endif(DOXYGEN_FOUND)
endif()