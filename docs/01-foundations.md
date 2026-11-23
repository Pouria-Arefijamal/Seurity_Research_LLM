# 01 — Foundations: static vs. dynamic, and why "all states" is the wrong frame

## 1. The two questions that get confused

Security testing is asked to answer one of two very different questions:

- **Detection (existential):** "Is there *one* reachable state that violates a security property?" — needs a single witness.
- **Verification (universal):** "Do *all* reachable states satisfy the property?" — needs coverage of everything.

Nearly every practical tool (SAST, DAST, fuzzing, LLM review) answers the **existential** question approximately. Only **formal verification** attempts the **universal** one, and only for a bounded *model*, under explicit *assumptions*, relative to a *specification*.

This single distinction resolves most confusion in the field:

| Claim | Actually means |
|---|---|
| "We fuzzed it for a week" | "Under this oracle, on sampled inputs, at this revision, no bug was detected." |
| "Coverage is 100%" | "Every *line* was executed at least once" — not every path, state, or interleaving. |
| "It is formally verified" | "Property P holds for model M under assumptions A" — not "the deployed binary is safe." |
| "The LLM found nothing" | "No hypothesis survived verification" — not "no bug exists." |

## 2. SAST: approximation of state

**Static Application Security Testing** reads source/bytecode/binaries **without running** the program.

How it works:
- parse to an AST;
- build control-flow and data-flow graphs;
- apply pattern rules, taint tracking (does untrusted input reach a sink?), semantic rules;
- optionally use **symbolic execution**, **abstract interpretation**, or **model checking** to approximate execution paths.

What it can and cannot do:
- It has no real heap, I/O, threads, clock, network, configuration, or environment.
- Full path/state enumeration is **undecidable** (halting problem / Rice's theorem) and exponential, so it *must* approximate using abstractions (points-to, summaries, widening).
- Consequences: **false positives** (over-approximation counts infeasible paths) and **false negatives** (under-approximation or invisible code such as reflection, generated code, native blobs).

So SAST is **broad but imprecise**, and it never proves exploitability or reachability in the deployed system.

## 3. DAST: sampling of real state

**Dynamic Application Security Testing** drives the **running** application from outside.

How it works:
- crawl or record real traffic;
- maintain session state (cookies, tokens);
- script multi-step workflows (macros);
- inject payloads and observe responses, timing, errors, side effects.

Because it executes the real server, there is **no internal state model to get wrong** — the server *is* the state machine. It is **precise but partial**: it only tests flows it can discover or that you script. Hidden endpoints, states behind auth, and destructive actions remain untested.

### How "a request between two clicks" is handled
DAST does not model it — it *performs* it:
1. A recorder/browser captures the actual HTTP requests produced by the UI.
2. Session and CSRF tokens are preserved so injections land in the same authenticated context.
3. Macros/sequences replay the ordered steps (login → cart → checkout) and attack each step.
4. Stateful scanning correlates and replays request sequences.

The limit: coverage equals what is discovered or scripted. Modern research (e.g., Atropos, USENIX Security 2024, see Chapter 03) improves this with **snapshots** of application state so the fuzzer can return to a state and explore deeper, rather than relying on fragile replay.

## 4. Fuzzing: sampling under an oracle

Fuzzing is dynamic testing that feeds malformed/random/structured inputs and watches for a violation signal.

- **Black-box:** no instrumentation.
- **Grey-box / coverage-guided:** instrumented target; coverage feedback steers toward new paths.
- **White-box / concolic:** symbolic constraint solving plus concrete runs — where fuzzing merges with static analysis.
- **Mutation vs. generation:** mutate seeds, or generate grammar-valid inputs.

A fuzzer needs two things beyond inputs:
1. a **running target** (or in-process harness), and
2. an **oracle** (crash, sanitizer, assertion, differential, side effect).

Coverage-guided fuzzing explores *paths*, not all inputs; it stops when coverage plateaus. It is excellent for memory-safety and parser bugs with crash/sanitizer oracles, and structurally weak for **logic and chain-based** bugs with no crash signal (see Chapter 02).

## 5. Formal verification: the universal side

Formal verification (model checking, theorem proving such as in Lean/Coq, refinement) can prove **properties** about a **model**.

The gaps that let bugs survive even a machine-checked proof:
1. **Specification gap** — the code matches a spec that omits the security property (or encodes the wrong one).
2. **Model/refinement gap** — you verify a model, not the compiled artifact; the TCB (compiler, runtime, FFI, OS, hardware) is outside the proof.
3. **Assumption gap** — axioms, trusted components, crypto assumptions, attacker model.
4. **Side-channel gap** — timing, cache, power, speculative execution are usually out of scope.
5. **Environment/operational gap** — config, secrets, dependencies, IAM, human operation.
6. **Maintenance gap** — the proof tracks one revision; code and deps drift.

Verification is **scope-narrowing**: it raises confidence in a property but relocates risk to spec, TCB, assumptions, side channels, and operations.

## 6. The state-space reality (the core mental model)

| Approach | Question | Method | Complete? |
|---|---|---|---|
| Formal verification | universal | exhaust a *model* | only relative to spec/model |
| SAST | existential | approximate all paths | no |
| Fuzzing | existential | sample executions, oracle | no |
| DAST | existential | sample live states, oracle | no |
| LLM review | existential | reason over source/graph + oracle | no |

No practical method enumerates the whole runtime state space. Detection needs **one witness**; verification needs **everything**, and even then only about a model.

## 7. Why "new" bugs appear in unchanged code

A later finding is almost always one of:

1. **Newly introduced** — code, feature, refactor.
2. **Newly reachable** — new integration, route, auth path.
3. **Newly understood** — a new attack class (timing, type juggling, speculative execution, prompt injection) becomes a recognized property.
4. **Newly detectable** — a better **oracle** now exists for states that were always reachable.

Category 4 is the one people underestimate: much of "new vulnerability research" is **new oracle research**.

## 8. Where LLMs fit

LLMs add **semantics** at three points — spec mining, oracle shaping, exploit reasoning — while execution supplies truth. They do not escape the coverage/decidability wall; they make the *search over meaning* cheaper. The detailed argument is in Chapter 04.
