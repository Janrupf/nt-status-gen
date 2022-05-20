# NtStatusGen

This project helps to generate a static lookup table of `NTSTATUS` codes to their
respective names. Using this approach software can look up the names of status codes,
which are usually not available at runtime.

## Building

This project uses [CMake](https://cmake.org/) and requires at least version 3.10 of it.
Building should be straight forward:
```powershell
mkdir build # ALWAYS use a separate build directory
cd build
cmake ..

# Depend on your case, choose ONE of the following:

# 1. Generate NtStatusNames.h and NtStatusNames.cpp in the build directory
cmake --build . --target nt-status-source-files

# 2. Generate NtStatusNames.zip in the build directory (includes the source files)
cmake -- build . --target nt-status-zip-library
```

## Usage

### Using the raw sources

See the [Building](#building) section for generating the source files. Afterwards you
can copy the generated source files into your project, they are standalone C++ sources.

### Using CMake projects

Download this repository or add it as a submodule into your CMake project, then add
the following to your `CMakeLists.txt`:
```cmake
# Add NtStatusGen library
add_subdirectory(/path/to/nt-status-gen)

# From here on you can use nt-status-names as a library dependency
target_link_libraries(your-executable PRIVATE nt-status-names)
```

### API

The entire API is represented by a single function in the namespace `NtStatus`:
```cpp
/**
 * Looks up an NTSTATUS code name based on its numerical representation.
 * 
 * The output parameters is only written if a valid code was supplied.
 * 
 * @param code the NTSTATUS code to look up
 * @param nameOut a reference to which the name will be written
 * @return true if the code could be found, false otherwise
 */
bool lookupNtStatusCodeName(uint64_t code, std::string &nameOut);
```
