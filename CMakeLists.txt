cmake_minimum_required(VERSION 3.5)
project(serial)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Find dependencies
find_package(ament_cmake REQUIRED)

# Handle platform-specific libraries
if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
elseif(UNIX AND NOT APPLE)
    # Linux: need rt and pthread
    set(PLATFORM_LIBRARIES rt pthread)
elseif(WIN32)
    set(PLATFORM_LIBRARIES setupapi)
endif()

# Platform-specific source files
if(APPLE)
    set(serial_SRCS
        src/serial.cc
        src/impl/unix.cc
        src/impl/list_ports/list_ports_osx.cc
    )
elseif(UNIX AND NOT APPLE)
    # Linux
    set(serial_SRCS
        src/serial.cc
        src/impl/unix.cc
        src/impl/list_ports/list_ports_linux.cc
    )
elseif(WIN32)
    # Windows
    set(serial_SRCS
        src/serial.cc
        src/impl/win.cc
        src/impl/list_ports/list_ports_win.cc
    )
else()
    # Fallback (should not happen on supported platforms)
    set(serial_SRCS src/serial.cc)
endif()

# Create the library
add_library(${PROJECT_NAME} ${serial_SRCS})

# Include directories
target_include_directories(${PROJECT_NAME}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# Link platform-specific libraries
if(APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX AND NOT APPLE)
    target_link_libraries(${PROJECT_NAME} ${PLATFORM_LIBRARIES})
elseif(WIN32)
    target_link_libraries(${PROJECT_NAME} ${PLATFORM_LIBRARIES})
endif()

# Create example executable
add_executable(serial_example examples/serial_example.cc)
target_link_libraries(serial_example ${PROJECT_NAME})
target_include_directories(serial_example PRIVATE include)

# Install library
install(TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

# Install headers
install(DIRECTORY include/
  DESTINATION include
)

# Install example
install(TARGETS serial_example
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

# Export package
ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_include_directories(include)

# Testing (optional)
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
  
  # If you have tests, uncomment and adapt:
  # add_subdirectory(tests)
endif()

# Suppress unused variable warning from colcon/catkin compatibility
set(CATKIN_INSTALL_INTO_PREFIX_ROOT "" CACHE INTERNAL "")

# Package
ament_package()