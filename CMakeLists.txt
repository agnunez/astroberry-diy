cmake_minimum_required(VERSION 2.4.7)

if(COMMAND cmake_policy)
    cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

PROJECT(astroberry CXX C)
include(ExternalProject)

set (VERSION_MAJOR 1)
set (VERSION_MINOR 0)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules/")
set(BIN_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/bin")
set(CMAKE_CXX_FLAGS "-std=c++0x ${CMAKE_CXX_FLAGS}")


find_package(INDI REQUIRED)

#configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h )

include_directories( ${CMAKE_CURRENT_BINARY_DIR})
include_directories( ${CMAKE_CURRENT_SOURCE_DIR})
include_directories( ${INDI_INCLUDE_DIR})

include_directories(${CMAKE_SOURCE_DIR}/libs/bcm2835-1.52/src)

add_library(bcm2835 STATIC IMPORTED)
set_property(TARGET bcm2835 PROPERTY IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/libs/bcm2835-1.52/src/libbcm2835.a)
add_dependencies(bcm2835 libbcm2835)

################ libbcm2835 ################

ExternalProject_Add(libbcm2835
    URL "http://www.airspayce.com/mikem/bcm2835/bcm2835-1.52.tar.gz"
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/libs/bcm2835-1.52
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/bcm2835-1.52
    CONFIGURE_COMMAND ./configure
    BUILD_COMMAND ${MAKE}
    INSTALL_COMMAND ""
    BUILD_IN_SOURCE 1
)


################ Astroberry Board ################

set(indi_rpibrd_SRCS
        ${CMAKE_CURRENT_SOURCE_DIR}/rpi_brd.cpp
   )

add_executable(indi_rpibrd ${indi_rpibrd_SRCS})
target_link_libraries(indi_rpibrd indidriver bcm2835)
install(TARGETS indi_rpibrd RUNTIME DESTINATION bin )
install(FILES indi_rpibrd.xml DESTINATION ${INDI_DATA_DIR})

################ Astroberry AltIMU ################

set(indi_rpialtimu_SRCS
        ${CMAKE_CURRENT_SOURCE_DIR}/rpi_altimu.cpp
   )

add_executable(indi_rpialtimu ${indi_rpialtimu_SRCS})
target_link_libraries(indi_rpialtimu indidriver RTIMULib)
install(TARGETS indi_rpialtimu RUNTIME DESTINATION bin )
install(FILES indi_rpialtimu.xml DESTINATION ${INDI_DATA_DIR})

################ Astroberry GPS ################

set(indi_rpigps_SRCS
        ${CMAKE_CURRENT_SOURCE_DIR}/rpi_gps.cpp
   )

add_executable(indi_rpigps ${indi_rpigps_SRCS})
target_link_libraries(indi_rpigps indidriver gps nova)
install(TARGETS indi_rpigps RUNTIME DESTINATION bin )
install(FILES indi_rpigps.xml DESTINATION ${INDI_DATA_DIR})

################ Astroberry Focuser ################

set(indi_rpifocus_SRCS
        ${CMAKE_CURRENT_SOURCE_DIR}/rpi_focus.cpp
   )

add_executable(indi_rpifocus ${indi_rpifocus_SRCS})
target_link_libraries(indi_rpifocus indidriver bcm2835)
install(TARGETS indi_rpifocus RUNTIME DESTINATION bin )
install(FILES indi_rpifocus.xml DESTINATION ${INDI_DATA_DIR})

