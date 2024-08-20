cmake_minimum_required(VERSION 3.14...3.22)

# ---- Project ----

# Note: update this to your new project's name and version
project(
  AddBoost.cmake
  VERSION 3.1
  LANGUAGES CXX
)

function(SUBDIRLIST result_var curdir)
  file(
    GLOB children
    RELATIVE ${curdir}
    ${curdir}/*)
  set(result "")
  foreach(child ${children})
    if(IS_DIRECTORY ${curdir}/${child})
      list(APPEND result ${child})
    endif()
  endforeach()
  set(${result_var}
      ${result}
      PARENT_SCOPE)
endfunction()

macro(add_boost TRY_BOOST_VERSION BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED
      BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED)
  message(
    DEBUG
    "add_boost called with next arguments:
    \$\{\$\{TRY_BOOST_VERSION\}\} : ${${TRY_BOOST_VERSION}}
    \$\{\$\{BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED\}\} : ${${BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}}
    \$\{\$\{BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED\}\} : ${${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}}
  ")
  set(targets_to_link_to "${ARGN}")
  message(DEBUG "targets_to_link_to: \"${targets_to_link_to}\"")
  set(BOOST_INCLUDE_LIBRARIES
      "${${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}};${${BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}}"
  )

  set(BOOST_URL
      "https://github.com/boostorg/boost/releases/download/boost-${${TRY_BOOST_VERSION}}/boost-${${TRY_BOOST_VERSION}}-cmake.tar.xz"
  )
  set(boost_is_old FALSE)
  if(${${TRY_BOOST_VERSION}} STRLESS "1.85.0")
    set(BOOST_URL
        "https://github.com/boostorg/boost/releases/download/boohttps://annas-archive.org/search?index=&page=1&q=c%2B%2B+concurrency+in+action&sort=st-${${TRY_BOOST_VERSION}}/boost-${${TRY_BOOST_VERSION}}.tar.xz"
    )
  endif()
  if(${${TRY_BOOST_VERSION}} STRLESS "1.81.0.beta1")
    set(boost_is_old TRUE)
  endif()

  set(patches_for_boost "")

  file(GLOB global_patches_for_boost CONFIGURE_DEPENDS
       "${CMAKE_CURRENT_SOURCE_DIR}/patches/boost/*.patch")
  list(APPEND patches_for_boost ${global_patches_for_boost})
  if(DEFINED BOOST_ADD_MY_PATCHES)
    file(GLOB global_patches_for_boost CONFIGURE_DEPENDS
         "${BOOST_ADD_MY_PATCHES}/patches/boost/*.patch")
    list(APPEND patches_for_boost ${global_patches_for_boost})

  endif()

  subdirlist(subdirs_boost_patches "${CMAKE_CURRENT_LIST_DIR}/patches/boost")
  if(subdirs_boost_patches)
    foreach(subdir ${subdirs_boost_patches})
      if(${subdir} STREQUAL ${${TRY_BOOST_VERSION}})
        file(GLOB_RECURSE patches_with_max_applicable_version_up_to
             CONFIGURE_DEPENDS
             "${CMAKE_CURRENT_SOURCE_DIR}/patches/boost/${subdir}/*.patch")
        list(APPEND patches_for_boost
             ${patches_with_max_applicable_version_up_to})
      endif()
    endforeach()
  endif()
  if(DEFINED BOOST_ADD_MY_PATCHES)
    subdirlist(subdirs_boost_patches "${BOOST_ADD_MY_PATCHES}/patches/boost")
    if(subdirs_boost_patches)
      foreach(subdir ${subdirs_boost_patches})
        if(${subdir} STREQUAL ${${TRY_BOOST_VERSION}})
          file(GLOB_RECURSE patches_with_max_applicable_version_up_to
               CONFIGURE_DEPENDS
               "${CMAKE_CURRENT_SOURCE_DIR}/patches/boost/${subdir}/*.patch")
          list(APPEND patches_for_boost
               ${patches_with_max_applicable_version_up_to})
        endif()
      endforeach()
    endif()
  endif()

  string(REPLACE ";" "\;"
                 ESCAPED_BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED
                 "${${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}}")
  message(
    DEBUG
    "\$\{ESCAPED_BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED\} : ${ESCAPED_BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}"
  )

  if(DEFINED BOOST_USE_MY_BOOST_DIRECTORY)
    cpmaddpackage(
      NAME
      Boost
      VERSION
      ${${TRY_BOOST_VERSION}}
      SOURCE_DIR
      ${BOOST_USE_MY_BOOST_DIRECTORY}
      PATCHES
      ${patches_for_boost}
      FIND_PACKAGE_ARGUMENTS
      "COMPONENTS ${ESCAPED_BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}"
      OPTIONS
      "BOOST_ENABLE_CMAKE ON;BOOST_SKIP_INSTALL_RULES OFF;${BOOST_MY_OPTIONS}")
  elseif(patches_for_boost AND NOT boost_is_old) # is it 1.80.0 < x < 1.84.0
    cpmaddpackage(
      NAME
      Boost
      VERSION
      ${${TRY_BOOST_VERSION}}
      URL
      ${BOOST_URL}
      PATCHES
      ${patches_for_boost}
      FIND_PACKAGE_ARGUMENTS
      "COMPONENTS ${ESCAPED_BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}"
      OPTIONS
      "BOOST_ENABLE_CMAKE ON;BOOST_SKIP_INSTALL_RULES OFF;${BOOST_MY_OPTIONS}")
  elseif(NOT boost_is_old) # is it 1.85.0+ ?
    cpmaddpackage(
      NAME
      Boost
      VERSION
      ${${TRY_BOOST_VERSION}}
      URL
      ${BOOST_URL}
      FIND_PACKAGE_ARGUMENTS
      "COMPONENTS ${ESCAPED_BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}"
      OPTIONS
      "BOOST_ENABLE_CMAKE ON;BOOST_SKIP_INSTALL_RULES OFF;${BOOST_MY_OPTIONS}")
  else() # is it <1.80.0 ?
    cpmaddpackage(
      NAME
      Boost
      VERSION
      ${${TRY_BOOST_VERSION}}
      GIT_REPOSITORY
      "https://github.com/boostorg/boost"
      GIT_TAG
      "boost-${${TRY_BOOST_VERSION}}"
      PATCHES
      ${patches_for_boost}
      FIND_PACKAGE_ARGUMENTS
      "COMPONENTS ${ESCAPED_BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}"
      OPTIONS
      "BOOST_ENABLE_CMAKE ON;BOOST_SKIP_INSTALL_RULES OFF;${BOOST_MY_OPTIONS}")
  endif()

  # set(IS_BOOST_LOCAL OFF) endif()

  # ====================== link and install part =======================

  # Link dependencies
  foreach(target IN LISTS targets_to_link_to)
    message(DEBUG "Boost_ADDED is defined : ${Boost_ADDED}")
    if(NOT DEFINED ADDBOOSTCMAKE_LINK_TYPE)
      get_target_property(type ${target} TYPE)
      if("${type}" STREQUAL "INTERFACE_LIBRARY")
        set(ADDBOOSTCMAKE_LINK_TYPE "INTERFACE")
      else()
        set(ADDBOOSTCMAKE_LINK_TYPE "PUBLIC")
      endif()
    endif()
    if(Boost_ADDED STREQUAL "")
      target_link_libraries(${target} ${ADDBOOSTCMAKE_LINK_TYPE} Boost::boost)
    else()
      foreach(a_lib ${${BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}})
        target_link_libraries(${target} ${ADDBOOSTCMAKE_LINK_TYPE}
                              Boost::${a_lib})
      endforeach()
    endif()

    foreach(a_lib ${${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}})
      target_link_libraries(${target} ${ADDBOOSTCMAKE_LINK_TYPE}
                            Boost::${a_lib})
    endforeach()
  endforeach()
  if(Boost_ADDED STREQUAL "")
    set(ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS
        "Boost ${${TRY_BOOST_VERSION}} REQUIRED ${${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}}"
    )
  else()
    set(ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS "")
    set(BOOST_ALL_DEPENDENCIES
        "${${BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}};${${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}}"
    )
    foreach(dep ${BOOST_ALL_DEPENDENCIES})
      string(APPEND ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS
             "boost_${dep} ${${TRY_BOOST_VERSION}};")
    endforeach()
    string(LENGTH "${ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS}"
                  ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS_LENGTH)
    math(EXPR ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS_LENGTH
         "${ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS_LENGTH}-1"
         OUTPUT_FORMAT DECIMAL)
    string(SUBSTRING "${ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS}" 0
                     ${ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS_LENGTH}
                     ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS)
  endif()

  message(
    DEBUG
    "ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS: ${ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS}"
  )

endmacro()
