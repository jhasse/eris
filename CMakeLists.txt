cmake_minimum_required(VERSION 3.19)

# Lua 5.3 with Eris built in, as a static library called lua. Its sources are all in src/, but for
# lua.c and luac.c, the standalone interpreter and compiler, which are left out.
project(eris C)

file(GLOB LUA_SOURCES CONFIGURE_DEPENDS src/*.c)
list(FILTER LUA_SOURCES EXCLUDE REGEX "/luac?\\.c$")
add_library(lua STATIC ${LUA_SOURCES})
target_include_directories(lua PUBLIC src)
if(NOT MSVC)
	# eris.c typedefs its own bool, which C23 (GCC's default since 15) has made a keyword. And no
	# fused multiply-adds, so that scripts do their sums the same on every platform.
	target_compile_options(lua PRIVATE -std=gnu99 -ffp-contract=off)
endif()
if(UNIX)
	target_compile_definitions(lua PRIVATE LUA_USE_POSIX) # e.g. mkstemp instead of tmpnam
endif()
