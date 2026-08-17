cmake_minimum_required(VERSION 3.15)
project(imgui LANGUAGES CXX)

# Options
option(BUILD_SHARED_LIBS "Build shared libraries instead of static" OFF)

# Backend Selection
option(IMGUI_BACKEND_GLFW "Enable GLFW backend" OFF)
option(IMGUI_BACKEND_SDL2 "Enable SDL2 backend" OFF)
option(IMGUI_BACKEND_SDL3 "Enable SDL3 backend" OFF)
option(IMGUI_BACKEND_OPENGL3 "Enable OpenGL3 backend" OFF)
option(IMGUI_BACKEND_VULKAN "Enable Vulkan backend" OFF)
option(IMGUI_BACKEND_DX11 "Enable DirectX 11 backend" OFF)
option(IMGUI_BACKEND_DX12 "Enable DirectX 12 backend" OFF)

# Core ImGui sources
set(IMGUI_SOURCES
    imgui.cpp
    imgui_demo.cpp
    imgui_draw.cpp
    imgui_tables.cpp
    imgui_widgets.cpp
)

set(IMGUI_HEADERS
    imconfig.h
    imgui.h
    imgui_internal.h
    imstb_rectpack.h
    imstb_textedit.h
    imstb_truetype.h
)

# Target Definition
add_library(imgui ${IMGUI_SOURCES} ${IMGUI_HEADERS})
add_library(imgui::imgui ALIAS imgui)

# Public Includes
target_include_directories(imgui
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/backends>
)

# C++ Standard
target_compile_features(imgui PUBLIC cxx_std_17)

# Handle Shared Library Export Rules (Windows/DLL)
if(BUILD_SHARED_LIBS)
    target_compile_definitions(imgui 
        PUBLIC IMGUI_API=__declspec(dllexport)
        INTERFACE IMGUI_API=__declspec(dllimport)
    )
endif()

# Window / Input Backends
if(IMGUI_BACKEND_GLFW)
    if(NOT TARGET glfw AND NOT TARGET glfw3 AND NOT TARGET glfw::glfw)
        find_package(glfw3 REQUIRED)
    endif()
    
    target_sources(imgui PRIVATE 
        backends/imgui_impl_glfw.cpp 
        backends/imgui_impl_glfw.h
    )

    if(TARGET glfw::glfw)
        target_link_libraries(imgui PUBLIC glfw::glfw)
    elseif(TARGET glfw)
        target_link_libraries(imgui PUBLIC glfw)
    elseif(TARGET glfw3)
        target_link_libraries(imgui PUBLIC glfw3)
    endif()
endif()

if(IMGUI_BACKEND_SDL2)
    if(NOT TARGET SDL2::SDL2 AND NOT TARGET SDL2)
        find_package(SDL2 REQUIRED)
    endif()

    target_sources(imgui PRIVATE 
        backends/imgui_impl_sdl2.cpp 
        backends/imgui_impl_sdl2.h
    )

    if(TARGET SDL2::SDL2)
        target_link_libraries(imgui PUBLIC SDL2::SDL2)
    elseif(TARGET SDL2)
        target_link_libraries(imgui PUBLIC SDL2)
    endif()
endif()

if(IMGUI_BACKEND_SDL3)
    if(NOT TARGET SDL3::SDL3 AND NOT TARGET SDL3)
        find_package(SDL3 REQUIRED)
    endif()

    target_sources(imgui PRIVATE 
        backends/imgui_impl_sdl3.cpp 
        backends/imgui_impl_sdl3.h
    )

    if(TARGET SDL3::SDL3)
        target_link_libraries(imgui PUBLIC SDL3::SDL3)
    elseif(TARGET SDL3)
        target_link_libraries(imgui PUBLIC SDL3)
    endif()
endif()

# Render Backends
if(IMGUI_BACKEND_OPENGL3)
    if(NOT TARGET OpenGL::GL)
        find_package(OpenGL REQUIRED)
    endif()

    target_sources(imgui PRIVATE 
        backends/imgui_impl_opengl3.cpp 
        backends/imgui_impl_opengl3.h
    )
    target_link_libraries(imgui PUBLIC OpenGL::GL)
endif()

if(IMGUI_BACKEND_VULKAN)
    if(NOT TARGET Vulkan::Vulkan)
        find_package(Vulkan REQUIRED)
    endif()

    target_sources(imgui PRIVATE 
        backends/imgui_impl_vulkan.cpp 
        backends/imgui_impl_vulkan.h
    )
    target_link_libraries(imgui PUBLIC Vulkan::Vulkan)
endif()

if(IMGUI_BACKEND_DX11)
    target_sources(imgui PRIVATE 
        backends/imgui_impl_dx11.cpp 
        backends/imgui_impl_dx11.h
    )
    if(WIN32)
        target_link_libraries(imgui PUBLIC d3d11 d3dcompiler dxgi)
    endif()
endif()

if(IMGUI_BACKEND_DX12)
    target_sources(imgui PRIVATE 
        backends/imgui_impl_dx12.cpp 
        backends/imgui_impl_dx12.h
    )
    if(WIN32)
        target_link_libraries(imgui PUBLIC d3d12 d3dcompiler dxgi)
    endif()
endif()