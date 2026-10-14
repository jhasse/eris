cmake_minimum_required(VERSION 3.21)

# Lua 5.3 with Eris built in, as a static library called lua. Its sources are all in src/, but for
# lua.c and luac.c, the standalone interpreter and compiler, which are left out.
project(eris C)

file(GLOB LUA_SOURCES CONFIGURE_DEPENDS src/*.c)
list(FILTER LUA_SOURCES EXCLUDE REGEX "/luac?\\.c$")
add_library(lua STATIC ${LUA_SOURCES})
target_include_directories(lua PUBLIC src)
target_compile_features(lua PRIVATE c_std_23) # for its bool keyword, which eris.c uses
if(NOT MSVC)
	# No fused multiply-adds, so that scripts do their sums the same on every platform.
	target_compile_options(lua PRIVATE -ffp-contract=off)
endif()
if(UNIX)
	target_compile_definitions(lua PRIVATE LUA_USE_POSIX) # e.g. mkstemp instead of tmpnam
	target_link_libraries(lua PUBLIC m)
endif()

include(CTest)
if(BUILD_TESTING)
	foreach(test IN ITEMS persist unpersist)
		add_executable(${test} test/${test}.c)
		target_link_libraries(${test} PRIVATE lua)
		add_test(NAME ${test} COMMAND ${test} "${CMAKE_CURRENT_SOURCE_DIR}/test/${test}.lua"
		                              "${CMAKE_CURRENT_BINARY_DIR}/test.eris")
	endforeach()
	set_tests_properties(persist PROPERTIES FIXTURES_SETUP eris_data)
	set_tests_properties(unpersist PROPERTIES FIXTURES_REQUIRED eris_data)
endif()
