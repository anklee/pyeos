file(GLOB HEADERS "include/eosio/chain/*.hpp" "include/eosio/chain/contracts/*.hpp")

find_package(leveldb)

## SORT .cpp by most likely to change / break compile
add_library( eosio_chain
             chain_config.cpp
             merkle.cpp
             name.cpp
             transaction.cpp
             block.cpp
             wast_to_wasm.cpp
             wasm_interface.cpp

             evm_interface.cpp
             micropython_interface.cpp
#             wasm_eosio_constraints.cpp

             wasm_eosio_validation.cpp
             wasm_eosio_injection.cpp

             apply_context.cpp
             rate_limiting.cpp

             fork_database.cpp
             get_config.cpp
             block_log.cpp
             asset.cpp


             global_property_object.cpp
             chain_controller.cpp

             contracts/eosio_contract.cpp
             contracts/chain_initializer.cpp
             contracts/genesis_state.cpp
             contracts/abi_serializer.cpp

             micropython/mpeoslib.cpp

             evm/EosState.cpp
             evm/EosExecutive.cpp
             evm/EosDB.cpp
             evm/EosExtVM.cpp

             webassembly/wavm.cpp
             webassembly/binaryen.cpp

             ${HEADERS}
             transaction_metadata.cpp)

include_directories (${CPP_ETHEREUM_DIR})
include_directories (${CPP_ETHEREUM_DIR}/evmjit/include)
include_directories (${CPP_ETHEREUM_DIR}/utils/include)
include_directories (${CPP_ETHEREUM_DIR}/utils)
include_directories (jsoncpp/include)

#add_library(secp256k STATIC IMPORTED)
#set_property(TARGET secp256k PROPERTY IMPORTED_LOCATION "${CMAKE_SOURCE_DIR}/build/deps/lib/libsecp256k1.a")

#set (ETH_LIBRARY devcore ethereum evm ethashseal ethcore leveldb ${SECP256K1_LIBRARY})
set (ETH_LIBRARY devcore ethereum evm ethashseal ethcore)

message("++++++++++++++++++"${CMAKE_CURRENT_LIST_LINE}":"${LevelDB_INCLUDE})
message("++++++++++++++++++"${CMAKE_CURRENT_LIST_FILE})

target_link_libraries( eosio_chain eos_utilities fc chainbase appbase Logging IR WAST WASM Runtime 
        wasm asmjs passes cfg ast emscripten-optimizer support softfloat  micropython ${ETH_LIBRARY} )

target_include_directories( eosio_chain
                            PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include" "${CMAKE_CURRENT_BINARY_DIR}/include"
                                   "${CMAKE_CURRENT_SOURCE_DIR}/../wasm-jit/Include"
                                   "${CMAKE_CURRENT_SOURCE_DIR}/../../externals/binaryen/src"
                            PUBLIC ${CMAKE_SOURCE_DIR}/libraries/micropython
                            PUBLIC ${CMAKE_SOURCE_DIR}/libraries/micropython/py
                            PUBLIC ${CMAKE_SOURCE_DIR}/libraries/micropython/extmod
                            PUBLIC ${CMAKE_SOURCE_DIR}/libraries/micropython/extmod/eoslib
                            PUBLIC ${CMAKE_SOURCE_DIR}/libraries/micropython/ports/unix
                            PUBLIC ${CMAKE_SOURCE_DIR}/libraries/micropython/ports/unix/build
                            PRIVATE ${LevelDB_INCLUDE}
                            )

if(MSVC)
  set_source_files_properties( db_init.cpp db_block.cpp database.cpp block_log.cpp PROPERTIES COMPILE_FLAGS "/bigobj" )
endif(MSVC)

add_subdirectory(test)

#add_executable( test test.cpp )
#target_link_libraries( test eosio_chain ${CMAKE_DL_LIBS} ${PLATFORM_SPECIFIC_LIBS} ${Intl_LIBRARIES} )

