# minimum version of CMake is 3.3
cmake_minimum_required(VERSION 3.3 FATAL_ERROR)

# define project
project( libecpint
         VERSION 0.9
         LANGUAGES C CXX)

# we need at least C++11
set (CMAKE_CXX_STANDARD 11)

# code generator variables
set (LIBECPINT_MAX_L "5" CACHE STRING "Maximum angular momentum")
set (LIBECPINT_MAX_UNROL "2" CACHE STRING "Maximum L for unrolling")

# configure the config header to pass the above variables to the program
configure_file (
	      "${PROJECT_SOURCE_DIR}/include/libecpint/config.hpp.in"
	      "${PROJECT_BINARY_DIR}/include/libecpint/config.hpp"
)
include_directories(${PROJECT_BINARY_DIR}/include/libecpint)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include/libecpint)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/external/Faddeeva)

add_subdirectory(src)

enable_testing()
add_subdirectory(tests)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# <<< Export Config to allow other projects to automagically import this >>>
set(PN ${PROJECT_NAME})
# GNUInstallDirs "DATADIR" wrong here; CMake search path wants "share".
set(CMAKECONFIG_INSTALL_DIR "share/cmake/${PN}")
configure_package_config_file(cmake/${PN}Config.cmake.in
                              "${CMAKE_CURRENT_BINARY_DIR}/${PN}Config.cmake"
                              INSTALL_DESTINATION ${CMAKECONFIG_INSTALL_DIR})
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${PN}ConfigVersion.cmake
                                 VERSION ${${PN}_VERSION}
                                 COMPATIBILITY AnyNewerVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PN}Config.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/${PN}ConfigVersion.cmake
              cmake/FindTargetLAPACK.cmake
        DESTINATION ${CMAKECONFIG_INSTALL_DIR})
install(EXPORT "${PN}Targets"
        NAMESPACE "${PN}::"
        DESTINATION ${CMAKECONFIG_INSTALL_DIR})

if(NOT TARGET ${PN}::ecpint)
    include("${CMAKE_CURRENT_LIST_DIR}/${PN}Targets.cmake")
endif()
install(TARGETS ecpint
        EXPORT "${PN}Targets"
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

install (DIRECTORY include/libecpint DESTINATION include)
install (FILES include/libecpint.hpp DESTINATION include)
install (FILES "${PROJECT_BINARY_DIR}/include/libecpint/config.hpp" DESTINATION include/libecpint)
