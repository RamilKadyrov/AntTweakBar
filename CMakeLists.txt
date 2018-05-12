#TODO Linux and OSX versions

cmake_minimum_required(VERSION 3.5)
project(AntTweakBar VERSION 1.16)

##############################################
# Declare dependencies

find_package(OpenGL 4.1 REQUIRED)

##############################################
# Create target and set properties

set( CPP_FILES
src/LoadOGL.cpp
src/LoadOGLCore.cpp
src/TwBar.cpp
src/TwColors.cpp
src/TwDirect3D10.cpp
src/TwDirect3D11.cpp
src/TwDirect3D9.cpp
src/TwEventSFML.cpp
src/TwFonts.cpp
src/TwMgr.cpp
src/TwOpenGL.cpp
src/TwOpenGLCore.cpp
src/TwPrecomp.cpp)

set( C_FILES
    src/TwEventGLFW.c
    src/TwEventGLUT.c    
    src/TwEventSDL.c
    src/TwEventSDL12.c
    src/TwEventSDL13.c
    src/TwEventWin.c)

set ( HEADER_FILES
    include/AntTweakBar.h
    src/AntPerfTimer.h
    src/LoadOGL.h
    src/LoadOGLCore.h
    src/MiniGLFW.h
    src/MiniGLUT.h
    src/MiniSDL12.h
    src/MiniSDL13.h
    src/MiniSFML16.h
    src/resource.h
    src/TwBar.h
    src/TwColors.h
    src/TwDirect3D10.h
    src/TwDirect3D11.h
    src/TwDirect3D9.h
    src/TwFonts.h
    src/TwGraph.h
    src/TwMgr.h
    src/TwOpenGL.h
    src/TwOpenGLCore.h
    src/TwPrecomp.h
    src/res/TwXCursors.h
)
set ( RESOURCE_FILES
    src/AntTweakBar.rc
    src/res/FontChars.txt
    src/res/cur00000.cur
    src/res/cur00001.cur
    src/res/cur00002.cur
    src/res/cur00003.cur
    src/res/cur00004.cur
    src/res/cur00005.cur
    src/res/cur00006.cur
    src/res/cur00007.cur
    src/res/cur00008.cur
    src/res/cur00009.cur
    src/res/cur00010.cur
    src/res/cur00011.cur
    src/res/cur00012.cur
    src/res/cur00013.cur
    src/res/FontFixed1.pgm
    src/res/FontLargeAA.pgm
    src/res/FontNormal.pgm
    src/res/FontSmall.pgm
)

set( SHADER_FILE
    src/TwDirect3D11.hlsl
)


add_library(${PROJECT_NAME} SHARED ${CPP_FILES} ${C_FILES} ${HEADER_FILES})

#Add an alias so that library can be used inside the build tree, e.g. when testing
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

if(MSVC)
if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
      set(H_PATH
        ${CMAKE_CFG_INTDIR}64
      )
    else()
      set(H_PATH
        ${CMAKE_CFG_INTDIR}32
      )
    endif()
    add_custom_command(TARGET ${PROJECT_NAME}
                 PRE_BUILD
                 COMMENT "Shader compilation"
                 COMMAND if not exist ${H_PATH} mkdir ${H_PATH} 
                 COMMAND fxc /T vs_4_0_level_9_1 /E LineRectVS /Fh ${H_PATH}/TwDirect3D11_LineRectVS.h ${CMAKE_CURRENT_SOURCE_DIR}/src/TwDirect3D11.hlsl
		 COMMAND fxc /T vs_4_0_level_9_1 /E LineRectCstColorVS /Fh ${H_PATH}/TwDirect3D11_LineRectCstColorVS.h ${CMAKE_CURRENT_SOURCE_DIR}/src/TwDirect3D11.hlsl
                 COMMAND fxc /T ps_4_0_level_9_1 /E LineRectPS /Fh ${H_PATH}/TwDirect3D11_LineRectPS.h ${CMAKE_CURRENT_SOURCE_DIR}/src/TwDirect3D11.hlsl
                 COMMAND fxc /T vs_4_0_level_9_1 /E TextVS /Fh ${H_PATH}/TwDirect3D11_TextVS.h ${CMAKE_CURRENT_SOURCE_DIR}/src/TwDirect3D11.hlsl
                 COMMAND fxc /T vs_4_0_level_9_1 /E TextCstColorVS /Fh ${H_PATH}/TwDirect3D11_TextCstColorVS.h ${CMAKE_CURRENT_SOURCE_DIR}/src/TwDirect3D11.hlsl
                 COMMAND fxc /T ps_4_0_level_9_1 /E TextPS /Fh ${H_PATH}/TwDirect3D11_TextPS.h ${CMAKE_CURRENT_SOURCE_DIR}/src/TwDirect3D11.hlsl
    )
endif() # MSVC

#Set target properties
add_definitions(-DTW_EXPORTS)

target_include_directories(${PROJECT_NAME}
    PUBLIC 
        $<INSTALL_INTERFACE:include>    
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
	${CMAKE_CURRENT_SOURCE_DIR}/src/res
        ${CMAKE_CURRENT_BINARY_DIR}
)

target_link_libraries(${PROJECT_NAME}
        ${OPENGL_LIBRARY}
    )

target_compile_features(${PROJECT_NAME} PRIVATE cxx_auto_type)
target_compile_options(${PROJECT_NAME} PRIVATE -Wall)


##############################################
# Installation instructions

include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

#Export the targets to a script
install(EXPORT ${PROJECT_NAME}-targets
  FILE
    ${PROJECT_NAME}Targets.cmake
  NAMESPACE
    ${PROJECT_NAME}::
  DESTINATION
    ${INSTALL_CONFIGDIR}
)

#Create a ConfigVersion.cmake file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

#Install the config, configversion and custom find modules
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
)

##############################################
## Exporting from the build tree


export(EXPORT ${PROJECT_NAME}-targets FILE ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake NAMESPACE ${PROJECT_NAME}::)

#Register package in user's package registry
export(PACKAGE ${PROJECT_NAME})
