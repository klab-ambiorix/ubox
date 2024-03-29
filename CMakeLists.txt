cmake_minimum_required(VERSION 2.6)

PROJECT(ubox C)
ADD_DEFINITIONS(-Os -ggdb -Wall -Werror --std=gnu99 -Wmissing-declarations)

SET(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")

IF(APPLE)
  INCLUDE_DIRECTORIES(/opt/local/include)
  LINK_DIRECTORIES(/opt/local/lib)
ENDIF()

find_library(json NAMES json-c json)

IF(DEBUG)
  ADD_DEFINITIONS(-DDEBUG -g3)
ENDIF()

INCLUDE (CheckSymbolExists)
CHECK_SYMBOL_EXISTS(SYS_getrandom syscall.h getrandom)

IF(getrandom)
  ADD_EXECUTABLE(getrandom getrandom.c)
  INSTALL(TARGETS getrandom
          RUNTIME DESTINATION bin
  )
ELSE()
  message( FATAL_ERROR "Kernel too old, missing SYS_getrandom system call")
ENDIF()

ADD_EXECUTABLE(kmodloader kmodloader.c)
TARGET_LINK_LIBRARIES(kmodloader ubox)

INSTALL(TARGETS kmodloader
	RUNTIME DESTINATION sbin
)

ADD_EXECUTABLE(lsbloader lsbloader.c)
TARGET_LINK_LIBRARIES(lsbloader ubox ubus)

INSTALL(TARGETS lsbloader
	RUNTIME DESTINATION sbin
)

ADD_LIBRARY(validate SHARED validate/validate.c)
INSTALL(TARGETS validate
	LIBRARY DESTINATION lib
)

FIND_PATH(uci_include_dir uci.h)
INCLUDE_DIRECTORIES(${uci_include_dir})

ADD_EXECUTABLE(validate_data validate/cli.c)
TARGET_LINK_LIBRARIES(validate_data ubox uci validate)
INSTALL(TARGETS validate_data
	RUNTIME DESTINATION sbin
)

ADD_EXECUTABLE(logd log/logd.c log/syslog.c)
TARGET_LINK_LIBRARIES(logd ubox ubus udebug)
INSTALL(TARGETS logd
	RUNTIME DESTINATION sbin
)

ADD_EXECUTABLE(logread log/logread.c)
TARGET_LINK_LIBRARIES(logread ubox ubus ${json} blobmsg_json)
INSTALL(TARGETS logread
	RUNTIME DESTINATION sbin
)
