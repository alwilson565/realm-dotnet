set(SOURCES
    debug.cpp
    error_handling.cpp
    event_loop_signal_cs.cpp
    list_cs.cpp
    marshalling.cpp
    object_cs.cpp
    query_cs.cpp
    realm-csharp.cpp
    results_cs.cpp
    schema_cs.cpp
    shared_realm_cs.cpp
    table_cs.cpp
)

set(HEADERS
    debug.hpp
    error_handling.hpp
    marshalable_sort_clause.hpp
    marshalling.hpp
    object_cs.hpp
    realm_error_type.hpp
    realm_export_decls.hpp
    schema_cs.hpp
    shared_realm_cs.hpp
)

if(REALM_ENABLE_SYNC)
    list(APPEND SOURCES
        sync_manager_cs.cpp
        sync_session_cs.cpp
        sync_user_cs.cpp
    )

    list(APPEND HEADERS
        sync_session_cs.hpp
    )
endif()

if(CMAKE_SYSTEM_NAME MATCHES "^Windows")
    list(APPEND SOURCES win32_polyfill.cpp)
endif()

set(INCLUDE_DIRS
    ${REALM_CORE_INCLUDE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR})

if(CMAKE_SYSTEM_NAME MATCHES "^Windows")
    list(APPEND INCLUDE_DIRS ${REALM_CORE_INCLUDE_DIR}/win32/pthread)
endif()

if(${CMAKE_CXX_COMPILER_ID} MATCHES "Clang" OR ${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU")
  add_compile_options(-Wno-missing-prototypes)
endif()

add_library(realm-wrappers SHARED ${SOURCES} ${HEADERS})
target_compile_definitions(realm-wrappers PRIVATE ${PLATFORM_DEFINES})
target_include_directories(realm-wrappers PUBLIC ${INCLUDE_DIRS})
target_link_libraries(realm-wrappers realm-object-store)

if(NOT CMAKE_GENERATOR_PLATFORM STREQUAL "")
    set(runtime_target ${CMAKE_SYSTEM_NAME}/$<CONFIG>-${CMAKE_GENERATOR_PLATFORM})
else()
    set(runtime_target ${CMAKE_SYSTEM_NAME}/$<CONFIG>)
endif()

install(TARGETS realm-wrappers
    RUNTIME DESTINATION ${runtime_target}
    LIBRARY DESTINATION ${runtime_target}
)
if(MSVC)
    install(FILES $<TARGET_PDB_FILE:realm-wrappers> DESTINATION ${runtime_target} OPTIONAL)
endif()