cmake_minimum_required(VERSION 3.12)

# Prohibit in-source build
set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
if ("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")
	message(FATAL_ERROR "In-source build prohibited.")
endif()

project(LinearFold
	VERSION "1.0.0"
	DESCRIPTION "Liquid column chromatography simulator"
	HOMEPAGE_URL "https://github.com/modsim/CADET"
	LANGUAGES CXX C)

# Hide symbols by default
set(CMAKE_C_VISIBILITY_PRESET hidden)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)

# Enable folders for IDEs
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Option that allows users to build release or debug version
if (NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel" FORCE)
	message(STATUS "Build type: ${CMAKE_BUILD_TYPE} (default)")
endif()

include(FeatureSummary)

option(ENABLE_OPENMP "Enable OpenMP if compiler supports it" ON)
add_feature_info(ENABLE_OPENMP ENABLE_OPENMP "Enable OpenMP if compiler supports it")

option(ENABLE_IPO "Enable interprocedural optimization if compiler supports it" ON)
add_feature_info(ENABLE_IPO ENABLE_IPO "Enable interprocedural optimization if compiler supports it")

set(IPO_AVAILABLE OFF)
if (ENABLE_IPO)
	include(CheckIPOSupported)
	check_ipo_supported(RESULT IPO_RESULT OUTPUT IPO_OUT LANGUAGES CXX)
	if (IPO_RESULT)
		set(IPO_AVAILABLE ON)
		set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
	else()
		message(WARNING "IPO is not supported: ${IPO_OUT}")
	endif()
	unset(IPO_RESULT)
	unset(IPO_OUT)
endif()


add_library(LinearFold::CompileOptions INTERFACE IMPORTED)
target_compile_features(LinearFold::CompileOptions INTERFACE cxx_std_11)
set(CMAKE_CXX_EXTENSIONS OFF)

if (WIN32)
	target_compile_definitions(LinearFold::CompileOptions INTERFACE NOMINMAX)
endif()

if (ENABLE_OPENMP)
	find_package(OpenMP)
	if(OpenMP_CXX_FOUND)
		target_link_libraries(LinearFold::CompileOptions INTERFACE OpenMP::OpenMP_CXX)
	endif()
endif()


add_executable(linearfold_c ${CMAKE_SOURCE_DIR}/src/LinearFold.cpp)
target_compile_definitions(linearfold_c PRIVATE is_cube_pruning)
target_compile_definitions(linearfold_c PRIVATE is_candidate_list)
target_include_directories(linearfold_c PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/src/Utils)
target_link_libraries(linearfold_c PRIVATE LinearFold::CompileOptions)

add_executable(linearfold_v ${CMAKE_SOURCE_DIR}/src/LinearFold.cpp)
target_compile_definitions(linearfold_v PRIVATE is_cube_pruning)
target_compile_definitions(linearfold_v PRIVATE is_candidate_list)
target_compile_definitions(linearfold_v PRIVATE lv)
target_include_directories(linearfold_v PRIVATE ${CMAKE_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/src/Utils)
target_link_libraries(linearfold_v PRIVATE LinearFold::CompileOptions)


# ---------------------------------------------------
#   Setup installation
# ---------------------------------------------------

install(CODE "MESSAGE(\"\nInstall LinearFold\n\")")
install(TARGETS linearfold_c RUNTIME)
install(TARGETS linearfold_v RUNTIME)



# Packaging support
include(CPack)
set(CPACK_PACKAGE_VENDOR "LinearFold")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY ${PROJECT_DESCRIPTION})
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_STRIP_FILES ON)

set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/README.md")

set(CPACK_SOURCE_IGNORE_FILES
	/.git
	/\\\\.DS_Store
)

feature_summary(WHAT ALL)

message("------------------------------- Summary -------------------------------")
message("C++ compiler name: ${CMAKE_CXX_COMPILER_ID} at ${CMAKE_CXX_COMPILER}")
message("Build type: ${CMAKE_BUILD_TYPE}")
message("Source dir: ${CMAKE_SOURCE_DIR}")
message("Binary dir: ${CMAKE_BINARY_DIR}")
message("Install dir: ${CMAKE_INSTALL_PREFIX}")
message("C Flags: ${CMAKE_C_FLAGS}")
message("C++ Flags: ${CMAKE_CXX_FLAGS}")
message("IPO enabled: ${IPO_AVAILABLE}")
if (ENABLE_OPENMP)
	message("OpenMP: ${OpenMP_CXX_FOUND}")
	message("  Flags: ${OpenMP_CXX_FLAGS}")
endif()
