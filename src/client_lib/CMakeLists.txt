file("GLOB"
    source_files
    ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)

add_definitions(-DDSN_ENABLE_THRIFT_RPC -DDSN_USE_THRIFT_SERIALIZATION -fPIC)

# include_dir DSN_ROOT/include & DSN_THIRDPARTY_ROOT/include is added by the dsn.cmake framework
include_directories(${CMAKE_SOURCE_DIR}/include)
include_directories(${CMAKE_SOURCE_DIR}/base)
add_library(pegasus_client_impl_objects OBJECT ${source_files})
add_library(pegasus_client_impl $<TARGET_OBJECTS:pegasus_client_impl_objects>)

# both shared & static version of pegasus client depends on boost_system,boost_filesystem,aio,dl
#
# link the static lib of pegasus_client:
# with pegasus_client_impl pegasus.base thrift
set(pegasus_clientlib_static_path ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}/libpegasus_client_static.a)
add_custom_target(clientlib_target
    COMMAND ar -x ${DSN_ROOT}/lib/libdsn_runtime.a
    COMMAND ar -x ${DSN_THIRDPARTY_ROOT}/lib/libthrift.a
    COMMAND ar -x libpegasus_client_impl.a
    COMMAND ar -x libpegasus.base.a
    COMMAND ar -qcs ${pegasus_clientlib_static_path} *.o
    COMMAND rm -rf *.o
    WORKING_DIRECTORY ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}
    DEPENDS pegasus_client_impl pegasus.base
    COMMENT "generate pegasus client static library"
    )

add_library(pegasus_client_static STATIC IMPORTED GLOBAL)
add_dependencies(pegasus_client_static clientlib_target)
set_target_properties(pegasus_client_static
    PROPERTIES
    IMPORTED_LOCATION ${pegasus_clientlib_static_path})
install(FILES ${pegasus_clientlib_static_path} DESTINATION "lib")

# link the shared lib of pegasus client
add_library(pegasus_client_shared SHARED $<TARGET_OBJECTS:pegasus_client_impl_objects>)
target_link_libraries(pegasus_client_shared PRIVATE
    pegasus.base
    dsn_runtime
    thrift)
install(TARGETS pegasus_client_shared DESTINATION "lib")
