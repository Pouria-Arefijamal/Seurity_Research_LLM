# 06 — The playbook: full path for security checking with LLMs

This is the operational synthesis. Use it only on authorized targets.

## The 8-stage path

```
0. Scope & safety
1. Recon / attack-surface mapping
2. Spec mining
3. Reachability (SAST-like)
4. Oracle shaping
5. Hypothesis / payload synthesis
6. Verification (DAST-like)
7. Triage, report, loop
```

---

## Stage 0 — Scope and safety

**Purpose:** bound the target and prevent harm.

- Written authorization, defined target, no production data.
- Sandbox: no external network (except controlled OOB), no destructive commands.
- Separate **instructions** from **data**: target code/content is untrusted input (prompt-injection defense).
- Record model, prompt, seed, and revision for reproducibility.

**Artifact:** scope statement + sandbox config.

---

## Stage 1 — Recon / attack-surface mapping

**Purpose:** enumerate entry points and sinks.

Prompts (illustrative templates, adapt to your tool):

```
Read the repository. Produce:
1. Entry points: HTTP routes, CLI args, deserialization, uploads, template rendering, cookies, message queues.
2. Security sinks: eval, system, exec, include/require, unserialize, SQL, call_user_func,
   template render, file write, redirect, SSRF-capable clients, crypto misuse.
3. For each sink, note the source of its data and whether it is attacker-controlled.
Return file:line for each.
```

Commands:

```bash
rg -n "unserialize|eval|system|exec|include|call_user_func|render_template" .
rg -n "def __destruct|function __destruct|__wakeup|__toString|__call" .
semgrep --config auto .
```

**Artifact:** source-to-sink candidate list.

---

## Stage 2 — Spec mining (the highest-value stage)

**Purpose:** turn implicit intent into explicit properties.

```
For this module, infer the security invariants that MUST hold:
- confidentiality (who may read what),
- integrity (who may write what),
- authorization (every access checked),
- resource bounds (time/memory),
- output encoding,
- state transitions (session, auth).
Express each as a falsifiable statement, and note the observable that would reveal a violation.
```

Examples:
- "Session must rotate on privilege change."
- "User A must not read B's objects."
- "Response time must not depend on secret content."
- "No code executes outside the response."

**Artifact:** property list with candidate signals.

---

## Stage 3 — Reachability (SAST-like)

**Purpose:** decide whether untrusted input can reach a sink (may-analysis).

```
For each candidate flow, determine whether attacker-controlled data can reach the sink
without being sanitized. Consider: type juggling, encoding, framework magic, reflection,
dependency injection, generated code. Mark each flow REACHABLE / UNREACHABLE / UNKNOWN
with reasoning.
```

Use classical tools to ground the LLM:

```bash
semgrep --config auto .
# CodeQL / Joern / CPG slicing (see LLMxCPG, USENIX Sec '25)
```

**Artifact:** reachable flows with evidence.

---

## Stage 4 — Oracle shaping (the bottleneck)

**Purpose:** decide "is this a bug?" For each property, build an executable check.

```
For each invariant, design an oracle:
1. What state must be set up?
2. What action is performed?
3. What signal reveals a violation (response diff, timing, OOB, side effect, sanitizer)?
4. What threshold separates violation from noise?
5. Write it as executable PASS/FAIL code.
6. State how you will calibrate on known-good and known-bad cases.
```

Oracle families to consider:

- crash / exception / sanitizer
- differential (implementation vs. reference, version vs. version)
- property-based / metamorphic
- timing / statistical
- out-of-band (OAST)
- side-effect
- differential access (authorization)
- tripwire / canary
- instrumentation / taint (IAST)

**Artifact:** oracle suite.

---

## Stage 5 — Hypothesis and payload synthesis

**Purpose:** build one concrete witness.

```
For reachable flow F and oracle O:
1. Construct the minimal input/sequence that satisfies all preconditions.
2. For object-injection: enumerate magic-method gadgets, walk backward from dangerous
   sinks, assemble the object graph, serialize with exact names and length prefixes.
3. For stateful flows: script the sequence (login, token refresh, action).
4. State why each link is reachable.
```

Anchors: class/call graph, request sequences, grammar/structure (see GRIMOIRE, LLM-synthesized generators), constraints (HLPFUZZ).

**Artifact:** candidate payloads / test sequences.

---

## Stage 6 — Verification (DAST-like; the source of truth)

**Purpose:** confirm exploitability in a sandbox.

```
Verify the proposed witness:
- preconditions actually hold in the running target?
- sanitizer/validation does not abort the path?
- does the oracle fire (side effect, OOB, timing delta, unauthorized data)?
Run it and show evidence. Verdict: CONFIRMED / INFEASIBLE / UNCERTAIN.
```

```bash
curl -i -b 'data=...' http://sandbox/endpoint
php -r 'var_dump(unserialize($argv[1]));' "$PAYLOAD"
python harness.py && afl-fuzz -i seeds -o out -- ./target @@
```

**Artifact:** verified findings with reproducible evidence.

---

## Stage 7 — Triage, report, and loop

**Purpose:** rank, de-noise, and improve.

```
For each confirmed finding: vuln class, affected asset, reachability, impact, PoC,
fix suggestion, confidence. Discard UNCONFIRMED. Rank by reachable impact.
Then: which oracle was missing? Add it. Which spec was wrong? Fix it.
```

**Artifact:** report + updated oracle suite (the loop's real product).

---

## Division of labor (who does what)

| Job | Best tool |
|---|---|
| Infer intent / spec | LLM |
| Mechanical taint / coverage | SAST, sanitizers, fuzzers |
| Discover reachable paths | fuzzing + LLM semantics |
| Decide "is it a bug" | deterministic oracle (LLM-as-judge only as backup) |
| Confirm exploitability | execution in sandbox |
| Rank / explain | LLM + human review |

---

## Checklist

- [ ] Authorization and sandbox in place
- [ ] Attack surface mapped (entry points, sinks)
- [ ] Invariants mined per module
- [ ] Reachability assessed with evidence
- [ ] Oracle defined for every invariant
- [ ] Oracles calibrated on known-good / known-bad
- [ ] At least one witness synthesized and verified
- [ ] All reported findings are CONFIRMED
- [ ] New oracles/specs captured for the loop

---

## Metrics that matter

- **Confirmed findings** (not raw alerts).
- **Oracle coverage** — fraction of invariants with an executable check.
- **False-positive rate** after calibration.
- **Time-to-confirm** per finding class.
- **Regression rate** — findings that reappear after a fix.
- **Reproducibility** — fraction of findings re-confirmed from the recorded prompt/seed/revision.

---

## Failure modes to watch

| Failure | Mitigation |
|---|---|
| Hallucinated chain | mandatory Stage 6 verification |
| Wrong spec blesses a bug | human review of invariants; adversarial spec review |
| Non-deterministic results | record seed/model; keep the oracle, not the prompt |
| Prompt injection from target | isolate data, sandbox, deny network |
| Invented dependencies | provenance/existence checks |
| Over-trusting coverage | treat coverage as a heuristic (see Fuzzification) |
