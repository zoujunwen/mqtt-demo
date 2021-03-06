#*******************************************************************************
#  Copyright (c) 2015 logi.cals GmbH
# 
#  All rights reserved. This program and the accompanying materials
#  are made available under the terms of the Eclipse Public License v1.0
#  and Eclipse Distribution License v1.0 which accompany this distribution. 
# 
#  The Eclipse Public License is available at 
#     http://www.eclipse.org/legal/epl-v10.html
#  and the Eclipse Distribution License is available at 
#    http://www.eclipse.org/org/documents/edl-v10.php.
# 
#  Contributors:
#     Rainer Poisel - initial version
#*******************************************************************************/

# Note: on OS X you should install XCode and the associated command-line tools

## compilation/linkage settings
INCLUDE_DIRECTORIES(
    .
    ${CMAKE_BINARY_DIR}
    )

CONFIGURE_FILE(VersionInfo.h.in
    ${CMAKE_BINARY_DIR}/VersionInfo.h
    @ONLY
    )

SET(common_src
    MQTTProtocolClient.c
    Clients.c
    utf-8.c
    StackTrace.c
    MQTTPacket.c
    MQTTPacketOut.c
    Messages.c
    Tree.c
    Socket.c
    Log.c
    MQTTPersistence.c
    Thread.c
    MQTTProtocolOut.c
    MQTTPersistenceDefault.c
    SocketBuffer.c
    Heap.c
    LinkedList.c
    )

IF (CMAKE_SYSTEM_NAME MATCHES "Windows")
    SET(LIBS_SYSTEM ws2_32)
ELSEIF (CMAKE_SYSTEM_NAME MATCHES "Linux")
    SET(LIBS_SYSTEM dl)
ENDIF()

## common compilation for libpaho-mqtt3c and libpaho-mqtt3a
ADD_LIBRARY(common_obj OBJECT ${common_src})
SET_PROPERTY(TARGET common_obj PROPERTY POSITION_INDEPENDENT_CODE ON)

ADD_EXECUTABLE(MQTTVersion MQTTVersion.c)

IF (PAHO_BUILD_SHARED)
    ADD_LIBRARY(paho-mqtt3c SHARED $<TARGET_OBJECTS:common_obj> MQTTClient.c)
    ADD_LIBRARY(paho-mqtt3a SHARED $<TARGET_OBJECTS:common_obj> MQTTAsync.c)
    TARGET_LINK_LIBRARIES(paho-mqtt3c pthread ${LIBS_SYSTEM})
    TARGET_LINK_LIBRARIES(paho-mqtt3a pthread ${LIBS_SYSTEM})
ENDIF()

IF (PAHO_BUILD_STATIC)
    ADD_LIBRARY(paho-mqtt3c-static STATIC $<TARGET_OBJECTS:common_obj> MQTTClient.c)
    ADD_LIBRARY(paho-mqtt3a-static STATIC $<TARGET_OBJECTS:common_obj> MQTTAsync.c)
    TARGET_LINK_LIBRARIES(paho-mqtt3c-static pthread ${LIBS_SYSTEM})
    TARGET_LINK_LIBRARIES(paho-mqtt3a-static pthread ${LIBS_SYSTEM})
ENDIF()

TARGET_LINK_LIBRARIES(MQTTVersion paho-mqtt3a paho-mqtt3c ${LIBS_SYSTEM})
SET_TARGET_PROPERTIES(
    paho-mqtt3c paho-mqtt3a PROPERTIES
    VERSION ${CLIENT_VERSION}
    SOVERSION ${PAHO_VERSION_MAJOR})
INSTALL(TARGETS paho-mqtt3c paho-mqtt3a MQTTVersion
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib)
INSTALL(FILES MQTTAsync.h MQTTClient.h MQTTClientPersistence.h
    DESTINATION include)

IF (PAHO_WITH_SSL)
    SET(OPENSSL_LIB_SEARCH_PATH "" CACHE PATH "Directory containing OpenSSL libraries")
    SET(OPENSSL_INC_SEARCH_PATH "" CACHE PATH "Directory containing OpenSSL includes")
    SET(OPENSSL_LIBRARIES ssl crypto)
    LINK_DIRECTORIES(
        ${OPENSSL_LIB_SEARCH_PATH}
        )
    INCLUDE_DIRECTORIES(
        ${OPENSSL_INC_SEARCH_PATH}
        )
    ## common compilation for libpaho-mqtt3cs and libpaho-mqtt3as
    ## Note: SSL libraries must be recompiled due ifdefs
    ADD_LIBRARY(common_ssl_obj OBJECT ${common_src})
    SET_PROPERTY(TARGET common_ssl_obj PROPERTY POSITION_INDEPENDENT_CODE ON)

    IF (PAHO_BUILD_SHARED)
        ADD_LIBRARY(paho-mqtt3cs SHARED $<TARGET_OBJECTS:common_ssl_obj> MQTTClient.c SSLSocket.c)
        ADD_LIBRARY(paho-mqtt3as SHARED $<TARGET_OBJECTS:common_ssl_obj> MQTTAsync.c SSLSocket.c)
        TARGET_LINK_LIBRARIES(paho-mqtt3cs pthread ${OPENSSL_LIBRARIES} ${LIBS_SYSTEM})
        TARGET_LINK_LIBRARIES(paho-mqtt3as pthread ${OPENSSL_LIBRARIES} ${LIBS_SYSTEM})
    ENDIF()

    IF (PAHO_BUILD_STATIC)
        ADD_LIBRARY(paho-mqtt3cs-static STATIC $<TARGET_OBJECTS:common_ssl_obj> MQTTClient.c SSLSocket.c)
        ADD_LIBRARY(paho-mqtt3as-static STATIC $<TARGET_OBJECTS:common_ssl_obj> MQTTAsync.c SSLSocket.c)
        TARGET_LINK_LIBRARIES(paho-mqtt3cs-static pthread ${OPENSSL_LIBRARIES} ${LIBS_SYSTEM})
        TARGET_LINK_LIBRARIES(paho-mqtt3as-static pthread ${OPENSSL_LIBRARIES} ${LIBS_SYSTEM})
    ENDIF()

    SET_TARGET_PROPERTIES(
        paho-mqtt3cs paho-mqtt3as PROPERTIES
        VERSION ${CLIENT_VERSION}
        SOVERSION ${PAHO_VERSION_MAJOR})
    INSTALL(TARGETS paho-mqtt3cs
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib)
    INSTALL(TARGETS paho-mqtt3as
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib)
ENDIF()

