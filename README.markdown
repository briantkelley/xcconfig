# xcode-build-utilities

xcode-build-utilites provides xcconfig files for using the latest and strict build settings for C, C++, Objective-C, and Swift, along with a few source files to accomplish trivial tasks.

## source

Use of static libraries can improve boot performance and reduce bundle size relative to dynamic libraries and frameworks. But, without building an executable image, verifying link dependencies for a static library is difficult. `source` contains empty source files for each language supported by Xcode, which are used to guide the build system to link the necessary language support when producing a dynamic library directly from a static library.

In Xcode, create a new Dynamic Library target, add the static library as a build dependency, add an Empty source file matching the language(s) used in static library, and link the static library and its dependencies. In addition to verifying dependency correctness, this strategy helps prevent circular dependencies in static libraries.

## xcconfig

`common__config__debug.xcconfig` and `common__config__release.xcconfig` are designed to be used as the project level build settings files for the Debug and Release configurations, respectively. `common__target__ios.xcconfig` and `common__target__osx.xcconfig` are designed to be included in a Target's xcconfig file or, less preferably, set as the Target's build settings file in conjunction with build settings specified directly in the Xcode project.

The build settings use the latest language standards and enable a majority of the available warnings, which is intended to encourage the writing of high quality code.

**Note::** If the `source` files are not relevant to your project's configuration, use the `xcconfig` branch to include only the `xcconfig` files.
