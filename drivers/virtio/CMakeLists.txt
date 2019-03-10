
add_executable(VirtIOConsole 
        lib/_udivsi3.S
        lib/stdlib.c
        lib/fifo.c
        domain.c
        serial.c
        main.S)

target_compile_definitions(VirtIOConsole PRIVATE 
        __KERNEL__
        TEXT_BASE=0x00200000)


set_target_properties(VirtIOConsole PROPERTIES LINK_FLAGS "-nostdlib -Bstatic -T ${CMAKE_CURRENT_SOURCE_DIR}/domain.ld")

target_include_directories(VirtIOConsole PRIVATE include/)

set_target_properties(VirtIOConsole PROPERTIES POSITION_INDEPENDENT_CODE OFF)

set_target_properties(VirtIOConsole PROPERTIES SUFFIX ".dom")

set_target_properties(VirtIOConsole PROPERTIES OUTPUT_NAME "virtio_console")

set_target_properties(VirtIOConsole PROPERTIES
                      RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/os)

