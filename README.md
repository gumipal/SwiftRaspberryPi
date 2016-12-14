# SwiftRaspberryPi
Patches and Images for Swift on the Raspberry Pi 2 and 3

## Compiling Swift

This is what worked for me to compile a pre-release Swift 3.0.2.

Use the scripts at the bottom of the following web pages
 - http://dev.iachieved.it/iachievedit/keeping-up-with-open-source-swift/
 - http://dev.iachieved.it/iachievedit/building-swift-3-0-on-an-armv7-system/
 - https://github.com/swift-arm

E.g.:
```bash
sudo apt-get install git cmake ninja-build clang uuid-dev libicu-dev icu-devtools libbsd-dev libedit-dev libxml2-dev libsqlite3-dev swig libpython-dev libncurses5-dev pkg-config
git clone https://github.com/iachievedit/package-swift
cd package-swift
./get.sh
```

Apply the following patches (from the `patches` directory on https://github.com/gumipal/SwiftRaspberryPi):
```
NSGeometry.swift.patch
NSXMLNode.swift.patch
```

### libdispatch
To make libdispatch compile for ARM (https://github.com/IBM-Swift/Aphid/wiki/Compiling-an-Aphid-Application-on-the-Pi-3):

Add the following in `libdispatch src/shims/linux_stubs.h`:

```
#ifndef PAGE_SIZE
#define PAGE_SIZE 4096
#endif
```

Then add the following to `<path_to_swift>/usr/lib/swift/clang/include/stdarg.h`:

```
#ifndef _VA_LIST
#include <_G_config.h>                 // <--- Add this line
typedef __builtin_va_list va_list;
#define _VA_LIST
#endif
```

Build core libs and `libdispatch` first (the below should all be one line):
```bash
./swift/utils/build-script --build-subdir buildbot_linux -R --lldb --llbuild --libdispatch -- --install-swift --install-lldb --install-llbuild --install-libdispatch --install-prefix=/usr '--swift-install-components=autolink-driver;compiler;clang-builtin-headers;stdlib;swift-remote-mirror;sdk-overlay;dev' --build-swift -static-stdlib --build-swift-static-sdk-overlay --skip-test-lldb --install-destdir=${INSTALL_DIR}-pre
```

Then install to `/usr/local` and link to `/usr/lib`:
```bash
sudo rsync -av `pwd`/install-pre/usr/ /usr/local/
sudo ln -s /usr/local/lib/swift/linux/libswift*.so /usr/lib
```

### Building Swift
Then build and install as normal, i.e.,
```bash
./package.sh
sudo rsync -av `pwd`/install/usr/ /usr/local/
```

And (if you installed in /usr/local) remove the, now unnecessary links:
```bash
sudo rm /usr/lib/libswift*.so
```

## Binaries

Pre-built images can be found in
https://github.com/gumipal/SwiftRaspberryPi/images

