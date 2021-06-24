cmake_minimum_required(VERSION 3.15)

project(fmtlog CXX)

if(MSVC)
  add_compile_options(/O2 /std:c++latest)
else()
  set(CMAKE_CXX_STANDARD 17)
  add_compile_options(-Wall -O3 -std=c++17)
  find_package(Threads REQUIRED)
endif()

link_directories(.)

include(FetchContent)
FetchContent_Declare(
  fmt
  GIT_REPOSITORY https://github.com/fmtlib/fmt.git
  GIT_SHALLOW TRUE
  GIT_TAG "8.0.0"
)
if(NOT fmt_POPULATED)
  FetchContent_Populate(fmt)
  include_directories(SYSTEM ${fmt_SOURCE_DIR}/include)
  add_subdirectory(${fmt_SOURCE_DIR} ${fmt_BINARY_DIR})
endif()

if(NOT FMTLOG_STATIC_ONLY)
  add_library(fmtlog-shared SHARED fmtlog.cc)
  target_include_directories(fmtlog-shared SYSTEM PUBLIC ${fmt_SOURCE_DIR}/include)
  if(MSVC)
    target_link_libraries(fmtlog-shared fmt)
  else()
    target_link_libraries(fmtlog-shared fmt-header-only Threads::Threads)
  endif()
  install(TARGETS fmtlog-shared)
endif(NOT FMTLOG_STATIC_ONLY)

if(NOT FMTLOG_SHARED_ONLY)
  add_library(fmtlog-static fmtlog.cc)
  target_include_directories(fmtlog-static SYSTEM PUBLIC ${fmt_SOURCE_DIR}/include)
  if(MSVC)
    target_link_libraries(fmtlog-static fmt)
  else()
    target_link_libraries(fmtlog-static fmt Threads::Threads)
  endif()
  install(TARGETS fmtlog-static)
endif(NOT FMTLOG_SHARED_ONLY)

if(NOT FMTLOG_JUST_LIBRARY)
  add_subdirectory(test)
  if(NOT MSVC)
    add_subdirectory(bench)
  endif(NOT MSVC)
endif(NOT FMTLOG_JUST_LIBRARY)
