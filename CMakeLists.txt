cmake_minimum_required(VERSION 3.8)
project(serial)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)

# Platform-specific libraries and sources
if(APPLE)
  find_library(IOKIT_LIBRARY IOKit)
  find_library(FOUNDATION_LIBRARY Foundation)
  set(serial_SRCS
    src/serial.cc
    src/impl/unix.cc
    src/impl/list_ports/list_ports_osx.cc
  )
  set(PLATFORM_LIBRARIES ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX AND NOT APPLE)
  set(serial_SRCS
    src/serial.cc
    src/impl/unix.cc
    src/impl/list_ports/list_ports_linux.cc
  )
  set(PLATFORM_LIBRARIES rt pthread)
elseif(WIN32)
  set(serial_SRCS
    src/serial.cc
    src/impl/win.cc
    src/impl/list_ports/list_ports_win.cc
  )
  set(PLATFORM_LIBRARIES setupapi)
else()
  set(serial_SRCS src/serial.cc)
  set(PLATFORM_LIBRARIES "")
endif()

add_library(${PROJECT_NAME} ${serial_SRCS})
target_include_directories(${PROJECT_NAME}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
target_link_libraries(${PROJECT_NAME} ${PLATFORM_LIBRARIES})

add_executable(serial_example examples/serial_example.cc)
target_link_libraries(serial_example ${PROJECT_NAME})
target_include_directories(serial_example PRIVATE include)

install(TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

install(DIRECTORY include/
  DESTINATION include
)

install(TARGETS serial_example
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_include_directories(include)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
  # add_subdirectory(tests) # Uncomment if you have tests
endif()

ament_package()