# Gazi QUBO supplementary material R1

Code, data and reproducibility notes accompanying manuscript **2032946**, “Tek makine ağırlıklı erken/geç tamamlanma çizelgeleme problemi için üç matematiksel model ve QUBO dönüşümleri” (three mathematical formulations and QUBO transformations for single-machine weighted earliness/tardiness scheduling).

Author: Oğuzhan Ahmet Arık.

## Start here

- **[Download the complete R1 supplement](Gazi_QUBO_Ek_Materyal_R1.zip)** for source code, archived results, coefficients, variable maps, parameters and revised figures.
- **[Read E1 supplementary methods and reproducibility](E1_REPRODUCIBILITY.md)** directly on GitHub.
- **[Supplementary implementation details](E1_REPRODUCIBILITY.md#supplementary-implementation-details)** is the final section of E1 and explains the C# classes and file flow.
- [Download checksums](DOWNLOAD_SHA256SUMS.txt) verify the files hosted at the repository root. The archive also contains its own `SHA256SUMS.txt` for the extracted supplement files.

Türkçe: Makalede geçen “Ek E1 Supplementary implementation details”, yukarıdaki E1 belgesinin son bölümüdür. `.md` dosyası GitHub üzerinde doğrudan okunabilir. Bütün kod ve verilere erişmek için ZIP dosyasını indirip açın.

## Archive contents

| Path inside the ZIP | Contents |
| --- | --- |
| `E1_REPRODUCIBILITY.md` | Instance data, bounded encoding, energy convention, experiment settings, reproduction commands and provenance limits |
| `source/SmWetSolver` | Supplied v5.5 C#/.NET 8 solution and Python scripts; the PowerShell summary filename check is corrected |
| `results` | 29 original archived result files, unchanged |
| `derived` | Independently checked tables, QUBO coefficients, variable maps, initial/optimized parameters and statistics |
| `figures` | Four revised figures, as 300-dpi PNG and vector SVG |
| `scripts` | Independent NumPy/SciPy audit, figure reproduction and current-environment collection |
| `SHA256SUMS.txt` | Checksums for all other files inside the archive |

The archive contains 138 files, including the manifest. The historical pilot folders inside `source/SmWetSolver` are distinguished from the manuscript results in E1.

## Independent verification

After extracting the ZIP, run the following from its extracted root in a Python environment with NumPy and SciPy:

```powershell
python .\scripts\audit_gazi_qubo.py --source .\source\SmWetSolver --results .\results --out .\audit-recomputed
```

To recreate the revised figures, also install Matplotlib:

```powershell
python .\scripts\reproduce_figures.py --results .\results --out .\figures-recomputed
```

These checks do not rerun the optimization experiments. The independent audit reconstructs six QUBO coefficient sets, checks integer schedules and archived solution constraints, and recomputes the initial/final statevector expectations for 150 QAOA records. The maximum final expectation discrepancy is approximately 4.1e-13.

Rebuilding the C# models and rerunning Gurobi requires .NET 8, Gurobi and an appropriate license. Full ideal-grid and noisy-Aer commands are provided in E1.

## Scope and archived records

The experiments concern two small three-job instances; ideal QAOA is evaluated on the 14-variable time-indexed QUBO. The two position-based `CERTIFIED_TARGET_REACHED` results use an encoded classical-optimal warm start together with the sufficient penalty condition. They are not independent Gurobi branch-and-bound optimality proofs.

The original complete dependency freeze and full shot histograms are not archived. The supplement retains recorded parameters, aggregate statistics, available sample records and independently checked energies. Recreating missing original histograms bit for bit is not claimed. No quantum advantage or real quantum hardware result is claimed.

## R1 snapshot integrity

The complete ZIP is the unchanged R1 supplement supplied with the revision. Its SHA-256 digest is:

```text
5d45ddec49726b029d645006babd8cdd608dbda0fbcb0dbff4c929cb4d6c9e30
```
