cmake_minimum_required(VERSION 3.14...3.22)

# ---- Project ----

# Note: update this to your new project's name and version
project(
  AddBoost.cmake
  VERSION 1.1
  LANGUAGES CXX
)

function(SUBDIRLIST result_var curdir)
  file(
    GLOB children
    RELATIVE ${curdir}
    ${curdir}/*
  )
  set(result "")
  foreach(child ${children})
    if(IS_DIRECTORY ${curdir}/${child})
      list(APPEND result ${child})
    endif()
  endforeach()
  set(${result_var}
      ${result}
      PARENT_SCOPE
  )
endfunction()

function(add_boost TRY_BOOST_VERSION BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED
         BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED target_to_link_to
         return_var_name_boost_install_targets
)
  set(IS_BOOST_LOCAL OFF)
  if(${CPM_LOCAL_PACKAGES_ONLY})
    message(STATUS "Trying to find Boost with version at least ${TRY_BOOST_VERSION}")
    find_package(
      Boost ${TRY_BOOST_VERSION} REQUIRED
      COMPONENTS ${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}
    )
    set(IS_BOOST_LOCAL ON)
  elseif(${CPM_USE_LOCAL_PACKAGES} OR NOT ${CPM_DOWNLOAD_ALL})
    message(STATUS "Trying to use local Boost with version at least ${TRY_BOOST_VERSION}")
    find_package(
      Boost ${TRY_BOOST_VERSION} COMPONENTS ${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}
    )
    if(${BOOST_FOUND})
      set(IS_BOOST_LOCAL ON)
      message(DEBUG "boost include dir: ${Boost_INCLUDE_DIRS}")
    endif()
  endif()

  if(NOT (${BOOST_FOUND}) OR (NOT DEFINED BOOST_FOUND))
    message(STATUS "Trying to download Boost ${TRY_BOOST_VERSION}")

    set(BOOST_INCLUDE_LIBRARIES
        "${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED};${BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}"
    )

    set(BOOST_URL
        "https://github.com/boostorg/boost/releases/download/boost-${TRY_BOOST_VERSION}/boost-${TRY_BOOST_VERSION}-cmake.tar.xz"
    )
    set(boost_is_old FALSE)
    if(${TRY_BOOST_VERSION} STRLESS "1.85.0")
      set(BOOST_URL
          "https://github.com/boostorg/boost/releases/download/boohttps://annas-archive.org/search?index=&page=1&q=c%2B%2B+concurrency+in+action&sort=st-${TRY_BOOST_VERSION}/boost-${TRY_BOOST_VERSION}.tar.xz"
      )
    endif()
    if(${TRY_BOOST_VERSION} STRLESS "1.81.0.beta1")
      set(boost_is_old TRUE)
    endif()

    set(patches_for_boost "")

    file(GLOB global_patches_for_boost CONFIGURE_DEPENDS
         "${CMAKE_CURRENT_SOURCE_DIR}/patches/boost/*.patch"
    )
    list(APPEND patches_for_boost ${global_patches_for_boost})
    if(DEFINED BOOST_ADD_MY_PATCHES)
      file(GLOB global_patches_for_boost CONFIGURE_DEPENDS
           "${BOOST_ADD_MY_PATCHES}/patches/boost/*.patch"
      )
      list(APPEND patches_for_boost ${global_patches_for_boost})

    endif()

    subdirlist(subdirs_boost_patches "${CMAKE_CURRENT_LIST_DIR}/patches/boost")
    if(subdirs_boost_patches)
      foreach(subdir ${subdirs_boost_patches})
        if(${subdir} STREQUAL ${TRY_BOOST_VERSION})
          file(GLOB_RECURSE patches_with_max_applicable_version_up_to CONFIGURE_DEPENDS
               "${CMAKE_CURRENT_SOURCE_DIR}/patches/boost/${subdir}/*.patch"
          )
          list(APPEND patches_for_boost ${patches_with_max_applicable_version_up_to})
        endif()
      endforeach()
    endif()
    if(DEFINED BOOST_ADD_MY_PATCHES)
      subdirlist(subdirs_boost_patches "${BOOST_ADD_MY_PATCHES}/patches/boost")
      if(subdirs_boost_patches)
        foreach(subdir ${subdirs_boost_patches})
          if(${subdir} STREQUAL ${TRY_BOOST_VERSION})
            file(GLOB_RECURSE patches_with_max_applicable_version_up_to CONFIGURE_DEPENDS
                 "${CMAKE_CURRENT_SOURCE_DIR}/patches/boost/${subdir}/*.patch"
            )
            list(APPEND patches_for_boost ${patches_with_max_applicable_version_up_to})
          endif()
        endforeach()
      endif()
    endif()

    if(patches_for_boost AND NOT boost_is_old)
      message(
        STATUS
          "Downloading boost release from github in form of a .tar.xz tarball... If there's nothing for too long, set at CMake's configuring -DFETCHCONTENT_QUIET=OFF to see download progress"
      )
      CPMAddPackage(
        NAME Boost
        VERSION ${TRY_BOOST_VERSION}
        URL ${BOOST_URL} PATCHES ${patches_for_boost}
        OPTIONS "BOOST_ENABLE_CMAKE ON" "BOOST_SKIP_INSTALL_RULES OFF"
      )
    elseif(NOT boost_is_old)
      message(
        STATUS
          "Downloading boost release from github in form of a .tar.xz tarball... If there's nothing for too long, set at CMake's configuring -DFETCHCONTENT_QUIET=OFF to see download progress"
      )
      CPMAddPackage(
        NAME Boost
        VERSION ${TRY_BOOST_VERSION}
        URL ${BOOST_URL}
        OPTIONS "BOOST_SKIP_INSTALL_RULES OFF"
      )
    else()
      message(
        STATUS
          "Since there's no release at github for such old version, cloning boost from github with GIT_TAG \"boost_${TRY_BOOST_VERSION}\"... If there's nothing for too long, set at CMake's configuring -DFETCHCONTENT_QUIET=OFF to see download progress"
      )
      CPMAddPackage(
        NAME Boost
        VERSION ${TRY_BOOST_VERSION}
        GIT_REPOSITORY "https://github.com/boostorg/boost"
        GIT_TAG "boost-${TRY_BOOST_VERSION}" PATCHES ${patches_for_boost}
        OPTIONS "BOOST_ENABLE_CMAKE ON" "BOOST_SKIP_INSTALL_RULES OFF"
      )

    endif()

    set(IS_BOOST_LOCAL OFF)
  endif()

  # ====================== link and install part =======================

  # Link dependencies
  if(${IS_BOOST_LOCAL})
    target_link_libraries(${PROJECT_NAME} PUBLIC Boost::boost)
  else()
    foreach(a_lib ${BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED})
      target_link_libraries(${PROJECT_NAME} PUBLIC Boost::${a_lib})
    endforeach()
  endif()

  foreach(a_lib ${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED})
    target_link_libraries(${PROJECT_NAME} PUBLIC Boost::${a_lib})
  endforeach()

  if(${IS_BOOST_LOCAL})
    set(OUR_BOOST_INSTALL_TARGETS
        "Boost ${TRY_BOOST_VERSION} REQUIRED ${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}"
    )
  else()
    set(BOOST_DOWNLOADED_INSTALL_TARGETS
        ""
        CACHE STRING "hello_doc..."
    )
    set(BOOST_ALL_DEPENDENCIES
        "${BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED};${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}"
    )
    foreach(dep ${BOOST_ALL_DEPENDENCIES})
      string(APPEND BOOST_DOWNLOADED_INSTALL_TARGETS "boost_${dep} ${TRY_BOOST_VERSION};")
    endforeach()
    string(LENGTH "${BOOST_DOWNLOADED_INSTALL_TARGETS}" BOOST_DOWNLOADED_INSTALL_TARGETS_LENGTH)
    math(EXPR BOOST_DOWNLOADED_INSTALL_TARGETS_LENGTH "${BOOST_DOWNLOADED_INSTALL_TARGETS_LENGTH}-1"
         OUTPUT_FORMAT DECIMAL
    )
    string(SUBSTRING "${BOOST_DOWNLOADED_INSTALL_TARGETS}" 0
                     ${BOOST_DOWNLOADED_INSTALL_TARGETS_LENGTH} BOOST_DOWNLOADED_INSTALL_TARGETS
    )
    message(
      DEBUG
      "BOOST_DOWNLOADED_INSTALL_TARGETS ${BOOST_DOWNLOADED_INSTALL_TARGETS_LENGTH}: ${BOOST_DOWNLOADED_INSTALL_TARGETS}"
    )
    set(OUR_BOOST_INSTALL_TARGETS ${BOOST_DOWNLOADED_INSTALL_TARGETS})
  endif()

  message(DEBUG "OUR_BOOST_INSTALL_TARGETS: ${OUR_BOOST_INSTALL_TARGETS}")

  set(${return_var_name_boost_install_targets}
      "${OUR_BOOST_INSTALL_TARGETS}"
      PARENT_SCOPE
  )

endfunction()
