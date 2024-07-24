# AddBoost.CMake

## Find or download Boost from CMake. 

Versions tested: from 1.79.0 upto 1.85.0 .

## Examples

- [x] [Arniiiii/ModernCppStarterExampleBoostCmake](https://github.com/Arniiiii/ModernCppStarterExampleBoostCmake)

## How to use:

1. Add [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake?tab=readme-ov-file#adding-cpm) in your project somehow (or if you know how, use ExternalProject or FetchContent).
2. Download `AddBoost.CMake`:

```cmake
CPMAddPackage(
  NAME AddBoost.CMake
  GIT_TAG main
  GITHUB_REPOSITORY Arniiiii/AddBoost.cmake
)
```
3. Use the `add_boost` function:
```cmake
set(TRY_BOOST_VERSION "1.85.0")
set(BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED "thread")
set(BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED "asio")

add_boost(
  ${TRY_BOOST_VERSION} ${BOOST_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED}
  ${BOOST_NOT_HEADER_ONLY_COMPONENTS_THAT_YOU_NEED} your_target_name boost_install_targets
)
```
4. Add Boost's install target (notice `${boost_install_targets}`):
```cmake
string(TOLOWER ${PROJECT_NAME}/version.h VERSION_HEADER_LOCATION)
string(TOLOWER ${PROJECT_NAME}/export.h EXPORT_HEADER_LOCATION)

set_property(TARGET ${PROJECT_NAME} PROPERTY VERSION ${PROJECT_VERSION})
set_property(TARGET ${PROJECT_NAME} PROPERTY SOVERSION 1)


CPMAddPackage(
  NAME PackageProject.cmake
  VERSION 1.11.2
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
  DEPENDENCIES "${boost_install_targets}"
)

```


## Features:  

 - [x] Find local Boost
 - [x] Download boost if there's with lower version than needed or if `-DCPM_DOWNLOAD_ALL=1` 
 - [x] Links Boost to what target you need by itself
 - [x] If you don't want to download Boost multiple times, set `-DCPM_SOURCE_CACHE=./.cache/cpm`
 - [x] Gives appropriate string for you to add to [`PackageProject.cmake`](https://github.com/TheLartians/PackageProject.cmake)
 - [x] Makes Boost generate appropriate install targets
 - [x] If you download Boost, you can add additional configuring options just by setting them before calling function.
 - [x] Well tested at [Arniiiii/ModernCppStarterExampleBoostCmake](https://github.com/Arniiiii/ModernCppStarterExampleBoostCmake)
 - [x] You can apply your patches to Boost. Define variable `BOOST_ADD_MY_PATCHES` to be a path to folder in which there's  `*.patch` in such layout:
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
