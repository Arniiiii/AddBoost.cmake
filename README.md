# AddBoost.CMake

## Find or download Boost from CMake. 

Versions tested: from 1.79.0 upto 1.88.0.beta1 .

## Examples

- [x] [Arniiiii/ModernCppStarterExampleBoostCmake](https://github.com/Arniiiii/ModernCppStarterExampleBoostCmake)

## How to use:

1. Add [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake?tab=readme-ov-file#adding-cpm) in your project.
2. Download `AddBoost.CMake` via CPM: (or install AddBoost.cmake system-wide, check [Features section](#features) of the document)

```cmake
CPMAddPackage(
  NAME AddBoost.CMake
  GIT_TAG 3.7.2
  GITHUB_REPOSITORY Arniiiii/AddBoost.cmake
)
```
3. Use the `add_boost` macro. Notice *NOT* wrapping variables like TRY_BOOST_VERSION and so on in `"${}"` when sending arguments to the macro
```cmake
set(TRY_BOOST_VERSION "1.88.0.beta1")
set(BOOST_MY_OPTIONS "BOOST_ENABLE_PYTHON ON;")
set(BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED "thread;system")
set(BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED "asio;beast;uuid;container;cobalt")

add_boost(
  TRY_BOOST_VERSION BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED
  BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED your_target_name your_target_name2 your_target_name...
)
```
4. Add Boost's install target (notice `${ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS}`): (Assumed that your target is named `${PROJECT_NAME}`)
```cmake
string(TOLOWER ${PROJECT_NAME}/version.h VERSION_HEADER_LOCATION)
string(TOLOWER ${PROJECT_NAME}/export.h EXPORT_HEADER_LOCATION)

set_property(TARGET ${PROJECT_NAME} PROPERTY VERSION ${PROJECT_VERSION})
set_property(TARGET ${PROJECT_NAME} PROPERTY SOVERSION ${PROJECT_VERSION})


CPMAddPackage(
  NAME PackageProject.cmake
  VERSION 1.12.0
  GITHUB_REPOSITORY "TheLartians/PackageProject.cmake"
)

packageProject(
  NAME ${PROJECT_NAME}
  VERSION ${PROJECT_VERSION}
  NAMESPACE ${PROJECT_NAME}
  BINARY_DIR ${PROJECT_BINARY_DIR}
  INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include
  INCLUDE_DESTINATION include/${PROJECT_NAME}
  VERSION_HEADER "${VERSION_HEADER_LOCATION}" 
  EXPORT_HEADER "${EXPORT_HEADER_LOCATION}"
  COMPATIBILITY "AnyNewerVersion" DISABLE_VERSION_SUFFIX ON
  DEPENDENCIES "${ADDBOOSTCMAKE_PACKAGEPROJECT_INSTALL_TARGETS}"
)

```

## Features

 - [x] Finds local Boost ( you can set `-DCPM_USE_LOCAL_PACKAGES=1` for "looking for local Boost. if failed: download" mode. Set `-DCPM_LOCAL_PACKAGES_ONLY=1` to only look for installed Boost and emit error if failed to find)
 - [x] Downloads boost, if `-DCPM_USE_LOCAL_PACKAGES=1` and there's installed Boost with lower version than needed or if installed Boost doesn't exist on the system or if `-DCPM_DOWNLOAD_ALL=1` is set.
 - [x] Links Boost to what targets you need by itself
 - [x] If you don't want to download Boost multiple times (when you deleted `build` folder and going to reconfigure), set `-DCPM_SOURCE_CACHE=./.cache/cpm` or something like.
 - [x] Gives appropriate string for you to add to [`PackageProject.cmake`](https://github.com/TheLartians/PackageProject.cmake)
 - [x] Makes Boost generate appropriate install targets
 - [x] If you download Boost, you can add additional configuring options just by either setting them before calling ~~function~~ macro `add_boost(...)` or by setting `BOOST_MY_OPTIONS` to something like `"OPTION value;OPTION2 value;"` for example `BOOST_ENABLE_PYTHON ON;` .
 - [x] Little bit tested at [Arniiiii/ModernCppStarterExampleBoostCmake](https://github.com/Arniiiii/ModernCppStarterExampleBoostCmake) and at some of my projects.
 - [x] If you have your own Boost directory, set `BOOST_USE_MY_BOOST_DIRECTORY` to be the path with your Boost before calling the ~~function~~ macro `add_boost(...)`.
 - [x] If you want, you can link Boost libs yourself, since the code is macro, not a function: copy and paste some last parts of the main CMakeLists.txt of the project and adjust for yourself.
 - [x] You can link Boost libs automagically to multiple targets just by adding them to the end of the `add_boost(...)` macro.
 - [x] You can use `ADDBOOSTCMAKE_LINK_TYPE` to override default behaviour of linking: if target is INTERFACE, use INTERFACE, if else: PUBLIC
 - [x] Correctly handles attempts to get beta versions like `boost-1.88.0.beta1`, since `find_package` considers such version as incorrect, but we understand what you mean ^_^
 - [x] You can install the AddBoost.cmake on your SYSTEM !!! Use `-DAddBoost.cmake_INSTALL=ON` command line option and enjoy! And yes, it supports versioning!
 - [ ] Internally, this uses my fork of CPM with better logging handling. Waiting until PRs for CPM are going to be reviewed...
 - [ ] Cannot check existence of header only components of installed Boost, since it relies on functionality of `find_package(Boost ${VERSION} ${COMPONENTS})`. Therefore, set version of Boost you need so that it have the library.
 - [x] You can install it as a utility module. Use `AddBoost.cmake_INSTALL` option at configuring and use `find_package(AddBoost.cmake)`
 - [x] You can apply your patches to Boost. Before calling the ~~function~~ macro `add_boost(...)`, define variable `BOOST_ADD_MY_PATCHES` to be a path to folder in which there's  `*.patch` in such layout:
```
patches/
└── boost
    ├── 1.80.0
    │   ├── a.patch
    │   └── b.patch
    ├── 1.80.0.beta1
    │   └── c.patch
    ├── 1.81.0
    │   └── d.patch
    ├── 1.81.0.beta1
    │   └── d.patch
    ├── 1.82.0
    │   └── e.patch
    ├── 1.83.0
    │   └── f.patch
    ├── 1.84.0
    │   └── g.patch
    ├── patch_for_all_versions1.patch
    └── i_dont_care_to_what_boost_version_itll_apply.patch
```

Where in folder with specific version will be patches, that will apply only to this version, if it's specified.

## License
MIT
