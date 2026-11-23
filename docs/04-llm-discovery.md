# 04 — How LLMs find vulnerabilities (and what they cannot do)

## 1. The one-sentence model

**An LLM does semantic, probabilistic static analysis, synthesizes oracles, and reasons out exploit paths — but it does not prove absence, enumerate states, or confirm findings without execution.**

It sits between classical SAST and a human pentester:

| Layer | Classical SAST | LLM review | Human pentester |
|---|---|---|---|
| Reachability | graph algorithms | semantic may-analysis | experience |
| Exploit reasoning | none | chain/payload synthesis | yes |
| Oracle shaping | fixed rules | spec mining + harness synthesis | yes |
| Confirmation | none | via tools/execution | yes |
| Determinism | high | low | medium |
| Coverage guarantee | none | none | none |

## 2. Why it does not need "all states"

- Finding is **existential**: one reachable witness suffices.
- The relevant structure is **small and finite**: classes, methods, magic methods, properties, sinks — an attack graph, not the runtime state space.
- Reachability is a **may-analysis** over data flow, not an enumeration of executions.
- Priors **prune** the search: the model recognizes the *shape* of known bug classes from training data.

So "it cannot know all states" is true and irrelevant: nobody needs all states to find a bug.

## 3. What it actually contributes (three roles)

### Role A — Spec mining
Infer intended invariants from code, comments, tests, and docs:
- "the session must rotate after login,"
- "user A must never read B's record,"
- "response time must not depend on secret contents,"
- "no code outside the response may execute."

### Role B — Oracle synthesis
Turn each invariant into an executable check: property → signal → harness → threshold → calibration. See Chapter 02.

### Role C — Exploit reasoning and synthesis
Anchor on a sink and walk backward:
1. enumerate magic methods / dangerous callbacks,
2. find edges to dangerous sinks,
3. assemble a valid object graph / request sequence,
4. synthesize a concrete payload,
5. propose a verification.

This is why an LLM reaches a deserialization → gadget-chain → RCE that a crash fuzzer cannot: it searches the **class graph for a semantic construction**, while the fuzzer searches **bytes for a crash**.

## 4. The architecture that works: hybrid, graph-guided, verified

```
        +-------------------+
        |  Code Property    |
        |  Graph / call     |----> slice (LLMxCPG-style) ---+
        |  graph            |                               |
        +-------------------+                               v
                                                    +-------------+
  docs/specs/tests ----> spec mining -------------> |    LLM      |
                                                    |  reasoner   |
  prior crashes/SAST ---> context -----------------> +-------------+
                                                          |
                              oracle + payload synthesis  |
                                                          v
                                            +--------------------------+
                                            | executor / fuzzer / DAST |
                                            | (ground-truth signal)    |
                                            +--------------------------+
                                                          |
                                                          v
                                                 verified findings
```

Key design choices backed by the USENIX literature (Chapter 03):

- **Slice before you prompt.** Context limits are solved with program-analysis slicing, not bigger prompts (LLMxCPG, USENIX Sec '25).
- **LLM writes structure; a real fuzzer explores.** LLM-synthesized input generators plus AFL++ (USENIX Sec '25); LLM plus crash reuse (USENIX Sec '24).
- **LLM solves hard input constraints.** HLPFUZZ (USENIX Sec '25) uses the model where symbolic solving is expensive.
- **LLM infers specs across heterogeneous code.** PANGOLIN (USENIX Sec '26) uses an agent for cross-language parameter specs.
- **LLM handles non-crash bugs.** mGPTFuzz (USENIX Sec '24) derives semantic oracles for application-layer bugs.

## 5. What an LLM cannot do

1. **Prove absence.** No amount of prompting proves "no bug exists."
2. **Enumerate states.** It samples hypotheses over a compressed model.
3. **Guarantee reproducibility.** Output is non-deterministic; keep the oracle, not just the prompt.
4. **Avoid hallucination.** It can invent infeasible chains, non-existent classes, and non-existent packages (see "Package Hallucinations," USENIX ;login:).
5. **Escape a wrong spec.** If the invariant is wrong, the oracle blesses the bug.
6. **Be trusted without execution.** Confirmation must be dynamic or by rigorous review.
7. **Be assumed safe.** The model is an attack surface (unsafe code generation per "Lost at C," USENIX Sec '23; jailbreaks per USENIX Sec '25; prompt injection from untrusted targets).

## 6. Controlling the risks

- **Sandbox everything.** No network, no destructive commands, isolated filesystem.
- **Separate instructions from data.** Treat target code/content as untrusted input, never as instructions (prompt-injection defense).
- **Deterministic verification.** A finding counts only if an executable oracle confirms it.
- **Provenance checks.** Verify that referenced packages/files/classes exist.
- **Calibration set.** Maintain known-good and known-bad cases to catch false positives and flakiness.
- **Human review** for high-impact or ambiguous findings.

## 7. Bottom line

LLMs lower the cost of the three hardest semantic tasks — **specification, oracle shaping, exploit reasoning** — and therefore find some bugs that crash-oracle fuzzers structurally cannot (logic, chains, multi-step, non-crash application bugs). They do not change the epistemology: **detection is existence, verification is universal, and truth comes from execution.**
