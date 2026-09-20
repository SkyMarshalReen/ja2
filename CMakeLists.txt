cmake_minimum_required(VERSION 3.25)

project(JaggedAlliance2 LANGUAGES C CXX)

if(NOT WIN32)
  message(FATAL_ERROR "This source release currently supports Windows only.")
endif()

if(NOT MSVC)
  message(FATAL_ERROR
    "The bundled Miles Sound System and Smacker libraries require a 32-bit "
    "MSVC-compatible build. Configure with a Visual Studio Win32 preset.")
endif()

if(NOT CMAKE_SIZEOF_VOID_P EQUAL 4)
  message(FATAL_ERROR
    "JA2 must be configured for Win32. The bundled mss32.lib and smackw32.lib "
    "are 32-bit libraries; use --preset vs2022-debug or vs2022-release.")
endif()

set(CMAKE_C_STANDARD 90)
set(CMAKE_C_STANDARD_REQUIRED OFF)
set(CMAKE_CXX_STANDARD 98)
set(CMAKE_CXX_STANDARD_REQUIRED OFF)
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

set(JA2_BUILD_DIR "${CMAKE_CURRENT_SOURCE_DIR}/ja2/Build")
set(JA2_SGP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/Standard Gaming Platform")

set(JA2_MODULE_DIRS
  "${JA2_BUILD_DIR}/Editor"
  "${JA2_BUILD_DIR}/Laptop"
  "${JA2_BUILD_DIR}/Strategic"
  "${JA2_BUILD_DIR}/Tactical"
  "${JA2_BUILD_DIR}/TacticalAI"
  "${JA2_BUILD_DIR}/TileEngine"
  "${JA2_BUILD_DIR}/Utils"
)

# CONFIGURE_DEPENDS keeps the historical source tree convenient to modify while
# still causing CMake to regenerate when a source file is added or removed.
file(GLOB JA2_ROOT_SOURCES CONFIGURE_DEPENDS
  "${JA2_BUILD_DIR}/*.c"
  "${JA2_BUILD_DIR}/*.cpp"
)
file(GLOB SGP_SOURCES CONFIGURE_DEPENDS
  "${JA2_SGP_DIR}/*.c"
  "${JA2_SGP_DIR}/*.cpp"
)

set(JA2_MODULE_SOURCES)
foreach(module_dir IN LISTS JA2_MODULE_DIRS)
  file(GLOB module_sources CONFIGURE_DEPENDS
    "${module_dir}/*.c"
    "${module_dir}/*.cpp"
  )
  list(APPEND JA2_MODULE_SOURCES ${module_sources})
endforeach()

# These files were present in the source drop but were not members of the VC6
# workspace. Keep that distinction until each dormant implementation has been
# reviewed and deliberately enabled.
list(REMOVE_ITEM SGP_SOURCES
  "${JA2_SGP_DIR}/Compression.c"
  "${JA2_SGP_DIR}/Flic.c"
  "${JA2_SGP_DIR}/Ja2 Libs.c"
)
list(REMOVE_ITEM JA2_MODULE_SOURCES
  "${JA2_BUILD_DIR}/Laptop/GunEmporium.c"
  "${JA2_BUILD_DIR}/TileEngine/Render Z.c"
  "${JA2_BUILD_DIR}/Utils/_TaiwaneseText.c"
  "${JA2_BUILD_DIR}/Utils/Win Util.c"
)

add_executable(ja2 WIN32
  ${JA2_ROOT_SOURCES}
  ${JA2_MODULE_SOURCES}
  ${SGP_SOURCES}
)

target_include_directories(ja2 PRIVATE
  "${JA2_BUILD_DIR}"
  "${JA2_BUILD_DIR}/Editor"
  "${JA2_BUILD_DIR}/Laptop"
  "${JA2_BUILD_DIR}/Strategic"
  "${JA2_BUILD_DIR}/Tactical"
  "${JA2_BUILD_DIR}/TacticalAI"
  "${JA2_BUILD_DIR}/TileEngine"
  "${JA2_BUILD_DIR}/Utils"
  "${JA2_SGP_DIR}"
)

target_compile_definitions(ja2 PRIVATE
  WIN32
  _WINDOWS
  JA2
  CALLBACKTIMER
  NO_ZLIB
  NO_ZLIB_COMPRESSION
  _CRT_SECURE_NO_WARNINGS
  _CRT_NONSTDC_NO_WARNINGS
  $<$<CONFIG:Debug>:_DEBUG>
  $<$<NOT:$<CONFIG:Debug>>:NDEBUG>
)

# The code predates standard-conforming wchar_t and for-loop scoping. These
# switches retain the VC6 ABI and language behavior needed by serialized data
# and the bundled binary libraries.
target_compile_options(ja2 PRIVATE
  /W3
  /Zc:wchar_t-
  /Zc:forScope-
  /wd4013
  /wd4244
  /wd4996
)

set(JA2_MSS_LIBRARY "${JA2_SGP_DIR}/mss32.lib")
set(JA2_SMACK_LIBRARY "${JA2_SGP_DIR}/SMACKW32.LIB")

foreach(required_library IN ITEMS JA2_MSS_LIBRARY JA2_SMACK_LIBRARY)
  if(NOT EXISTS "${${required_library}}")
    message(FATAL_ERROR "Required legacy library not found: ${${required_library}}")
  endif()
endforeach()

target_link_libraries(ja2 PRIVATE
  "${JA2_MSS_LIBRARY}"
  "${JA2_SMACK_LIBRARY}"
  ddraw
  dxguid
  dsound
  winmm
  kernel32
  user32
  gdi32
  winspool
  comdlg32
  advapi32
  shell32
  ole32
  oleaut32
  uuid
)

set_target_properties(ja2 PROPERTIES
  OUTPUT_NAME "ja2"
  VS_DEBUGGER_WORKING_DIRECTORY "$<TARGET_FILE_DIR:ja2>"
)

source_group(TREE "${JA2_BUILD_DIR}" PREFIX "JA2" FILES
  ${JA2_ROOT_SOURCES}
  ${JA2_MODULE_SOURCES}
)
source_group(TREE "${JA2_SGP_DIR}" PREFIX "Standard Gaming Platform" FILES
  ${SGP_SOURCES}
)

install(TARGETS ja2 RUNTIME DESTINATION .)
