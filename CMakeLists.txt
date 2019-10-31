cmake_minimum_required(VERSION 3.0)
project(resid)

set(SRC_VER 0.16)
set(MAJOR_VER 5)
set(MINOR_VER 0)
set(PATCH_VER 0)

option(BUILD_STATIC "Build a static library instead of dynamic" OFF)

if (APPLE)
  option(BUILD_FRAMEWORK "Build a Mac OS X framework instead of a shared library" OFF)
  set(CMAKE_OSX_ARCHITECTURES "i386;x86_64")
endif ()

file(GLOB SOURCES *.cc)
set(API_HEADERS envelope.h extfilt.h filter.h pot.h sid.h siddefs.h spline.h voice.h wave.h)

if (BUILD_STATIC AND NOT BUILD_FRAMEWORK)
  add_library(${PROJECT_NAME} STATIC ${SOURCES} ${API_HEADERS})
else()
  add_library(${PROJECT_NAME} SHARED ${SOURCES} ${API_HEADERS})

  if(WIN32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE RESID_DLL=1 RESID_EXPORTS=1)
  endif()
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES
  VERSION ${SRC_VER}
  SOVERSION ${MAJOR_VER}
  CXX_STANDARD 11)

target_include_directories(${PROJECT_NAME} PRIVATE
  ${CMAKE_BINARY_DIR})

if (APPLE AND BUILD_FRAMEWORK)
  set_target_properties(${PROJECT_NAME} PROPERTIES
    OUTPUT_NAME ${PROJECT_NAME}
    FRAMEWORK TRUE
    FRAMEWORK_VERSION C
    MACOSX_FRAMEWORK_IDENTIFIER net.zimmers)
  set_source_files_properties(${API_HEADERS} PROPERTIES
    MACOSX_PACKAGE_LOCATION Headers)
  set(CMAKE_OSX_DEPLOYMENT_TARGET "10.9")
endif ()

target_compile_definitions(${PROJECT_NAME} PRIVATE VERSION="${MAJOR_VER}.${MINOR_VER}.${PATCH_VER}")

install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  FRAMEWORK DESTINATION "/Library/Frameworks"
  PUBLIC_HEADER DESTINATION include/${PROJECT_NAME})

if (NOT APPLE OR NOT BUILD_FRAMEWORK)
  install(FILES ${API_HEADERS} DESTINATION include/${PROJECT_NAME})
endif ()
