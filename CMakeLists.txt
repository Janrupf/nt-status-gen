#################
# Project setup #
#################
cmake_minimum_required(VERSION 3.10)
project(nt-status-gen)

#####################
# Set CMake options #
#####################
set(CMAKE_CXX_STANDARD 17)

###########
# Targets #
###########

# All source nt-status-gen files
set(NT_STATUS_GEN_BUILD_TOOL_SOURCE
        build-src/main.cpp)

# Add executable target
add_executable(nt-status-gen-build-tool ${NT_STATUS_GEN_BUILD_TOOL_SOURCE})


# Dummy input file to the preprocessor
set(NT_STATUS_GEN_PREPROCESSOR_INPUT
        build-src/ntstatus.preprocess.h)

# Preprocessed output file, this will be read by nt-status-gen
set(NT_STATUS_GEN_PREPROCESSOR_OUT
        ${CMAKE_CURRENT_BINARY_DIR}/ntstatus.preprocess.h.target)

get_filename_component(NT_STATUS_GEN_PREPROCESSOR_INPUT_FULL
        ${NT_STATUS_GEN_PREPROCESSOR_INPUT}
        REALPATH BASE_DIR ${CMAKE_CURRENT_LIST_DIR}
        )

# Add a target for preprocessing
add_custom_target(nt-status-gen-preprocess-ntstatus)

if(MSVC)
    add_custom_command(
            COMMAND ${CMAKE_C_COMPILER} /P /Fi${NT_STATUS_GEN_PREPROCESSOR_OUT} ${NT_STATUS_GEN_PREPROCESSOR_INPUT_FULL}
            DEPENDS ${NT_STATUS_GEN_PREPROCESSOR_INPUT}
            COMMENT "Generating preprocessed ntstatus.h"
            TARGET "nt-status-gen-preprocess-ntstatus"
    )
else()
    # If not MSVC, assume GCC/Clang compatible compiler
    add_custom_command(
            COMMAND ${CMAKE_C_COMPILER} -MF ${NT_STATUS_GEN_PREPROCESSOR_OUT} -M ${NT_STATUS_GEN_PREPROCESSOR_INPUT_FULL}
            DEPENDS ${NT_STATUS_GEN_PREPROCESSOR_INPUT}
            COMMENT "Generating preprocessed ntstatus.h"
            TARGET "nt-status-gen-preprocess-ntstatus"
    )
endif()

# Make building the generator depend on preprocessing ntstatus.h
add_dependencies(nt-status-gen-build-tool nt-status-gen-preprocess-ntstatus)
target_compile_definitions(nt-status-gen-build-tool PRIVATE
        NT_STATUS_GEN_PREPROCESSOR_OUT=\"${NT_STATUS_GEN_PREPROCESSOR_OUT}\"
        NT_STATUS_GEN_COMPILER=\"${CMAKE_CXX_COMPILER_ID}\"
        NT_STATUS_GEN_COMPILER_VERSION=\"${CMAKE_CXX_COMPILER_VERSION}\"
        NT_STATUS_GEN_TARGET_OS=\"${CMAKE_SYSTEM_NAME}\"
        NT_STATUS_GEN_TARGET_OS_VERSION=\"${CMAKE_SYSTEM_VERSION}\"
)

# Define output files for generated source code
set(NT_STATUS_GEN_HEADER "${CMAKE_CURRENT_BINARY_DIR}/NtStatusNames.hpp")
set(NT_STATUS_GEN_SOURCE "${CMAKE_CURRENT_BINARY_DIR}/NtStatusNames.cpp")

# Add command to generate source code
add_custom_command(
        COMMAND nt-status-gen-build-tool ${NT_STATUS_GEN_HEADER} ${NT_STATUS_GEN_SOURCE}
        COMMENT "Generating NTSTATUS lookup table"
        OUTPUT ${NT_STATUS_GEN_HEADER} ${NT_STATUS_GEN_SOURCE}
)

# This final library can be used to access the lookup
add_library(
        nt-status-names ${NT_STATUS_GEN_SOURCE}
)
target_include_directories(nt-status-names PUBLIC ${CMAKE_CURRENT_BINARY_DIR})

# Add a target to create outputs only
add_custom_target(nt-status-source-files
        DEPENDS ${NT_STATUS_GEN_HEADER} ${NT_STATUS_GEN_SOURCE}
        COMMENT "Writing output to ${NT_STATUS_GEN_SOURCE} and ${NT_STATUS_GEN_HEADER}"
)

# Add a target to create a distribution zip
set(NT_STATUS_LIBRARY_ZIP "${CMAKE_CURRENT_BINARY_DIR}/NtStatusNames.zip")

add_custom_target(nt-status-zip-library
        DEPENDS ${NT_STATUS_GEN_HEADER} ${NT_STATUS_GEN_SOURCE}
        COMMENT "Creating output ZIP at ${NT_STATUS_LIBRARY_ZIP}"
        COMMAND
            ${CMAKE_COMMAND} -E tar "cvf" ${NT_STATUS_LIBRARY_ZIP} --format=zip
            "${NT_STATUS_GEN_HEADER}"
            "${NT_STATUS_GEN_SOURCE}"
)
