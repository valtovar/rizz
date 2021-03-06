cmake_minimum_required(VERSION 3.1)
project(rizz)

set(SOURCE_FILES "app.c"
                 "core.c" 
                 "plugin.cpp"
                 "graphics.c"
                 "reflect.c"
                 "asset.c" 
                 "camera.c" 
                 "http.c" 
                 "vfs.c")

set(INCLUDE_FILES "../../include/rizz/types.h"
                  "../../include/rizz/app.h" 
                  "../../include/rizz/entry.h" 
                  "../../include/rizz/core.h" 
                  "../../include/rizz/plugin.h" 
                  "../../include/rizz/config.h" 
                  "../../include/rizz/graphics.h"
                  "../../include/rizz/_sg-types.h"
                  "../../include/rizz/asset.h"
                  "../../include/rizz/reflect.h"
                  "../../include/rizz/camera.h"
                  "../../include/rizz/http.h"
                  "../../include/rizz/util.h"
                  "../../include/rizz/vfs.h")

set(EXTERNAL_INCLUDES ../../3rdparty/sokol/sokol_app.h 
                      ../../3rdparty/sokol/sokol_gfx.h
                      ../../3rdparty/sokol/sokol_audio.h
                      ../../3rdparty/cr/cr.h
                      ../../include/sjson/sjson.h 
                      ../../include/dds-ktx/dds-ktx.h
                      ../../3rdparty/stb/stb_image.h
                      ../../3rdparty/stb/stb_image_resize.h
                      ../../3rdparty/sort/sort.h
                      ../../3rdparty/mattias/http.h)

if (APPLE)
    set_source_files_properties(app.c graphics.c PROPERTIES COMPILE_FLAGS "-fobjc-arc -fmodules -x objective-c") 
endif()

# bundle: make static lib to link it to the game.exe later
# no-bundle: make rizz.exe and run the game.dll as a plugin
if (BUNDLE)
    add_library(rizz STATIC ${SOURCE_FILES} ${EXTERNAL_INCLUDES} ${INCLUDE_FILES})
else()
    add_executable(rizz ${EXEC_TYPE} ${SOURCE_FILES} ${EXTERNAL_INCLUDES} ${INCLUDE_FILES})
endif()

target_include_directories(rizz PRIVATE ../../3rdparty)
target_link_libraries(rizz PUBLIC sx PRIVATE remotery utf)
target_compile_definitions(rizz PRIVATE -DRIZZ_INTERNAL_API) # Expose internal API structs when build the library
if (ENABLE_HOT_LOADING)
    target_compile_definitions(rizz PRIVATE -DRIZZ_CONFIG_HOT_LOADING=1)
    target_link_libraries(rizz PRIVATE efsw)
else()
    target_compile_definitions(rizz PRIVATE -DRIZZ_CONFIG_HOT_LOADING=0)
endif()

# Link system libs
if (ANDROID)
    target_link_libraries(rizz PRIVATE EGL android)
elseif (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    target_link_libraries(rizz PRIVATE GL GLEW PUBLIC X11)
elseif (WIN32)
    target_link_libraries(rizz PRIVATE dxgi d3d11)
endif()

bundle_plugins_internal(rizz imgui)

# shaders
set(shaders shaders/debug.vert 
            shaders/debug.frag)
set_source_files_properties(${shaders} PROPERTIES GLSLCC_OUTPUT_DIRECTORY "shaders_h")
glslcc_target_compile_shaders_h(rizz "${shaders}")
