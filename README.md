# Somco Agent Skills

Somco Software forked version of The Qt Company agentic skills:
https://github.com/TheQtCompanyRnD/agent-skills

AI-powered skills for Qt development and quality assurance,
designed for Claude Code and compatible AI coding tools.

> These skills use AI and can make mistakes. Always review the output.

## Skills

### Somco Skills

| Skill | Description | Example trigger |
|-------|-------------|-----------------|
| `somco-qt6-porting-review` | Assesses Qt5-to-Qt6 porting readiness using clazy, an inline pattern linter, and deep analysis. Read-only. | `/somco-qt6-porting-review` in your Qt5 project |
| `somco-qt6-project-audit` | Four-phase Qt6 project audit: house-style conventions (QML/C++/CMake), CMake quality, unit-test presence, clang-tidy/clazy static analysis, and qmllint. Read-only. | `/somco-qt6-project-audit` in your Qt6 project |

## Installation

For full multi-tool support details, see the
[upstream documentation](https://github.com/TheQtCompanyRnD/agent-skills#multi-tool-support).

### Claude Code CLI (recommended)

```bash
# Or copy into a project for team use
cp -r skills/somco-qt6-project-audit .claude/skills/somco-qt6-project-audit
```

## Usage Examples

Skills are triggered with `/skill-name` in your AI coding tool.
You can pass additional instructions after the skill name to
customize the output.

### Basic audit

```
/somco-qt6-project-audit
```

Runs the full four-phase audit on the current project directory.

### HTML report output

```
/somco-qt6-project-audit output only html report
```

Generates the audit report as a standalone HTML file instead of Markdown.

### Audit specific commits only

```
/somco-qt6-project-audit check only commits: a1b2c3d, e4f5g6h, i7j8k9l
```

Limits the audit scope to files changed in the listed commits.

### Migration readiness check

```
/somco-qt6-porting-review
```

Scans the project for Qt5-to-Qt6 porting issues and produces a readiness report.

### Migration check on specific files

```
/somco-qt6-porting-review check only src/widgets/MainWindow.cpp src/qml/main.qml
```

## License

BSD-3-Clause - see [LICENSE](LICENSE) for details.
