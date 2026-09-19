# Rx compiler test cases

Welcome to the Rx Compiler course! This repository contains the test cases for the Rx compiler course. The files are organized by the compiler stage or kind of workload they exercise. A test runner discovers test cases from `manifest.json` files.

### Test families

- **`lexer/`** contains tokenization tests. `accept/` holds inputs that should
  lex successfully and `reject/` holds malformed inputs that should be rejected.
  The suite manifest uses `stage: "lex"`.
- **`parser/`** contains syntax tests with the same `accept/` and `reject/`
  split. The suite manifest uses `stage: "parse"`.
- **`semantic/`** contains feature-oriented programs that are checked through
  semantic analysis. A manifest can contain both accepted and rejected
  programs, distinguished by `compilation_success`.
- **`codegen/`** contains programs that must compile and run. Each runtime
  fixture supplies stdin and the expected stdout for one execution.
- **`optimization/`** contains larger or optimization-sensitive runtime
  workloads. These cases are organized separately for coverage, but their
  manifests currently use `stage: "codegen"` and follow the same runtime
  contract as `codegen/`.

## Files in a test directory

- **`.rx`** — an Rx source program. 
- **`.in`** — stdin for one runtime execution. A manifest may use `null` when a
  case needs empty stdin instead of a file.
- **`.out`** — the exact stdout expected from the corresponding execution. Use
  an empty file when the program should print nothing.
- **`manifest.json`** — a nonempty JSON array describing the cases in that
  directory. Paths in the manifest are relative to the manifest's directory.

The usual runtime naming pattern keeps fixtures beside their source, for
example:

```text
codegen/integer-arithmetic/
├── manifest.json
├── acc-...-i32.rx
├── acc-...-i32.mixed.in
└── acc-...-i32.mixed.out
```

One source can have several `io` pairs, which is useful for smoke, boundary,
large, or otherwise distinct input scenarios.

## Manifest format

`manifest.schema.json` is the JSON Schema for version 1 of the manifest format.
Each entry contains:

```json
{
  "source": "program.rx",
  "stage": "semantic",
  "compilation_success": true,
  "description": "Optional explanation",
  "metadata": {}
}
```

The fields are:

- **`source`** — exactly one source file for the entry.
- **`stage`** — one of `lex`, `parse`, `semantic`, or `codegen`.
- **`compilation_success`** — whether the source is expected to pass that
  stage. A normal rejection is a valid negative result; a crash, signal, or
  timeout is not.
- **`io`** — one or more `{ "input", "output" }` pairs. This is required for
  `codegen` entries, which must compile successfully before their generated
  program is run.
- **`description`** — an optional human-readable purpose or explanation for a
  case, especially a negative case.
- **`metadata`** — optional authoring or provenance information. It can record
  hashes, upstream origins, tokens, and similar details.
