# Copyright 2025 Arniiiii lg3dx6fd@gmail.com MIT license

cmake_minimum_required(VERSION 3.14...3.22)

# ---- Project ----

set(AddBoost.cmake_VERSION 3.7.3)

option(${PROJECT_NAME}_INSTALL "Install the script at local system?" NO)

include(CMakePackageConfigHelpers)

include(GNUInstallDirs)

configure_package_config_file(
  ${CMAKE_CURRENT_LIST_DIR}/cmake/AddBoost.cmake/AddBoost.cmakeConfig.cmake.in
  ${CMAKE_CURRENT_LIST_DIR}/cmake/AddBoost.cmake/AddBoost.cmakeConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_DATADIR}/cmake/AddBoost.cmake
)

write_basic_package_version_file(${CMAKE_CURRENT_LIST_DIR}/cmake/AddBoost.cmake/AddBoost.cmakeConfigVersion.cmake VERSION ${AddBoost.cmake_VERSION} COMPATIBILITY AnyNewerVersion ARCH_INDEPENDENT)


include(${CMAKE_CURRENT_LIST_DIR}/cmake/AddBoost.cmake/AddBoost.cmakeConfig.cmake)

if(${AddBoost.cmake_INSTALL})
  include(GNUInstallDirs)

  set(CMAKEDIR
      ${CMAKE_INSTALL_DATADIR}/cmake/AddBoost.cmake/
      CACHE
        PATH
        "Path to a folder to which install the script AddBoostConfig.cmake . By default: ${CMAKE_INSTALL_DATADIR}/cmake/AddBoost.cmake/"
  )

  install(FILES "${CMAKE_CURRENT_LIST_DIR}/cmake/AddBoost.cmake/AddBoost.cmakeConfig.cmake"
          DESTINATION "${CMAKEDIR}"
  )

  install(FILES "${CMAKE_CURRENT_LIST_DIR}/cmake/AddBoost.cmake/AddBoost.cmakeConfigVersion.cmake"
          DESTINATION "${CMAKEDIR}"
  )
  message(STATUS "Use `find_package(AddBoost.cmake ${AddBoost.cmake_VERSION})` to use it")

endif()
