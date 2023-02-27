# xcconfig

Xcode configuration settings files to enable the latest C, C++, Objective-C, and Swift best practices.

## Overview

This repository provides a central definition of Xcode configuration settings files for best practices, with updates for each release as necessary. These settings use the latest language standards, enable most warnings, and enable all static analysis checks to support writing and maintaining high-quality code.

Configuration settings files have a few advantages compared to specifying settings directly in an Xcode project file:

* **Changes are easier to review.** Xcode project files can be challenging to read and can become very large. Extracting configuration settings into dedicated files reduces the cogitative overhead required to understand the change.
* **Build settings are reusable.** Maintaining configuration settings consistency across multiple targets and Xcode projects is error-prone and tedious. Configuration settings files refactor settings into reusable components, like other well-designed code, to create a single source of truth.
* **It's easier to maintain.** Code search results with well-scoped configuration settings files are usually easier to understand than multiple matches throughout an Xcode project file. Employing tools that automate code changes is less complex for configuration settings files when compared to Xcode's arcane property list format.

For more information about Xcode configuration settings files, check out the WWDC 2021 session [Explore advanced project configuration in Xcode](https://developer.apple.com/videos/play/wwdc2021/10210/).

### SDK Versions

The `SDKROOT` build setting specifies the path of the SDK to use for the build. As a convenience, Xcode has always accepted an SDK's *canonical name* (e.g., `iphoneos12.0`) as shorthand and a machine-independent way to specify the SDK path. In Xcode 6 for iOS and Xcode 8 for macOS, Xcode added support for simply specifying an SDK's base name (e.g., `iphoneos`) as a convenience to eliminate a paper cut encountered when updating Xcode versions. (tvOS and watchOS have always supported this convenience.)

In my opinion, however, this convenience is a disservice to the long-term maintainability of a code base because source history no longer defines the SDK used at a particular commit. This information can be vital when bisecting a regression because Apple implements backward compatibility by checking the SDK version used to build an app; simply linking against a new version can cause observable changes in an app's runtime behavior. So, the configuration settings files in this repository specify the SDK version to assist with root-causing regressions due to SDK version changes.

## Naming Convention

The configuration settings files in the [`xcconfig`](xcconfig) directory use a <tt>*dimension*\_*value*.xcconfig</tt> naming convention with all letters in lowercase.

`config_` files specify settings for a particular **build configuration**, typically *Debug* or *Release*. As a special case, [`config_all.xcconfig`](xcconfig/config_all.xcconfig) contains configuration settings used in all dimensions and values. (The rationale for this approach is twofold: Xcode requires a build configuration value for each build, and this dimension is more general than the others.)

`target_` files specify a **target destination**'s configuration settings, like *iOS* or *macOS*.

A logical product (e.g., a To Do app) typically acts as a configuration settings file dimension where each Xcode build target is a value (e.g., *app*, *unit tests*, *library*, etc.).

## How to Use

Let's assume your repository has an Xcode project for a To Do app and has the following directory structure:

<pre>
&lt;root&gt;
|-- ToDo/
    |-- ToDo.xcodeproj/
</pre>

First, add this repository's [`xcconfig`](xcconfig) directory as a subtree to your repository. I prefer using a subtree in this scenario because copying the configuration settings files into the repository eliminates the dependency on this repository's stability, enables modifying the configuration settings files in-tree, and simplifies bisecting and reviewing source history.

```sh
git remote add --fetch xcconfig-upstream https://github.com/briantkelley/xcconfig.git
git checkout xcconfig-upstream/main
git subtree split --prefix=xcconfig --branch xcconfig-split
git checkout main # the repo's primary development branch
git subtree add --prefix=xcconfig --squash xcconfig-split
git branch -D xcconfig-split
```

Next, create a configuration settings file for your app target, for example, `todo_app.xcconfig`. Then, use `#include` to incorporate the build settings file for its target destination.

```
#include "../../xcconfig/target_ios.xcconfig"

// Packaging
INFOPLIST_FILE = ToDo/Info.plist
PRODUCT_BUNDLE_IDENTIFIER = com.example.todo
PRODUCT_NAME = To Do

// Signing
CODE_SIGN_ENTITLEMENTS = ToDo/ToDo.entitlements
CODE_SIGN_IDENTITY = Apple Development
CODE_SIGN_STYLE = Automatic
DEVELOPMENT_TEAM = 5FH2HO38V3
```

After completing the above steps, the repository's directory structure should look like this:

<pre>
&lt;root&gt;
|-- xcconfig/
    |-- &lt;this repo's xcconfig files&gt;
|-- ToDo/
    |-- ToDo.xcodeproj/
    |-- xcconfig/
        |-- todo_app.xcconfig
</pre>

Finally, to configure the Xcode project to use the new configuration settings files:

1. Select the *ToDo* Xcode project in the Project Navigator.
2. Switch to the *Info* tab in the Editor.
3. Select the *ToDo* Xcode project in the Editor's left column in the *Project* section at the top.
4. Expand the *Debug* configuration in the *Configurations* section.
5. In the *ToDo* Xcode project row, select the `config_debug.xcconfig` file from the popup in the *Based on Configuration File* column.
6. Expand the *ToDo* Xcode project row from step 5.
7. In the *ToDo* target row, select the `todo_app.xcconfig` file from the popup in the *Based on Configuration File* column.
8. Repeat steps 4-7 for the *Release* build configuration.
9. Switch to the *Build Settings* tab in the Editor.
10. Select *All* and *Levels* in the view settings bar located directly under the tab bar in the Editor.
11. Delete any build settings set in the Xcode project level. To preserve a setting, select it in the Editor, copy its value (via the Edit, Copy menu item or ⌘C), and paste it into your `.xcconfig` file.
12. Select the *ToDo* app target in the Editor's left column in the *Targets* section under the *Project* section.
13. Repeat steps 9-11 for build settings set in the app target level.

### Updating the Subtree

When it's time to update Xcode versions, use the following steps to merge the latest changes from this repository with the `xcconfig` subtree. This process similar to the previous steps used to create the subtree, with the primary difference being the use of `git subtree merge` in place of `git subtree add`.

```sh
git fetch xcconfig-upstream
git checkout xcconfig-upstream/main
git subtree split --prefix=xcconfig --branch xcconfig-split
git checkout main # the repo's primary development branch
git subtree merge --prefix=xcconfig --squash xcconfig-split
git branch -D xcconfig-split
```
