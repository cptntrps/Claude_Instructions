# Worked Example: WORK-1018 — pytest Console Import Failure

**Observed:** 2026-09-16

**Workflow:** [Bug Fix v2](../workflows/bug-fix.md)

**Outcome:** Import defect fixed and published; full suite retained one independent failure

This example records an actual small configuration fix. It explains the decisions
that informed workflow v2. It does not claim that the expanded workflow has been
benchmarked across production incidents, concurrency faults, or data migrations.

## 1. Report and acceptance

The `agents` repository README advertised this test command:

```bash
pytest -q
```

An existing test, `test_obsidian_po_is_registered`, imported the control-plane
package and failed with `ModuleNotFoundError: tools` under the console launcher.
The report also recorded that the Python module invocation worked.

The task-specific acceptance condition was to reach and pass the registration
assertion through the documented console invocation without ad hoc environment
changes. Publication was in scope; no service deployment was needed.

The report's historical count of 117 passing module-invoked tests was a prior
observation. It was not accepted as proof that the current checkout was green.

## 2. Inspect the baseline

The shared checkout contained extensive unrelated edits. Its local `main` was
behind published `origin/main`. The target `pytest.ini` had no existing local edit.

The relevant configuration was:

```ini
[pytest]
testpaths = tests
asyncio_default_fixture_loop_scope = function
```

The project configuration also declared `package = false` for its uv environment.
The repository source was therefore not installed through that project setup.

The first investigation used the canonical checkout. Publication later used a
separate worktree based on the current remote revision. Unrelated work remained
untouched, and only `pytest.ini` entered the published commit.

## 3. Reproduce using the existing test

From the repository root:

```bash
pytest -q tests/test_obsidian_steward.py::test_obsidian_po_is_registered
```

Observed result:

```text
from tools.control_plane.catalog import load_bundle
ModuleNotFoundError: No module named 'tools'
1 failed
```

Compare the other invocation against the same test:

```bash
python3 -m pytest -q tests/test_obsidian_steward.py::test_obsidian_po_is_registered
```

Observed result:

```text
1 passed
```

The existing test already detected the reported failure. A new test checking the
literal contents of `pytest.ini` would not improve behavioral regression coverage.

## 4. Diagnose the difference

The failing test imported `tools.control_plane.catalog` from the repository root.
The console launcher did not make that root available on the Python import path.
The module invocation did, explaining the different outcomes.

The cause was test import configuration, not a broken agent registration. Renaming
the import, weakening the registration assertion, or changing the agent manifest
would address the wrong component.

## 5. Apply the focused correction

Add the repository root to pytest's configured Python path:

```diff
 [pytest]
 testpaths = tests
+pythonpath = .
 asyncio_default_fixture_loop_scope = function
```

No product logic, agent identity, dependency, or test assertion changed.

## 6. Verify and investigate the remaining failure

The targeted console invocation passed after the edit. The documented full command
then produced:

```text
116 passed, 1 failed
```

The remaining failure was:

```text
test_control_plane_cli_has_no_projection_drift
control-plane: FAIL: projection drift: <user-home>/.codex/config.toml
```

This was not dismissed because the patch was small. The comparison disabled only
the new import-path setting through pytest's supported override:

```bash
pytest -q -o pythonpath= \
  tests/test_obsidian_steward.py::test_obsidian_po_is_registered \
  tests/test_smoke.py::test_control_plane_cli_has_no_projection_drift
```

Both tests failed:

- The registration test again failed to import `tools`.
- The control-plane check again reported the same generated configuration drift.

The drift test executed a separate projection-checking process. Its reported cause
was a generated configuration mismatch, independent of this pytest import setting.
This supplied both a baseline comparison and a distinct causal path.

The independent defect was filed as WORK-1021 with an owner, dates, reproduction
details, and acceptance criteria. The active Codex configuration was not rewritten
as an incidental part of this import fix.

## 7. Verify the publication checkout

A separate worktree was created from published revision `eb7fa44`. Before the
edit, the same focused test failed with the same import error. After applying the
one-line correction:

```text
pytest console invocation: PASS
python3 -m pytest invocation: PASS
git diff --check: PASS
Diff: pytest.ini, one insertion
```

These checks verified the target behavior on the publication base. They did not
establish that every test in that worktree passed. The recorded full-suite result
belongs to the canonical checkout and remains explicitly qualified.

The change was committed and pushed to the repository's authorized `main` branch.
The remote branch reference was checked against the intended commit:

[042797d — resolve repository imports in the pytest console launcher](https://github.com/cptntrps/agents/commit/042797d5b9b44c769a5d4373b56e76ddef0f1c02).

## 8. Completion record

```text
Outcome: Import defect verified fixed and published.
Cause: Repository root absent from console pytest's import path.
Change: Add pythonpath = . to pytest.ini.
Before: Existing registration test fails with ModuleNotFoundError: tools.
After: Registration test passes under both launchers on publication source.
Full canonical suite: 116 passed, 1 independent configuration-drift failure.
Follow-up: WORK-1021, assigned for separate diagnosis and reconciliation.
Delivery: Commit 042797d verified on origin/main; no deployment required.
Limitation: Full suite is not green; no claim of complete platform health.
```

WORK-1018 was marked Done with this evidence under the task-specific closure rule.
If green full-suite CI had been a required acceptance or publication gate, the
remaining failure would have blocked that endpoint. A follow-up card would not
override the requirement.

## What this example supports

- Reproducing the exact failing invocation prevented an unrelated code change.
- Comparing two invocations narrowed the cause to startup/import context.
- An existing test supplied meaningful before-and-after evidence.
- The full suite exposed a separate issue without invalidating the targeted fix.
- Baseline comparison justified the separate issue; patch size alone did not.
- An isolated publication checkout preserved unrelated work.
- The completion claim distinguished a fixed bug from a completely green suite.

The expanded v2 procedure is a generalization from these observations and the
framework review. Its broader effectiveness remains to be measured on other bugs.
