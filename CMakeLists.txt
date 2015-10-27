cmake_minimum_required(VERSION 2.6)

project(natpmp)

add_definitions( -DENABLE_STRNATPMPERR )
add_definitions( -DNATPMP_STATICLIB )
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

set(SRCS natpmp.c getgateway.c)

if(WIN32)
	list(APPEND SRCS wingettimeofday.c)
endif()

add_library(natpmp STATIC ${SRCS})
