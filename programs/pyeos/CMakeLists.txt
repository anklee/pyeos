add_subdirectory(contracts)

if( UNIX AND NOT APPLE )
  set(rt_library rt )
endif()

find_package( Gperftools QUIET )
if( GPERFTOOLS_FOUND )
    message( STATUS "Found gperftools; compiling steemd with TCMalloc")
    list( APPEND PLATFORM_SPECIFIC_LIBS tcmalloc )
endif()


add_library(python3.6m STATIC IMPORTED)
set_property(TARGET python3.6m PROPERTY IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/libraries/python/libpython3.6m.a)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/wrap/eosapi.wrap.cpp
    COMMAND echo python3 -m cython --cplus ${CMAKE_CURRENT_SOURCE_DIR}/cython/eosapi.pyx -o ${CMAKE_CURRENT_SOURCE_DIR}/wrap/eosapi.wrap.cpp
    COMMAND python3 -m cython --cplus ${CMAKE_CURRENT_SOURCE_DIR}/cython/eosapi.pyx -o ${CMAKE_CURRENT_SOURCE_DIR}/wrap/eosapi.wrap.cpp
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/cython/eosapi.pyx ${CMAKE_CURRENT_SOURCE_DIR}/cython/eostypes_.pxd
)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/wrap/wallet.wrap.cpp
    COMMAND python3 -m cython --cplus ${CMAKE_CURRENT_SOURCE_DIR}/cython/wallet.pyx -o ${CMAKE_CURRENT_SOURCE_DIR}/wrap/wallet.wrap.cpp
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/cython/wallet.pyx ${CMAKE_CURRENT_SOURCE_DIR}/cython/eostypes_.pxd
)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/wrap/pyobject.wrap.cpp
    COMMAND python3 -m cython --cplus ${CMAKE_CURRENT_SOURCE_DIR}/cython/pyobject.pyx -o ${CMAKE_CURRENT_SOURCE_DIR}/wrap/pyobject.wrap.cpp
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/cython/pyobject.pyx
)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/wrap/database.wrap.cpp
    COMMAND python3 -m cython --cplus ${CMAKE_CURRENT_SOURCE_DIR}/cython/database.pyx -o ${CMAKE_CURRENT_SOURCE_DIR}/wrap/database.wrap.cpp
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/cython/database.pyx ${CMAKE_CURRENT_SOURCE_DIR}/cython/eostypes_.pxd ${CMAKE_CURRENT_SOURCE_DIR}/cython/database_object.pxd
)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/wrap/blockchain.wrap.cpp
    COMMAND python3 -m cython --cplus ${CMAKE_CURRENT_SOURCE_DIR}/cython/blockchain.pyx -o ${CMAKE_CURRENT_SOURCE_DIR}/wrap/blockchain.wrap.cpp
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/cython/blockchain.pyx ${CMAKE_CURRENT_SOURCE_DIR}/cython/eostypes_.pxd
)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/wrap/eostypes.wrap.cpp
    COMMAND python3 -m cython --cplus ${CMAKE_CURRENT_SOURCE_DIR}/cython/eostypes.pyx -o ${CMAKE_CURRENT_SOURCE_DIR}/wrap/eostypes.wrap.cpp
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/cython/eostypes.pyx ${CMAKE_CURRENT_SOURCE_DIR}/cython/eostypes_.pxd
)


add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/wrap/util.wrap.cpp
    COMMAND python3 -m cython --cplus ${CMAKE_CURRENT_SOURCE_DIR}/cython/util.pyx -o ${CMAKE_CURRENT_SOURCE_DIR}/wrap/util.wrap.cpp
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/cython/util.pyx
)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/wrap/debug.wrap.cpp
    COMMAND python3 -m cython --cplus ${CMAKE_CURRENT_SOURCE_DIR}/cython/debug.pyx -o ${CMAKE_CURRENT_SOURCE_DIR}/wrap/debug.wrap.cpp
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/cython/debug.pyx
)

add_executable(pyeos main.cpp py_plugin.cpp interface/json.cpp interface/eosapi_.cpp 
    interface/wallet_.cpp interface/pyobject.cpp interface/database_.cpp
    interface/math_.cpp  
    interface/common.cpp
    interface/codelimit.c 
    wrap/eosapi.wrap.cpp 
    wrap/database.wrap.cpp 
    wrap/wallet.wrap.cpp
    wrap/pyobject.wrap.cpp
    wrap/debug.wrap.cpp)

#    util/db_handling.cpp
#    wrap/util.wrap.cpp
#    interface/util_.cpp
#   interface/blockchain_.cpp
#    wrap/eostypes.wrap.cpp
#    wrap/blockchain.wrap.cpp


target_link_libraries( pyeos appbase account_history_api_plugin account_history_plugin chain_api_plugin 
    producer_plugin chain_plugin net_plugin http_plugin fc ${CMAKE_DL_LIBS} ${PLATFORM_SPECIFIC_LIBS} 
    wallet_plugin wallet_api_plugin
    chainbase eos_utilities eos_egenesis_none fc
    python3.6m eosio_chain intl)

target_include_directories( pyeos PUBLIC ${CMAKE_SOURCE_DIR}/contracts 
                                  PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include"
                                  PUBLIC ${CMAKE_SOURCE_DIR}/libraries/python/Include
                                  PUBLIC ${CMAKE_SOURCE_DIR}/libraries/python
                                  PUBLIC  ${CMAKE_SOURCE_DIR}/plugins/wallet_plugin/include
                                  PUBLIC  ${CMAKE_SOURCE_DIR}/plugins/wallet_plugin/include
                                  PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/interface"
                                  PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/util"
                                  )



install( TARGETS
   pyeos

   RUNTIME DESTINATION bin
   LIBRARY DESTINATION lib
   ARCHIVE DESTINATION lib
)
