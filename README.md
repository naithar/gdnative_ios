# iOS example for GDNative

This repo contains a *starter* XCode project and SCons configuration to build GDNative library for iOS platform.  
XCode project allows to build **.framework** or **.xcframework** (using specific script).
SCons configuration builds a static (**.a**) or dynamic (**.dylib**) iOS library.  
All resulting products can be used for creating GDNative library in Godot project. (`master` [#40050](https://github.com/godotengine/godot/pull/40050) and current `3.2` branches [#39996](https://github.com/godotengine/godot/pull/39996))

I would recomend using either **.xcframework** or **.a** library.  
While **.a** require static **gdnative** library linked to iOS project and fake dynamic table, it would result in smallest application size.  
**.xcframework** provides multi-architecture framework product. This safes you from manually striping **simulator** architecture before App Store upload (like **.framework** would). It also results in smaller application size then **.dylib**, but bigger than **.a** library.

# Initial Setup

## Compiling GDNative

To generate **libgdnative.fat.a** perform inside **godot-cpp** directory:

```
scons platform=ios ios_arch=armv7 target=<target> -j4  
scons platform=ios ios_arch=arm64 target=<target> -j4  
scons platform=ios ios_arch=x86_64 target=<target> -j4  

lipo -create bin/libgodot-cpp.ios.<target>.arm64.a bin/libgodot-cpp.ios.<target>.armv7.a bin/libgodot-cpp.ios.<target>.x86_64.a -output bin/libgdnative.fat.a
```

For more detailed information about building **gdnative** binary and using resulting dynamic or static library go to [Godot Docs](https://docs.godotengine.org/en/stable/tutorials/plugins/gdnative/gdnative-cpp-example.html)

# Working with XCode

Change **Bundle Identifier** to some unique value. Each framework should have different bundle identifiers.  
Change **Product Name** parameter in **Build Settings** tab to the name you would want to use.  
Add source files to project.

# Building XCFramework

## Requirements

Building **XCFramework** might require additionaly building **gdnative** library for **i386** architecture, which is not present at **SConstruct** file by default.  
Simple workaround would be changing **Architecture** from **$(ARCHS_STANDARD)** to **armv7 x86_64 arm64** or any other value that works for you.  
This also could be worked around by adding additional parameters and conditions for **ios_arch** in SConstruct file in **godot-cpp** directory.   


To create a **.xcframework** run:
```
bash xcframework_build.sh <project_name> <scheme_name> <product_name>
```
Where **<product_name>** is the name you've set in **Product Name** at **Build Settings** tab.

Example:
```
bash xcframework_build.sh gdnative_ios.xcodeproj library gdexample
```

# Building Static library

## Requirements

Using static (**.a**) library will require adding and linking **libgdnative.fat.a** that was previously built to exported iOS app. This can be simplified by moving **libgdnative.fat.a** to godot project folder and adding it as dependency in **.gdnlib** file.

To build **.a** library run:
```
scons platform=ios arch=<arch> target_name=<library_name>
```

# Building Dynamic library

To build **.dylib** library run:
```
scons platform=ios arch=<arch> target_name=<library_name> mode=dynamic
```
