# Reengineering `isort`

### Architecture analysis · God Class decomposition · Behaviour-preserving refactoring

**University of Sheffield · MSc Advanced Computer Science · Software Reengineering (Group 10, 2026)**

A practical investigation of the maintainability of **[isort](https://github.com/PyCQA/isort)**, the open-source Python import sorter. We combined repository mining, static and dynamic analysis, targeted architectural restructuring, and regression testing to assess whether a complex configuration subsystem could be made more modular **without changing its public behaviour**.

> **Portfolio archive:** The project source and analysis artefacts are provided in the ZIP file attached to this repository. Extract the archive to browse the Python implementation, Jupyter notebooks and testing configuration. This is an academic group project, not an official isort release.

## At a glance

| Measure | Reported outcome |
| :-- | :-- |
| Target system | PyCQA `isort`, based on commit [`6e70bb60`](https://github.com/PyCQA/isort/commit/6e70bb60c586f83c7e6245ba5e1d98e6a3d12215) |
| Primary architectural issue | Overloaded `Config` class and fragmented skip logic |
| Extracted responsibility components | **5** |
| Baseline unit-test coverage | **99%** (3,062 statements) |
| Post-restructuring unit-test coverage | **99%** (3,277 statements) |
| Automated debloating experiment | **102 lines removed** (~1.0% of 9,946 LOC) |
| Regression outcome | Previously passing baseline tests continued to pass; no new failures reported |

*These are results reported in the accompanying Group 10 project report; coverage percentages describe the selected unit-test suite, not exhaustive verification of all possible behaviour.*

## Research questions

1. Which architectural and maintenance problems are visible through code metrics, repository history and runtime behaviour?
2. Can responsibilities be extracted from `Config` while preserving compatibility with existing callers?
3. Do automated regression tests support the safety of the restructuring?
4. How far can execution-guided debloating reduce the codebase without breaking the selected usage tests?

## 1. Diagnosing the architecture

We investigated `isort` using complementary methods:

- **Repository evolution:** PyDriller-based analysis of **4,462 commits**, including lines-of-code trends and commit categories.
- **Static analysis:** class and method size, cyclomatic complexity, weighted methods per class, cohesion, fan-out, inheritance and potential code smells.
- **Clone analysis:** Jaccard similarity and dot plots to investigate duplicated code at file and method level.
- **Dynamic analysis:** Python execution tracing, call graphs, module-interaction heatmaps and test hit matrices.
- **Potential dead code:** Vulture findings reviewed against runtime traces and dynamic dispatch behaviour.

### The central finding: a `Config` God Class

The report identified the configuration subsystem as the highest-priority restructuring target. Before refactoring, its constructor had **233 lines** and **cyclomatic complexity 46**, while the class showed **WMC 90** and **fan-out 178**. Responsibilities such as loading configuration, resolving patterns, checking file types and applying skip rules were concentrated in one area.

This was not simply a style issue: changes to configuration behaviour could affect many dependent modules, while the skip-related logic was distributed across multiple methods.

## 2. Architectural restructuring

Rather than replacing the public configuration API, we applied **Extract Function**, responsibility extraction and a **delegating-wrapper approach**. The existing `Config` interface was retained where needed, with internal work delegated to focused components.

| Extracted component | Responsibility |
| :-- | :-- |
| `FileTypeChecker` | Supported and blocked file-extension checks |
| `SkipRules` | Skip patterns, ignored paths and related rules |
| `ConfigLoader` | Configuration discovery, loading, merging and validation |
| `PatternMatcher` | Import sections, patterns and associated comments |
| `SortResolver` | Resolution of sorting functions |

The refactoring also addressed practical compatibility issues, including mutable state in configuration objects, circular dependencies and serialisation of extracted providers.

**Design objective:** improve separation of concerns while keeping existing callers and regression tests working.

## 3. Testing and evaluation

We used **pytest** with coverage reporting, together with a **GitHub Actions** regression workflow configured for Python **3.10, 3.11 and 3.12**.

| Evaluation | Before | After |
| :-- | --: | --: |
| Statements in coverage report | 3,062 | 3,277 |
| Missed statements | 25 | 40 |
| Overall reported coverage | **99%** | **99%** |

The report states that tests passing in the baseline continued to pass after restructuring. Three environment-dependent tests were excluded from the regression workflow because they already failed before the changes; the reported outcome should be interpreted with that qualification.

## 4. Execution-guided program debloating

We explored whether code unused by a defined set of public-API scenarios could be removed automatically. An initial AST-based approach caused failures because some functions were referenced across modules or reached through dynamic dispatch.

The pipeline was refined with **cross-module import protection**, **compilation-error rollback** and explicit protections for dynamically accessed functions. In the selected nine-scenario evaluation, the report records **three functions removed** and a **102-line (~1.0%) reduction**. This is a bounded experiment, not proof that the reduced package supports every possible isort use case.

## Implementation and project artefacts

The uploaded ZIP contains the original project folder. Notable paths include:

```text
README.md                       # Original university group README
RepoAnalysis.ipynb              # Repository evolution analysis
staticAnalysis.ipynb            # Static code metrics
code_clone_analysis.ipynb       # Clone investigation
dynamic_analysis.ipynb          # Runtime tracing and analysis
analyse.py                      # Repository mining support
code_clones.py                  # Clone-analysis support
.github/workflows/
  regression-testing.yml        # Automated regression tests
isort_6e70bb60/                # isort snapshot and project changes
```

The archive also contains visualisation outputs such as call graphs, class hierarchy images and code-analysis plots. Some experiments may require additional Python dependencies and environment setup; the ZIP is provided as a research artefact, not a guaranteed one-command installation.

## Limitations

- Results depend on the selected isort snapshot, analysis methods and test inputs.
- Dynamic tracing cannot establish that unexecuted functions are universally unreachable.
- Preserving baseline test results provides useful regression evidence, but does not prove semantic equivalence for every configuration or integration.
- The project was completed collaboratively; individual contributions should be assessed from the relevant code history or supporting coursework evidence.

## Authors and attribution

**Ali Gasimzade · Oliver Smith · Xinlu Zhou**  
University of Sheffield — Software Reengineering, Group 10 (2026)

The underlying **isort** tool is developed and maintained by **[PyCQA](https://github.com/PyCQA/isort)**. Our academic work analyses and modifies a snapshot of that existing open-source project; it does not claim authorship of the original tool. See the upstream project's licence and notices for applicable reuse terms.

---

**Interested in the implementation?** Download the project ZIP from this repository, extract it, and inspect the notebooks, Python modules and regression workflow. The project report provides the full methodology, figures and evaluation details.
