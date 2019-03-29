# Author: Sébastien Gonzalve (2019)
# 
# LICENSE:
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
# 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA 02111-1307, USA.
# 
cmake_minimum_required(VERSION 3.11)

set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/toolchain.cmake)

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

set(KNOW_BOARD beagle realview versatile)
if(NOT DEFINED BOARD OR NOT BOARD IN_LIST KNOW_BOARD)
    message(FATAL_ERROR "Please set BOARD to a value in ${KNOW_BOARD}")
endif()

message("Compiling for board=${BOARD}")

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

add_subdirectory(arch/${ARCH}/common/)
add_subdirectory(arch/${ARCH}/lib/)
add_subdirectory(arch/${ARCH}/${BOARD}/)
add_subdirectory(domains/)


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



