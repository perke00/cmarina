# Building ATLAS (Windows / Linux)

Requires CMake >= 3.22 and a C++20 compiler (Visual Studio 2022 or g++/clang on Linux).
For the Windows setup .exe, NSIS must be installed (https://nsis.sourceforge.io) and on PATH.
macOS is not supported.

## Windows (Developer PowerShell / x64 Native Tools prompt)

    cmake -S . -B build -G "Visual Studio 17 2022" -A x64
    cmake --build build --config Release
    ctest --test-dir build -C Release --output-on-failure
    cmake --build build --config Release --target package

Output: `build/ATLAS-0.1.0-Windows-setup.exe` (installer) and `.zip` (portable).

## Linux

    cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
    cmake --build build && ctest --test-dir build --output-on-failure
    cmake --build build --target package
