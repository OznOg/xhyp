

file(STRINGS config.domains CONFIG_DOMAINS REGEX "^[^ #]")

file(GLOB DOMAINS_DIR domain0?)

set(OFST "0x00100000")
set(DOM_ID "0")
foreach(domain ${DOMAINS_DIR})
    list(GET CONFIG_DOMAINS ${DOM_ID} config)
    math(EXPR DOM_ID "${DOM_ID}+1")
    get_filename_component(DOMAIN_NAME ${domain} NAME)
    add_custom_target(
            ${DOMAIN_NAME} ALL
            COMMAND ${CMAKE_COMMAND} -E make_directory ${DOMAIN_NAME}
            COMMAND ${CMAKE_OBJCOPY} ${domain}/os -O binary ${DOMAIN_NAME}/domain.bin
            COMMAND ${CMAKE_OBJCOPY} --input-target binary --output-target elf32-littlearm
                                     --binary-architecture arm --rename-section '.data=.dm${DOM_ID}'
                                     ${DOMAIN_NAME}/domain.bin ${DOMAIN_NAME}/domain.ho
            COMMENT "Building ${domain}")
    list(APPEND DOMAINS ${CMAKE_CURRENT_BINARY_DIR}/${DOMAIN_NAME}/domain.ho)

    # original file uses \t and spaces to indent fields, need to remove all this
    # before processing inputs
    string(REGEX REPLACE "[ \t]+" ";" config ${config})
    list(GET config 0 name)
    list(GET config 1 base)
    list(GET config 2 virt)
    list(GET config 3 size)
    list(GET config 4 flags)
    list(GET config 5 prio)
    list(GET config 6 state)
    list(GET config 7 dom_type)
    list(GET config 8 drv_type)
    list(GET config 9 path)
    set(start "${base} + ${OFST}")
    if(dom_type STREQUAL "DTYPE_GPOS")
       set(offset "0xc")
       set(device "0 /* not set */")
       set(device_size "0 /* not set */")
    elseif(dom_type STREQUAL "DTYPE_DRV")
       set(offset "0 /* not set */")
       set(device "CONFIG_PERIPH_BASE")
       set(device_size "0x00200000")
    else()
       set(offset "0 /* not set */")
       set(device "0 /* not set */")
       set(device_size "0 /* not set */")
     endif()
    configure_file(domain_table.c.entry.in domain_table.c.entry.${DOM_ID})
    list(APPEND generated_entries ${CMAKE_CURRENT_BINARY_DIR}/domain_table.c.entry.${DOM_ID})

    #loader script stuff
    string(APPEND PHDRS "  dom${DOM_ID} PT_LOAD;\n")
    string(APPEND SECTIONS "      .dm${DOM_ID} : { } > ram_dom${DOM_ID} : dom${DOM_ID}\n")
    string(APPEND MEMORY_LAYOUT "  ram_ctl${DOM_ID} : org = ${base}, l = ${OFST}\n")
    string(APPEND MEMORY_LAYOUT "  ram_dom${DOM_ID} : org = ${base} + ${OFST}, l = ${size} - ${OFST}\n")
endforeach()

configure_file(xhyp.ld.in xhyp.ld)

configure_file(domain_table.c.header.in domain_table.c)
foreach(entry ${generated_entries})
    file(READ ${entry} content)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/domain_table.c "${content}")
endforeach()

file(READ domain_table.c.footer.in footer)
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/domain_table.c "${footer}")

add_library(domain_table STATIC ${CMAKE_CURRENT_BINARY_DIR}/domain_table.c)
target_compile_definitions(domain_table PRIVATE 
        __KERNEL__
        TEXT_BASE=0x00200000
        CONFIG_ARM __ARM__
        TEXT_BASE=0x00200000)

target_include_directories(domain_table PRIVATE ${CMAKE_SOURCE_DIR}/include/)
