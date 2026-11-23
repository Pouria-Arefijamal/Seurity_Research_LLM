# 03 — What USENIX Security methods contribute

This chapter surveys relevant USENIX Security work (2019–2026) and extracts the transferable method from each: **how they get a better oracle, a better input model, or better state coverage.** Links and full citations are in [`../references.md`](../references.md).

> Note: 2026 entries are pre-publication/preprint according to USENIX. Claims are summarized at the level of the published abstracts.

## Summary table

| Work | Venue | Core lever | Transferable lesson |
|---|---|---|---|
| GRIMOIRE | Sec '19 | structure synthesis from coverage | valid structure is a prerequisite for deep inputs |
| Fuzzification | Sec '19 | anti-fuzzing | coverage feedback is an attackable signal |
| MUZZ | Sec '20 | thread-aware grey-box | concurrency is a separate state dimension |
| FuzzJIT | Sec '23 | differential oracle | "correct" needs a reference to compare to |
| Fuzztruction | Sec '23 | fault-injection fuzzing | reuse the producer to reach consumer states |
| Atropos | Sec '24 | stateful web fuzzing via snapshots | model *state*, not request replay |
| mGPTFuzz | Sec '24 | LLM extracts spec from docs | non-crash bugs need semantic oracles |
| BusyBox + LLM | Sec '24 | LLM + crash reuse | prior crashes seed new targets |
| LLMxCPG | Sec '25 | CPG slicing + LLM | reduce code to vulnerability-relevant context |
| HLPFUZZ | Sec '25 | LLM constraint solving | LLMs can solve hard input constraints |
| Non-textual fuzzing | Sec '25 | LLM synthesizes input generators | grammar-aware generation at scale |
| PANGOLIN | Sec '26 | LLM cross-language agent | infer parameter specs across languages |
| AIxCC SoK | Sec '26 | autonomous cyber reasoning | end-to-end find-patch-validate systems |
| Lost at C | Sec '23 | LLM code assistant study | LLMs also *introduce* vulnerabilities |
| Package hallucinations | ;login: | LLM invents packages | supply-chain risk of LLM output |

---

## A. Oracles and structure

### FuzzJIT — Oracle-Enhanced Fuzzing for JavaScript Engine JIT Compilers (USENIX Security 2023)
- **Problem:** JIT compiler bugs rarely crash; they produce *wrong results*. A crash oracle never fires.
- **Method:** a **differential oracle** — run the same program in the interpreter and the JIT, and flag divergent behavior.
- **Lesson:** many bug classes need a **reference** to compare against. "Correct" is only meaningful relative to something. This is the same idea behind differential-access oracles (IDOR) and cross-version oracles.

### GRIMOIRE — Synthesizing Structure while Fuzzing (USENIX Security 2019)
- **Problem:** mutation fuzzing breaks syntax; deep states are guarded by format checks.
- **Method:** infer the input grammar from coverage feedback, then synthesize structured inputs.
- **Lesson:** **valid structure is a gate** to deep execution. This is exactly why fuzzers fail on serialized object graphs: they cannot produce a valid typed structure without a grammar/model.

### Fuzztruction — Fault-Injection-Based Fuzzing (USENIX Security 2023)
- **Problem:** generating valid inputs to a consumer (parser) is hard.
- **Method:** instrument the **producer** of the data and inject faults so it emits malformed-but-realistic inputs to the consumer.
- **Lesson:** reuse a real component to reach states that are hard to synthesize directly. Analogous to using a real session/flow to reach authenticated states in DAST.

### Fuzzification — Anti-Fuzzing Techniques (USENIX Security 2019)
- **Problem:** attackers fuzz released binaries.
- **Method:** degrade the coverage signal so fuzzing is less effective.
- **Lesson:** coverage feedback is a **signal that can be gamed**. Treat coverage as a heuristic, not truth. Adversarial targets can starve the oracle/feedback loop.

---

## B. State and concurrency

### Atropos — Effective Fuzzing of Web Applications for Server-Side Vulnerabilities (USENIX Security 2024)
- **Problem:** web apps are stateful; naive replay is fragile and shallow.
- **Method:** take **snapshots** of application state so the fuzzer can restore and explore alternative branches, instead of relying on replaying request sequences.
- **Lesson:** the right model of "state" is the lever. Replay models *sequence*; snapshots model *state*. This is the direct answer to "how are multi-step flows handled" — not by modeling state in the scanner, but by capturing and restoring real state and exploring from there.

### MUZZ — Thread-Aware Grey-Box Fuzzing for Multithreaded Programs (USENIX Security 2020)
- **Problem:** concurrency bugs depend on interleavings, not inputs.
- **Method:** thread-aware instrumentation (coverage, thread-context, schedule intervention).
- **Lesson:** **interleaving is a separate input dimension.** Sequential test suites structurally cannot cover it; you need a concurrency oracle (TSan, invariant violation).

---

## C. LLM-assisted fuzzing

### HLPFUZZ — Hybrid Language Processor Fuzzing via LLM-Based Constraint Solving (USENIX Security 2025)
- **Problem:** language processors guard behavior with hard input constraints; random generation rarely satisfies them.
- **Method:** an LLM solves the constraints to produce inputs that reach deeper program states; divergence analysis and language-server integration refine context on failure.
- **Lesson:** LLMs are useful precisely where symbolic solving is expensive and syntax must be valid. They become a **constraint solver over semantics**.

### mGPTFuzz — From One Thousand Pages of Specification to Unveiling Hidden Bugs (USENIX Security 2024)
- **Problem:** IoT specs are huge; protocol behavior and *non-crash* application-layer bugs are missed.
- **Method:** an LLM extracts information from the specification to drive a black-box fuzzer; the system targets non-crash bugs.
- **Lesson:** **specification → oracle/constraints** is a legitimate LLM task, and non-crash bugs require semantic oracles. Directly relevant to "why the fuzzer missed the logic bug."

### Low-Cost and Comprehensive Non-textual Input Fuzzing via LLM-Synthesized Input Generators (USENIX Security 2025)
- **Problem:** non-textual formats need grammar-aware generation.
- **Method:** the LLM writes and mutates **Python input generators** that conform to the format grammar; a traditional fuzzer (AFL++) mutates the outputs to explore further.
- **Lesson:** the strongest pattern is **hybrid**: LLM supplies structure/semantics, a conventional fuzzer supplies high-throughput exploration and the crash oracle. Same pattern as Big Sleep / AIxCC-style systems.

### Fuzzing BusyBox: Leveraging LLM and Crash Reuse (USENIX Security 2024)
- **Method:** use LLM assistance plus **crash reuse** across related targets to find crashes in a new version without full traditional fuzzing.
- **Lesson:** prior findings are a **seed corpus of oracles and states**; reuse beats starting from zero.

### PANGOLIN — Fuzzing Multilingual IoT Firmware with LLM-Driven Code Analysis (USENIX Security 2026, preprint)
- **Method:** an LLM agent performs **cross-language analysis**, generates input-parameter specifications, and uses response-driven feedback to correct them, enabling semantics-aware fuzzing.
- **Lesson:** **agentic spec inference** across heterogeneous code is a real, current research direction. It is the industrial version of "LLM mines the spec, then the fuzzer explores."

---

## D. LLM detection and autonomous agents

### LLMxCPG — Context-Aware Vulnerability Detection Through Code Property Graphs (USENIX Security 2025)
- **Problem:** LLMs have limited context and get lost in large codebases; naive prompting is noisy.
- **Method:** build **Code Property Graph** slices that reduce code by ~68–91% while preserving vulnerability-relevant context, then feed the slice to the LLM.
- **Lesson:** the fix for context limits is not a bigger prompt; it is **program-analysis-guided slicing**. This is the practical bridge between classical SAST and LLM semantics: use the graph to pick the context, use the LLM to reason about it.

### SoK: DARPA's AI Cyber Challenge (AIxCC) — Competition Design, Results, Lessons (USENIX Security 2026, preprint)
- **What it is:** a systematization of the AIxCC competition, where autonomous systems **find, patch, and validate** vulnerabilities in real open-source targets.
- **Observed design constraints:** many targets needed fuzzing harnesses; organizers onboarded projects to OSS-Fuzz; scoring required validated findings.
- **Lesson:** the state of the art is **end-to-end autonomous cyber reasoning**, but it still leans on **fuzzing infrastructure and oracles**. Autonomy changes orchestration, not the underlying epistemology.

---

## E. Evaluation, risk, and caution

### Lost at C — Security Implications of LLM Code Assistants (USENIX Security 2023)
- **Method:** user study (N=58) of student programmers assisted by LLMs, measuring the security of produced code.
- **Lesson:** LLMs **produce** vulnerable code as readily as they find it. Any pipeline that uses LLMs must also evaluate LLM-authored artifacts; this is a two-sided tool.

### We Have a Package for You! Package Hallucinations (USENIX ;login:)
- **Finding:** code-generating LLMs hallucinate package names, creating a **supply-chain attack surface** (slopsquatting).
- **Lesson:** an LLM that "fixes" code may invent dependencies. Oracle design must include **dependency existence and provenance checks**.

### Exploiting Task-Level Vulnerabilities: An Automatic Jailbreak Attack (USENIX Security 2025)
- **Method:** automatic construction of jailbreak attacks that exploit task-level vulnerabilities in LLMs.
- **Lesson:** the LLM **is itself an attack surface**. An agent that reads untrusted code or web content can be prompt-injected by the target. Sandboxing and instruction/data separation are mandatory.

### Quantifying Large Language Model Attacks Through the Lens of Model Internals (USENIX Security 2026, preprint)
- **Method:** layer-wise probing of internal representations to detect harmful prompting before generation.
- **Lesson:** LLM safety/security is increasingly **mechanistic**; for security tooling, this suggests monitoring model internals as a defense-in-depth layer, not relying on output filters alone.

---

## Cross-cutting conclusions from the literature

1. **Oracles are the recurring bottleneck.** FuzzJIT invents a differential oracle; mGPTFuzz invents a spec-derived oracle for non-crash bugs; Atropos invents a state oracle via snapshots.
2. **Structure beats brute force.** GRIMOIRE, LLM-synthesized generators, and CPG slicing all say: get the input/structure model right, then explore.
3. **State is a first-class dimension.** Atropos (snapshots) and MUZZ (interleavings) show that "state coverage" is not "line coverage."
4. **The winning pattern is hybrid.** LLM for semantics and constraints; conventional fuzzer/executor for throughput and ground-truth signal. AIxCC systems and Google's Big Sleep/Naptime follow this pattern.
5. **LLMs are dual-use.** They find bugs and create them (Lost at C), hallucinate packages, and can be jailbroken. Controls are part of the method, not an afterthought.
