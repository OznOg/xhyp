
add_executable(console 
        lib/_udivsi3.S
        lib/stdlib.c
        lib/fifo.c
        domain.c
        serial.c
        main.S)

target_compile_definitions(console PRIVATE 
        __KERNEL__
        TEXT_BASE=0x00200000
        CONFIG_ARM __ARM__
        TEXT_BASE=0x00200000)


set_target_properties(console PROPERTIES LINK_FLAGS "-nostdlib -Bstatic -T ${CMAKE_CURRENT_SOURCE_DIR}/domain.ld")

target_include_directories(console PRIVATE include/)

set_target_properties(console PROPERTIES PREFIX "")
set_target_properties(console PROPERTIES POSITION_INDEPENDENT_CODE OFF)

set_target_properties(console PROPERTIES SUFFIX ".dom")

