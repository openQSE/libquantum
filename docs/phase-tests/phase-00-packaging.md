# Packaging Gate

## Scope

This gate verifies the implementation plan's Packaging section and the
operator requirement that libquantum remain installable, configurable, and
runnable as an ordinary Python distribution.

## Prerequisites

- Python 3.10 or newer
- pip, setuptools, and pytest available to the active Python interpreter
- repository root as the working directory

No SDK, compiler, QFw, DEFw, credential, network connection, allocation, or
reservation is required.

## Fixture Inputs

The gate uses the repository's `pyproject.toml`, an isolated pytest temporary
directory, and an `InitConfig` containing a representative site compiler path,
enabled compiler ID, and compiler initialization configuration.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_00_packaging.py
```

## Assertions and Expected Output

The gate installs the local project without dependencies or build isolation,
starts Python outside the repository, imports libquantum from the isolated
installation, verifies distribution metadata, and constructs the designed
configuration record. It also verifies that build metadata declares no
mandatory runtime dependencies and includes the complete package namespace.

The expected result is all tests passing with no skips or expected failures.

## Cleanup

pytest removes the isolated installation directory after the gate exits.

## Failure Interpretation

A failure means the source tree is not a valid local Python distribution, the
installed module differs from the source package, configuration cannot be
constructed after installation, or the core acquired an undeclared mandatory
integration dependency.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_00_packaging.py`

Execution date: 2026-10-29

Environment: Python 3.10.12, pip 25.2, setuptools 59.6.0, and pytest 9.1.1

Result: `2 passed in 1.28s`
