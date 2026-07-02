# ⚡ Global Power Plant Data Management

> **Object-Oriented Design · Advanced Data Structures · NumPy Vectorisation**
>
> Modelling the World Resources Institute's Global Power Plant Database as a Python class hierarchy, then benchmarking pure-Python implementations against NumPy-vectorised equivalents for search, aggregation, and distance operations.

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-Vectorised-013243?logo=numpy&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?logo=pandas&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualisation-11557C?logo=matplotlib&logoColor=white)
![License](https://img.shields.io/badge/License-CC%20BY%204.0%20(data)-lightgrey)

---

## 📌 Introduction

This project takes a real-world, tabular energy dataset and re-imagines it as a structured domain model. Rather than manipulating rows in a DataFrame, each power plant becomes a first-class Python object with validated attributes and polymorphic behaviour. On top of that model, a **registry** provides multi-indexed lookups, and a suite of **NumPy-vectorised routines** performs the numerical heavy-lifting (k-NN search, pairwise Haversine distances, correlation, z-score normalisation).

The project is organised as three connected layers:

| Layer | Focus | Techniques |
|---|---|---|
| **1. Domain Model** | Represent plants as typed objects | OOP · Encapsulation · Inheritance · Polymorphism · Factory Pattern |
| **2. Registry & Search** | Efficient querying at scale | Multi-indexing · Heap-based k-NN · Radius search · Benchmarking |
| **3. Numerical Engine** | High-performance analytics | NumPy broadcasting · Vectorised aggregation · Memory profiling |

---

## 📊 Dataset Overview

- **Source:** [WRI Global Power Plant Database](http://datasets.wri.org/dataset/globalpowerplantdatabase) — CC BY 4.0
- **Scope:** ~80% of the world's grid-connected generating capacity (≥1 MW)
- **Coverage:** **28,664 plants** across **164 countries** · **22 attributes**

| Attribute | Description |
|---|---|
| `country_long`, `name` | Country and facility name |
| `gppd_idnr` | Unique global plant identifier |
| `capacity_mw` | Installed nameplate capacity in megawatts |
| `latitude`, `longitude` | Geographic coordinates |
| `fuel1` – `fuel4` | Primary and secondary fuel types |
| `commissioning_year` | Year of first operation |
| `generation_gwh_2013`–`2016` | Annual generation (GWh) |
| `estimated_generation_gwh` | WRI-modelled generation estimate |
| `owner`, `source`, `url` | Provenance information |

---

## 🏗️ Class Architecture

```
                    ┌──────────────────┐
                    │   PowerPlant     │  ← base class, validated properties
                    │   (abstract)     │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼─────────────────────┐
        │                    │                     │
┌───────▼────────┐  ┌────────▼────────┐   ┌────────▼───────┐
│ RenewablePlant │  │ FossilFuelPlant │   │  NuclearPlant  │
└────────────────┘  └─────────────────┘   └────────────────┘
        ▲                    ▲                     ▲
        └────────────────────┼─────────────────────┘
                             │
                    ┌────────┴─────────┐
                    │ PowerPlantFactory│  ← chooses subclass from `fuel1`
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │PowerPlantRegistry│  ← indexed by id / country / fuel
                    └──────────────────┘
```

**Design patterns used:** Factory Method · Encapsulation with property decorators · Template Method (`capacity_factor`, `calculate_environmental_impact` overrides) · Multi-index Registry.

---

## 🔬 Approach

### 1. Object-Oriented Modelling
- `PowerPlant` base class with `@property` getters/setters that reject invalid data (negative capacity, out-of-range coordinates).
- Three subclasses — `RenewablePlant`, `FossilFuelPlant`, `NuclearPlant` — each overriding the capacity-factor and environmental-impact calculations.
- `PowerPlantFactory` inspects the `fuel1` column and returns the correct subclass, keeping construction logic out of client code.

### 2. Registry, Indexing & Search
- `PowerPlantRegistry` maintains three internal dictionaries (`plants_by_id`, `plants_by_country`, `plants_by_fuel`) for **O(1)** grouped lookups.
- **k-Nearest Neighbour** and **radius search** implemented three ways — pure loop, list comprehension, NumPy — with `timeit`-driven benchmarking.
- Heap-based k-NN achieves **O(n log k)** compared to sorting the full array.

### 3. NumPy Optimisation
- Extract numerical attributes into contiguous NumPy arrays for cache-friendly computation.
- Vectorised fuel-type aggregation via `np.unique` + `np.add.at`.
- Full pairwise Haversine distance matrix via broadcasting — no nested loops.
- Z-score normalisation and correlation analysis using built-in NumPy statistics.

---

## 📈 Findings

### Search performance (query point: London, 51.5074°N, -0.1278°E)

| Operation | Loop | List Comprehension | NumPy | Speedup (NumPy vs Loop) |
|---|---:|---:|---:|---:|
| **K-Nearest Neighbour (k=5)** | 2.42 ms | 1.61 ms | **0.17 ms** | **~14×** |
| **Radius Search (100 km)**    | 1.50 ms | 1.49 ms | **0.15 ms** | **~10×** |

### Scaling of loop-based k-NN

| Dataset Size | Avg. Time (ms) |
|---:|---:|
| 100    | 0.17 |
| 1,000  | 0.58 |
| 3,000  | 1.54 |

Confirms **O(n)** scaling behaviour of the naive implementation.

### Memory footprint (capacity data)

| Structure | Memory (bytes) | Relative |
|---|---:|---:|
| Python list | 119,496 | 100% |
| **NumPy array** | **24,000** | **~20%** |

NumPy's contiguous storage cuts memory usage by roughly **5×** compared to a native Python list holding the same numeric values.

---

## 🛠️ Libraries Used

| Library | Purpose |
|---|---|
| **pandas** | CSV loading, initial inspection, DataFrame benchmarking helpers |
| **NumPy** | Vectorised arithmetic, broadcasting, memory-efficient arrays |
| **Matplotlib** | Bar charts, scaling plots, distance-matrix heatmaps |
| **heapq** | Efficient k-nearest-neighbour selection |
| **timeit** | Precise micro-benchmarking of alternative implementations |

---

## 🚀 Getting Started

```bash
git clone https://github.com/DataScienceVishal/global-power-plant-data-management.git
cd global-power-plant-data-management
pip install pandas numpy matplotlib
jupyter notebook Power_Plant_Data_Management.ipynb
```

The dataset (`global_power_plant_database.csv`) is included in the repository, so the notebook runs end-to-end with no additional downloads.

---

## 🧭 Repository Structure

```
global-power-plant-data-management/
├── Power_Plant_Data_Management.ipynb   ← main notebook (design + benchmarks + analysis)
├── global_power_plant_database.csv     ← WRI dataset, v1.1.0 (Oct 2019)
├── DATASET_INFO.txt                    ← original WRI README (attribution & methodology)
├── DATASET_RELEASE_NOTES.txt           ← WRI release notes
└── README.md
```

---

## ✨ Conclusion

Combining an object-oriented domain model with NumPy-vectorised analytics is a practical pattern for scientific data work: the OOP layer keeps heterogeneous entities (renewable vs. fossil vs. nuclear plants) clean and self-validating, while the numerical layer keeps the hot paths fast. The benchmarks demonstrate an order-of-magnitude speedup for search operations and a ~5× reduction in memory footprint — concrete evidence that the "hybrid OOP + NumPy" strategy scales gracefully as the dataset grows.

Further extensions could include:
- 🌍 An interactive geospatial dashboard (Folium / Plotly) built on top of the `PowerPlantRegistry`
- 🔭 Emissions modelling per subclass, with country-level rollups
- 🧪 Unit tests for the class hierarchy using `pytest`

For detailed implementation and results, refer to the Jupyter notebook. **Feel free to explore, fork, and contribute!**

---

## 📚 Citation

> Global Energy Observatory, Google, KTH Royal Institute of Technology in Stockholm, Enipedia, World Resources Institute. 2018. *Global Power Plant Database.* Published on Resource Watch and Google Earth Engine.
