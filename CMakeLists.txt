cmake_minimum_required(VERSION 3.1.0)

include(env.cmake)
set(CMAKE_PREFIX_PATH ${QT_ROOT}/lib/cmake)

project(YK_FileTransfer VERSION 1.0.1)

include(InstallRequiredSystemLibraries)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

set(CMAKE_VERBOSE_MAKEFILE ON)

if(CMAKE_VERSION VERSION_LESS "3.7.0")
    set(CMAKE_INCLUDE_CURRENT_DIR ON)
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

#add_definitions(-DUNICODE -D_UNICODE)
add_definitions(-DUNICODE -D_UNICODE -DWIN32_LEAN_AND_MEAN)

find_package(Qt5 COMPONENTS Widgets REQUIRED)
find_package(Qt5 COMPONENTS WebSockets REQUIRED)
find_package(Qt5 COMPONENTS Sql REQUIRED)
find_package(Qt5 COMPONENTS WinExtras Charts REQUIRED)

if(${CMAKE_SIZEOF_VOID_P} STREQUAL "4")
	set(PLATFORM_WIN64 FALSE)
else()
	set(PLATFORM_WIN64 TRUE)
endif()

include_directories(${CMAKE_SOURCE_DIR}
					${CMAKE_SOURCE_DIR}/src
					${CMAKE_SOURCE_DIR}/3rdparty
					${CMAKE_SOURCE_DIR}/3rdparty/asio2/include
					${CMAKE_SOURCE_DIR}/3rdparty/asio2/3rd
					${CMAKE_SOURCE_DIR}/3rdparty/leveldb/include
					${CMAKE_SOURCE_DIR}/3rdparty/EventBus/lib/src
					${CMAKE_SOURCE_DIR}/3rdparty/boost/libs/variant/include
					${CMAKE_SOURCE_DIR}/3rdparty/boost/libs/signals2/include
					${CMAKE_SOURCE_DIR}/3rdparty/protobuf/src
					${CMAKE_SOURCE_DIR}/3rdparty/spdlog/include/
					${CMAKE_SOURCE_DIR}/3rdparty/spdlog/include/spdlog
)

set(src_SOURCES
	src/main.cpp
	ico.rc
	resources.qrc
)

add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")

if(MSVC)
	add_compile_options($<IF:$<CONFIG:Debug>,/MTd,/MT>)
	add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/bigobj>")
endif()

add_compile_definitions(ASIO2_ENABLE_SSL ASIO2_ENABLE_LOG)

add_subdirectory(3rdparty)

include_directories(
				${CMAKE_SOURCE_DIR}/3rdparty/boost/libs/locale/include
				${CMAKE_BINARY_DIR}
				)

link_directories(
)

### Protobuf start
set(Protobuf_SRC_ROOT_FOLDER ${CMAKE_SOURCE_DIR}/3rdparty/protobuf)
if(CMAKE_BUILD_TYPE MATCHES "Release" OR CMAKE_BUILD_TYPE MATCHES "RelWithDebInfo")
	set(Protobuf_LIBRARIES ${CMAKE_BINARY_DIR}/3rdparty/protobuf/cmake/libprotobufmt.lib)
else()
	set(Protobuf_LIBRARIES ${CMAKE_BINARY_DIR}/3rdparty/protobuf/cmake/libprotobufmtd.lib)
endif()
find_package(Protobuf REQUIRED)
protobuf_generate_cpp(FILE_TRANS_PROTO_SRCS FILE_TRANS_PROTO_HDRS
		${CMAKE_SOURCE_DIR}/protocol/file_trans.proto
)
message("Protobuf path: ${FILE_TRANS_PROTO_SRCS}" )
include_directories(${Protobuf_INCLUDE_DIRS})
### Protobuf end

set(GENERATE_SETUP false)

if(${GENERATE_SETUP})
	#WIN32
	add_executable(${PROJECT_NAME} WIN32
		${src_SOURCES} ${FILE_TRANS_PROTO_SRCS}
	)
else()
	add_executable(${PROJECT_NAME}
		${src_SOURCES} ${FILE_TRANS_PROTO_SRCS}
	)
endif()

SET_TARGET_PROPERTIES(${PROJECT_NAME} PROPERTIES LINK_FLAGS "/MANIFESTUAC:\"level=\'requireAdministrator\' uiAccess=\'false\'\"")

#if(PLATFORM_WIN64)
#	target_link_libraries(${PROJECT_NAME} libssl64MT.lib libcrypto64MT.lib)
#else()
#	target_link_libraries(${PROJECT_NAME} libssl32MT.lib libcrypto32MT.lib)
#endif(PLATFORM_WIN64)

find_package(Qt5 COMPONENTS Svg REQUIRED)
target_link_libraries(${PROJECT_NAME} Qt5::Widgets Qt5::WebSockets Qt5::Network Qt5::Svg Qt5::WinExtras)
target_link_libraries(${PROJECT_NAME} libprotobuf)
target_link_libraries(${PROJECT_NAME} Boost::beast Boost::url Boost::variant2 Boost::process leveldb spdlog::spdlog)

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
	add_custom_command(
					TARGET ${PROJECT_NAME}
					POST_BUILD
					COMMAND ${QT_ROOT}/bin/windeployqt.exe --debug ${PROJECT_NAME}.exe
					WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
	)
else()
	add_custom_command(
					TARGET ${PROJECT_NAME}
					POST_BUILD
					COMMAND ${QT_ROOT}/bin/windeployqt.exe --release ${PROJECT_NAME}.exe
					WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
	)
endif()
