# tree-sitter-nim (GRAVITON fork)

This is a fork of [alaviss/tree-sitter-nim](https://github.com/alaviss/tree-sitter-nim)
(all grammar, scanner, and query credit belongs there — see that repo for
the real project), pinned to commit
[`ac72ba30d16edf0be021588a9301ede4accd6cf4`](https://github.com/alaviss/tree-sitter-nim/commit/ac72ba30d16edf0be021588a9301ede4accd6cf4),
with exactly **one line changed**.

## Why this fork exists

[GRAVITON](https://github.com/RhaB17369/Graviton) is a local, offline
code-intelligence CLI that links ~65 tree-sitter grammars into one Rust
binary, including a real Nim import/include resolver. The only
`tree-sitter-nim` published on crates.io (`0.1.0`, from
`github.com/undivisible/tree-sitter-nim`) has **no import-related node
type in its grammar at all** — confirmed by inspecting its real
`node-types.json` — so there is nothing for a real import resolver to
extract. That crate was also published by a brand-new, zero-star account
that batch-published several other language grammars the same day, a
pattern worth being cautious about for a tool whose own purpose includes
security tooling.

`alaviss/tree-sitter-nim` is an actively maintained, substantially richer
grammar with real `import_statement`/`import_from_statement`/
`include_statement` nodes — verified directly by parsing real Nim source
and inspecting the resulting tree, not assumed from the grammar source
alone. The only problem: its Rust binding (`bindings/rust/lib.rs` +
`Cargo.toml`) depends on the full `tree-sitter` crate directly, pinned to
`~0.25`. Cargo's `links = "tree-sitter"` uniqueness rule means only one
version of that crate can appear in an entire dependency graph — a real,
confirmed build error when linked alongside GRAVITON's own `tree-sitter
0.26`.

## The patch

`Cargo.toml`:

```diff
-tree-sitter = "~0.25"
+tree-sitter = "0.26"
```

Nothing else. This was verified to work correctly, not just assumed:
parsing real Nim source (`import`, `from ... import`, `include`, in
several forms) against tree-sitter `0.26`'s actual runtime produces
identical, error-free trees to what the same grammar produces against
`0.25` — the C-level ABI a generated `parser.c` exposes doesn't depend on
which Rust crate version generated the binding glue around it. Only the
*declared* version requirement was overly strict, not a real
incompatibility.

## License

MPL-2.0, same as upstream (see `LICENSE.txt`, `LICENSES/`). The
modification above is disclosed here per MPL-2.0 section 3.3's
modification-notice requirement.

## Using this fork

```toml
[dependencies]
tree-sitter-nim = { git = "https://github.com/RhaB17369/tree-sitter-nim", rev = "<commit-sha-of-the-patch-commit>" }
```

If upstream ever relaxes its own version pin (or rebases onto a newer
tree-sitter core GRAVITON also adopts), this fork should be re-evaluated
for whether it's still needed at all.
