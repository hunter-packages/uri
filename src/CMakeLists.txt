# Copyright (c) Glyn Matthews 2012, 2013.
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE_1_0.txt or copy at
# http://www.boost.org/LICENSE_1_0.txt)

sugar_include("./") # URI_SOURCES

if(CPP-NETLIB_URI_BUILD_SHARED_LIBS OR BUILD_SHARED_LIBS)
  set(URI_TYPE SHARED)
  set(Boost_USE_STATIC_LIBS OFF)
else()
  set(URI_TYPE STATIC)
  set(Boost_USE_STATIC_LIBS ON)
endif()
set(Boost_USE_MULTITHREADED ON)

# Install config
set(LIB_DEST lib)
set(INCL_DEST include)
set(CONF_DEST lib/cmake/CppNetlibUri)

set(uri_config "${CMAKE_CURRENT_BINARY_DIR}/CppNetlibUriConfig.cmake")

include(CMakePackageConfigHelpers)

configure_package_config_file(
    "CppNetlibUriConfig.cmake.in"
    "${uri_config}"
    INSTALL_DESTINATION ${CONF_DEST}
    PATH_VARS CONF_DEST
)

find_package(Boost 1.53 REQUIRED system filesystem)

add_library(network-uri ${URI_TYPE} ${URI_SOURCES})
target_include_directories(
    network-uri
    PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>"
    "$<INSTALL_INTERFACE:${INCL_DEST}>"
)
target_include_directories(network-uri SYSTEM PUBLIC ${Boost_INCLUDE_DIR})
target_link_libraries(network-uri PUBLIC ${Boost_LIBRARIES})

if(MSVC)
  target_compile_definitions(network-uri PUBLIC BOOST_ALL_NO_LIB=1)
endif()

if(MSVC)
  target_compile_definitions(
      network-uri
      PUBLIC
      -D_SCL_SECURE_NO_WARNINGS -D_VARIADIC_MAX=10
  )
endif()

if(WIN32)
  target_compile_definitions(
      network-uri
      PUBLIC
      -D_WIN32_WINNT=0x0501
  )
endif()

install(TARGETS network-uri DESTINATION ${LIB_DEST} EXPORT CppNetlibUriTargets)
install(
    DIRECTORY network DESTINATION ${INCL_DEST} FILES_MATCHING PATTERN "*.hpp"
)
install(EXPORT CppNetlibUriTargets DESTINATION ${CONF_DEST})
install(FILES "${uri_config}" DESTINATION ${CONF_DEST})
