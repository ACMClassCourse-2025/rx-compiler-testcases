# Optimization workloads

These eleven original Rx programs exercise substantial runtime work and
different compiler decisions. Each bundle has a descriptive `acc-*.rx`, a
manifest, one small correctness input (`smoke`), and two larger inputs
(`large` and `large-variant`). All three inputs have exact expected outputs.
The generator and independent Python calculations are in
[`tools/suite_optimization.py`](../tools/suite_optimization.py).

| Bundle | Largest input | Main optimization opportunities |
| --- | --- | --- |
| [matrix-multiply](matrix-multiply/manifest.json) | 160 × 160 matrices; 4,096,000 multiply-accumulates | Induction variables, invariant addresses, register allocation |
| [floyd-warshall](floyd-warshall/manifest.json) | 144 vertices; 2,985,984 relaxations | Load forwarding, conditional updates, safe alias analysis |
| [jacobi-stencil](jacobi-stencil/manifest.json) | 112 × 112 grid, 240 steps; 3,010,560 cell visits | Loop optimization, address reuse, strength reduction |
| [merge-sort](merge-sort/manifest.json) | 131,071 signed keys | Recursive calls, branches, indexing, scratch-buffer reuse |
| [knapsack](knapsack/manifest.json) | 320 items, capacity 10,000 | Loop-carried memory dependencies, descending induction, scalar promotion |
| [prime-sieve](prime-sieve/manifest.json) | Candidates through 1,000,000 | Boolean storage, strided stores, loop invariants |
| [graph-bfs](graph-bfs/manifest.json) | 8,192 vertices, 65,536 edges, 64 searches | Irregular loads, branches, queue and visited-state access |
| [integer-mixing](integer-mixing/manifest.json) | 1,800,000 dependent rounds | Inlining, constant materialization, wrapping integer operations |
| [recursive-heap-tree](recursive-heap-tree/manifest.json) | 16,383 nodes, deep clone, 128 traversals | Recursive calls, field addressing, typed heap operations |
| [scalar-optimization](scalar-optimization/manifest.json) | 1,200,000 aggregate updates | SROA, mem2reg, inlining, GVN, SCCP, DCE, LICM |
| [large-control-flow](large-control-flow/manifest.json) | 96 functions, 1,152 branch diamonds, 1,474,560 dynamic diamonds | Compiler scalability, inlining budgets, CFG handling, branch relaxation |

The numerical kernels use integers because Rx has no floating-point types.
They are workload adaptations, not numerically equivalent ports of the
floating-point scientific benchmarks. The large-control-flow program is
1,638 source lines; the other workloads obtain scale primarily from
runtime iteration and data size. This separates compiler scalability from
the quality of generated hot-loop code.

## Keeping the measurements meaningful

Dimensions, iteration counts, keys, seeds, and source vertices come from
`getInt`. Computation contributes to printed results or a checksum over the
complete result sequence. Kernels perform no input/output inside their hot
loops. Buffers are reused across repeated steps where the algorithm permits
it; there is no intentional unbounded allocation or deep linear recursion.

Checksums use explicitly wrapping 32-bit arithmetic. Some programs also print
counts, endpoint values, or unchanged-original witnesses. The ordinary
feature tests cover the underlying behavior directly; a checksum alone is
not intended to replace those fine-grained tests.

Smoke inputs catch correctness problems cheaply. Large inputs change both
size and data, and exercise non-power-of-two boundaries where appropriate.
All inputs obey indexing, division, borrowing, ownership and lifetime
guarantees. No performance input relies on signed division overflow, an
out-of-bounds access, illegal aliasing, or a particular allocation address.

Bundle entries in [`suite-index.json`](../suite-index.json) record sizing notes.
Under four-byte integers, the
reference aggregate layout, and capacity doubling with old buffers retained,
the largest flat-buffer workloads need only a few MiB. The tree has bounded
depth and about 1.1 MiB of typed payload for both copies, before allocator
overhead. These estimates provide headroom under the 256 MiB execution/1 MiB
stack configuration; they are not measurements of an arbitrary student
allocator. The specification's detailed heap-budget policy was still TBD
when these fixtures were authored.

## Comparing compilers

1. Require successful compilation, exact output, and exit status 0 for every
   selected input. A wrong answer has no valid performance measurement.
2. Pin the compiler options, runtime, Clang version if used, REIMU version,
   and simulator weights. Use the same backend route for the baseline and
   candidate. Do not change cache/predictor or cost-model options between
   runs.
3. Record REIMU `Total cycles` separately for every input. The included
   runner also records compilation time and source artifact sizes. Assembly
   text bytes are only a size proxy, not loaded machine-code bytes.
4. If staff want a single comparison number, compute each baseline/candidate
   cycle ratio, combine the two performance inputs within each benchmark,
   then take a geometric mean across benchmark families. This is a suggested
   analysis method, not a mandated grading formula. Preserve per-case results
   so improvements and regressions remain visible.
5. Use repeated runs to check reproducibility. A pinned deterministic
   simulator should normally report identical guest cycle counts; host wall
   time can vary. For native timing experiments, use warmups and multiple
   samples, and report the measurement method separately.

No speedup target or cutoff has been invented. The committed outputs were
validated on 32-bit Rust at `-O0` and `-O2`, but the **complete performance
corpus has not been calibrated on a course reference Rx compiler in REIMU**.
Before assigning scores or hard time limits, run the course baseline and
check dynamic instruction counts, cycles, stack usage, and actual allocator
behavior. The runner's timeout options are configurable operational limits.

## Sources consulted

Retrieved online on 2026-09-19. These informed workload selection and
measurement principles; no benchmark implementation was copied.

- [LLVM test-suite guide](https://llvm.org/docs/TestSuiteGuide.html): reference
  outputs for correctness, and separate runtime, compilation-time, and code-size
  metrics; complementary single-source and larger application workloads.
- [PolyBench/C](https://www.cs.colostate.edu/~pouchet/software/polybench/):
  matrix/graph/stencil kernel families, parametric loop bounds, nontrivial
  initialization, and observable live-out data to prevent whole-kernel
  dead-code elimination.
- [Embench IoT](https://github.com/embench/embench-iot): a varied collection of
  integer and embedded workloads, with speed and size treated as distinct
  measurements.
- [LLVM Stanford benchmarks](https://github.com/llvm/llvm-test-suite/tree/main/SingleSource/Benchmarks/Stanford):
  compact algorithmic workloads complementing regular numerical kernels.
- [Rx backend contract](../../RCompiler-Spec/src/backend.md#optimization-and-execution-cost):
  RV32IM/ILP32 and REIMU `Total cycles` with default weights.
