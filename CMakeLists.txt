# nVidia DirectDraw Fix
# Copyright (c) 2009 Julien Langer
# 
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
# 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

PROJECT( nvidia_ddraw_fix )
cmake_minimum_required(VERSION 2.6)

if (COMMAND cmake_policy)
	cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

OPTION(LOGGING
	"enable logging build"
	OFF
)

# override this with cmake -D CMAKE_BUILD_TYPE:STRING="Debug" CMakeLists.txt
IF (NOT CMAKE_BUILD_TYPE)
	SET(CMAKE_BUILD_TYPE Debug CACHE STRING "build type" FORCE)
ENDIF(NOT CMAKE_BUILD_TYPE)

if (NOT WINDRES)
	set(WINDRES windres)
endif (NOT WINDRES)

SET( NMAKE FALSE )

# add the build type (Release/Debug) to the output path
# this isn't required for MSVC as cmake does this automatically
IF(MSVC AND NOT CMAKE_BUILD_TOOL MATCHES "nmake")
	SET ( LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/bin )
	SET (NOT_USING_MSVC_IDE TRUE)
ELSE(MSVC AND NOT CMAKE_BUILD_TOOL MATCHES "nmake")
	SET( NMAKE TRUE )
	SET ( LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE} )
ENDIF(MSVC AND NOT CMAKE_BUILD_TOOL MATCHES "nmake")

SET ( EXECUTABLE_OUTPUT_PATH ${LIBRARY_OUTPUT_PATH} )

# MinGW specific
IF(CMAKE_COMPILER_IS_MINGW)
	SET(CMAKE_SHARED_LINKER_FLAGS "-Wl,--enable-auto-image-base,--enable-runtime-pseudo-reloc,--add-stdcall-alias,--output-def,${LIBRARY_OUTPUT_PATH}/out.def")
ENDIF(CMAKE_COMPILER_IS_MINGW)

# increase warning level
SET(CMAKE_CXX_WARNING_LEVEL 4)

#IF(CMAKE_BUILD_TOOL MATCHES "(msdev|devenv|nmake)")
IF(MSVC)
   # Use the highest warning level for visual studio.
   SET(CMAKE_CXX_WARNING_LEVEL 4)
   IF(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
     STRING(REGEX REPLACE "/W[0-4]" "/W4"
            CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
   ELSE(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
     SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
   ENDIF(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
   #ENDIF(CMAKE_BUILD_TOOL MATCHES "(msdev|devenv|nmake)")

   # don't warn when using the C runtime
   ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS)
ENDIF(MSVC)

SET (GCC FALSE)
if (CMAKE_COMPILER_IS_GNUCXX OR CMAKE_COMPILER_IS_CYGWIN OR CMAKE_COMPILER_IS_MINGW)
	SET (GCC TRUE)
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
endif (CMAKE_COMPILER_IS_GNUCXX OR CMAKE_COMPILER_IS_CYGWIN OR CMAKE_COMPILER_IS_MINGW)

# these defines are automatically added in the VC project files, so only do this for
# nmake and gcc builds
if (GCC OR NMAKE)
	if (CMAKE_BUILD_TYPE MATCHES "Release")
		ADD_DEFINITIONS(-DNDEBUG)
	endif (CMAKE_BUILD_TYPE MATCHES "Release")
	
	if (CMAKE_BUILD_TYPE MATCHES "Debug")
		ADD_DEFINITIONS(-D_DEBUG)
	endif (CMAKE_BUILD_TYPE MATCHES "Debug")
endif (GCC OR NMAKE)

if (LOGGING)
	ADD_DEFINITIONS(-DNVIDIA_DDRAW_FIX_LOGGING)
endif (LOGGING)

INCLUDE (CheckIncludeFiles)
INCLUDE( CheckFunctionExists )
CHECK_INCLUDE_FILES (stdint.h HAVE_STDINT_H)
CHECK_FUNCTION_EXISTS ( strncasecmp HAVE_STRNCASECMP )
CHECK_FUNCTION_EXISTS ( strnicmp HAVE_STRNICMP )

INCLUDE (cmake/add_rc_file.cmake)
CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)
include_directories ( ${CMAKE_CURRENT_SOURCE_DIR} )

add_subdirectory(src)

