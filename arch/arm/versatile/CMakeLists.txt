
add_library(start STATIC 
        start.S)
target_compile_definitions(start PRIVATE 
        __KERNEL__ __ASSEMBLY__
        CONFIG_ARM __ARM__
        TEXT_BASE=0x00200000)
target_include_directories(start PRIVATE ${CMAKE_SOURCE_DIR}/include/)
