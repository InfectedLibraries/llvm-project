# LLVM Compiler Infrastructure - Pathogen Fork

This fork extends the functionality of libclang. Amonth other things, it primarily allows inspecting the memory and vtable layout of records (structs, classes, and unions.) It is currently based on [LLVM 10.0.0](https://github.com/llvm/llvm-project/tree/d32170dbd5b0d54436537b6b75beaf44324e0c28).

All functionality provided by this fork can be found in [`PathogenExtensions.cpp`](clang/tools/libclang/PathogenExtensions.cpp).

Essentially it exposes information provided by `ASTRecordLayout` and `MicrosoftVTableContext`/`ItaniumVTableContext`.
Both sets of information are intended to be ABI-agnostic.

The information provided for record layouts is largely based on the behavior of the `-fdump-record-layouts` switch.

The information provided for vtable layouts is somewhat based on the bahvior of the `-fdump-vtable-layouts` switch, but the implementation of this switch for Itanium and Microsoft ABIs is basically completely separate (the information provided by each isn't even consistent.) The codebase meant to be processed by this fork never has multiple inheritance or virtual bases, so it may be lacking in information in that department.

This fork also provides:

* `pathogen_Location_isFromMainFile`
  * A variant of `clang_Location_isFromMainFile` that uses `SourceManager::isInMainFile` instead of `SourceManager::isWrittenInMainFile`.
  * In particular this function will consider cursors created from a macro expansion in the main file to be in the main file.
* `pathogen_getOperatorOverloadInfo`
  * Provides information about operator overloads (and whether a given function is an operator overload.)
* `pathogen_getArgPassingRestrictions`
  * Returns whether the given type is able to be passed in registers for by-value arguments (or return values.)
  * Note that the underlying method for this function (`RecordDecl::getArgPassingRestrictions`) only cares about C++ restrictions, it does not consider size-related restrictions.
* `pathogen_ComputeConstantValue`
  * Tries to compute the constant value of an expression or a variable's initializer and returns the constant value.

This fork was never really intended to be merged into libclang proper. The API shape doesn't match exactly what libclang provides, and it only exists to support [ClangSharp.Pathogen](https://github.com/InfectedLibraries/ClangSharp.Pathogen) (and as such are accessed via C# bindings, hence the lack of a header file.)

---------------

# The LLVM Compiler Infrastructure

This directory and its subdirectories contain source code for LLVM,
a toolkit for the construction of highly optimized compilers,
optimizers, and runtime environments.

The README briefly describes how to get started with building LLVM.
For more information on how to contribute to the LLVM project, please
take a look at the
[Contributing to LLVM](https://llvm.org/docs/Contributing.html) guide.

## Getting Started with the LLVM System

Taken from https://llvm.org/docs/GettingStarted.html.

### Overview

Welcome to the LLVM project!

The LLVM project has multiple components. The core of the project is
itself called "LLVM". This contains all of the tools, libraries, and header
files needed to process intermediate representations and converts it into
object files.  Tools include an assembler, disassembler, bitcode analyzer, and
bitcode optimizer.  It also contains basic regression tests.

C-like languages use the [Clang](http://clang.llvm.org/) front end.  This
component compiles C, C++, Objective C, and Objective C++ code into LLVM bitcode
-- and from there into object files, using LLVM.

Other components include:
the [libc++ C++ standard library](https://libcxx.llvm.org),
the [LLD linker](https://lld.llvm.org), and more.

### Getting the Source Code and Building LLVM

The LLVM Getting Started documentation may be out of date.  The [Clang
Getting Started](http://clang.llvm.org/get_started.html) page might have more
accurate information.

This is an example workflow and configuration to get and build the LLVM source:

1. Checkout LLVM (including related subprojects like Clang):

     * ``git clone https://github.com/llvm/llvm-project.git``

     * Or, on windows, ``git clone --config core.autocrlf=false
    https://github.com/llvm/llvm-project.git``

2. Configure and build LLVM and Clang:

     * ``cd llvm-project``

     * ``mkdir build``

     * ``cd build``

     * ``cmake -G <generator> [options] ../llvm``

        Some common generators are:

        * ``Ninja`` --- for generating [Ninja](https://ninja-build.org)
          build files. Most llvm developers use Ninja.
        * ``Unix Makefiles`` --- for generating make-compatible parallel makefiles.
        * ``Visual Studio`` --- for generating Visual Studio projects and
          solutions.
        * ``Xcode`` --- for generating Xcode projects.

        Some Common options:

        * ``-DLLVM_ENABLE_PROJECTS='...'`` --- semicolon-separated list of the LLVM
          subprojects you'd like to additionally build. Can include any of: clang,
          clang-tools-extra, libcxx, libcxxabi, libunwind, lldb, compiler-rt, lld,
          polly, or debuginfo-tests.

          For example, to build LLVM, Clang, libcxx, and libcxxabi, use
          ``-DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi"``.

        * ``-DCMAKE_INSTALL_PREFIX=directory`` --- Specify for *directory* the full
          pathname of where you want the LLVM tools and libraries to be installed
          (default ``/usr/local``).

        * ``-DCMAKE_BUILD_TYPE=type`` --- Valid options for *type* are Debug,
          Release, RelWithDebInfo, and MinSizeRel. Default is Debug.

        * ``-DLLVM_ENABLE_ASSERTIONS=On`` --- Compile with assertion checks enabled
          (default is Yes for Debug builds, No for all other build types).

      * Run your build tool of choice!

        * The default target (i.e. ``ninja`` or ``make``) will build all of LLVM.

        * The ``check-all`` target (i.e. ``ninja check-all``) will run the
          regression tests to ensure everything is in working order.

        * CMake will generate build targets for each tool and library, and most
          LLVM sub-projects generate their own ``check-<project>`` target.

        * Running a serial build will be *slow*.  To improve speed, try running a
          parallel build. That's done by default in Ninja; for ``make``, use
          ``make -j NNN`` (NNN is the number of parallel jobs, use e.g. number of
          CPUs you have.)

      * For more information see [CMake](https://llvm.org/docs/CMake.html)

Consult the
[Getting Started with LLVM](https://llvm.org/docs/GettingStarted.html#getting-started-with-llvm)
page for detailed information on configuring and compiling LLVM. You can visit
[Directory Layout](https://llvm.org/docs/GettingStarted.html#directory-layout)
to learn about the layout of the source code tree.
