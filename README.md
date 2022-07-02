## **MINGW toolchain for bazel projects on windows**
### This project shows how to use [platforms](https://bazel.build/docs/platforms) to define a custom MINGW toolchain. This project defines a x86_64-compatible C++ MINGW toolchain for windows (`//toolchains:mingw_cc_toolchain`) and a custom platform advertising x86_64 (`//platforms:windows_platform`).

## **How to use:**
for example let's start by creating the following bazel project that we want to build with MINGW toolchain

    bazel_project
        ├── .bazelrc
        ├── BUILD
        ├── main.cpp
        └── WORKSPACE

**BUILD** file
```py
cc_binary(
    name = "main",
    srcs = [
        "main.cpp"
    ],
)
```

**main.cpp** file
```cpp
#include <cstdio>

int main(int argc, char **argv)
{
   #ifdef __MINGW32__
   printf("Build is done using MINGW toolchain with GCC version %d.%d.%d at %s %s\n",
          __GNUC__,
          __GNUC_MINOR__,
          __GNUC_PATCHLEVEL__,
          __TIME__,
          __DATE__);
   #else /* ifdef __MINGW32__ */
   printf("Build is done using default toolchain at %s %s\n",
          __TIME__,
          __DATE__);
   #endif /* ifdef __MINGW32__ */

   return(0);
}
```

and leave **.bazelrc** & **WORKSPACE** files empty for now
#
Now let's try to build & run with the **default toolchain** by typing the following command
```bash
bazel run //:main
```

Output
```cmd
Build is done using default toolchain at 14:00:23 Jul  1 2022
```
#
Now let's try to build & run with the **local MINGW toolchain**

First lets define MINGW toolchain as external dependency for our bazel project by Adding the following lines to **WORKSPACE** file
```py
load("@bazel_tools//tools/build_defs/repo:git.bzl", "git_repository")

git_repository(
    name = "mingw",
    remote = "https://github.com/EhabHelaly/bazel_mingw_toolchain.git",
    branch = "master"
)
```

Second lets define build configurations to use the **MINGW toolchain** by adding the following lines to **.bazelrc** file and replace values of **MINGW_PATH** & **GCC_VERSION**  with compiler's local path & GCC version respectively
```py
# build configurations
build --incompatible_enable_cc_toolchain_resolution             # allow bazel to use custom toolchains
build --platforms=@mingw//platforms:windows_platform            # tell bazel to build using the custom platform
build --extra_toolchains=@mingw//toolchains:mingw_cc_toolchain  # tell bazel to build using the custom toolchain
build --define=MINGW_PATH="C:/toolchains/TDM-GCC-64"            # replace with correct local toolchain path 
build --define=GCC_VERSION="10.3.0"                             # replace with correct toolchain GCC version
```

Now let's try to build & run with the **local MINGW toolchain** by typing the following command
```bash
bazel run //:main
```

Output
```cmd
Build is done using MINGW toolchain with GCC version 10.3.0 at 14:03:27 Jul  1 2022
```
#
Note:
- the toolchain is not hermetic as the toolchain files are referenced from outside this project and they shall be pre-installed
- for a hermetic version of toolchain; the toolchain files can be moved into this project, toolchains_config.bzl to be updated to reference toolchain files within


