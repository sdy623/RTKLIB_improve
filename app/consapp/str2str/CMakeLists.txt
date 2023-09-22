cmake_minimum_required(VERSION 3.16.3)

project(str2str)

set (RTKLIB_SRC ${CMAKE_CURRENT_SOURCE_DIR}/../../../src)
include_directories(${RTKLIB_SRC})

set (SRC
    ${CMAKE_CURRENT_SOURCE_DIR}/str2str.c
    ${RTKLIB_SRC}/stream.c
    ${RTKLIB_SRC}/streamsvr.c
    ${RTKLIB_SRC}/rtkcmn.c
    ${RTKLIB_SRC}/solution.c
    ${RTKLIB_SRC}/sbas.c
    ${RTKLIB_SRC}/geoid.c
    ${RTKLIB_SRC}/rcvraw.c
    ${RTKLIB_SRC}/rcv/novatel.c
    ${RTKLIB_SRC}/rcv/ublox.c
    ${RTKLIB_SRC}/rcv/ss2.c
    ${RTKLIB_SRC}/rcv/crescent.c
    ${RTKLIB_SRC}/rcv/skytraq.c
    ${RTKLIB_SRC}/rcv/javad.c
    ${RTKLIB_SRC}/rcv/nvs.c
    ${RTKLIB_SRC}/rcv/binex.c
    ${RTKLIB_SRC}/rcv/rt17.c
    ${RTKLIB_SRC}/rtcm.c
    ${RTKLIB_SRC}/rtcm2.c
    ${RTKLIB_SRC}/rtcm3.c
    ${RTKLIB_SRC}/rtcm3e.c
    ${RTKLIB_SRC}/preceph.c
    ${RTKLIB_SRC}/rcv/septentrio.c
)

set (TARGET_NAME str2str)

add_executable(${TARGET_NAME} ${SRC})

if(MSVC)
    add_definitions("/W0")
endif()


if(WIN32)
  target_link_libraries(${TARGET_NAME} wsock32 ws2_32 winmm)
endif()

target_compile_definitions(${TARGET_NAME} PRIVATE TRACE ENAGLO ENAQZS ENAGAL ENACMP ENAIRN NFREQ=5)

if(WIN32)
   target_compile_options(${TARGET_NAME} PRIVATE -Wall -O3 -ansi -pedantic -g)
else()
   target_compile_options(${TARGET_NAME} PRIVATE -Wall -O3 -ansi -pedantic -Wno-unused-but-set-variable -g)
endif()

#target_link_libraries(${TARGET_NAME} m rt pthread)

set (DEVICE ttyS1:115200:8:n:1:off)

add_test(NAME test1 COMMAND ${TARGET_NAME} "-in" "serial://$(DEVICE)" "-out" "file://test1.out" "-t" "2")
add_test(NAME test2 COMMAND ${TARGET_NAME} "-in" "serial://$(DEVICE)" "-out" "tcpsvr://:2102" "-c" "../../../data/oem4_raw_10hz.cmd" "-t" "5")
add_test(NAME test3 COMMAND ${TARGET_NAME} "-in" "serial://$(DEVICE)" "-out" "ntrips://:sesam@localhost:8000/BUCU0" "-c" "../../../data/oem4_raw_10hz.cmd" "-t" "5")
add_test(NAME test4 COMMAND ${TARGET_NAME} "-in" "../oem6_20121107.gps#nov" "-out" "oem6_20121107.rtcm3#rtcm3" "-sta" "123" "-msg" "\"1077(10),1087(10),1097(10),1107(10),1117(10),1019,1020\"")
add_test(NAME test5 COMMAND ${TARGET_NAME} "-in" "../oem6_20121107.gps#nov" "-out" "oem6_20121107_gal.rtcm3#rtcm3" "-msg" "\"1097(10)\"")
add_test(NAME test6 COMMAND ${TARGET_NAME} "-in" "serial://ttyS54:115200#nov" "-out" "tcpsvr://:2103#rtcm3" "-c" "../../../data/oem4_raw_10hz.cmd")

install(TARGETS ${TARGET_NAME} DESTINATION /usr/local/bin)