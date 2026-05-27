# Biqbin analysis

The software that I ran is available here https://github.com/Rudolfovoorg/parallel_biqbin_maxcut
It is a parallel branch and bound MPI program.

---

### Raw results

Solutions were computed using Vega HPC cpu partition https://izum.si/vega_slv/

There are 3 solutions folders:
- org_solutions is for Biqbin version 0.0.0
- new_solutinos is for Biqbin version 2.0.0
- newest_solutions is for Biqbin development branch, not yet deployed (41-expose-babnode-evaluation-to-python branch)


### Analysis

Run notebooks in this order:

1. `00_preprocess.ipynb`
2. `01_basic_scaling.ipynb`
3. `02_amdahl_analysis.ipynb`
4. `03_karp_flatt_analysis.ipynb`
5. `04_bab_nodes_analysis.ipynb` (newest_results only)

### Analysis files

All of them are under analysis_output under separate subforlders