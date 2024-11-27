cmake_minimum_required(VERSION 3.10)

set(CMAKE_SYSTEM_NAME Generic)

set(CMAKE_OBJCOPY riscv32-unknown-elf-objcopy)
set(CMAKE_C_COMPILER riscv32-unknown-elf-gcc)
set(CMAKE_ASM_COMPILER riscv32-unknown-elf-gcc)
set(_CMAKE_TOOLCHAIN_PREFIX riscv32-unknown-elf-)

set(CMAKE_C_FLAGS "-march=rv32ima_zifencei -mabi=ilp32 -O0")
set(CMAKE_ASM_FLAGS "-march=rv32ima_zifencei -mabi=ilp32 -O0")
set(CMAKE_EXE_LINKER_FLAGS "-march=rv32ima_zifencei -nostartfiles -T ${CMAKE_CURRENT_SOURCE_DIR}/memory.ld")

set_property(
        DIRECTORY
        APPEND
        PROPERTY ADDITIONAL_CLEAN_FILES out
)

project(embive_tests C ASM)

file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/out")

file(GLOB source_files "tests/rv32*/*.S")

foreach(source_file IN LISTS source_files)
get_filename_component(target_name ${source_file} NAME_WE)
get_filename_component(target_dir_abs "${source_file}/.." ABSOLUTE)
get_filename_component(target_dir_name "${target_dir_abs}" NAME)

file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/out/${target_dir_name}")

add_executable(${target_name}.elf "${source_file}")
target_include_directories(${target_name}.elf PRIVATE ./include/ )

add_custom_command(TARGET ${target_name}.elf POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -O binary  ${target_name}.elf  "out/${target_dir_name}/${target_name}.bin"
        COMMENT "Invoking: Objcopy")
endforeach()
