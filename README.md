# SHEFF Second-Party Mesh (`sheff-second-party-mesh`)

`sheff-second-party-mesh` is a second-party module in the **Scalable Hierarchical Enrichment Finite-element Framework (SHEFF)** that packages **Triangle** (2D quality Delaunay triangulation) and **TetGen** (3D quality tetrahedral mesh generation and Delaunay tetrahedralization) as a unified CMake static library with targeted stability patches for finite element sub-discretization.

## Capabilities

- **2D Constrained Delaunay Triangulation (`second_party_mesh/triangle/triangle.h`)**: Provides Jonathan Shewchuk's Triangle library interface for 2D polygon triangulation, constrained segment insertion, and quality mesh generation.
- **3D Constrained Tetrahedralization (`second_party_mesh/tetgen/tetgen.h`)**: Provides Hang Si's TetGen library compiled in library mode (`TETLIBRARY`) for 3D polyhedral decomposition, surface-conforming tetrahedralization, and adaptive quadrature sub-cell generation in enriched XFEM elements.
- **SHEFF CMake & Diagnostics Integration**: Integrates both libraries under `SecondPartyMesh::second_party_mesh` with SHEFF exception/logging utilities (`Utility::utility`).

## Project Structure

- `include/second_party_mesh/triangle/triangle.h`: Public Triangle C/C++ API header.
- `include/second_party_mesh/tetgen/tetgen.h`: Public TetGen C++ library header (`TETLIBRARY`).
- `source/`: Patched Triangle and TetGen source implementations.
- `unit_tests/`: Unit tests exercising 2D triangulation and 3D tetrahedralization routines.
- **CMake Package / Target**: `SecondPartyMesh` / `SecondPartyMesh::second_party_mesh` (static library).

## Dependencies

- **Build System**: [`sheff-build-tools`](https://github.com/afrl/sheff-build-tools) (`SHEFF_TOOLS`), CMake 3.17+, C++17/C++20 compiler.
- **First-Party SHEFF Modules**: [`sheff-utility`](https://github.com/afrl/sheff-utility) (`Utility::utility`).

## Building

When building within a top-level SHEFF project (such as [SPAWC](https://github.com/afrl/spawc)), this module is built automatically via the top-level `make.py`.

To build standalone (after installing required dependencies or within a SHEFF workspace with `SHEFF_TOOLS` configured):

```bash
python3 make.py --config --build --make-tests --run-tests
```

## Public Release Information

Distribution Statement A. Approved for public release: distribution is unlimited. Case #: AFRL-2026-4513
