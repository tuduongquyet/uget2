cmake_minimum_required(VERSION 2.8)

if(CMAKE_VERSION VERSION_LESS "3.0")
	project(uglib C CXX)
	set(PROJECT_VERSION ${UGET_VERSION})
else()
	cmake_policy(SET CMP0042 NEW)
	cmake_policy(SET CMP0048 NEW)
	project(uglib VERSION ${UGET_VERSION} LANGUAGES C CXX)
endif()

include_directories(${CURL_INCLUDE_DIR}
	${CMAKE_CURRENT_SOURCE_DIR}
)

set(UGLIB_SOURCES
	UgStdio.c
	UgString.c
	UgThread.c
	UgSocket.c
	UgUtil.c
	UgFileUtil.c
	UgArray.c
	UgList.c
	UgSLink.c
	UgOption.c
	UgUri.c
	UgNode.c
	UgData.c
	UgInfo.c
	UgRegistry.c
	UgValue.c
	UgEntry.c
	UgBuffer.c
	UgJson.c
	UgJson-custom.c
	UgJsonFile.c
	UgJsonrpc.c
	UgJsonrpcSocket.c
	UgJsonrpcCurl.c
	UgHtml.c
	UgHtmlEntry.c
	UgHtmlFilter.c
)

add_library(uglib STATIC ${UGLIB_SOURCES})
