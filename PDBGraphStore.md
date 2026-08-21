# PDB Graph Store

---

## Overview

PDB Graph Store is a high-productivity Python library and memory-optimization framework designed for compressing and managing large-scale protein graph datasets derived from Protein Data Bank (PDB) structures. The project focuses on reducing the heavy memory footprint associated with protein graph modeling in scientific computing, bioinformatics pipelines, and machine learning workflows.

By reorganizing redundant graph structures into shared, compact representations, PDB Graph Store enables research pipelines to load, store, and manipulate thousands of complex protein graphs with significantly reduced RAM usage while preserving exact graph topology and attribute fidelity.

---

## Problem

In structural bioinformatics and graph machine learning (e.g., Graph Neural Networks for drug discovery and protein property prediction), protein structures are commonly represented as NetworkX graphs where nodes correspond to amino acid residues or atoms, and edges represent spatial or chemical interactions.

However, standard Python graph representations suffer from severe memory bloat:

- **Redundant Structural Data**: Multiple protein graphs share identical residue labels, atomic connectivity patterns, and global attributes (e.g., amino acid properties, Meiler embeddings).
- **High In-Memory Overhead**: Standard Python dictionaries and objects in libraries like NetworkX add significant memory allocation overhead for every node and edge.
- **Scalability Bottlenecks**: Large datasets (such as `ligand_PO4` with over 940,000 nodes and 7,000,000 edges) require gigabytes of RAM when uncompressed, making parallel graph loading, processing, and GNN model training memory-prohibitive on standard hardware.

---

## Solution

To address these memory challenges, I designed and implemented PDB Graph Store, a specialized graph compression and storage layer.

The system employs a two-tier compression strategy using deduplicated dictionary mappings and bitwise set representations:

- **Topology Deduplication with Roaring Bitmaps**: Graph nodes and edges are mapped to global integer IDs via bidirectional dictionaries (`bidict`). PDB graph topologies are stored as 64-bit Roaring Bitmaps (`pyroaring.BitMap64`), providing high-speed, compact bitset storage for node and edge ID sets.
- **Attribute Separation and Deduplication**: Node and edge attributes are split into global attributes (shared across graphs, such as residue names and Meiler vectors) and local attributes (graph-specific properties like 3D spatial coordinates and B-factors). Attribute values are deduplicated in central value stores and referenced by integer keys.
- **Lossless Reconstruction**: The store provides an `extract()` API that losslessly reconstructs standard NetworkX graph objects on demand, ensuring full compatibility with downstream bioinformatics tools and GNN frameworks like PyTorch Geometric.

Additionally, the system provides modular operations to manipulate compressed datasets, including graph removal (`remove_graph_from_store`), dataset splitting (`split_graph_store`), and merging heterogeneous graph stores (`merge_graph_stores`).

---

## Workflow

The repository is structured to support graph loading, compression, memory profiling, and interactive experimentation.

The overall directory structure is shown below:

```
.
├── docker-compose.yaml
├── jupyter-lab/
│   └── docker-compose.yaml
├── lab
├── build
├── requirements.txt
├── src/
│   ├── Builder.py
│   ├── main.py
│   ├── MemoryMeasuring.py
│   ├── PDBGraphStore_notebooks/
│   ├── Graphein_notebooks/
│   └── pkg/
│       ├── PDBGraphStore.py
│       ├── edge_functions_Model.py
│       ├── operations.py
│       └── __init__.py
├── data/
├── results/
├── min_max_results/
├── memory_footprint_results/
└── times/
```

The core operational workflow follows these steps:

1. **Ingestion & Graph Construction**: PDB structural files are retrieved and constructed into raw protein graphs using Graphein and NetworkX based on defined granularities ($C_\alpha$ or atom level) and interaction functions (e.g., Delaunay triangulation, aromatic contacts).
2. **Compression Pipeline**: Raw graphs pass through `Builder.compress_pdb_graphs()`, mapping topology and attributes into compact Roaring Bitmaps and integer lookup arrays within `PDBGraphStore`.
3. **Storage & Manipulation**: Compressed graph stores support dataset splitting, merging, and graph removal without full uncompression.
4. **On-Demand Extraction**: Graphs are extracted back into native NetworkX objects whenever required by downstream machine learning loaders.
5. **Interactive Analysis**: A containerized Jupyter Lab session (`sh lab`) provides interactive execution and benchmark visualization tools.

---

## Technologies Used

The project uses a focused and efficient scientific Python stack:

- Python 3.11 – core programming language
- Graphein – protein graph construction from PDB structural files
- NetworkX – graph structure modeling and representation
- Pyroaring (`pyroaring.BitMap64`) – fast 64-bit compressed Roaring Bitmaps for topology set storage
- Bidict (`bidict`) – bidirectional mapping between string labels and integer IDs
- Pympler & Memory Profiler – fine-grained memory consumption measuring and profiling
- Docker & Docker Compose – containerized environment setup for reproducible execution

---

## Technical Decisions

Key technical decisions were made to maximize memory efficiency and maintain pipeline usability:

- **Use 64-bit Roaring Bitmaps**: Replaced standard Python `set` objects with `pyroaring.BitMap64` for storing node and edge ID sets, drastically lowering memory overhead for dense graph collections.
- **Decompose Global and Local Attributes**: Separated node attributes into global immutable properties (stored once in a unified 2D NumPy integer matrix) and local coordinates/B-factors (indexed via tuple keys), eliminating redundant attribute copies across protein instances.
- **Adopt Bidirectional ID Mapping**: Utilized `bidict` structures for $O(1)$ bidirectional lookups between human-readable PDB node/edge labels and compressed integer IDs.
- **Containerize the Workbench**: Used Docker Compose to package all dependencies, Graphein tools, and Jupyter Lab into a reproducible container environment accessible via a single shell script (`sh lab`).

---

## Technical Challenges

- **Lossless Reconstruction of Complex Data Types**: Protein graph attributes contain diverse data structures, such as 3D NumPy coordinate arrays, Pandas Series Meiler embeddings, and set-based edge categories. Guaranteeing exact attribute equality (`nx.utils.nodes_equal` and `nx.utils.edges_equal`) upon extraction required designing custom serialization and casting logic.
- **Accurate In-Memory Profiling**: Measuring true RAM usage in Python is complex due to object overhead and internal memory management. Integrated `pympler.asizeof` and memory profiling tools to track individual components (dictionaries, bitmaps, serialized sizes) across datasets.
- **In-Memory Store Operations**: Implementing graph removal, store splitting, and store merging directly on compressed structures required managing global ID spaces carefully to avoid memory spikes or index fragmentation.

---

## Results

The performance of PDBGraphStore was evaluated across two main scenarios: a comparative memory compression benchmark on standard ccPDB datasets, and a practical impact assessment integrated into three real-world protein graph machine learning workflows.

In both benchmarks, the compression ratio is defined as:

$$\text{Compression Ratio (\%)} = 1 - \left( \frac{\text{Memory}_{\text{after}}}{\text{Memory}_{\text{before}}} \right) \times 100$$

### 1. Comparative Performance & Memory Compression

Evaluated on an Intel Core 14th gen i9 CPU with 64 GB DDR5 RAM using residue-level ($C_\alpha$) protein graphs with Delaunay, aromatic, and aromatic–sulphur edges across 8 ccPDB datasets:

| Dataset | PDBs | Avg Compression Time (s) | Memory Before (MB) | Memory After (MB) | Compression Ratio (%) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Ligand_PLP (plp)** | 63 | 19.68 ± 0.12 | 356.26 | 285.01 | **19.99%** |
| **Nucleotides_UDP (udp)** | 68 | 19.46 ± 0.16 | 343.07 | 280.03 | **18.37%** |
| **Nucleotides_NAD (nad)** | 139 | 39.53 ± 0.30 | 721.77 | 573.59 | **20.53%** |
| **Nucleotides_FAD (fad)** | 168 | 53.56 ± 0.23 | 962.55 | 705.91 | **26.66%** |
| **Ligand_BME (bme)** | 187 | 30.49 ± 0.27 | 543.17 | 483.46 | **10.99%** |
| **Metals_CO (co)** | 196 | 35.39 ± 0.30 | 627.26 | 527.11 | **15.96%** |
| **Nucleic_DNA (dna)** | 533 | 106.13 ± 0.76 | 1,978.96 | 1,649.90 | **16.62%** |
| **Ligand_PO4 (po4)** | 1,259 | 263.27 ± 2.27 | 4,932.00 | 4,072.15 | **17.43%** |

Key findings:
- PDBGraphStore consistently reduced memory consumption across all evaluated datasets, achieving compression ratios between **10.99%** and **26.66%**.
- Compression efficiency is primarily driven by structural redundancy (overlap of shared nodes and edges across protein structures) rather than simple dataset size.

### 2. Practical Impact on Real-World Machine Learning Workflows

PDBGraphStore was integrated into three graph machine learning pipelines on an Intel Core 11th gen i5 CPU with 16 GB DDR4 RAM running inside Docker containers:

- **Baseline (PSCDB Graph Classification)**: 788 paired protein structures across 7 classes using GCN, GAT, and GraphSAGE models.
- **TDC-Developability (TDC)**: Binary classification of 2,426 antibody structures on SAbDab-Chen dataset using EGNN.
- **GNN-SCOPe (GNN)**: Classification of 3,000 protein structures across SCOPe 2.08 superfamilies using GraphSAGE.

#### Execution Time & Overhead Comparison

| Workflow | Graphein Time (s) | PDBGraphStore Time (s) | Time Overhead (%) |
| :--- | :---: | :---: | :---: |
| **Baseline** | 541.20 ± 9.74 | 581.58 ± 4.66 | **+7.46%** |
| **TDC** | 1,230.93 ± 52.08 | 1,748.48 ± 68.21 | **+42.04%** |
| **GNN** | 1,882.95 ± 27.78 | 2,649.72 ± 71.67 | **+40.72%** |

#### Memory Footprint Comparison

| Workflow | Metric | Graphein (GB) | PDBGraphStore (GB) | Compression Ratio (%) |
| :--- | :--- | :---: | :---: | :---: |
| **Baseline** | Graph Structure | 0.76 GB | 0.68 GB | **10.34%** |
| | Peak Workflow RAM | 2.33 GB | 2.18 GB | **6.68%** |
| **TDC** | Graph Structure | 4.80 GB | 3.40 GB | **29.18%** |
| | Peak Workflow RAM | 7.31 GB | 6.09 GB | **16.69%** |
| **GNN** | Graph Structure | 5.00 GB | 4.01 GB | **21.24%** |
| | Peak Workflow RAM | 7.95 GB | 7.18 GB | **9.69%** |

Overall trade-off:
- PDBGraphStore delivers consistent memory savings between **10.34% and 29.18%** for standalone graph structures, and **6.68% to 16.69%** for overall peak workflow RAM.
- In exchange, total execution time increases by 7.4% to 42% due to internal compression and extraction overhead, making it a highly practical solution for memory-constrained protein graph applications.

---

## Key learnings

Through this project, I gained practical experience with:

- Designing memory-efficient data structures and compression algorithms in Python
- Leveraging bitwise set representations (Roaring Bitmaps) for large-scale graph topology indexing
- Profiling object memory footprints and optimizing dynamic data structures
- Building reproducible scientific environments using Docker and containerized workflows
- Structuring modular Python libraries for bioinformatics and graph machine learning applications

---

Link to project repository: [graphein-on-ray](https://github.com/dcc-ufla-graph-mining/graphein-graph-store)

