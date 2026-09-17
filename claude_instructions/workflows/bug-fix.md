# Bug Fix Workflow

**Workflow version:** 2.0.0

**Updated:** 2026-09-16

**Status:** Revised procedure; limited execution evidence, not a benchmarked guarantee

**Owner:** The implementing engineer or agent, subject to the project's authority

Use this workflow to investigate and correct an existing defect. It covers
application behavior, test failures, configuration, integrations, and operational
faults. A bug report is a claim to investigate, not proof of a particular cause.

**Sequence:** establish the contract → reproduce → diagnose → make a focused fix
→ verify → classify remaining failures → publish evidence → close at the agreed
delivery boundary.

## Scope and precedence

For bug-fix tasks, this document replaces this instruction package's older
procedural defaults for testing, coverage, heartbeat reminders, and closure.
That includes conflicting summaries in the main instructions, quick reference,
initialization, modes, and linked testing guides. Consulting a supporting guide
does not silently change the selected bug-fix contract.

Explicit user instructions, applicable project rules, permissions, and enforced
repository or deployment gates still govern. This document cannot waive them.
An existing required CI failure still blocks merge when the repository requires
green CI, even if the failure predates this change.

This is a procedure for a person or agent. Markdown does not install a scheduler,
persist a counter, run tests, or enforce a gate. Load the relevant sections when
needed. Do not claim automatic enforcement without an actual runtime mechanism.

## Completion contract

A completed bug fix must have:

- A specific description of expected and observed behavior.
- Evidence that establishes the defect, or an explicit reproduction limitation.
- A causal explanation supported by observations rather than guesswork.
- The smallest complete correction, including necessary consumer changes.
- Passing verification of the reported behavior and relevant neighboring behavior.
- A result for each required gate: passed, failed, or not run with a reason.
- Classification and disposition of every remaining failure encountered.
- Publication and deployment evidence appropriate to the agreed scope.
- A concise completion record with limitations and follow-up references.

Do not change assertions, delete tests, suppress errors, broaden permissions, or
weaken gates merely to make the result green. Changing an incorrect assertion
requires an independently established behavior contract.

An existing regression test can satisfy the test requirement. Do not add a second
test that only repeats the first or checks the spelling of the implementation.

## Inputs and initial decisions

Collect only information needed for the next decision. Read the issue, comments,
linked evidence, repository instructions, and current state before asking for facts
already available. Continue useful independent work while clarification is pending.

| Input | What to establish | If missing |
|---|---|---|
| Target | Repository, component, version, environment | Locate it from evidence; ask if targets remain ambiguous |
| Symptom | Actual result and user impact | Inspect logs or obtain a concrete example |
| Contract | Expected result and acceptance criteria | Resolve competing interpretations before dependent edits |
| Reproduction | Command, request, fixture, or observed sequence | Construct the smallest safe reproducer |
| Authority | Allowed effects and delivery destination | Continue ordinary authorized work; ask only at an unresolved boundary |
| Ownership | Current assignee and active work | Avoid racing another implementation |
| Delivery | Local fix, submitted PR, merged source, or deployed behavior | Use the request and project conventions; report the boundary explicitly |

Opening task record, when useful:

```text
Issue: <identifier>
Expected: <observable behavior>
Observed: <observable failure>
Target: <repository, revision, environment>
Acceptance: <checks that establish resolution>
Delivery: <required endpoint>
Known constraints: <permissions, data, outage, active work>
```

Use the existing issue or task record. Do not create a second reporting system for
a small fix. Use the project's approved issue interface for updates.

## Choose the investigation depth

Choose depth from impact, uncertainty, and affected boundaries. Line count alone
does not establish risk: one authorization condition can be critical.

| Situation | Appropriate depth | Typical evidence |
|---|---|---|
| Typo or documentation defect | Direct inspection and relevant validation | Corrected output, link check, or rendered result |
| Deterministic code/configuration defect | Focused reproduction and neighboring checks | Same failure before; expected result after |
| Shared library or interface change | Consumer analysis and broader regression checks | Producer and consumer results |
| Authorization or data integrity defect | Isolated fixtures, negative cases, integrity checks | Allowed and denied behavior; preserved data |
| Intermittent or concurrent failure | Controlled timing, load, or repeated observations | Failure rate, conditions, residual uncertainty |
| Production incident | Containment plus recovery and observation | User-facing recovery, rollback readiness, root-cause follow-up |

Do not impose a universal coverage percentage or mutation score. Honor an existing
project threshold, then assess whether the affected behavior is actually tested.
Coverage shows execution; it does not establish that assertions detect this defect.

## 1. Inspect the working state

Before editing, establish the baseline and preserve existing work:

```bash
git status --short
git diff --stat
git diff --cached --stat
git branch --show-current
git rev-parse HEAD
git remote -v
```

Inspect output locally. Do not copy credential-bearing remote URLs or sensitive
diffs into task records. Record revisions and relevant paths, not whole environments.

Identify which changes belong to this task. Never reset, stash, commit, or delete
another person's changes simply to obtain a clean checkout. If the target overlaps
active work, coordinate ownership or use an isolated checkout.

When publication needs current remote source while the shared checkout is dirty:

```bash
git fetch origin
git worktree add -b fix/issue-123 ../issue-123-fix origin/main
```

Use the actual remote and base branch; `main` is illustrative. Keep the original
checkout intact. Check for an existing task branch before creating another.
Private worktrees may lack ignored configuration, dependencies, or generated files;
record those differences rather than misclassifying them as product regressions.

**Output:** a known revision, task-owned paths, and a safe editing location.

## 2. Establish and reproduce the failure

Start with the reported invocation or user path. A nearby command that passes is
useful comparison evidence, but does not by itself reproduce or fix the report.

Record:

1. Working directory and code revision.
2. Relevant dependency/runtime versions and non-secret configuration differences.
3. Input data or fixture identity, redacted where necessary.
4. Exact command or interaction sequence.
5. Expected result, observed result, exit code, and failure location.

Prefer an existing test that already exercises the behavior. Otherwise create a
focused regression test or bounded reproducer before the fix where practical.
Check why it fails: an unrelated import error does not prove an authentication
assertion catches a defect. If the report itself is an import error, that failure
is the relevant observation.

### Reproduction routes

| Evidence available | First useful action |
|---|---|
| Stack trace | Run the failing path; inspect the named frame and callers |
| Broken UI behavior | Repeat the interaction; inspect request and state transitions |
| Wrong data | Build a small fixture with independently known expected results |
| Configuration mismatch | Compare the reported and working invocations |
| Performance regression | Measure comparable input, environment, and timing before editing |
| External outage | Separate local behavior from upstream availability or authorization |

Use test data and approved environments. Do not reproduce a destructive report
against live data when an isolated fixture can answer the question.

### When reproduction is incomplete

Do not invent a successful reproduction. State what was attempted and what is
missing: data, environment, timing, permission, or access to the affected version.

Historical logs, traces, and code analysis may support a diagnosis. Label that
evidence accurately. A plausible correction can be prepared, but do not claim a
verified resolution if a required behavior check remains unavailable.

For an intermittent issue, set a bounded observation plan before repeated runs.
Record attempts, failures, workload, and conditions. One passing retry does not
establish either recovery or a fixed race.

**Output:** a repeatable failure, or a precise statement of the evidence limit.

## 3. Diagnose the cause

Explain the causal chain: input or environment → executing path → incorrect
assumption or operation → observed failure. Avoid starting with the desired patch.

Use short, falsifiable hypotheses:

| Hypothesis | Discriminating check | Interpretation |
|---|---|---|
| Invocation changes imports | Compare console launcher with module invocation | Different outcomes narrow the fault to startup/import context |
| Wrong filter mixes account data | Add two accounts to a fixture | Cross-account totals expose the missing boundary |
| Ordering causes a race | Control execution at the contested boundary | A forced interleaving supports the timing hypothesis |
| External dependency is unavailable | Probe its documented health/auth path without secret output | Distinguishes local implementation from upstream failure |

Change one relevant variable at a time where possible. Read contracts, call sites,
configuration precedence, and existing tests. Inspect dependency code or official
documentation when behavior remains uncertain.

Debug output must be narrow and redacted. Do not dump complete user records,
credentials, connection strings, or authorization headers. Remove temporary probes
after diagnosis, or convert a useful one into appropriate production observability.

Before implementation, answer:

- Why does the failure occur under the reported conditions?
- Why does the proposed correction address that cause?
- Which neighboring behavior could the correction change?
- What observation would disprove the diagnosis?

**Output:** a supported cause and a bounded correction plan.

## 4. Define verification and rollback

Choose checks before changing code. The plan can be a few lines in the task record;
it does not require a separate design document.

### Verification layers

1. **Reported behavior:** the exact reproducer or affected assertion.
2. **Neighboring behavior:** normal use, boundary conditions, relevant failure cases.
3. **Affected consumers:** callers, integrations, alternate invocation paths.
4. **Repository gates:** applicable tests, lint, types, build, policy checks.
5. **Deployed behavior:** smoke checks and observation if deployment is in scope.

Run the full suite when project policy requires it, when shared behavior is affected,
or when it is practical and adds useful regression coverage. Otherwise record why
the selected subset covers the change. Never describe a subset as the full suite.

For a reversible documentation or simple configuration change, an existing check
may be enough. For a behavioral fix, add or strengthen a regression test if existing
coverage does not detect it. Test expected outcomes, not the exact implementation.

Assertions about existence are valid when existence is the contract. Exact values,
authorization outcomes, state transitions, or side effects are needed when those
are the contract. Neither a matcher-name ban nor a coverage badge replaces judgment.

### Rollback proportional to effects

| Change | Rollback or recovery preparation |
|---|---|
| Local source/configuration | Reviewable diff and known prior revision |
| Published source | Revert path consistent with repository policy |
| Deployed service | Previous artifact, compatible configuration, verification plan |
| Persistent data | Tested restore/compensation plan, backup boundary, integrity checks |

A code rollback does not necessarily reverse a data migration. Never assume a
DOWN migration restores lost data or that a backup restore preserves writes made
since the backup. Resolve that boundary before the material effect.

**Output:** a verification plan and an appropriate recovery path.

## 5. Implement the smallest complete correction

Prefer the existing design and the component that owns the defective behavior.
Do not add abstractions, dependencies, fallback paths, or broad refactors unless
the cause requires them.

Smallest complete correction does not always mean one file. A contract change may
require both a producer and its consumers to change in the same unit of work.
Finish those necessary parts; track unrelated improvements separately.

Protect the observed failure evidence. Do not overwrite the baseline result with
a passing result. Do not weaken the reproducer to accommodate the implementation.

If investigation shows the test is wrong, establish the intended contract from an
independent source before editing it. Explain why the original expectation was
incorrect and how the replacement still detects meaningful faults.

Search for the same causal pattern in related paths. Repair directly coupled
instances required for correctness; record independent defects as separate work.
Avoid expanding a login fix into an unsolicited authentication redesign.

### Example: a focused data-filter correction

For this illustrative example, amounts are integer units and rows belong to accounts.

```python
# Before: totals include every account.
def total_for_account(rows, account_id):
    return sum(row["amount"] for row in rows)
```

The reproducer defines the contract before the correction:

```python
def test_total_excludes_other_accounts():
    rows = [
        {"account_id": "a", "amount": 30},
        {"account_id": "a", "amount": -5},
        {"account_id": "b", "amount": 900},
    ]
    assert total_for_account(rows, "a") == 25
    assert total_for_account(rows, "missing") == 0
```

The faulty implementation returns 925 for account `a`. The correction belongs in
the selection logic, not in a test expectation or display-time adjustment:

```python
def total_for_account(rows, account_id):
    return sum(row["amount"] for row in rows if row["account_id"] == account_id)
```

If incorrect totals were persisted, this code fix alone does not repair them.
Investigate affected records and handle any authorized repair as an explicit part
of delivery or a clearly stated unresolved boundary.

**Output:** a focused patch and any necessary regression coverage.

## 6. Verify the correction and its neighbors

Run the same reproducer after the fix, under comparable conditions. Confirm the
expected assertion executes and the original symptom is gone.

Then run the selected neighboring, consumer, and repository checks. Preserve each
command's exit status. An application printing “OK” is not sufficient if its process
exits unsuccessfully, and exit zero alone does not prove correct behavior.

Simple sequential gates should stop on failure:

```bash
# Use the project's actual commands.
npm test && npm run lint && npm run typecheck && npm run build
```

If independent checks are run concurrently, inspect every result. Bare `wait` does
not aggregate child failures. This Bash example preserves them:

```bash
#!/usr/bin/env bash
set -u

pids=()
names=(tests lint types)
npm test & pids+=("$!")
npm run lint & pids+=("$!")
npm run typecheck & pids+=("$!")

failed=0
for i in "${!pids[@]}"; do
  if wait "${pids[$i]}"; then
    printf '%s: PASS\n' "${names[$i]}"
  else
    result=$?
    printf '%s: FAIL (exit %s)\n' "${names[$i]}" "$result" >&2
    failed=1
  fi
done
exit "$failed"
```

Do not parallelize checks that modify shared fixtures, ports, databases, generated
outputs, or process state without isolation. A build may depend on earlier results.

Once required checks pass, stop repeating them unless a new edit, failure, changed
environment, or unresolved concern justifies another run. Re-run affected checks
after any subsequent code change or conflict resolution.

**Output:** actual results tied to the tested revision and environment.

## 7. Classify every remaining failure

A failed check is a finding to investigate. It is not automatically caused by the
patch, and it is not automatically unrelated because the patch is small.

| Classification | Evidence needed | Required action |
|---|---|---|
| Regression caused by the change | Failure depends on this patch under comparable conditions | Fix it and re-run affected checks |
| Existing independent defect | Same failure without this patch, plus a separate causal path | Record evidence and route ownership; keep gate status explicit |
| Environment/dependency failure | Specific unavailable dependency, access, or setup condition | Repair within scope or state what verification remains blocked |
| Nondeterministic failure | Recorded variation under stated conditions | Investigate or bound uncertainty; do not retry until green |
| Stale or incorrect test | Independent contract contradicts the assertion | Correct the test with justification and verify meaningful behavior |
| Unknown | Evidence cannot distinguish causes | Treat verification as unresolved; continue investigation or report a blocker |

### Establishing an independent baseline failure

Prefer a clean checkout of the relevant baseline, with the same dependencies and
test inputs. Alternatively, disable only the proposed setting through a supported
test override, or compare an isolated copy with and without the patch.

Do not revert other people's working changes to obtain a baseline. Record any
environment differences that weaken the comparison. “It was already broken”
requires evidence, not a comment or assumption.

Example from WORK-1018:

```text
Without import-path fix:
  Registration test: FAIL, ModuleNotFoundError: tools
  Control-plane smoke check: FAIL, generated Codex configuration drift

With import-path fix:
  Registration test: PASS
  Full suite: 116 passed, 1 failed
  Remaining failure: same generated configuration drift
```

The correct statement is “the import defect is fixed; the suite still has one
independent failure.” Neither “everything passes” nor “the import fix failed”
accurately describes this evidence.

### When closure with an independent failure is allowed

All of the following must hold:

1. The reported defect's acceptance checks pass.
2. The remaining failure is independently reproduced and causally separated.
3. It does not prevent required verification of this fix.
4. It does not block the agreed delivery under project policy.
5. It has an owner and disposition, with a linked issue when follow-up is needed.
6. The completion record reports the failure and its effect on confidence.

If the task explicitly requires a green full suite, a failure anywhere in that
required suite leaves the task incomplete. Creating a follow-up issue does not
waive acceptance or make a required gate pass.

## 8. Review and publish the actual change

Review the final diff for correctness, scope, secrets, temporary probes, and
unintended changes. Confirm that the evidence still matches the final code.

```bash
git diff --check
git diff -- path/to/changed-file path/to/relevant-test
git add -- path/to/changed-file path/to/relevant-test
git diff --cached --stat
git diff --cached
git commit -m 'fix(component): describe the corrected behavior'
```

Use actual task-owned paths. Check for existing staged changes before staging; do
not commit someone else's index contents. Use an isolated worktree when necessary.
Let repository hooks run. Do not bypass them to hide a failed check.

Publish using the established project path: PR, authorized direct push, or another
documented delivery mechanism. A commit is not proof of a successful push; a push
is not proof of merge; merge is not proof of deployment.

If the remote moved, fetch and integrate the new base safely. Review conflicts and
re-run checks affected by integration. Never force an unrelated branch backward.
Confirm the destination contains the intended commit and record its identifier.

### Independent review

Use independent review when the user or project requires it, or when a specific
substantial risk cannot be adequately resolved by the builder's checks. State the
risk and the exact question the reviewer must answer.

File count, use of a lifecycle workflow, and closure itself are not sufficient
reasons to add an approval stage. Simulating several reviewer personas in the same
session does not establish independent verification.

## 9. Verify deployment when deployment is in scope

Use the project's deployment procedure and authorized access. Validate the target
environment, artifact revision, configuration, migration compatibility, and recovery
path before applying material changes.

After deployment, test the affected user path, relevant health signals, and error
behavior. Observe for a duration justified by the failure mode. A successful restart
or health endpoint alone does not verify a corrected business operation.

If the change causes a regression, stop promotion and use the prepared recovery
path. Verify the recovered state. Record what was rolled back and what persisted.

Do not claim deployed resolution for a source-only change. Conversely, do not add
a deployment to a task whose authorized endpoint is a local patch or submitted PR.

## 10. Record evidence and close honestly

Use the existing issue, PR, or task artifact as the canonical completion record.
Record facts concise enough to review without replaying the conversation.

```text
Outcome: <verified fixed / implemented but verification blocked / mitigated /
          unresolved / rolled back>
Issue and cause: <symptom and supported cause>
Change: <what changed and why>
Before: <reproducer, failure, baseline revision>
After: <same check, result, final revision>
Regression checks: <scope and actual passed/failed/not-run results>
Remaining failures: <classification, evidence, owner, follow-up>
Delivery: <local / PR submitted / merged / deployed, with reference>
Recovery: <available or performed, when relevant>
Limitations: <what the evidence does not establish>
```

For a small fix, this can be a short paragraph. Keep detailed logs only when they
support a material finding; store them through the project's approved, redacted
evidence path. Do not publish credentials or unrelated personal data.

| Outcome | Meaning | Close the defect? |
|---|---|---|
| Verified fixed | Acceptance and agreed delivery completed; remaining failures satisfy Section 7 | Yes, under project policy |
| Implemented but verification blocked | Patch exists; required evidence is unavailable | No |
| Mitigated | Impact reduced; cause or durable correction remains | Keep defect/root-cause work open; incident recovery may close separately |
| Unresolved | Cause or correction not established | No |
| Rolled back | Attempted change reversed and recovery checked | No, unless the authorized resolution was the rollback itself |

An authorized builder can record evidence and close their own completed issue.
Do not invent a mandatory separate verifier or queue-manager approval. Honor an
explicit project requirement where one actually exists.

## Practical checkpoints and interruption recovery

Use checkpoints at decisions, not a guessed message counter:

- **Before editing:** expected behavior, baseline, cause, and scope are clear.
- **Before publication:** required results are available and all failures classified.
- **Before closure:** delivery and remaining limitations match the claim.
- **After interruption:** reload the task record, inspect repository state, and
  reconcile prior evidence before resuming.

Do not replay every completed check automatically after interruption. Repeat checks
whose inputs changed or whose recorded results cannot be trusted. Do not treat a
stale summary as proof that a command ran or a deployment succeeded.

Provide progress updates when diagnosis, scope, or confidence changes. Report the
fact and the next discriminating check. Routine framework-status announcements
do not substitute for investigation.

## Special cases

### Production incident

Separate immediate containment from durable correction. Within existing authority,
consider rollback, disabling the affected path, or another reversible mitigation.
Preserve enough evidence to diagnose the cause without prolonging user impact.

Do not require a new automated test before an urgent authorized rollback. Verify
recovery immediately, then reproduce and test the durable correction safely.
An emergency does not authorize arbitrary data loss, secret exposure, or bypassing
the user's deployment permissions.

### Intermittent race or performance regression

Prefer deterministic synchronization or a captured workload over arbitrary sleeps.
For performance, compare the same data, resources, warm-up, and measurement method.
Record sample counts and distributions when meaningful. Passing a few runs narrows
uncertainty; it does not prove the defect can never recur.

### No useful automated test

A hardware interaction, visual defect, documentation error, or unavailable external
environment may need bounded manual verification. Specify the steps, expected
observable result, observed result, and remaining limitation. Do not manufacture a
trivial automated assertion solely to claim test coverage.

### Existing test already catches the bug

Use it as the regression check. Add coverage only for a distinct risk or missing
boundary. Configuration fixes such as WORK-1018 often need no new test file.

### Report appears incorrect

Compare the report with the authoritative contract and observed behavior. Explain
the discrepancy with evidence. Do not change working behavior simply to satisfy an
unsupported interpretation. Record the resolution according to project policy.

## Common failure patterns to avoid

| Pattern | Why it is inadequate | Better action |
|---|---|---|
| Patch the first suspicious line | No causal evidence | Reproduce and test a discriminating hypothesis |
| Re-run until green | Hides nondeterminism | Record variation and investigate the conditions |
| Weaken an assertion | Can conceal the original defect | Establish the contract independently |
| Add a test that only checks config text | Does not verify the reported behavior | Exercise the actual invocation or outcome |
| Demand whole-repository cleanup | Expands scope without proving necessity | Separate coupled fixes from independent findings |
| Call an unrelated failure “pre-existing” | Assumes independence | Reproduce it without the patch |
| Declare success after bare `wait` | Loses individual failure status | Inspect every concurrent command's result |
| Treat a heartbeat as enforcement | Depends on the same agent remembering it | Record real checks at meaningful decisions |
| Equate passing tests with proof of all behavior | Overstates test coverage | Name the exercised behavior and evidence limits |
| Close after commit when merge is required | Stops before agreed delivery | Verify the actual destination and gate results |

## Decision scenarios for reviewing this workflow

These are expected decisions, not claims that all scenarios were executed.

| Scenario | Expected decision |
|---|---|
| Console pytest fails; module invocation passes | Compare import context; fix the documented invocation; verify both |
| Focused test passes; independent optional check still fails | Establish baseline, track the failure, evaluate closure conditions |
| Same situation but full green CI is required | Keep merge/closure blocked; follow-up does not waive the gate |
| A new related test fails only with the patch | Treat it as a regression and fix before publication |
| Test passes against previously untested correct behavior | It may be valid characterization coverage; assess fault detection |
| A bug reproducer fails only because its fixture is broken | Repair the fixture before claiming the product defect is reproduced |
| Another agent has dirty overlapping changes | Isolate or coordinate; do not stash or overwrite their work |
| Production users lose access after release | Use authorized containment, verify recovery, preserve root-cause work |
| Reproducer requires unavailable hardware | State verification blocked; do not mark verified fixed |
| Pass count rises but targeted defect still reproduces | Continue; aggregate counts do not satisfy acceptance |
| Remote base changes after successful tests | Integrate safely and re-run checks affected by the new base |
| Same agent role-plays three reviewers | Treat as self-review, not independent evidence |

## Final checklist

- [ ] Expected and observed behavior are explicit.
- [ ] Baseline, reproduction, and evidence limits are recorded.
- [ ] The causal explanation supports the correction.
- [ ] Unrelated work and sensitive data are preserved.
- [ ] The patch is complete for its scope and introduces no known regression.
- [ ] The reported path and relevant neighboring behavior pass verification.
- [ ] Every required gate has an honest result.
- [ ] Remaining failures have evidence, classification, and disposition.
- [ ] No required acceptance or delivery condition is waived by a follow-up.
- [ ] Publication/deployment reaches the agreed endpoint.
- [ ] The completion record matches the evidence.

## Worked example and revision history

See [WORK-1018: pytest console import failure](../examples/bug-fix-example.md)
for the observed small-fix execution that informed this revision.

**2.0.0 — 2026-09-16:** Added explicit baselines, proportional verification,
independent-failure classification, publication boundaries, interruption recovery,
and closure rules. Replaced message-count heartbeat requirements with operational
checkpoints. Corrected assumptions about matcher names and universal coverage.

The core sequence has evidence from one small configuration defect. The expanded
incident, data-repair, and concurrency guidance remains guidance; its inclusion
is not evidence of end-to-end validation in those settings.
