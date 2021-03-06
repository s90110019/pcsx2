# Check that people use the good file
if(NOT TOP_CMAKE_WAS_SOURCED)
    message(FATAL_ERROR "
    You did not 'cmake' the good CMakeLists.txt file. Use the one in the top dir.
    It is advice to delete all wrongly generated cmake stuff => CMakeFiles & CMakeCache.txt")
endif(NOT TOP_CMAKE_WAS_SOURCED)


# plugin name
set(Output spu2x-2.0.0)

set(CommonFlags
    -fvisibility=hidden
    -Wall
    )

set(OptimizationFlags
    -O2
    -DNDEBUG
    )

# Debug - Build
if(CMAKE_BUILD_TYPE STREQUAL Debug)
	# add defines
    add_definitions(${CommonFlags} -g)
endif(CMAKE_BUILD_TYPE STREQUAL Debug)

# Devel - Build
if(CMAKE_BUILD_TYPE STREQUAL Devel)
	# add defines
	add_definitions(${CommonFlags} ${OptimizationFlags})
endif(CMAKE_BUILD_TYPE STREQUAL Devel)

# Release - Build
if(CMAKE_BUILD_TYPE STREQUAL Release)
	# add defines
	add_definitions(${CommonFlags} ${OptimizationFlags})
endif(CMAKE_BUILD_TYPE STREQUAL Release)

# spu2x sources
set(spu2xSources
    ADSR.cpp
    Debug.cpp
    DplIIdecoder.cpp
    Dma.cpp
    Lowpass.cpp
    Mixer.cpp
    PrecompiledHeader.cpp
    PS2E-spu2.cpp
    ReadInput.cpp
    RegLog.cpp
    RegTable.cpp
    Reverb.cpp
    SndOut.cpp
    SndOut_Portaudio.cpp
    spu2freeze.cpp
    Spu2replay.cpp
    spu2sys.cpp
    Timestretcher.cpp
    Wavedump_wav.cpp
    WavFile.cpp
    )

# spu2x headers
set(spu2xHeaders
    Config.h
    Debug.h
    defs.h
    Dma.h
    DPLII.h
    Global.h
    Lowpass.h
    Mixer.h
    PS2E-spu2.h
    regs.h
    SndOut.h
    spdif.h
    Spu2replay.h
    WavFile.h
)


# spu2x Linux sources
set(spu2xLinuxSources
    Linux/AboutBox.cpp
    Linux/Alsa.cpp
    Linux/CfgHelpers.cpp
    Linux/Config.cpp
    Linux/ConfigDebug.cpp
    Linux/ConfigSoundTouch.cpp
    Linux/Dialogs.cpp)

# spu2x Linux headers
set(spu2xLinuxHeaders
    Linux/Alsa.h
    Linux/Config.h
    Linux/Dialogs.h)

# add additional include directories
include_directories(.  Linux)

### Don't use yet c11 feature. I will bump gcc requirement to 4.7 when 4.9 is out.
### Note: actually it might work on 4.6 too
# # If compiling on GCC Version 4.7 or higher we build SDLMod in C++11 mode
# if("${CMAKE_CXX_COMPILER_ID}" MATCHES "GNU")
#     execute_process(COMMAND ${CMAKE_C_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)
#     if(GCC_VERSION VERSION_GREATER 4.7)
#         set_source_files_properties(SndOut_SDL.cpp PROPERTIES COMPILE_FLAGS -std=c++11)
#         message(STATUS "GCC has C++11 support for SDLAudioMod")
#     endif(GCC_VERSION VERSION_GREATER 4.7)
# endif()

if(SDL_FOUND)
    list(APPEND spu2xSources SndOut_SDL.cpp)
    if(SDL2_API)
        add_definitions(-DSPU2X_SDL2)
    else()
        add_definitions(-DSPU2X_SDL)
    endif()
  #elseif(SDL2_FOUND)
  #  add_definitions(-DSPU2X_SDL2)
endif()

# add library
add_library(${Output} SHARED
			${spu2xSources}
			${spu2xHeaders}
			${spu2xLinuxSources}
			${spu2xLinuxHeaders})

# link target with project internal libraries
target_link_libraries(${Output} Utilities)

# link target with various backend
target_link_libraries(${Output} ${ALSA_LIBRARIES})
target_link_libraries(${Output} ${PORTAUDIO_LIBRARIES})
if (SDL2_API)
    target_link_libraries(${Output} ${SDL2_LIBRARY})
else()
    target_link_libraries(${Output} ${SDL_LIBRARY})
endif()

# link target with SoundTouch
target_link_libraries(${Output} ${SOUNDTOUCH_LIBRARIES})

if(Linux)
	# link target with gtk2
	target_link_libraries(${Output} ${GTK2_LIBRARIES})
endif(Linux)

# User flags options
if(NOT USER_CMAKE_LD_FLAGS STREQUAL "")
    target_link_libraries(${Output} "${USER_CMAKE_LD_FLAGS}")
endif(NOT USER_CMAKE_LD_FLAGS STREQUAL "")

if(PACKAGE_MODE)
    install(TARGETS ${Output} DESTINATION ${PLUGIN_DIR})
else(PACKAGE_MODE)
    install(TARGETS ${Output} DESTINATION ${CMAKE_SOURCE_DIR}/bin/plugins)
endif(PACKAGE_MODE)
