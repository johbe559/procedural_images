cmake_minimum_required(VERSION 2.6.0)
project(terrain)

IF(UNIX)
SET(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} "-std=c++11 -Werror -Wall -Werror=sign-compare -Wno-error=unused-variable -Wno-error=unused-but-set-variable")
ENDIF(UNIX)

FILE(GLOB HEADERS  ./*.h*)

FILE(GLOB SRCS  ./*.cpp)

option (USE_OPENGL
  "Use openGL to render scene while calculating" ON)

list(APPEND terrain_INCLUDE_DIRS
	${CMAKE_SOURCE_DIR}/externals
	${CMAKE_SOURCE_DIR}/externals/glm
	${CMAKE_SOURCE_DIR}/externals/assimp
 	${CMAKE_SOURCE_DIR}/externals/FreeImage  
    ${CMAKE_SOURCE_DIR}/externals/tinyobjloader
)

if (USE_OPENGL)
  list(APPEND terrain_INCLUDE_DIRS
	${CMAKE_SOURCE_DIR}/externals/GL
	${CMAKE_SOURCE_DIR}/externals/GL/GL)
  set(EXTRA_LIBS ${EXTRA_LIBS} GL glfw glut)
  add_definitions(-DUSE_OPENGL)
endif (USE_OPENGL)

# Random generator stuff
list(APPEND terrain_INCLUDE_DIRS
	${CMAKE_SOURCE_DIR}/sfmt
)

#FILE(GLOB_RECURSE SFMT_H ${CMAKE_SOURCE_DIR}/sfmt/*.h*)
#FILE(GLOB_RECURSE SFMT_CPP ${CMAKE_SOURCE_DIR}/sfmt/*.c)

FILE(GLOB_RECURSE TNYOBJ_H ${CMAKE_SOURCE_DIR}/externals/tinyobjloader/*.h*)
FILE(GLOB_RECURSE TNYOBJ_CPP ${CMAKE_SOURCE_DIR}/externals/tinyobjloader/*.cc)


list(APPEND HEADERS ${SFMT_H} ${TNYOBJ_H})

list(APPEND SRCS ${SFMT_CPP} ${TNYOBJ_CPP})

add_definitions(-DSFMT_MEXP=19937)

####

include_directories(	
	${terrain_INCLUDE_DIRS}
	)

link_directories(
	${CMAKE_SOURCE_DIR}/externals/libs
	) 

set(SHADER_FILES
  ${CMAKE_SOURCE_DIR}/shaders/mainshader.frag
  ${CMAKE_SOURCE_DIR}/shaders/mainshader.vert
  ${CMAKE_SOURCE_DIR}/shaders/tessellation.vert
  ${CMAKE_SOURCE_DIR}/shaders/tessellation.tcs
  ${CMAKE_SOURCE_DIR}/shaders/tessellation.tes
  ${CMAKE_SOURCE_DIR}/shaders/tessellation.frag
)

source_group("The Shaders" FILES ${SHADER_FILES})

file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/export)

if(CMAKE_CURRENT_BINARY_DIR)
	add_definitions(-DCMAKE_BINARY_DIR=\"${CMAKE_CURRENT_BINARY_DIR}\")
else()
	add_definitions(-DCMAKE_BINARY_DIR=\"\")
endif()
get_filename_component(PARENT_DIR ${CMAKE_SOURCE_DIR} PATH)
add_definitions(-DCMAKE_SOURCE_DIR=\"${PARENT_DIR}\")

#add_executable(schumacher ${HEADERS} ${SRCS})
add_executable(terrain ${HEADERS} ${SRCS} ${SHADER_FILES})

if(WIN32)
	target_link_libraries(terrain GLFW.lib FreeImage.lib freeglut.lib assimp.lib opengl32.lib glew32.lib)
endif(WIN32)

if(UNIX)
	target_link_libraries(terrain freeimage assimp ${EXTRA_LIBS})
endif(UNIX)
