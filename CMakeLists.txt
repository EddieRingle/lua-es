project(lua52 C)

cmake_minimum_required(VERSION 2.8.12)

#
# Options
#

option(LUA_BUILD_BINARIES "Build Lua binaries" ON)

#
# Set up some variables
#

set(LUA_ROOT_DIR "${PROJECT_SOURCE_DIR}")
set(LUA_CONTRIB_DIR "${LUA_ROOT_DIR}/contrib")
set(LUA_SCRIPTS_DIR "${LUA_ROOT_DIR}/scripts")
set(LUA_SOURCE_DIR "${LUA_ROOT_DIR}/src")
set(LUA_OUT_DIR "${LUA_ROOT_DIR}/out")

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE "Release")
endif()

list(APPEND CMAKE_MODULE_PATH "${LUA_SCRIPTS_DIR}/cmake/modules")

#
# Modules/macros
#

include(GroupByFolder)
include(MergeStaticLibs)
include(TargetArch)

#
# Target Arch/OS detection
#

if(ANDROID)
    set(LUA_TARGET_OS "android")
elseif(DREAMCAST)
    set(LUA_TARGET_OS "dreamcast")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	set(LUA_TARGET_OS "darwin")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	set(LUA_TARGET_OS "linux")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	set(LUA_TARGET_OS "windows")
else()
	set(LUA_TARGET_OS "${CMAKE_SYSTEM_NAME}")
endif()

target_architecture(LUA_TARGET_ARCH)

set(LUA_TARGET_OUT_DIR "${LUA_OUT_DIR}/${LUA_TARGET_OS}_${LUA_TARGET_ARCH}")
set(LUA_TARGET_CONTRIB_OUT_DIR "${LUA_TARGET_OUT_DIR}/contrib")

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
	set (CMAKE_INSTALL_PREFIX "${LUA_TARGET_OUT_DIR}" CACHE PATH "Default install path" FORCE)
endif()

#
# Dependencies
#

include("${LUA_CONTRIB_DIR}/CMakeLists.txt")

#
# Lua
#

set(lua52_SRC_CORE
	${LUA_SOURCE_DIR}/lapi.c
	${LUA_SOURCE_DIR}/lcode.c
	${LUA_SOURCE_DIR}/lctype.c
	${LUA_SOURCE_DIR}/ldebug.c
	${LUA_SOURCE_DIR}/ldo.c
	${LUA_SOURCE_DIR}/ldump.c
	${LUA_SOURCE_DIR}/lfunc.c
	${LUA_SOURCE_DIR}/lgc.c
	${LUA_SOURCE_DIR}/llex.c
	${LUA_SOURCE_DIR}/lmem.c
	${LUA_SOURCE_DIR}/lobject.c
	${LUA_SOURCE_DIR}/lopcodes.c
	${LUA_SOURCE_DIR}/lparser.c
	${LUA_SOURCE_DIR}/lstate.c
	${LUA_SOURCE_DIR}/lstring.c
	${LUA_SOURCE_DIR}/ltable.c
	${LUA_SOURCE_DIR}/ltm.c
	${LUA_SOURCE_DIR}/lundump.c
	${LUA_SOURCE_DIR}/lvm.c
	${LUA_SOURCE_DIR}/lzio.c
)

set(lua52_SRC_LIB
	${LUA_SOURCE_DIR}/lauxlib.c
	${LUA_SOURCE_DIR}/lbaselib.c
	${LUA_SOURCE_DIR}/lbitlib.c
	${LUA_SOURCE_DIR}/lcorolib.c
	${LUA_SOURCE_DIR}/ldblib.c
	${LUA_SOURCE_DIR}/liolib.c
	${LUA_SOURCE_DIR}/lmathlib.c
	${LUA_SOURCE_DIR}/loslib.c
	${LUA_SOURCE_DIR}/lstrlib.c
	${LUA_SOURCE_DIR}/ltablib.c
	${LUA_SOURCE_DIR}/linit.c
	${LUA_SOURCE_DIR}/loadlib.c
)

set(lua52_SRC_LUA
	${LUA_SOURCE_DIR}/lua.c
)

set(lua52_SRC_LUAC
	${LUA_SOURCE_DIR}/luac.c
)

set(lua52_INC_CORE
    ${LUA_SOURCE_DIR}/lapi.h
    ${LUA_SOURCE_DIR}/lcode.h
    ${LUA_SOURCE_DIR}/lctype.h
    ${LUA_SOURCE_DIR}/ldebug.h
    ${LUA_SOURCE_DIR}/ldo.h
    ${LUA_SOURCE_DIR}/lfunc.h
    ${LUA_SOURCE_DIR}/lgc.h
    ${LUA_SOURCE_DIR}/llex.h
    ${LUA_SOURCE_DIR}/llimits.h
    ${LUA_SOURCE_DIR}/lmem.h
    ${LUA_SOURCE_DIR}/lobject.h
    ${LUA_SOURCE_DIR}/lopcodes.h
    ${LUA_SOURCE_DIR}/lparser.h
    ${LUA_SOURCE_DIR}/lstate.h
    ${LUA_SOURCE_DIR}/lstring.h
    ${LUA_SOURCE_DIR}/ltable.h
    ${LUA_SOURCE_DIR}/ltm.h
    ${LUA_SOURCE_DIR}/lundump.h
    ${LUA_SOURCE_DIR}/lvm.h
    ${LUA_SOURCE_DIR}/lzio.h
)

set(lua52_INC_LUA
    ${LUA_SOURCE_DIR}/lua.h
    ${LUA_SOURCE_DIR}/luaconf.h
    ${LUA_SOURCE_DIR}/lualib.h
    ${LUA_SOURCE_DIR}/lauxlib.h
    ${LUA_SOURCE_DIR}/lua.hpp
)

if(NOT MINGW AND NOT MSVC)
  add_definitions(-DLUA_USE_POSIX -DLUA_USE_DLOPEN)
endif()

include_directories(SYSTEM ${LUA_SOURCE_DIR})
include_directories(SYSTEM "${LUA_TARGET_CONTRIB_OUT_DIR}/include")

link_directories("${LUA_TARGET_CONTRIB_OUT_DIR}/lib")

add_library(lua52 STATIC ${lua52_SRC_CORE} ${lua52_SRC_LIB})
add_dependencies(lua52 turfs)

install(TARGETS lua52 EXPORT lua52 ARCHIVE DESTINATION lib)
install(FILES ${lua52_INC_LUA} DESTINATION include/lua5.2)

set(lua52_LIBRARIES lua52 turfs)

if(LUA_BUILD_BINARIES)
    add_executable(lua ${lua52_SRC_LUA})
    target_link_libraries(lua ${lua52_LIBRARIES})
    install(TARGETS lua RUNTIME DESTINATION bin)
    add_executable(luac ${lua52_SRC_LUAC})
    target_link_libraries(luac ${lua52_LIBRARIES})
    install(TARGETS luac RUNTIME DESTINATION bin)
endif()

group_by_folder(${LUA_ROOT_DIR} "${lua52_SRC_CORE};${lua52_SRC_LIB};${lua52_SRC_LUA};${lua52_SRC_LUAC}")
group_by_folder(${LUA_ROOT_DIR} "${lua52_INC_CORE};${lua52_INC_LUA}")
