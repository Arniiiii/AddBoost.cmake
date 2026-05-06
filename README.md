## Important piece of advice

While this works, I would recommend using a real package manager like Conan, Gentoo's or Nix's ones, or vcpkg in a worse case. 
You should not use CPM or FetchContent. Why? Because these two are not package managers. CPM, while has letters PM that stands for "package manager", is not a package manager. It is a wrapper of FetchContent with better interface. And FetchContent should not be used as primary way of getting dependencies. Why? Because it uses rule "first declaration of dependencies is the way we'll do it" and it's impossible to override this totally incorrect behaviour without insane overengineering.

So, yeah, this project somehow works, but please, use a package manager instead, and not CPM, FetchContent or Hunter.

# AddBoost.cmake

## Find or download Boost from CMake. 

Versions tested: from 1.79.0 upto 1.88.0.beta1 .

## Examples

- [x] [Arniiiii/ModernCppStarterExampleBoostCmake](https://github.com/Arniiiii/ModernCppStarterExampleBoostCmake)

## How to use:

1. Add [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake?tab=readme-ov-file#adding-cpm) in your project.
2. Download `AddBoost.cmake` via CPM: (or install AddBoost.cmake system-wide, check [Features section](#features) of the document)

```cmake
set(AddBoost.cmake_VERSION 3.7.3)
CPMAddPackage(
  NAME AddBoost.cmake
  VERSION "${AddBoost.cmake_VERSION}"
  URL "https://github.com/Arniiiii/AddBoost.cmake/archive/refs/tags/${AddBoost.cmake_VERSION}.tar.gz"
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

 - [x] Finds locally installed Boost or downloads
   - If `-DCPM_LOCAL_PACKAGES_ONLY=1`, then only `find_package(Boost REQUIRED ...)` is used internally in CPM.
   - If `-DCPM_USE_LOCAL_PACKAGES=1`, then try `find_package(Boost ...)`. if failed, the macro downloads CMake version of a Boost's tarball of specified version from Boost's GitHub Release page.
   - If `-DCPM_DOWNLOAD_ONLY=1`, the macro only attempts to download.
 - [x] Links Boost to what targets you need by itself
 - [x] If you don't want to download Boost multiple times (when you deleted `build` folder and going to reconfigure), set `CPM_SOURCE_CACHE` to any folder e.g. `./.cache/cpm` .
 - [x] Gives appropriate string for you to add to [`PackageProject.cmake`](https://github.com/TheLartians/PackageProject.cmake)
 - [x] Makes Boost generate appropriate install targets
 - [x] If you download Boost, you can add additional configuring options just by either setting them before calling ~~function~~ macro `add_boost(...)` or by setting `BOOST_MY_OPTIONS` to something like `"OPTION value;OPTION2 value;"` for example `BOOST_ENABLE_PYTHON ON;` .
 - [x] Little bit tested at [Arniiiii/ModernCppStarterExampleBoostCmake](https://github.com/Arniiiii/ModernCppStarterExampleBoostCmake) and at some of my projects.
 - [x] If you have your own Boost directory, set `BOOST_USE_MY_BOOST_DIRECTORY` to be the path with your Boost before calling the ~~function~~ macro `add_boost(...)`.
 - [x] If you want, you can link Boost libs yourself, since the code is macro, not a function: copy and paste some last parts of the `cmake/AddBoost.cmake/AddBoost.cmakeConfig.cmake.in` and adjust for your needs.
 - [x] You can link Boost libs automagically to *multiple* targets just by adding them to the end of the `add_boost(...)` macro.
 - [x] You can use `ADDBOOSTCMAKE_LINK_TYPE` to override default behaviour of linking: if target is `INTERFACE`, use `INTERFACE`, if else: `PUBLIC`
 - [x] Correctly handles attempts to get beta versions like `boost-1.88.0.beta1`, since `find_package` considers such version as incorrect, but we understand what you mean ^_^
 - [x] You can install the AddBoost.cmake on your SYSTEM !!! Use `-DAddBoost.cmake_INSTALL=ON` command line option at configuring and install as a usual CMake package i.e. `cmake -S . -B ./build && cmake --build ./build && sudo cmake install ./build` . And yes, it supports versioning: `find_package(AddBoost.cmake 3.7.3)`
 - [ ] Internally, this uses my fork of CPM with a little bit better logging handling. Waiting until PRs for CPM are going to be reviewed seems futile at this point...
 - [ ] The macro does not check existence of header only components of installed Boost, since it relies on functionality of `find_package(Boost ${VERSION} ${COMPONENTS})`. Therefore, set minimal version of Boost you need so that it would have desired header-only library.
 - [ ] (CPM's problem) Almost impossible to get a reason why it failed to use an installed Boost if `-DCPM_USE_LOCAL_PACKAGES=1`. Temporary solution: `-DCPN_LOCAL_PACKAGES_ONLY=1` will show the reason, but it has a problem too: it will make all CPM to only find packages, not add. But it works, trust me.
 - [ ] (CPM's problem) To use patches or any version of Boost before 1.85.0, consider specifing any folder for caching downloaded sources via `-DCPM_SOURCE_CACHE=/path/to/a/folder/for/downloaded/sources/` . On Linux, you may use `~/.cache/cpm` (per user) , `./.cache/cpm` (per work folder) or whatever else.
 - [x] You can apply your patches to Boost. Before calling the macro `add_boost(...)`, define variable `BOOST_ADD_MY_PATCHES` to be a path to folder in which there's  `*.patch` in such layout:
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
