cmake_minimum_required(VERSION 3.5)

project(hello)

IF (CMAKE_COMPILER_IS_GNUCC)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -std=c++11 -fmessage-length=0")
ENDIF (CMAKE_COMPILER_IS_GNUCC)
IF (CMAKE_COMPILER_IS_GNUCXX)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=c++11 -fmessage-length=0")
ENDIF (CMAKE_COMPILER_IS_GNUCXX)

SET(CMAKE_CXX_STANDARD 11)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)

add_custom_target(build_clean
        COMMAND ${CMAKE_COMMAND} -P ${CMAKE_SOURCE_DIR}/cmake_files/build_clean.cmake
        )
add_custom_target(test_on
        COMMAND ${CMAKE_COMMAND} -Dtest=ON ..
        )
add_custom_target(test_off
        COMMAND ${CMAKE_COMMAND} -Dtest=OFF ..
        )

find_package(Doxygen)

if (DOXYGEN_FOUND)
    if (WIN32)
        set(CMAKE_DOT_PATH "\"C:\\Program Files (x86)\\Graphviz2.38\\bin\"")
    endif ()
    if (UNIX)
        set(CMAKE_DOT_PATH "")
    endif ()

#    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile @ONLY)
#    add_custom_target(doc
#            ${DOXYGEN_EXECUTABLE}
#            ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
#            WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
#            COMMENT "Gerando a documentacao da API com Doxygen" VERBATIM
#            )
else ()
    message("Doxygen ${Red}não${ColourReset} foi encontrado")
endif (DOXYGEN_FOUND)

SET(cleos_cmd "docker exec -it eosio /opt/eosio/bin/cleos --url http://127.0.0.1:7777 --wallet-url http://127.0.0.1:5555")

function(createContract contract_name contract_path files)
    add_custom_target("${contract_name}" ALL
            DEPENDS ${files}
            COMMAND ${CMAKE_COMMAND} -E cmake_echo_color --red "Compiling contract ${contract_name}"
            COMMAND eosio-cpp -o ${CMAKE_CURRENT_BINARY_DIR}/${contract_name}.wasm ${files} --abigen
            COMMENT "Compiling contract ${contract_name}"
            VERBATIM
            )

    add_custom_target("${contract_name}_set" ALL
            DEPENDS ${contract_name}
            COMMAND ${CMAKE_COMMAND} -E cmake_echo_color --red "Setting contract ${contract_name}"
            COMMAND docker exec -it eosio /opt/eosio/bin/cleos --url http://127.0.0.1:7777 --wallet-url http://127.0.0.1:5555 set contract ${contract_name} ${contract_path} -p ${contract_name}@active
            COMMENT "Setting contract ${contract_name}"
            VERBATIM
            )
endfunction()

function(pushContract contract_name action_name owner_name user_name)
    add_custom_target("${contract_name}_${action_name}_${owner_name}_${user_name}_push" ALL
            DEPENDS "${contract_name}_set"
            COMMAND ${CMAKE_COMMAND} -E cmake_echo_color --yellow "Pushing contract ${owner_name} - ${contract_name}.${action_name} - ${user_name}"
            COMMAND docker exec -it eosio /opt/eosio/bin/cleos --url http://127.0.0.1:7777 --wallet-url http://127.0.0.1:5555 push action ${contract_name} ${action_name} '["${owner_name}"]' -p ${user_name}@active
            COMMENT "Pushing contract ${owner_name} - ${contract_name}.${action_name} - ${user_name}"
            VERBATIM
            )
endfunction()

add_subdirectory(hello)
