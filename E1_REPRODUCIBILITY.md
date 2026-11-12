# E1 Supplementary methods and reproducibility

Article 2032946. This supplement accompanies the revised manuscript on three QUBO representations of single-machine weighted earliness/tardiness scheduling with optional inserted idle time. The archived optimization results are unchanged.

## File structure and provenance

- `source/SmWetSolver`: C#/.NET 8 solution and original Python experiment scripts, based on the supplied v5.5 source. The only change to the historical source is the expected summary filename in `run_revision_experiments.ps1`, from `qaoa-summary.csv` to `qaoa-summary-statistics.csv`. Scheduling, QUBO and QAOA algorithms are unchanged.
- `results`: all 29 extracted files of the supplied `revision-results.zip`, byte-for-byte unchanged. Its legacy four figures are retained for provenance; the revised figures are in `figures`.
- `derived`: independently checked instance tables, variable maps, coefficients, parameters and statistics. These are audit outputs, not new optimization experiments.
- `figures`: revised 300-dpi PNG and vector SVG figures.
- `scripts/audit_gazi_qubo.py`: an independent mathematical reconstruction using NumPy and SciPy; it does not call the C# implementation, Qiskit or Gurobi.
- `scripts/reproduce_figures.py`: revised figure generation from archived run/comparison CSVs.
- `scripts/collect_environment.ps1`: captures a current Windows environment without running optimization. A newly collected environment must not be described as a historical environment unless its continuity is established.
- `SHA256SUMS.txt`: SHA-256 digests for every file except the manifest itself.

The old `source/SmWetSolver/validation` and `source/SmWetSolver/qubo-results` folders were supplied with the source package and contain historical pilot artifacts. The manuscript's six formulation validations and 150-run grid are the records in the top-level `results` folder. The penalty-sensitivity pilot in the source package agrees with the independently reconstructed sensitivity CSV in `derived`.

## Complete instance data

All processing times and due dates use discrete time units. All weights are positive costs per time unit.

| Instance | Job | Processing time | Due date | Earliness weight | Tardiness weight |
| --- | ---: | ---: | ---: | ---: | ---: |
| Tiny | 1 | 1 | 2 | 2 | 3 |
| Tiny | 2 | 2 | 4 | 1 | 2 |
| Tiny | 3 | 1 | 5 | 3 | 1 |
| Paper | 1 | 2 | 3 | 1 | 3 |
| Paper | 2 | 3 | 6 | 2 | 1 |
| Paper | 3 | 1 | 5 | 1 | 2 |

Tiny: horizon H=5, feasible upper bound U=0, all penalty groups lambda=1. The start vector in job-ID order is (1,2,4), with completions (2,4,5) and objective 0. This certifies that the reduced horizon retains an optimum; H=5 is not the general dmax+P bound, which would be 9.

Paper: H=12, U=2, all penalty groups lambda=3. The bound comes from EDD order (1,3,2), shifted right by one unit, with sequence-ordered starts (1,3,4), completions (3,4,7), objective 2. The bound generator tests input, EDD and SPT orders at every integer block start from 0 to H-P. The paper's earlier illustrative U=3/lambda=4 choice uses the unshifted EDD schedule. It is valid but is not the penalty used in the archived six-formulation comparison.

## Integer encoding and preprocessing

For an integer with maximum K>0, use L=ceil(log2(K+1)) bits, weights 2^b for b=0,...,L-2, and final weight K-(2^(L-1)-1). For K=0, create no bits. For K=12, weights are (1,2,4,5). Their sum is exactly K; every bit string is in range and every integer in 0,...,K is representable. Some integers have multiple representations. No out-of-range bit strings are silently discarded by the unconstrained solver.

Bounds used by family are: completion and idle K=H; earliness K=d_i; tardiness K=max(0,H-d_i); precedence start K=H-p_i; each precedence slack K=2H. The assignment/product/precedence variables are ordinary binary variables. No cost-based start-window pruning or pair-specific big-M was applied to the archived comparisons.

There are no product auxiliaries in the time-indexed formulation. One-hot assignment penalties create a clique within each job's start variables; overlap penalties connect intersecting half-open intervals of different jobs. The original counts, respectively tiny/paper, are 14/33 variables for time-indexed, 65/87 for position, and 46/63 for precedence.

## Energy and bit order

The JSON convention is `constant + sum linear[i]*x_i + sum_{i<j} quadratic[i,j]*x_i*x_j`. Off-diagonal coefficients are stored once and must not be doubled. A symmetric matrix implementation should split each stored pair coefficient equally between its two matrix entries. The scalar constant is retained in QUBO energy and in the Ising offset.

Variable indices are zero-based. Qiskit measured bit strings have variable 0 at the right-hand end. `variables` records the job, time, position, bit weight and product/slack meaning. `derived/variable-map.csv` and `derived/qubo-coefficients.csv` flatten all six JSON files.

## Classical validation

The source classical reference enumerates job permutations and optimizes their integer timing by dynamic programming. The independent audit instead enumerates the Cartesian product of allowed integer starts and rejects overlaps. It finds 24 feasible schedules for the tiny instance, objective 0, and 504 feasible schedules for the paper instance, objective 2.

Gurobi receives a binary quadratic objective with MIPGap=0 and NonConvex=2. C# CLI options specify time limit and threads; those exact launch settings are not embedded in the archived validation JSONs. The JSONs do preserve status, runtime, MIP gap, node count and solution vector.

Both position rows have `CERTIFIED_TARGET_REACHED`, not an independent Gurobi optimality proof. The classical-optimal schedule is encoded as a consistent QUBO warm start, whose zero-penalty energy is verified. Together with lambda>U and the known classical optimum, this certifies the QUBO optimum mathematically. The intermediate solver MIP gap is not that certificate and the runtimes are not used to claim comparative solver speed.

## Ideal QAOA protocol

There are five depths p=1,...,5 and 30 seeds per depth, seeds 1,...,30. The same seed list is reused across depths. The grid consists of 150 distinct (p,seed) records; there is no assertion that all 150 are mutually independent random streams.

Qiskit's `qaoa_ansatz` uses its default all-plus initial state, the cost Hamiltonian and the standard X mixer. Parameters are in Qiskit's list order, beta[0],...,beta[p-1],gamma[0],...,gamma[p-1]. All initial angles are generated from `numpy.random.default_rng(seed).uniform(0.0, pi, 2*p)`. The same seed is supplied to StatevectorEstimator and StatevectorSampler. The ideal estimator uses exact statevector expectations; 4096 shots are used only for final sampling after optimization. No coefficient normalization is applied.

SciPy `minimize` uses COBYLA with `maxiter=200` and `rhobeg=0.5`. No `tol`, `catol`, `f_target` or explicit parameter bounds are passed; remaining behavior follows the installed SciPy defaults. The source requirement declares SciPy 1.18.1 but the historical result JSON does not contain a package freeze; a requirement declaration is not independently verified installation evidence. `maxiter` is a function-evaluation budget, not a proof of convergence. The budget is not multiplied by p.

Optimized parameters, objective history, number of evaluations, best sample, modal sample, first 20 most frequent strings, aggregate rates and runtime are preserved in `qaoa-experiment.json`. The initial angles were not saved as a separate original field. `derived/qaoa-parameters.csv` contains 900 reconstructed initial angles and 900 archived optimized angles; reconstruction is explicitly labelled. Their initial expected energies match the first stored objective for every run.

The independent audit applies diagonal exp(-i*gamma*QUBO_energy) and single-qubit exp(-i*beta*X) operations directly to a NumPy statevector. Using the full QUBO energy differs from using the offset-free Ising Hamiltonian only by an irrelevant global phase. Every final expected energy matches the archive to below 4.1e-13. This verifies the energy trajectory endpoints without rerunning the optimizer. It does not reconstruct every original sampled histogram.

## Statistics and figure definitions

Each run selects the lowest-energy string among the final 4096 samples, breaking an energy tie by sample frequency. `optimalBest` is the fraction of runs where that selected solution is feasible, has the classical-optimal scheduling cost and zero energy penalty. It is not a single-shot success probability. The optimum counts are 6,10,17,14,18 out of 30 for depths 1,...,5.

Sample standard deviation uses n-1. Quartiles use the inclusive quantile convention, equivalent to linear interpolation. Mean energy and runtime confidence intervals use Student t with n-1=29 degrees of freedom. Binary event rates use the two-sided Wilson score interval with z=1.959963984540054. Overlap of separate intervals is not a formal test for absence of a difference. No formal across-depth superiority test is reported.

Runtime is measured around `qaoa_solve`, including ideal optimization, final sampling and candidate assessment; the caller's subsequent decoding and file export are outside that timer. p=1,...,5 have 0,24,29,30,30 runs reaching 200 evaluations. Optimizer success counts are 30,6,1,0,0. A budget stop is distinct from failure to find an optimum among samples.

Figure 1 uses the tiny-instance rows of Table 8. Left: counts of variables and nonzero off-diagonal couplings; right: 100*couplings/binomial(m,2). Figure 2 uses run-level rates and Wilson intervals from the same CSV as Table 9. Figure 3 shows the 30 best energies per depth, median, Q1/Q3, mean, 1.5-IQR whiskers and outliers. Figure 4 shows mean runtime in seconds and Student t intervals. All axes are linear.

## Noisy Aer execution

The selected trained run is p=1, training seed=1. In radians, beta[0]=2.3404256775430574 and gamma[0]=4.007934650837959. These angles are bound directly; there is no noisy reoptimization. Both `seed_transpiler` and `seed_simulator` are 42, the backend script's explicit default. The result filename's `seed1` refers to training, not Aer sampling.

The source constructs `AerSimulator(noise_model=noise, seed_simulator=42)` without overriding method (automatic) and calls `transpile` with optimization_level=3 and seed_transpiler=42. No explicit coupling map or basis gate list is passed. The noise attaches one-qubit depolarizing_error(0.001,1) to x and sx, two-qubit depolarizing_error(0.01,2) to cx, and symmetric 2% readout error to all qubits. rz is noiseless in this specified model. There is no device calibration, T1/T2 model or physical topology.

Archived compiled operations are rz=116, cx=94, sx=42, measure=14 and barrier=1; depth=84 and size=266. With 4096 shots there are 9 feasible shots and 1 optimal shot. The selected ideal run has 9 feasible shots and 0 optimal shots. Neither a general noise benefit nor degradation can be inferred from this single comparison.

The source settings document the intended call; the result JSON does not separately preserve optimization_level, final effective simulator method, a basis-gate/coupling-map snapshot, a QPY circuit, full histogram or exact Aer version. The complete optimized angles are present. These provenance limitations are acknowledged in the revised manuscript.

## Software records and limitations

The result JSON explicitly records Qiskit 2.5.2. The historical source requirements declare qiskit==2.5.2, numpy==2.5.2, scipy==1.18.1, qiskit-aer>=0.17, qiskit-ibm-runtime>=0.44 and matplotlib>=3.10. This file is retained as supplied; its entries are not a verified full environment freeze. The Python version, exact Aer and plotting versions, complete OS/hardware record and exact solver patch version are not jointly archived. The current online documentation does not establish which versions were installed historically.

The full original shot histograms are absent. Original aggregate rates, best strings and top-20 frequencies are retained. Regenerating samples from saved angles is a new sampling event and should be stored separately. The supplement supports independent verification of coefficients, classical optima, expected energies and summary arithmetic; bit-for-bit recovery of unarchived original histograms is not claimed.

## Reproduction commands

For the independent audit, use a Python environment with NumPy and SciPy. From the extracted supplement directory:

```powershell
python .\scripts\audit_gazi_qubo.py --source .\source\SmWetSolver --results .\results --out .\audit-recomputed
python .\scripts\reproduce_figures.py --results .\results --out .\figures-recomputed
```

The second command also requires Matplotlib. These commands do not rerun optimization. To inspect the current Windows project environment, copy `scripts/collect_environment.ps1` to the original SmWetSolver folder and run it there; its default Python is `.venv/Scripts/python.exe`.

To perform a new ideal grid using the archived tiny QUBO, without rebuilding the C# models, use a Qiskit/SciPy environment and a new output directory:

```powershell
python .\source\SmWetSolver\qiskit_run\run_qaoa_grid.py .\results\qubo-tiny\time-indexed-qubo.json --instance .\source\SmWetSolver\instances\qubo-tiny-three-job.json --validation .\results\qubo-tiny\time-indexed-validation.json --reps 1:5 --seeds 1:30 --shots 4096 --maxiter 200 --no-resume --output-dir .\reproduction\qaoa-new
```

To resample the archived first trained run with the documented noise call, independently of any new optimization:

```powershell
python .\source\SmWetSolver\qiskit_run\sample_qaoa_backend.py .\results\qubo-tiny\time-indexed-qubo.json .\results\qaoa-30seed\qaoa-experiment.json --run-index 0 --instance .\source\SmWetSolver\instances\qubo-tiny-three-job.json --validation .\results\qubo-tiny\time-indexed-validation.json --mode noisy --seed 42 --optimization-level 3 --shots 4096 --one-qubit-error 0.001 --two-qubit-error 0.01 --readout-error 0.02 --output .\reproduction\noisy-new.json
```

These are new runs, not recovery of missing historical records. Their numerical sampling and optimizer stopping details can depend on installed package versions.

To rerun the original mathematical model generation, an appropriate .NET 8 SDK, Gurobi installation and license are needed. The historical `run_revision_experiments.ps1` is supplied with its filename check corrected. Activate the intended virtual environment explicitly before calling it. Use a new result directory for any new computation and retain existing results unchanged. The grid resume check identifies data by name and a subset of settings; it does not compare a QUBO content hash, so it must not be used to mix altered QUBOs with old runs.

## Supplementary implementation details

`SchedulingInstance` and `JobData` contain shared instance data. Three classical solver classes create position, time and disjunctive formulations. The three QUBO builders use `QuboAccumulator` to expand squared linear constraints; the position builder also adds Rosenberg terms. `QuboBuilderSupport` chooses bounded binary weights and the feasible upper bound. `QuboJsonExporter` writes sparse coefficients and metadata. `ExactSmallInstanceSolver` supplies classical references; `PositionBasedQuboWarmStart` encodes the position certificate. Python scripts load sparse JSON, map it to Ising, run ideal QAOA and optional backend sampling, and summarize archived records. These class design details are supplementary rather than part of the main experimental description.
