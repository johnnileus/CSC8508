cmake_minimum_required(VERSION 3.16.0 FATAL_ERROR)

set(CMAKE_SYSTEM_VERSION 10.0 CACHE STRING "" FORCE)

project(CSC8503 CXX)

################################################################################
# Set target arch type if empty. Visual studio solution generator provides it.
################################################################################
if(NOT CMAKE_VS_PLATFORM_NAME)
    set(CMAKE_VS_PLATFORM_NAME "x64")
endif()
message("${CMAKE_VS_PLATFORM_NAME} architecture in use")

if(NOT ("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x64"))
    message(FATAL_ERROR "${CMAKE_VS_PLATFORM_NAME} arch is not supported!")
endif()

################################################################################
# Global configuration types
################################################################################
set(CMAKE_CONFIGURATION_TYPES
    "Debug"
    "Release"
    CACHE STRING "" FORCE
)

################################################################################
# Global compiler options
################################################################################
set(CMAKE_CXX_STANDARD 20)
#set(CMAKE_CXX_STANDARD 23)

################################################################################
# Nuget packages function stub.
################################################################################
function(use_package TARGET PACKAGE VERSION)
    message(WARNING "No implementation of use_package. Create yours. "
                    "Package \"${PACKAGE}\" with version \"${VERSION}\" "
                    "for target \"${TARGET}\" is ignored!")
endfunction()

################################################################################
# Common utils
################################################################################
include(CMake/Utils.cmake)

################################################################################
# Additional Global Settings(add specific info there)
################################################################################
include(CMake/GlobalSettingsInclude.cmake OPTIONAL)

################################################################################
# Use solution folders feature
################################################################################
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
if(USE_VULKAN)
	find_package(Vulkan)
endif() 
set(ASSET_ROOT "${CMAKE_SOURCE_DIR}/Assets/" CACHE STRING "" FORCE)
add_compile_definitions(ASSETROOTLOCATION="${ASSET_ROOT}") 

set(USE_VULKAN CACHE BOOL FORCE)
if(USE_VULKAN)
    add_compile_definitions("USEVULKAN") 
	add_compile_definitions("VK_USE_PLATFORM_WIN32_KHR") 
endif() 

if(MSVC)
    add_compile_definitions("NOMINMAX")
    add_compile_definitions("WIN32_LEAN_AND_MEAN")  
endif()

#################################################################################
#Physics
#################################################################################
include_directories("${CMAKE_SOURCE_DIR}/ThirdParty/reactphysics/include")

file(GLOB_RECURSE RP3D_SRC "${CMAKE_SOURCE_DIR}/ThirdParty/reactphysics/src/*.cpp")

add_library(ReactPhysics3D STATIC ${RP3D_SRC})

#IMGUI 
include_directories("${CMAKE_SOURCE_DIR}/ThirdParty/ImGui/include")

file(GLOB_RECURSE IMGUI_SRC "${CMAKE_SOURCE_DIR}/ThirdParty/ImGui/src/*.cpp")

add_library(ImGui STATIC ${IMGUI_SRC})

################################################################################
# Sub-projects
################################################################################
add_subdirectory(NCLCoreClasses)
add_subdirectory(CSC8503CoreClasses)
add_subdirectory(OpenGLRendering)
add_subdirectory(CSC8503)
if(USE_VULKAN)
    add_subdirectory(VulkanRendering)
endif()

target_link_libraries(CSC8503 PRIVATE ReactPhysics3D)
target_link_libraries(CSC8503 PRIVATE ImGui)
target_link_libraries(CSC8503CoreClasses PRIVATE ReactPhysics3D)
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT CSC8503)








