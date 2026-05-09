
# Add source to this project's executable.
add_executable (engine 
  main.cpp
  vk_types.h
  vk_initializers.cpp
  vk_initializers.h
  vk_images.h
  vk_images.cpp 
  vk_descriptors.h
  vk_descriptors.cpp
  vk_pipelines.h
  vk_pipelines.cpp
  vk_engine.h
  vk_engine.cpp
  vk_loader.h
  vk_loader.cpp
  camera.cpp
  camera.h
)

target_compile_definitions(engine PUBLIC GLM_FORCE_DEPTH_ZERO_TO_ONE)
target_include_directories(engine PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}")
# SDL's CMake target may expose build-tree include dirs; clang-tidy/clangd
# work best when the source include dir is also available.
target_include_directories(engine PUBLIC "${PROJECT_SOURCE_DIR}/third_party/SDL/include")

set_property(TARGET engine PROPERTY CXX_STANDARD 23)
set_property(TARGET engine PROPERTY CXX_STANDARD_REQUIRED ON)
set_property(TARGET engine PROPERTY CXX_EXTENSIONS OFF)

option(VKGUIDE_ENABLE_CLANG_TIDY "Run clang-tidy for engine (slow)" OFF)
if(VKGUIDE_ENABLE_CLANG_TIDY)
  find_program(CLANG_TIDY_EXE NAMES clang-tidy clang-tidy.exe)
  if(CLANG_TIDY_EXE)
    set_property(TARGET engine PROPERTY CXX_CLANG_TIDY "${CLANG_TIDY_EXE}")
  else()
    message(WARNING "VKGUIDE_ENABLE_CLANG_TIDY=ON but clang-tidy not found in PATH")
  endif()
endif()

if(MSVC)
  target_compile_options(engine PRIVATE /W4 /WX /permissive- /Zc:__cplusplus)
else()
  target_compile_options(engine PRIVATE -Wall -Wextra -Wpedantic)
endif()

target_link_libraries(engine PUBLIC vma glm Vulkan::Vulkan stb_image SDL2::SDL2 vkbootstrap imgui fastgltf::fastgltf)

option(VKGUIDE_ENABLE_PCH "Use precompiled headers for engine" ON)
if(VKGUIDE_ENABLE_PCH)
  target_precompile_headers(engine PUBLIC <optional> <vector> <memory> <string> <vector> <unordered_map> <glm/mat4x4.hpp>  <glm/vec4.hpp> <vulkan/vulkan.h>)
endif()

if(WIN32)
  add_custom_command(TARGET engine POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_RUNTIME_DLLS:engine> $<TARGET_FILE_DIR:engine>
    COMMAND_EXPAND_LISTS
    )
endif()