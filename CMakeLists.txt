cmake_minimum_required(VERSION 3.5)

file(STRINGS VERSION PVER)

project(natpmp
        LANGUAGES C
        VERSION ${PVER}
        DESCRIPTION "A portable and fully compliant implementation of the NAT-PMP protocol"
        HOMEPAGE_URL "https://github.com/miniupnp/libnatpmp"
)

set (NATPMP_SOURCES
        natpmp.c
        getgateway.c
)

if (WIN32)
    set (NATPMP_SOURCES ${NATPMP_SOURCES} wingettimeofday.c)
endif (WIN32)

option(BUILD_SHARED_LIBS "Build using shared libraries" OFF)
option(BUILD_EXECUTABLES "Build executables" ON)

# Library itself
add_library(natpmp ${NATPMP_SOURCES})
target_include_directories(natpmp PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}> $<INSTALL_INTERFACE:include>)
target_compile_definitions(natpmp PRIVATE -DENABLE_STRNATPMPERR)

if (WIN32)
	target_link_libraries(natpmp PUBLIC ws2_32 iphlpapi)
	if (BUILD_SHARED_LIBS)
		target_compile_definitions(natpmp PUBLIC -DNATPMP_EXPORTS)
	else() # win static lib
		target_compile_definitions(natpmp PUBLIC -DNATPMP_STATICLIB)
	endif (BUILD_SHARED_LIBS)
endif (WIN32)

if (${CMAKE_SYSTEM_NAME} STREQUAL "Haiku")
	target_link_libraries(natpmp PUBLIC network)
endif ()

if(BUILD_EXECUTABLES)
	# Executables
	add_executable(natpmpc natpmpc.c)
	target_link_libraries(natpmpc natpmp)

	add_executable(testgetgateway
		testgetgateway.c
		getgateway.c)
	target_link_libraries(testgetgateway natpmp)
endif()

# natpmp.pc
include(GNUInstallDirs)
configure_file(natpmp.pc.in natpmp.pc @ONLY)

# install
install(TARGETS natpmp
        EXPORT libnatpmp-targets
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

if(BUILD_EXECUTABLES)
        install(TARGETS natpmpc
                EXPORT libnatpmp-targets
                DESTINATION ${CMAKE_INSTALL_BINDIR})
endif()

install(FILES natpmp.h natpmp_declspec.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/natpmp.pc
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

include(CMakePackageConfigHelpers)
configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/libnatpmp-config.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/libnatpmp-config.cmake"
  INSTALL_DESTINATION "share/libnatpmp"
)
install(
  FILES "${CMAKE_CURRENT_BINARY_DIR}/libnatpmp-config.cmake"
  DESTINATION "share/libnatpmp"
)

install(
    EXPORT libnatpmp-targets
    DESTINATION share/libnatpmp
    NAMESPACE libnatpmp::
)
