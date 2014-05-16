# xcode-build-utilities

xcode-build-utilites provides xcconfig files for using the latest and strict build settings for C, Objective-C and C++ along with a few source files to accomplish trivial tasks.

## source

With the advent of the App Store the advantage of dynamic libraries or frameworks has all but disappeared, leaving apps that use bundled dynamic libraries or frameworks with only a slower boot time. Converting dynamic libraries and frameworks to static libraries can improve boot performance and reduce bundle size, but, without building a binary executable, verifying link dependencies for a given library is very difficult. `source` contains empty source files for each language so a dynamic library can be built directly from a static library.

In Xcode, create a new Dynamic Library target, add the static library as a build dependency, add an Empty source file matching the language used in static library and link the static library, along with any other necessary dependencies. This strategy helps static libraries avoid circular dependencies on each other.

## xcconfig

`common__config__debug.xcconfig` and `common__config__release.xcconfig` are designed to be used as the project level build settings files for the Debug and Release configurations, respectively. `common__target__ios.xcconfig` and `common__target__osx.xcconfig` are designed to be included in a Target's xcconfig file or, less preferably, set as the Target's build settings file in conjunction with build settings specified directly int he Xcode project.

The build settings use the latest language standards and enable a majority of the available warnings, which is intended to encourage the writing of high quality code.
