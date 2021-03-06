####################### CMakeLists.txt (libopenshot) #########################
# @brief CMake build file for libopenshot (used to generate Ruby SWIG bindings)
# @author Jonathan Thomas <jonathan@openshot.org>
#
# @section LICENSE
#
# Copyright (c) 2008-2019 OpenShot Studios, LLC
# <http://www.openshotstudios.com/>. This file is part of
# OpenShot Library (libopenshot), an open-source project dedicated to
# delivering high quality video editing and animation solutions to the
# world. For more information visit <http://www.openshot.org/>.
#
# OpenShot Library (libopenshot) is free software: you can redistribute it
# and/or modify it under the terms of the GNU Lesser General Public License
# as published by the Free Software Foundation, either version 3 of the
# License, or (at your option) any later version.
#
# OpenShot Library (libopenshot) is distributed in the hope that it will be
# useful, but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with OpenShot Library. If not, see <http://www.gnu.org/licenses/>.
################################################################################

# Pick up our include directories from the parent context
include_directories(${OPENSHOT_INCLUDE_DIRS})

############### RUBY BINDINGS ################
FIND_PACKAGE(SWIG 3.0 REQUIRED)
INCLUDE(${SWIG_USE_FILE})

### Enable some legacy SWIG behaviors, in newer CMAKEs
if (POLICY CMP0078)
	cmake_policy(SET CMP0078 OLD)
endif()
if (POLICY CMP0086)
	cmake_policy(SET CMP0086 OLD)
endif()

FIND_PACKAGE(Ruby)
IF (RUBY_FOUND)

	### Include the Ruby header files
	INCLUDE_DIRECTORIES(${RUBY_INCLUDE_DIRS})
	INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})

	### Enable C++ in SWIG
	set_property(SOURCE openshot.i PROPERTY CPLUSPLUS ON)
	set_property(SOURCE openshot.i PROPERTY SWIG_MODULE_NAME openshot)

	### Suppress a ton of warnings in the generated SWIG C++ code
	set(SWIG_CXX_FLAGS "-Wno-unused-variable -Wno-unused-function -Wno-deprecated-copy -Wno-class-memaccess -Wno-cast-function-type \
-Wno-unused-parameter -Wno-catch-value -Wno-sign-compare -Wno-ignored-qualifiers")
	separate_arguments(sw_flags UNIX_COMMAND ${SWIG_CXX_FLAGS})
	set_property(SOURCE openshot.i PROPERTY GENERATED_COMPILE_OPTIONS ${sw_flags})

	### Add the SWIG interface file (which defines all the SWIG methods)
	if (CMAKE_VERSION VERSION_LESS 3.8.0)
		swig_add_module(rbopenshot ruby openshot.i)
	else()
		swig_add_library(rbopenshot LANGUAGE ruby SOURCES openshot.i)
	endif()

	### Set name of target (with no prefix, since Ruby does not like that)
	SET_TARGET_PROPERTIES(${SWIG_MODULE_rbopenshot_REAL_NAME} PROPERTIES
	                      PREFIX "" OUTPUT_NAME "openshot")

	### Link the new Ruby wrapper library with libopenshot
	target_link_libraries(${SWIG_MODULE_rbopenshot_REAL_NAME}
	                      ${RUBY_LIBRARY} openshot)

	### FIND THE RUBY INTERPRETER (AND THE LOAD_PATH FOLDER)
	EXECUTE_PROCESS(COMMAND ${RUBY_EXECUTABLE}
	                -r rbconfig -e "print RbConfig::CONFIG['vendorarchdir']"
	                OUTPUT_VARIABLE RUBY_VENDOR_ARCH_DIR)
	MESSAGE(STATUS "Ruby executable: ${RUBY_EXECUTABLE}")
	MESSAGE(STATUS "Ruby vendor arch dir: ${RUBY_VENDOR_ARCH_DIR}")
	MESSAGE(STATUS "Ruby include path: ${RUBY_INCLUDE_PATH}")

	############### INSTALL HEADERS & LIBRARY ################
	# Install Ruby bindings
	install(TARGETS ${SWIG_MODULE_rbopenshot_REAL_NAME}
	        LIBRARY DESTINATION ${RUBY_VENDOR_ARCH_DIR} )

ENDIF (RUBY_FOUND)
