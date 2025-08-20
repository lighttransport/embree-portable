# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Embree is Intel's high-performance ray tracing library (version 4.4.0) that provides optimized ray tracing kernels for CPU and GPU. It's a C++/C library with optional ISPC and SYCL support, targeting graphics applications and production rendering.

## Build System

This project uses **CMake** as its primary build system:

### Basic Build Commands
```bash
# Configure build (from project root)
mkdir build && cd build
cmake -D CMAKE_BUILD_TYPE=Release ..

# Build the project
cmake --build .

# Run tests (if available)
ctest
```

### Common Build Options
- `CMAKE_BUILD_TYPE`: Release, Debug, RelWithDebInfo
- `EMBREE_ISPC_SUPPORT`: Enable ISPC support (default ON)
- `EMBREE_TUTORIALS`: Build tutorial applications (default ON)
- `EMBREE_TESTING_PACKAGE`: Enable testing (default OFF)

### Testing
Run the test suite using Python script:
```bash
python3 scripts/test.py
```

## Code Architecture

### Core Components

1. **kernels/** - Main ray tracing implementation
   - `kernels/common/` - Core ray tracing API and device management
   - `kernels/bvh/` - Bounding Volume Hierarchy acceleration structures
   - `kernels/builders/` - BVH construction algorithms
   - `kernels/geometry/` - Primitive intersectors (triangles, curves, instances, etc.)
   - `kernels/subdiv/` - Catmull-Clark subdivision surface support
   - `kernels/sycl/` - SYCL GPU implementation

2. **common/** - Shared utilities and foundation
   - `common/math/` - Vector math and geometric primitives
   - `common/simd/` - SIMD instruction wrappers (SSE/AVX/AVX512)
   - `common/sys/` - System utilities (threads, memory, platform abstraction)
   - `common/tasking/` - Task scheduling with TBB backend
   - `common/algorithms/` - Parallel algorithms

3. **include/embree4/** - Public C/C++ and ISPC API headers

4. **tutorials/** - Example applications and API usage
   - Self-contained examples of various rendering techniques
   - Each tutorial has its own CMakeLists.txt and can be built independently

### Multi-Platform Support
- **CPUs**: x86 (SSE/AVX/AVX512), ARM (NEON), WASM
- **GPUs**: Intel GPUs via SYCL
- **OS**: Linux, Windows, macOS

### Key Design Patterns
- **Device abstraction**: CPU vs GPU code paths unified through device layer
- **SIMD vectorization**: Hand-optimized kernels with runtime ISA selection
- **Template specialization**: Heavy use of templates for performance-critical paths
- **ISPC integration**: Optional ISPC backend for automatic vectorization

## Development Workflow

### File Organization
- Header files use `.h` extension
- Implementation files use `.cpp` 
- ISPC files use `.ispc` extension
- Configuration templates use `.in` extension

### Adding New Geometry Types
1. Add primitive definition to `kernels/geometry/`
2. Implement intersector in same directory
3. Register with geometry factory in `kernels/common/scene_*.cpp`
4. Add tests in `tutorials/embree_tests/`

### Performance Considerations
- Use vectorized SIMD types from `common/simd/`
- Follow existing patterns for template specialization by ISA
- BVH builders are highly optimized - changes require careful benchmarking