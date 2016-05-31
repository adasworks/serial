cmake_minimum_required(VERSION 3.2)

project(serial)

# Enable debug postfix for all targets created in this project
set(CMAKE_DEBUG_POSTFIX "_d")

## Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)
if(APPLE)
	# If OSX
	list(APPEND serial_SRCS src/impl/unix.cc)
	list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(${PROJECT_NAME} ${serial_SRCS})
if(APPLE)
	find_library(IOKIT_LIBRARY IOKit)
	find_library(FOUNDATION_LIBRARY Foundation)
	target_link_libraries(${PROJECT_NAME} PRIVATE ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
	target_link_libraries(${PROJECT_NAME} PRIVATE rt pthread)
else()
	target_link_libraries(${PROJECT_NAME} setupapi)
endif()

# Set header search path for build, and install
target_include_directories(${PROJECT_NAME}
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/gen>
  PUBLIC $<INSTALL_INTERFACE:include>
)

# Install library, and generate config module
install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}-targets RUNTIME DESTINATION bin ARCHIVE DESTINATION lib LIBRARY DESTINATION lib)
install(DIRECTORY include/ DESTINATION include PATTERN "impl/*" EXCLUDE)
install(EXPORT ${PROJECT_NAME}-targets NAMESPACE "::" DESTINATION share/cmake/${PROJECT_NAME} FILE ${PROJECT_NAME}-config.cmake)

## Uncomment for example
add_executable(serial_example examples/serial_example.cc)
target_link_libraries(serial_example PRIVATE ${PROJECT_NAME})

## Tests
if(BUILD_TESTING)
    if (NOT CMAKE_CROSSOMPILING)
        enable_testing()
    endif()
    add_subdirectory(tests)
endif()
