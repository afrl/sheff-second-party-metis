# SHEFF Second-Party METIS (`sheff-second-party-metis`)

`sheff-second-party-metis` is a second-party module in the **Scalable Hierarchical Enrichment Finite-element Framework (SHEFF)** that wraps **METIS** and **GKlib** with cross-platform CMake build support for graph and finite element mesh partitioning.

## Capabilities

- **Unstructured Graph & Mesh Partitioning (`second_party_metis/metis.h`)**: Provides multilevel k-way and recursive bisection partitioning routines (`METIS_PartGraphKway`, `METIS_PartGraphRecursive`, `METIS_PartMeshDual`, `METIS_PartMeshNodal`) used by [`sheff-mesh`](https://github.com/afrl/sheff-mesh) to decompose finite element meshes across MPI ranks while minimizing inter-partition communication.
- **Fill-Reducing Sparse Matrix Ordering**: Supplies graph reordering routines (`METIS_NodeND`) usable by sparse direct and iterative linear algebra backends in [`sheff-sparse-algebra`](https://github.com/afrl/sheff-sparse-algebra).
- **Cross-Platform CMake Integration**: Packages METIS and GKlib cleanly on Linux and Windows without requiring a separate external METIS installation.

## Project Structure

- `include/second_party_metis/metis.h`: Public METIS API header.
- `source/`: METIS and `GKLib/` C implementation files configured for cross-platform builds.
- `unit_tests/`: Unit tests verifying graph and mesh partitioning calls.
- **CMake Package / Target**: `SecondPartyMetis` / `SecondPartyMetis::second_party_metis` (static library).

## Dependencies

- **Build System**: [`sheff-build-tools`](https://github.com/afrl/sheff-build-tools) (`SHEFF_TOOLS`), CMake 3.17+, C/C++ compiler.
- **SHEFF / External Dependencies**: None.

## Building

When building as part of [SPAWC](https://github.com/afrl/spawc) or another top-level SHEFF project, this module is configured and built automatically via `make.py`.

To build and test standalone (with `SHEFF_TOOLS` pointing to [`sheff-build-tools`](https://github.com/afrl/sheff-build-tools)):

```bash
python3 make.py --config --build --make-tests --run-tests
```

## Public Release Information

Distribution Statement A. Approved for public release: distribution is unlimited. Case #: AFRL-2026-4513
