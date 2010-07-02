cmake_minimum_required(VERSION 2.6)

project(minizip)

# needed packages
find_package(ZLIB REQUIRED)
link_directories(${ZLIB_LIBRARY_DIRS})

set(MINIZIP_LIB_MAJOR 1)
set(MINIZIP_LIB_MINOR 1)
set(MINIZIP_LIB_RELEASE 0)
set(MINIZIP_LIB_NAME minizip_${MINIZIP_LIB_MAJOR}-${MINIZIP_LIB_MINOR})
set(MINIZIP_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(MINIZIP_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

# to distinguish between debug and release lib
set(CMAKE_DEBUG_POSTFIX "d")


# command line options
option(BUILD_SHARED_LIBS "Set to OFF to build static libraries" ON )
option(MINIZIP_DEMOS  "Build minizip demos" ON)

include_directories(
    include 
    ${ZLIB_INCLUDE_DIRS}
)

file(
	GLOB_RECURSE
	MINIZIP_SOURCE_FILES
	${MINIZIP_SRC_DIR}/*
)
file(
    GLOB_RECURSE
    MINIZIP_INCLUDE_FILES
    ${MINIZIP_INCLUDE_DIR}/*
)


add_library(
        ${MINIZIP_LIB_NAME}

        ${MINIZIP_SOURCE_FILES}
        ${MINIZIP_INCLUDE_FILES}
    )
    
if(BUILD_SHARED_LIBS)
    set_target_properties(
        ${MINIZIP_LIB_NAME} PROPERTIES DEFINE_SYMBOL  "MINIZIP_EXPORTS" 
    )
else()
    # not yet available :'(
    MESSAGE(FATAL_ERROR "Building static lib is not yet available. "
      "Please select Build shared lib.")
endif()

target_link_libraries( ${MINIZIP_LIB_NAME} ${ZLIB_LIBRARIES} )



install( TARGETS ${MINIZIP_LIB_NAME}
    RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_BUILD_TYPE}/bin
    LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_BUILD_TYPE}/lib
    ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_BUILD_TYPE}/lib
)
install( FILES ${MINIZIP_INCLUDE_FILES} 
    DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_BUILD_TYPE}/include
)


if(MINIZIP_DEMOS)
    set(MINIUNZ_DEMO_SOURCES_FILES
        ${minizip_SOURCE_DIR}/demos/miniunz.c
    )
    set(MINIZIP_DEMO_SOURCES_FILES
        ${minizip_SOURCE_DIR}/demos/minizip.c
    )
    set(MINIUNZ_DEMO_NAME miniunz)
    set(MINIZIP_DEMO_NAME minizip)
    add_executable(${MINIUNZ_DEMO_NAME} ${MINIUNZ_DEMO_SOURCES_FILES})
    target_link_libraries(${MINIUNZ_DEMO_NAME} ${MINIZIP_LIB_NAME})
    
    add_executable(${MINIZIP_DEMO_NAME} ${MINIZIP_DEMO_SOURCES_FILES})
    target_link_libraries(${MINIZIP_DEMO_NAME} ${MINIZIP_LIB_NAME})
    
    install(TARGETS ${MINIUNZ_DEMO_NAME} ${MINIZIP_DEMO_NAME}
        RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_BUILD_TYPE}/demos)
endif()
