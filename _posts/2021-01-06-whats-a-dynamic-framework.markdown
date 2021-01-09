---
layout: post
title: "What's a Dynamic framework"
date: 2021-01-06 19:29:38 GMT
tags: Xcode Basics iOS DRY
---

All standard iOS libraries are dynamic frameworks, and they 
They are handled in Xcode in a bundle with the extension `.framework`. Although they are directories, Xcode recognizes them as special cases, and treat them as files. Is easier to move them between projects. 

They are usually shared across multiple applications, but because they are loaded dynamically, at runtime, the system can store only one copy, and let different process access it on demand 

Inside a framework, you will find a bunch of files, mostly metadata, two of them are especially interesting. 

### Module Map
It is a file that tells Xcode where to find headers for relevant objects. We can edit it to add functionalities from other frameworks. 

```
framework module Volonbolon {
  umbrella header "Volonbolon.h"

  export *
  module * { export * }
}

module Volonbolon.Swift {
    header "Volonbolon-Swift.h"
    requires objc
}
```

### Share Object
It is an executable that has the same name as the framework. It is compiled when the framework is created and is the object that can be shared amongst multiple applications. Because it can be shared across multiple apps and platforms, we end up with a leaner package. In fact, when linking the actual binary (.dylib) is not accepted, we need to link against text-based .dylib stubs (.tbd). 

A .tbd is a YAML that contains the name of all the symbols and the location of the binary, but not the implementation. As a result, its size is smaller. 

A typical .tbd would be something like 

```
--- !tapi-tbd
tbd-version:     4
targets:         [ x86_64-ios-simulator ]
uuids:
  - target:          x86_64-ios-simulator
    value:           3DF443DB-F355-321D-9A25-F240AAD0B145
flags:           [ not_app_extension_safe ]
install-name:    '@rpath/Volonbolon.framework/Volonbolon'
swift-abi-version: 7
exports:
  - targets:         [ x86_64-ios-simulator ]
    symbols:         [ '_$s10VolonbolonAAV10volonbolonABSS_tcfC', '_$s10VolonbolonAAV10volonbolonSSvg', 
                       '_$s10VolonbolonAAV10volonbolonSSvpMV', '_$s10VolonbolonAAVMa', 
                       '_$s10VolonbolonAAVMn', '_$s10VolonbolonAAVN', _VolonbolonVersionNumber, 
                       _VolonbolonVersionString ]
...
```

This is a very simple framework containing a struct. 

To generate the .tbd, we can use `tapi` from the Xcrun toolchain 

```
xcrun -r tapi stubify [lib]
```