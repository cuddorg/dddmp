# DDDMP Library (BSD-3-Clause Licensed)

The DDDMP library provides a set of serialization and deserialization functions for decision diagrams using the CUDD library.

This repository contains the DDDMP source code as originally released in the CUDD 3.0.0 bundle. While the code was previously distributed under a non-free educational license, it is now available under the BSD-3-Clause license through this repository.

## Build and Install

To build and install the library:

```
# Configure the project using CMake
cmake -B build

# Build the library
cmake --build build

# Install the library to the specified prefix (e.g., /usr/local)
cmake --install build --prefix /usr/local
```
