

file(STRINGS config.domains CONFIG_DOMAINS REGEX "^[^ #]")
foreach(config ${CONFIG_DOMAINS})
message("Coucou ${config}")
endforeach()

file(GLOB DOMAINS_DIR domain0?)

foreach(domain ${DOMAINS_DIR})
    get_filename_component(DOMAIN_NAME ${domain} NAME)
    add_custom_target(
            ${DOMAIN_NAME} ALL
            COMMAND ${CMAKE_COMMAND} -E make_directory ${DOMAIN_NAME}
            COMMAND ${CMAKE_OBJCOPY} ${domain}/os -O binary ${DOMAIN_NAME}/domain.bin
            COMMAND ${CMAKE_OBJCOPY} --input-target binary --output-target elf32-littlearm
                                     --binary-architecture arm --rename-section '.data=.dm1'
                                     ${DOMAIN_NAME}/domain.bin ${DOMAIN_NAME}/domain.ho
            COMMENT "Building ${domain}")
    list(APPEND DOMAINS ${CMAKE_CURRENT_BINARY_DIR}/${DOMAIN_NAME}/domain.ho)
endforeach()

