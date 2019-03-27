cmake_minimum_required(VERSION 3.11)

set(CMAKE_TOOLCHAIN_FILE toolchain.cmake)

enable_language(ASM)

set(project xhyp)

add_compile_options(-fno-common -ffixed-r8 -msoft-float -fno-builtin
        -ffreestanding -nostdinc -pipe -marm  -mabi=aapcs-linux -mno-thumb-interwork
        -march=armv5te -Wall -Wstrict-prototypes -fno-stack-protector -Werror)

if(CONFIG_SCHED STREQUAL "POSIX")
set(SCHED kernel/sched_posix.c)
elseif(CONFIG_SCHED STREQUAL "ARINC")
set(SCHED kernel/sched_arinc.c)
else()
   message(FATAL_ERROR "Unknown Scheduler ${CONFIG_SCHED}")
endif()

set(KNOW_ARCH arm leon mips sh4 x86)

if(NOT DEFINED ARCH OR NOT ARCH IN_LIST KNOW_ARCH)
   message(FATAL_ERROR "Please set ARCH to a value in ${KNOW_ARCH}")
endif()

message("Compiling for arch=${ARCH}")

add_library(kernel STATIC
        kernel/copypage.c
        kernel/domain.c
        kernel/event.c
        kernel/hypercalls.c
        kernel/io.c
        kernel/irq.c
        kernel/main.c
        kernel/mm.c
        kernel/panic.c
        ${SCHED}
        kernel/scheduler.c
        kernel/tag.c
        kernel/timer.c)
target_compile_definitions(kernel PRIVATE
                                       __KERNEL__
                                       TEXT_BASE=0x00200000
                                       CONFIG_ARM __ARM__
                                       TEXT_BASE=0x00200000)
target_include_directories(kernel PRIVATE include/)

add_library(lib STATIC 
        lib/fifo.c
        lib/ring.c
        lib/stdlib.c
        lib/_udivsi3.s
        lib/_udivsi3.S)
target_compile_definitions(lib PRIVATE 
                                       __KERNEL__
                                       TEXT_BASE=0x00200000
                                       CONFIG_ARM __ARM__
                                       TEXT_BASE=0x00200000)


target_include_directories(lib PRIVATE include/)
