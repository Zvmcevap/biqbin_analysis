## Results

The experiment compares three BiqBin solver versions on the same Max-Cut instance:

| Folder | Meaning |
|---|---|
| `org_solutions` | original BiqBin version `0.0.0` |
| `new_solutions` | BiqBin version `2.0.0` |
| `newest_solutions` | development branch `41-expose-babnode-evaluation-to-python` |

The benchmark was run on the Vega HPC CPU partition. The measured problem is small enough that all versions evaluate approximately the same number of branch-and-bound nodes, around **17.7k to 17.8k nodes**, so the runtime comparison is mostly a comparison of parallel execution overhead and implementation efficiency rather than a comparison of different search trees.

The main measured fields are:

- `cores`: number of MPI processes / cores used
- `time_mean_s`: runtime in seconds
- `bab_nodes_mean`: number of evaluated branch-and-bound nodes
- `speedup`: speedup relative to the smallest measured core count for that version
- `efficiency`: speedup divided by ideal speedup
- `bab_nodes_per_second`: branch-and-bound node throughput

> Note: speedup is computed relative to each version's own baseline. For `org_solutions` and `new_solutions`, the baseline is 4 cores. For `newest_solutions`, the baseline is 8 cores because no 4-core result is present.

---

### Runtime comparison

The most important result is runtime. Lower is better.

| Cores | org time [s] | new time [s] | newest time [s] |
|---:|---:|---:|---:|
| 4 | 2688.61 | 1897.60 | — |
| 8 | 1205.93 | 1095.91 | 1234.41 |
| 16 | 568.95 | 589.90 | 592.24 |
| 32 | 276.85 | 253.53 | — |
| 64 | 144.45 | 141.18 | 147.41 |
| 128 | 73.45 | 77.25 | 75.85 |
| 256 | 42.99 | 43.70 | 46.39 |
| 512 | 40.27 | 27.96 | 28.82 |
| 1024 | 29.48 | 19.00 | 16.59 |

At low and mid core counts, the three versions are relatively close. The original version is fastest at 16, 128, and 256 cores, while the new version is better at 4, 8, 32, 64, and 512 cores. At the largest tested size, **1024 cores**, the development version is fastest:

| Version | 1024-core runtime [s] |
|---|---:|
| `newest_solutions` | **16.59** |
| `new_solutions` | 19.00 |
| `org_solutions` | 29.48 |

At 1024 cores, the development branch is about **43.7% faster than the original version** and about **12.7% faster than the new 2.0.0 version**.

Runtime plots:

| Version | Runtime plot |
|---|---|
| `org_solutions` | ![](analysis_output/basic_scaling/basic_scaling_runtime_org.png) |
| `new_solutions` | ![](analysis_output/basic_scaling/basic_scaling_runtime_new.png) |
| `newest_solutions` | ![](analysis_output/basic_scaling/basic_scaling_runtime_newest.png) |

---

### Speedup

Speedup measures how much faster the solver becomes as more cores are added.

For the original and new versions, speedup is relative to 4 cores. For the newest development version, speedup is relative to 8 cores.

At the largest measured core count:

| Version | Baseline | Max measured cores | Speedup |
|---|---:|---:|---:|
| `org_solutions` | 4 cores | 1024 cores | 91.20x |
| `new_solutions` | 4 cores | 1024 cores | 99.86x |
| `newest_solutions` | 8 cores | 1024 cores | 74.41x |

The new 2.0.0 version reaches the largest measured relative speedup, while the development version has the lowest absolute runtime at 1024 cores. This difference is caused by the different baseline: the development version starts from an 8-core baseline instead of a 4-core baseline.

Speedup plots:

| Version | Speedup plot |
|---|---|
| `org_solutions` | ![](analysis_output/basic_scaling/basic_scaling_speedup_org.png) |
| `new_solutions` | ![](analysis_output/basic_scaling/basic_scaling_speedup_new.png) |
| `newest_solutions` | ![](analysis_output/basic_scaling/basic_scaling_speedup_newest.png) |

---

### Parallel efficiency

Efficiency is speedup divided by ideal speedup. A value near 1 means the program is close to linear scaling. A decreasing value means that additional cores contribute less useful work.

At high core counts, efficiency drops for all versions:

| Version | Efficiency at highest measured core count |
|---|---:|
| `newest_solutions` at 1024 cores | 0.581 |
| `new_solutions` at 1024 cores | 0.390 |
| `org_solutions` at 1024 cores | 0.356 |

The development branch keeps the best efficiency at the largest measured core count. This matches the runtime result: it scales better at the high end and achieves the fastest 1024-core runtime.

Efficiency plots:

| Version | Efficiency plot |
|---|---|
| `org_solutions` | ![](analysis_output/basic_scaling/basic_scaling_efficiency_org.png) |
| `new_solutions` | ![](analysis_output/basic_scaling/basic_scaling_efficiency_new.png) |
| `newest_solutions` | ![](analysis_output/basic_scaling/basic_scaling_efficiency_newest.png) |

---

### Amdahl analysis

Amdahl's Law estimates the effective serial fraction of the program. The fitted serial fraction is small for all versions:

| Version | Estimated serial fraction | Implied maximum speedup |
|---|---:|---:|
| `newest_solutions` | 0.00595 | 168.12x |
| `new_solutions` | 0.00636 | 157.25x |
| `org_solutions` | 0.00668 | 149.64x |

All three versions have an estimated serial fraction below 1%. This means that the program is highly parallelizable for this instance. The development branch has the smallest fitted serial fraction and the highest implied maximum speedup.

Amdahl fit plots:

| Version | Amdahl fit |
|---|---|
| `org_solutions` | ![](analysis_output/amhdal/amdahl_fit_org.png) |
| `new_solutions` | ![](analysis_output/amhdal/amdahl_fit_new.png) |
| `newest_solutions` | ![](analysis_output/amhdal/amdahl_fit_newest.png) |

---

### Karp-Flatt analysis

The Karp-Flatt metric estimates the effective serial fraction plus overhead from measured speedup. It is useful because it includes not only truly serial code, but also synchronization, communication, load imbalance, and other parallel overheads.

The observed Karp-Flatt trend is small for all versions:

| Version | Karp-Flatt slope |
|---|---:|
| `new_solutions` | -0.000065 |
| `newest_solutions` | 0.000027 |
| `org_solutions` | 0.000060 |

The values are close to zero, which suggests that the overhead trend is weak for this instance. The original and newest versions show a slight positive trend, meaning overhead grows mildly with more cores. The new version shows a slight negative trend, meaning the measured effective overhead decreases slightly as core count increases.

Karp-Flatt plots:

| Version | Karp-Flatt plot |
|---|---|
| `org_solutions` | ![](analysis_output/karp_flatt/karp_flatt_metric_org.png) |
| `new_solutions` | ![](analysis_output/karp_flatt/karp_flatt_metric_new.png) |
| `newest_solutions` | ![](analysis_output/karp_flatt/karp_flatt_metric_newest.png) |

---

### Branch-and-bound node throughput

The number of evaluated B&B nodes is almost constant across runs, staying near 17.8k nodes. This is useful because the solver is doing approximately the same amount of search work, so runtime differences mostly reflect implementation and parallel performance.

At 1024 cores:

| Version | B&B nodes / second |
|---|---:|
| `newest_solutions` | **1074.74** |
| `new_solutions` | 936.15 |
| `org_solutions` | 603.83 |

The development branch has the best node throughput at 1024 cores. It evaluates about **78% more B&B nodes per second than the original version** at the same core count.

---

### Per-rank branch-and-bound distribution

The development branch exposes additional per-rank branch-and-bound data. These plots show how branch-and-bound work is distributed across MPI ranks.

For low and medium core counts, the per-rank plots are useful for checking load balance. Ideally, most ranks should evaluate a similar number of nodes and spend a similar amount of time computing.

Representative per-rank plots:

| Metric | 16 cores | 64 cores | 1024 cores |
|---|---|---|---|
| B&B nodes per rank | ![](analysis_output/branch_and_bound/babnodes_per_rank_15.png) | ![](analysis_output/branch_and_bound/babnodes_per_rank_63.png) | ![](analysis_output/branch_and_bound/babnodes_per_rank_1023.png) |
| Compute time per rank | ![](analysis_output/branch_and_bound/compute_per_rank_15.png) | ![](analysis_output/branch_and_bound/compute_per_rank_63.png) | ![](analysis_output/branch_and_bound/compute_per_rank_1023.png) |
| Throughput per rank | ![](analysis_output/branch_and_bound/throughput_per_rank_15.png) | ![](analysis_output/branch_and_bound/throughput_per_rank_63.png) | ![](analysis_output/branch_and_bound/throughput_per_rank_1023.png) |

The per-rank analysis is important because branch-and-bound algorithms are naturally irregular: different subtrees may require different amounts of work. If some ranks receive harder parts of the search tree, the total runtime is limited by the slowest ranks. This is visible as load imbalance in the node-count, compute-time, or throughput plots.

---

## Conclusion

The measurements show that all three BiqBin versions scale well on this benchmark instance. The fitted Amdahl serial fraction is below 1% for all versions, confirming that the solver is highly parallelizable on hard instances that branch well.

The strongest result is at high core counts. At 1024 cores:

- `newest_solutions` is the fastest version with **16.59 s**
- `new_solutions` takes **19.00 s**
- `org_solutions` takes **29.48 s**

The development branch is therefore the best high-core-count version in this experiment. It also has the highest 1024-core B&B node throughput, reaching **1074.74 evaluated nodes per second**.

The branch-and-bound node counts remain almost constant across versions, so the runtime improvement is not mainly caused by solving a smaller search tree. Instead, the improvement appears to come from better parallel execution and lower high-core overhead.