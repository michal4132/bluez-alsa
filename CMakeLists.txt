project(bluez-alsa)
cmake_minimum_required(VERSION 3.18)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE Debug)
SET(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

add_compile_definitions(_GNU_SOURCE)
add_compile_definitions(PACKAGE_VERSION="1.0")
add_compile_definitions(PIC)

find_package(Bluez REQUIRED)
find_package(SBC REQUIRED)

find_package(PkgConfig REQUIRED)
pkg_check_modules(deps REQUIRED IMPORTED_TARGET glib-2.0 gio-unix-2.0 libopenaptx spandsp dbus-1 alsa)

file(GLOB SOURCES "src/*.c" "src/shared/a2dp-codecs.c" "src/shared/ffb.c" "src/shared/hex.c" "src/shared/log.c" "src/shared/nv.c" "src/shared/rt.c")

include_directories(
    "src"
    ${BLUEZ_INCLUDE_DIRS}
    ${SBC_INCLUDE_DIRS}
    ${deps_INCLUDE_DIRS}
)

add_executable(bluez-alsa ${SOURCES})
set_target_properties(bluez-alsa PROPERTIES ENABLE_EXPORTS 1)
target_link_libraries(bluez-alsa pthread m ${CMAKE_DL_LIBS} ${THINGS_TO_LINK} ${BLUEZ_LIBRARIES} ${SBC_LIBRARIES} PkgConfig::deps)

# libasound_module_ctl_bluealsa.so
SET(asound_module_ctl_bluealsa_SRCS
    "src/shared/a2dp-codecs.c"
    "src/shared/dbus-client.c"
    "src/asound/bluealsa-ctl.c"
)

add_library(asound_module_ctl_bluealsa MODULE ${asound_module_ctl_bluealsa_SRCS})
target_link_libraries(asound_module_ctl_bluealsa pthread m dl ${CMAKE_DL_LIBS} ${THINGS_TO_LINK} ${BLUEZ_LIBRARIES} ${SBC_LIBRARIES} PkgConfig::deps)

# libasound_module_pcm_bluealsa.so
SET(asound_module_pcm_bluealsa_SRCS
    "src/shared/a2dp-codecs.c"
    "src/shared/dbus-client.c"
    "src/shared/hex.c"
    "src/shared/log.c"
    "src/shared/rt.c"
    "src/asound/bluealsa-pcm.c"
)

add_library(asound_module_pcm_bluealsa MODULE ${asound_module_pcm_bluealsa_SRCS})
target_link_libraries(asound_module_pcm_bluealsa pthread m dl ${CMAKE_DL_LIBS} ${THINGS_TO_LINK} ${BLUEZ_LIBRARIES} ${SBC_LIBRARIES} PkgConfig::deps)


include(GNUInstallDirs)
install(TARGETS asound_module_pcm_bluealsa asound_module_ctl_bluealsa DESTINATION "/lib/${CMAKE_LIBRARY_ARCHITECTURE}/alsa-lib/")
