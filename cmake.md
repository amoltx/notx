# Make / CMake

Typical CMake execution flow
1. Create project source code
2. Create CMakeLists file
3. Creake an empty build directory: all cmake generated files will be stored here
4. cd build
5. cmake <location of CMakefile> : configure the project / generate makefiles
6. cmake --build . : start build, the . here implies that we are in build dir and want to build here
