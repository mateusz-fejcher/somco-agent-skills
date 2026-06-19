# Qt5-to-Qt6 Porting Review Skill

A read-only review skill that assesses Qt5-to-Qt6 porting readiness for C++/QML projects. It combines [clazy's Qt6 checks](https://www.qt.io/blog/porting-from-qt-5-to-qt-6-using-clazy-checks) with an inline pattern linter and a deep-analysis pass to produce a structured porting report.

## Requirements

### Required

- **Qt 5.x project** -- the codebase under review must be a Qt5 project (C++, QML, or both)

### Optional (recommended)

- **clazy v1.10+** -- enables the most accurate analysis via clazy-standalone's dedicated Qt6-porting checks. Without it, the skill falls back to pattern-based linting only.

  Install:
  ```bash
  # macOS
  brew install clazy

  # Ubuntu/Debian
  sudo apt install clazy

  # From source
  git clone https://github.com/kde/clazy
  ```

- **compile_commands.json** -- required by clazy to understand your build. Generate it with CMake:
  ```bash
  cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -B build
  ```
  The skill searches for this file in the project root and common build directories (`build/`, `cmake-build-debug/`, etc.).

### Configuration

If `clazy-standalone` is not on your `$PATH`, set the environment variable before running:

```bash
export CLAZY_STANDALONE_PATH=/path/to/clazy-standalone
```

## What it checks

| Phase | Tool | Dependencies | What it covers |
|---|---|---|---|
| 1 | clazy-standalone | clazy + compile_commands.json | Deprecated Qt5 APIs, moved headers, qHash signatures, forward declaration fixes, missing Q_OBJECT macros |
| 2 | Inline pattern linter | None | Removed classes (QRegExp, QStringRef, QDesktopWidget, ...), removed modules (QtScript, QtX11Extras, ...), stale CMake references (Qt5::), QML versioned imports, inline GLSL shaders, qmake detection |
| 3 | Deep analysis | None | QtMultimedia/QtWebEngine risk assessment, Qt5Compat bridge usage, setContextProperty patterns, qHash signature confirmation |

If clazy is not available, Phases 2 and 3 still run and provide useful coverage.

## Usage

Ask your AI agent to review your project for Qt6 readiness:

```
Review this project for Qt6 porting readiness
```

```
Check if my changes break Qt6 compatibility
```

```
Audit src/ for Qt5-to-Qt6 issues
```

The skill produces a structured report with findings categorized by severity (Blocking, Warning, Suggestion) and type (API, MOD, BLD, QML, HASH, etc.).

## Output

The report includes:

- **Clazy findings** -- from clazy-standalone (if available)
- **Lint findings** -- from the inline pattern linter
- **Deep-analysis findings** -- confirmed issues from semantic analysis
- **Investigation targets** -- medium-confidence items needing manual verification
- **Summary table** -- counts by category and severity

## Limitations

- This is static analysis only. It cannot detect runtime or visual regressions (QML RHI rendering changes, rewritten QtMultimedia backend, QtWebEngine API drift).
- The inline linter uses pattern matching -- it may flag occurrences in comments or strings. The deep-analysis pass filters false positives where possible.
- qmake projects are flagged with a migration-to-CMake recommendation, but this skill does not perform qmake-to-CMake conversion.

## License

BSD-3-Clause
