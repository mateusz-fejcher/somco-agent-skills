# somco-qt6-project-audit

A read-only audit skill for Qt6 projects that checks five dimensions in a single pass:

1. **Somco conventions** — house-style rules for QML, C++, and CMake (see `references/somco-conventions.md`)
2. **CMake quality** — structural and correctness checks for the build system
3. **Unit-test presence** — lightweight check for whether QML and C++ tests exist
4. **clang-tidy / clazy** — deterministic static analysis layer
5. **Code formatting** — clang-format and qmlformat dry-run checks

## Requirements

All external tools are **optional** — if a tool is not found, its checks are skipped with a warning and the audit continues with the remaining phases.

| Tool | Phase | If missing | How to install | Env var override |
|------|-------|------------|----------------|------------------|
| `clang-tidy` | Static Analysis | Skipped with warning | `brew install llvm` / `apt install clang-tidy` | `CLANG_TIDY_PATH` |
| `clazy-standalone` | Static Analysis | Skipped with warning | [github.com/KDE/clazy](https://github.com/KDE/clazy) | `CLAZY_STANDALONE_PATH` |
| `compile_commands.json` | Static Analysis | clang-tidy + clazy both skipped | `cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -B build` | — |
| `clang-format` | Formatting | Skipped with warning | `brew install clang-format` / `apt install clang-format` | `CLANG_FORMAT_PATH` |
| `qmlformat` | Formatting | Skipped with warning | Included with Qt6 dev tools | `QMLFORMAT_PATH` |

### Configuration files (auto-detected)

| File | Tool | If missing |
|------|------|------------|
| `.clang-format` | clang-format | Uses clang-format's built-in default style |
| `.qmlformat.ini` | qmlformat | Uses qmlformat's built-in default style |

## Usage

```
/somco-qt6-project-audit
```

## Output

A single combined Markdown report covering all five phases with a pass/fail verdict.
