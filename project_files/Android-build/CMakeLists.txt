###################################################################################
# Hedgewars, a free turn based strategy game
# Copyright (c) 2011 Richard Deurwaarder <xeli@xelification.com>
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA 02111-1307, USA
###################################################################################

find_program(ANT ant)

if(NOT ANDROID_NDK)
    find_path(ANDROID_NDK ndk-build)
endif()

if(IS_DIRECTORY "${ANDROID_NDK}")
    MESSAGE(STATUS "Detected the android NDK directory at: " ${ANDROID_NDK}) 
else ()
    MESSAGE(FATAL_ERROR "Couldn't detect the Android NDK directory")
endif()

if(NOT ANDROID_SDK)#Check if its defined at the cmdline
    find_path(ANDROID_SDK adb ) #assume they've added platform-tools to their path
    get_filename_component(ANDROID_SDK "${ANDROID_SDK}" PATH)
    set(ANDROID_SDK "${ANDROID_SDK}" CACHE PATH "Path to the android sdk" FORCE)
endif()

if( IS_DIRECTORY "${ANDROID_SDK}")
    MESSAGE(STATUS "Detected the android SDK directory at: " ${ANDROID_SDK}) 
else ()
    MESSAGE(FATAL_ERROR "Couldn't detect the Android SDK directory")
endif()

if( NOT FPC_DIR)
    find_path(FPC_DIR ppcrossarm)
    get_filename_component(FPC_DIR "${FPC_DIR}" PATH)
    set(FPC_DIR "${FPC_DIR}" CACHE PATH "Path to fpc dir used in the android port" FORCE)
endif()

if( IS_DIRECTORY "${FPC_DIR}")
    MESSAGE(STATUS "Detected the FreePascal directory at: " "${FPC_DIR}") 
else ()
    MESSAGE(FATAL_ERROR "Couldn't detect the FreePascal directory")
endif()

set(SDL_DIR /home/richard/Downloads/android-project)


set(ANDROID_SDK_API_LVL 8)
set(ANDROID_NDK_API_LVL 5)

MESSAGE(STATUS "Creating android scripts and configuration files")

configure_file(Templates/complete_build.sh .)
configure_file(Templates/default.properties SDL-android-project/)
configure_file(Templates/local.properties SDL-android-project/)
configure_file(Templates/Makefile.android .)
configure_file(Templates/pushToDevice.sh .)

if(SDL_DIR_TEMP_DISABLE)
    MESSAGE("Moving Android port of SDL to the proper directories")
    set(DirsToCopy
        SDL
        SDL_image
        SDL_mixer
        SDL_ttf
	jpeg
	png
         mikmod
        ) 
    foreach(DIR ${DirsToCopy})
        exec_program( 
            ${CMAKE_COMMAND} 
            ARGS -E copy_directory 
            ${SDL_DIR}/jni/${DIR}
            ${CMAKE_CURRENT_SOURCE_DIR}/SDL-android-project/jni/${DIR}
        )
        MESSAGE(STATUS "Moved ${DIR}")
    endforeach(DIR)
    exec_program(${HGCOMMAND}
        ARGS revert ${CMAKE_CURRENT_SOURCE_DIR}/SDL-android-project/jni/SDL/src/core/android/SDL_android.cpp
	)
    exec_program(${HGCOMMAND}
        ARGS revert ${CMAKE_CURRENT_SOURCE_DIR}/SDL-android-project/jni/SDL/src/main/android/SDL_android_main.cpp
	)
    exec_program(${HGCOMMAND}
        ARGS revert ${CMAKE_CURRENT_SOURCE_DIR}/SDL-android-project/jni/SDL/src/video/android/SDL_androidtouch.c
        )

    exec_program(${HGCOMMAND}
        ARGS revert ${CMAKE_CURRENT_SOURCE_DIR}/SDL-android-project/jni/SDL/src/video/android/SDL_androidtouch.h
	)
endif(SDL_DIR_TEMP_DISABLE)