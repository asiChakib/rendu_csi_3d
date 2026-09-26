# rendu_csi_3d

A Python implementation of a progressive 3D mesh-processing pipeline for the CSI3D project. The project loads a triangular mesh, builds a multiresolution hierarchy by simplifying the mesh, parametrizes removed vertices with barycentric coordinates, and writes the result as a progressive OBJA model.

> **Status:** Academic project (in a team of 4 students) (might be incomplete). The repository includes the implementation, unit tests, and the project report (`MAPS.pdf`). Some parts of the implementation may still require refinement for production use.

## Features

- Parse and generate the [OBJA](https://www-sop.inria.fr/reves/Basilic/OBJA/) mesh format.
- Convert an OBJA mesh into internal vertex/face data structures.
- Build mesh topology information:
  - vertex adjacency;
  - vertex stars and ordered one-rings;
  - edge-to-face relationships;
  - boundary detection and boundary loops;
  - Euler characteristic calculation.
- Estimate geometric properties such as triangle areas, face and vertex normals, curvature, and dihedral angles.
- Flatten vertex one-rings into 2D using a conformal-style mapping.
- Retriangulate holes created during simplification using ear clipping, with Delaunay/fan fallbacks.
- Construct a discrete-Kosaraju-style multiresolution mesh hierarchy.
- Encode the hierarchy as a progressive OBJA file, including newly introduced vertices and faces at each level.
- Validate the main modules with Python unit tests.

## How the pipeline works

The main pipeline in `pipeline.py` performs four stages:

1. **Load the input mesh** with the OBJA parser.
2. **Build a hierarchy** of progressively coarser meshes using `dk.py` and the topology helpers.
3. **Build a parametrization** by projecting removed vertices onto base-level triangles and storing barycentric coordinates.
4. **Generate a progressive OBJA file** containing the coarse base mesh followed by refinement levels.

The simplification process uses independent vertex sets, local one-ring flattening, hole retriangulation, and boundary-aware handling. Configuration constants such as the target base size and maximum number of levels are defined in `dk.py` and `pipeline.py`.

## Repository layout

| File | Description |
| --- | --- |
| `run.py` | Small command-line entry point for processing files in an `example/` directory. |
| `pipeline.py` | End-to-end mesh-to-progressive-OBJA pipeline and barycentric parametrization. |
| `dk.py` | Mesh hierarchy construction and vertex-removal/retriangulation logic. |
| `data_structures.py` | Vertices, faces, mesh levels, mesh hierarchies, and OBJA-to-mesh conversion. |
| `obja.py` | OBJA parser, model representation, and progressive OBJA writer. |
| `mesh_topology.py` | Adjacency, one-ring, boundary, and mesh-topology utilities. |
| `geometry_utils.py` | Geometric measurements, normals, curvature, and tangent bases. |
| `conformal_mapping.py` | One-ring flattening, polygon utilities, triangulation, and orientation checks. |
| `priority_queue.py` | Priority-based independent-set selection used during simplification. |
| `test_*.py` | Unit tests for the individual modules and pipeline behavior. |
| `MAPS.pdf` | Project documentation/report. |

## Requirements

- Python 3.9 or newer (Python 3.10+ recommended).
- NumPy.
- SciPy.

Install the Python dependencies in a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate       # Windows: .venv\\Scripts\\activate
python -m pip install --upgrade pip
python -m pip install numpy scipy
```

If you add a development dependency file such as `requirements.txt`, the installation can instead be performed with:

```bash
python -m pip install -r requirements.txt
```

## Input format

The parser expects an OBJA mesh file. At minimum, meshes should contain:

- `v x y z` vertex declarations;
- triangular `f v1 v2 v3` face declarations.

The parser also supports several project-specific instructions, including vertex and face edits, triangle strips, and hidden faces. Vertex indices in OBJA are one-based; internal Python indices are zero-based.

## Usage

### Command-line entry point

`run.py` expects two file names and automatically looks for both files inside an `example/` directory:

```bash
python run.py input.obja output.obja
```

For example, if `example/model.obja` exists:

```bash
python run.py model.obja model_progressive.obja
```

The command prints progress for loading, hierarchy construction, parametrization, and progressive OBJA generation.

### Python API

The pipeline can also be called directly from Python:

```python
from pipeline import process_obj2obja

process_obj2obja("input.obja", "output.obja")
```

To work with the lower-level data structures:

```python
from data_structures import obj2mesh
from dk import build_hierarchy

mesh = obj2mesh("input.obja")
hierarchy = build_hierarchy(mesh)
print(hierarchy.num_levels())
print(hierarchy.compression_ratio())
```

Despite the historical `obj` names used in a few functions and variables, the current parser and writer operate on the repository's OBJA-style format.

## Running the tests

Run the complete test suite from the repository root:

```bash
python -m unittest discover -p "test_*.py"
```

You can also run an individual test module:

```bash
python -m unittest test_pipeline.py
```

If the tests are written using `pytest` conventions in your local version, install pytest and run:

```bash
python -m pip install pytest
pytest
```

## Output format

The generated progressive OBJA file contains:

1. vertices and faces for the coarsest/base mesh;
2. comments identifying each refinement level;
3. additional vertices introduced by each level;
4. faces that become available after each refinement;
5. optional random face-color records emitted by the writer.

The output can be inspected with an OBJA-compatible viewer or processed by another tool that supports the format.

## Configuration

The following controls are currently implemented as module-level constants:

- target base mesh size;
- maximum hierarchy depth;
- maximum allowed vertex degree;
- minimum removal fraction;
- simplification/priority parameters.

For experimentation, update the constants in `dk.py` and `pipeline.py` and rerun the pipeline. Keeping the values consistent between the two modules is recommended.

## Known limitations

- The project is currently organized as a collection of Python modules rather than an installable package.
- No dependency lockfile or packaging configuration is provided yet.
- The command-line wrapper assumes input and output paths are relative to `example/`.
- The progressive format and parametrization are project-specific and may require an OBJA-aware consumer.
- Degenerate, non-manifold, or invalid meshes may produce incomplete simplification results or fallback triangulations.
- The implementation is intended primarily for experimentation and coursework rather than production-scale meshes.

## Development notes

When extending the project:

- preserve the distinction between one-based OBJA indices and zero-based internal indices;
- keep face visibility consistent when modifying topology;
- run the full test suite after changing topology or triangulation code;
- test both closed meshes and meshes with boundaries;
- validate generated meshes in an external viewer when changing `conformal_mapping.py` or `dk.py`.

## Contributing

1. Create a feature branch.
2. Make a focused change with tests where practical.
3. Run the test suite locally.
4. Document changes to the input/output format or algorithm parameters.
5. Open a pull request describing the motivation, implementation, and validation performed.

## License

No license file is currently included in this repository. Unless a license is added, the copyright holder retains all rights to the source code. Please contact the repository owner before redistributing or using it outside the intended project context.

## Acknowledgements

This repository was created for the CSI3D project and includes the accompanying `MAPS.pdf` documentation.
