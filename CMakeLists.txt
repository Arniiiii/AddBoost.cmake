# Copyright 2025 Arniiiii lg3dx6fd@gmail.com MIT license

cmake_minimum_required(VERSION 3.14...3.22)

# ---- Project ----

# Note: update this to your new project's name and version
project(
  AddBoost.cmake
  VERSION 3.5
  LANGUAGES CXX
)

option(${PROJECT_NAME}_INSTALL "Install the script at local system?" NO)

configure_file(
  ${CMAKE_CURRENT_LIST_DIR}/cmake/AddBoost.cmake.in ${CMAKE_CURRENT_LIST_DIR}/cmake/AddBoost.cmake
  @ONLY
)

include(${CMAKE_CURRENT_LIST_DIR}/cmake/AddBoost.cmake)

if(${${PROJECT_NAME}_INSTALL})
  include(GNUInstallDirs)

  set(CMAKEDIR
      ${CMAKE_INSTALL_DATADIR}/cmake
      CACHE
        PATH
        "Path to a folder to which install the script ArniiiiiAddBoost.cmake . By default: ${CMAKE_INSTALL_DATADIR}/cmake/"
  )

  install(FILES "${CMAKE_CURRENT_LIST_DIR}/cmake/AddBoost.cmake" DESTINATION "${CMAKEDIR}")

endif()
