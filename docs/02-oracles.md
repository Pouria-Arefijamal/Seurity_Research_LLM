# 02 — The oracle problem

## 1. Definition

An **oracle** is the decision procedure that answers: *"is this observed behavior a bug?"*

Tests and fuzzers generate inputs and executions. The oracle turns a run into a finding. Without an oracle, a triggered bug is invisible. This is why the oracle, not input generation, is the real bottleneck of security testing.

## 2. Oracle taxonomy

| Oracle | Decides by | Example |
|---|---|---|
| Crash / exception | process dies or throws | malformed input crashes parser |
| Sanitizer | ASan/UBSan/TSan/MSan flags UB, memory, race | race on session store |
| Assertion / invariant | a stated property holds | "user must be authenticated here" |
| Expected-value | output equals a specified value | wrong password returns 401 |
| Differential | two implementations/configs disagree | interpreter vs. JIT (FuzzJIT) |
| Metamorphic | a relation between outputs holds | doubling input doubles result |
| Property-based | random inputs checked against properties | "A never sees B's data" |
| Timing / statistical | measurable timing delta | valid vs. invalid user differ |
| Out-of-band (OAST) | a callback arrives | blind SSRF hits our listener |
| Side-effect | state changed outside the response | DB row written, file created |
| Differential access | principals see different data | IDOR: A reads B's record |
| Tripwire / canary | a secret surfaces | API key in an error page |
| Taint / instrumentation | taint reaches a sink | IAST/RASP flags unserialize |
| Human expert | judgment against known attack classes | reviewer recognizes timing flaw |

The human oracle matters: a **new attack class is a new oracle**. The field keeps "discovering" old code because oracles improve.

## 3. How a new oracle is shaped

Four required pieces:

1. **A security property (spec).** E.g., "a timing side channel must not reveal password content."
2. **An observable signal** correlated with violation: response diff, timing, OOB callback, state change, sanitizer report.
3. **A threshold / tripwire** that separates violation from noise: e.g., timing delta > 3 sigma across many samples.
4. **Calibration** on known-good and known-bad cases, tuned for **sensitivity** (catches real bugs) and **specificity** (does not flood with false positives).

Process: *property → signal → harness (setup/teardown) → threshold → validate.*

## 4. Worked examples of invented oracles

- **Spectre** — no crash, no wrong output. Oracle: cache-timing measurement revealing secret-dependent access.
- **Blind SSRF** — no response. Oracle: out-of-band DNS/HTTP callback.
- **Username enumeration** — oracle: statistical response-time or message difference between existing/non-existing accounts.
- **IDOR** — oracle: differential access (A requests B's object).
- **Race conditions** — oracle: invariant violation under concurrency, or TSan.
- **Deserialization RCE** — oracle: side effect (command output) or OOB callback, **not** a crash.
- **Prototype pollution** — oracle: property appears on a shared prototype / downstream behavior change.
- **Prompt injection** — oracle: canary leaks or model follows an attacker instruction.

## 5. Why this explains "the fuzzer missed it"

A payload like a PHP object-injection gadget chain requires:
1. a valid serialization format with exact length prefixes,
2. existing class names,
3. a specific nested property graph,
4. a multi-class chain of magic methods to a dangerous sink.

A crash-oracle fuzzer fails on all four:
- random bytes almost never form a valid typed object graph;
- success is `system('id')` returning HTTP 200 — **no crash, no signal**;
- the sink is already covered, so **no coverage reward** to craft the exploit;
- reaching the sink needs auth/state and a semantic construction, not mutation.

The LLM instead **anchors on the sink and walks backward** across the class graph — a search over a small, finite, readable structure, not the infinite runtime state space.

## 6. LLMs as oracle generators

An oracle is "a spec of safe behavior + a way to observe its violation." LLMs are good at both:

1. **Spec mining** — infer intended invariants from code, comments, tests, docs.
2. **Signal selection** — choose a violation observable.
3. **Harness synthesis** — write executable PASS/FAIL code.
4. **Calibration** — run on known-good and known-bad inputs.

Example (session fixation):

```python
def test_session_rotates_on_login(session):
    pre = session.cookies.get("SID")
    session.post("/login", data={"u": "alice", "p": "correct"})
    post = session.cookies.get("SID")
    assert post and post != pre, "session not rotated; fixation risk"
```

LLMs can also **be** the oracle ("model-as-judge") for fuzzy properties like PII leakage, but this is non-deterministic and gameable; it should back up deterministic checks, not replace them.

## 7. Honest limits

- "New oracle" from an LLM is usually **recombination** of known oracle patterns, not invention from nothing.
- If the **spec is wrong**, the oracle encodes the bug as "expected."
- Hallucinated invariants produce false positives; non-determinism produces flaky checks.
- Source-only reasoning cannot **confirm**; execution and calibration are required.
- A genuinely novel attack class with no known oracle pattern may still be missed.

The invariant: **LLM proposes the oracle, execution validates it, calibration keeps it honest.**
