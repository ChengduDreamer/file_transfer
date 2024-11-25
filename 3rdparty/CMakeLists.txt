set(BUILD_SHARED_LIBS OFF CACHE BOOL "BUILD_SHARED_LIBS")
set(SPDLOG_WCHAR_SUPPORT ON CACHE BOOL "SPDLOG_WCHAR_SUPPORT ON" FORCE)
set(SPDLOG_WCHAR_FILENAMES ON CACHE BOOL "SPDLOG_WCHAR_FILENAMES ON" FORCE)
add_definitions(-DSPDLOG_FMT_EXTERNAL) 
add_subdirectory(spdlog)
add_subdirectory(boost)
add_subdirectory(leveldb)

set(ENABLE_TEST OFF CACHE BOOL "Disable tests" FORCE)
add_subdirectory(EventBus)


set(protobuf_BUILD_TESTS OFF CACHE BOOL "disable protobuf_BUILD_TESTS" FORCE)
add_subdirectory(protobuf)


