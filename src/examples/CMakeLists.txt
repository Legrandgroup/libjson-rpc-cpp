file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/gen)

add_custom_command(
	OUTPUT ${CMAKE_BINARY_DIR}/gen/abstractstubserver.h
	COMMAND jsonrpcstub ARGS ${CMAKE_CURRENT_SOURCE_DIR}/spec.json --cpp-server=AbstractStubServer --cpp-server-file=${CMAKE_BINARY_DIR}/gen/abstractstubserver.h
	MAIN_DEPENDENCY spec.json
	DEPENDS jsonrpcstub
	COMMENT "Generating Server Stubfiles"
	VERBATIM
)

add_custom_command(
	OUTPUT ${CMAKE_BINARY_DIR}/gen/stubclient.h
	COMMAND jsonrpcstub ARGS ${CMAKE_CURRENT_SOURCE_DIR}/spec.json --cpp-client=StubClient --cpp-client-file=${CMAKE_BINARY_DIR}/gen/stubclient.h
	MAIN_DEPENDENCY spec.json
	DEPENDS jsonrpcstub
	COMMENT "Generating Client Stubfile"
	VERBATIM
)

add_custom_target(common_stubs
	DEPENDS ${CMAKE_BINARY_DIR}/gen/abstractstubserver.h ${CMAKE_BINARY_DIR}/gen/stubclient.h
)

add_custom_command(
	OUTPUT ${CMAKE_BINARY_DIR}/gen/xbmcremote.h
	COMMAND jsonrpcstub ARGS ${CMAKE_CURRENT_SOURCE_DIR}/xbmc_remote.json --cpp-client=XbmcRemoteClient --cpp-client-file=${CMAKE_BINARY_DIR}/gen/xbmcremote.h
	MAIN_DEPENDENCY xbmc_remote.json
	DEPENDS jsonrpcstub
	COMMENT "Generating Client XBMC Stubfile"
	VERBATIM
)

include_directories(..)
include_directories(${CMAKE_BINARY_DIR})
include_directories(${JSONCPP_INCLUDE_DIRS})
include_directories(${MHD_INCLUDE_DIRS})

if (COMPILE_EXAMPLES)
	if (HTTP_SERVER AND HTTP_CLIENT)
		add_executable(simpleserversample simpleserver.cpp)
		target_link_libraries(simpleserversample jsonrpcserver)
	
		add_executable(simpleclientsample simpleclient.cpp)
		target_link_libraries(simpleclientsample jsonrpcclient)
		
		if (COMPILE_STUBGEN)
			add_executable(stubclientsample stubclient.cpp ${CMAKE_BINARY_DIR}/gen/stubclient.h)
			target_link_libraries(stubclientsample jsonrpcclient)
	
			add_executable(stubserversample stubserver.cpp ${CMAKE_BINARY_DIR}/gen/abstractstubserver.h)
			target_link_libraries(stubserversample jsonrpcserver)
	
			add_executable(xbmcremote xbmcremote.cpp ${CMAKE_BINARY_DIR}/gen/xbmcremote.h)
			target_link_libraries(xbmcremote jsonrpcclient)
		endif()
	endif (HTTP_SERVER AND HTTP_CLIENT)
	
	if(UNIX)
		if (UNIX_DOMAIN_SOCKET_SERVER AND UNIX_DOMAIN_SOCKET_CLIENT)
			add_executable(unixdomainsocketserversample unixdomainsocketserver.cpp)
			target_link_libraries(unixdomainsocketserversample jsonrpcserver)
		
			add_executable(unixdomainsocketclientsample unixdomainsocketclient.cpp)
			target_link_libraries(unixdomainsocketclientsample jsonrpcclient)
		endif (UNIX_DOMAIN_SOCKET_SERVER AND UNIX_DOMAIN_SOCKET_CLIENT)
	endif(UNIX)

	
endif()
