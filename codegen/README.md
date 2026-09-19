# Shared IR and codegen correctness suite

Use these same 33 feature/application bundles for both stages: 46 accepted
programs with 88 input/output pairs. Every case must compile successfully.
Compile-error cases are covered by the [semantic suite](../semantic/).

- **IR:** compile valid sources to LLVM IR, lower with Clang 22 for
  `riscv32-unknown-elf -march=rv32im -mabi=ilp32`, and execute in REIMU.
- **Codegen:** compile valid sources with the student's backend to RV32
  assembly and execute in REIMU.

Run all inputs listed for each accepted case and compare exact stdout plus
normal exit status 0. This suite grades correctness, with no cycle threshold.
The large-frame and comprehensive programs check robustness without imposing
specific register allocation, struct layout, or internal calling conventions.

See the [root guide](../README.md) and [coverage map](../COVERAGE.md).
