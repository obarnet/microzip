cmake_minimum_required(VERSION 2.6)

project(microzip)

# needed packages
find_package(ZLIB REQUIRED)
find_package(Boost COMPONENTS date_time filesystem)

set(MICROZIP_LIB_MAJOR 0)
set(MICROZIP_LIB_MINOR 1)
set(MICROZIP_LIB_RELEASE 0)
set(MICROZIP_LIB_NAME microzip_${MICROZIP_LIB_MAJOR}-${MICROZIP_LIB_MINOR})
set(MICROZIP_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(MICROZIP_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

# to distinguish between debug and release lib
set(CMAKE_DEBUG_POSTFIX "d")


# command line options
option(BUILD_MINIZIP     "Set to OFF to use your minzip version" ON)
option(BUILD_SHARED_LIBS "Set to OFF to build static libraries"  ON )

if(BUILD_MINIZIP)
    set(MINIZIP_CMAKE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/minizip)
    add_subdirectory(${MINIZIP_CMAKE_DIR})
    get_directory_property(MINIZIP_LIB_NAME DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/minizip DEFINITION MINIZIP_LIB_NAME)
    get_directory_property(MINIZIP_INCLUDE_DIR DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/minizip DEFINITION MINIZIP_INCLUDE_DIR)
else()
    set(MINIZIP_INCLUDE_DIR CACHE PATH "Minizip include dirs")
    set(MINIZIP_LIB_NAME    CACHE FILEPATH "Minizip lib")
endif()

link_directories(
    ${ZLIB_LIBRARY_DIRS}
    ${Boost_LIBRARY_DIRS}
)
include_directories(
    include 
    ${Boost_INCLUDE_DIRS}
    ${ZLIB_INCLUDE_DIRS}
    ${MINIZIP_INCLUDE_DIR}
)

file(
	GLOB_RECURSE
	MICROZIP_SOURCE_FILES
	${MICROZIP_SRC_DIR}/*
)
file(
    GLOB_RECURSE
    MICROZIP_INCLUDE_FILES
    ${MICROZIP_SRC_DIR}/*
)


add_library(
        ${MICROZIP_LIB_NAME}

        ${MICROZIP_SOURCE_FILES}
        ${MICROZIP_INCLUDE_FILES}
    )
    
if(BUILD_SHARED_LIBS)
    set_target_properties(
        ${MICROZIP_LIB_NAME} PROPERTIES DEFINE_SYMBOL  "MICROZIP_EXPORTS" 
    )
else()
    # not yet available :'(
    MESSAGE(FATAL_ERROR "Building static lib is not yet available. "
      "Please select Build shared lib.")
endif()

target_link_libraries( ${MICROZIP_LIB_NAME} ${MINIZIP_LIB_NAME} ${Boost_LIBRARIES})


install( TARGETS ${MICROZIP_LIB_NAME}
    RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_BUILD_TYPE}/bin
    LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_BUILD_TYPE}/lib
    ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_BUILD_TYPE}/lib
)
install( FILES ${MICROZIP_INCLUDE_FILES} 
    DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_BUILD_TYPE}/include
)

