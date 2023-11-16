cmake_minimum_required(VERSION 2.8)

if(CMAKE_VERSION VERSION_LESS "3.0")
  project(uget C CXX)
  set(PROJECT_VERSION ${UGET_VERSION})
else()
  cmake_policy(SET CMP0042 NEW)
  cmake_policy(SET CMP0048 NEW)
  project(uget VERSION ${UGET_VERSION} LANGUAGES C CXX)
endif()

include_directories(${CURL_INCLUDE_DIR}
  ${OPENSSL_INCLUDE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/../uglib
)

set(UGET_SOURCES
  UgetSequence.c
  UgetRss.c
  UgetRpc.c
  UgetOption.c
  UgetData.c
  UgetFiles.c
  UgetNode.c
  UgetNode-compare.c
  UgetNode-filter.c
  UgetTask.c
  UgetHash.c
  UgetSite.c
  UgetApp.c
  UgetEvent.c
  UgetPlugin.c
  UgetA2cf.c
  UgetCurl.c
  UgetAria2.c
  UgetMedia.c
  UgetMedia-youtube.c
  UgetPluginCurl.c
  UgetPluginAria2.c
  UgetPluginMedia.c
  UgetPluginAgent.c
  UgetPluginMega.c
  UgetPluginEmpty.c
)

if(WITH_LIBPWMD)
  set(UGET_SOURCES ${UGET_SOURCES} pwmd.c)
endif()

add_library(uget STATIC ${UGET_SOURCES})
