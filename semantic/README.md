# Semantic / AST stage

Compile each `.rx` independently and compare acceptance/rejection with its
bundle manifest. No AST serialization or diagnostic wording is required.
There are 40 bundles: 55 accepted and 171 rejected programs.

Every negative source passes the supplied ANTLR grammar; failures target
required name, type, mutability, capability, constant, layout, receiver, or
entry checks. Tests do not ask for ownership, borrow, or lifetime analysis.
Valid but nonterminating helper functions are never called in the fixtures.

Accepted feature cases and five comprehensive programs are also present in
[`../codegen/`](../codegen/), where they gain input/output fixtures.
Semantic rejection cases are kept here; codegen contains only valid programs.
See the [root guide](../README.md) for the schema, runner, and validation rules.
