# 09 — GPU-accelerated fuzzing with an LLM copilot: a research path

## Lead

GPUs and LLMs attack the two halves of the fuzzing problem from opposite ends. **The GPU is a throughput engine** for anything data-parallel: executing millions of independent inputs, differential oracles, constraint solving, coverage hashing, and kernel/simulator workloads. **The LLM is a semantics engine**: it infers specs, writes harnesses, generates grammar-valid inputs, synthesizes oracles, and triages results. Today they are almost always used separately. The opportunity is a **closed loop** in which the LLM decides *what* to generate and *how* to judge it, and the GPU does the *mass execution* and *mass evaluation* in between. This chapter surveys what already exists, identifies the gap, proposes an architecture (the **Triad**), and lays out twelve concrete, testable ideas plus a phased path to find new bugs.

> Status note: Sections 1–2 are grounded in published work (arXiv IDs / venues cited). Sections 3–6 are **proposals and hypotheses**, not established results. Treat them as a research plan with falsifiable experiments.

---

## 1. What the state of the art already does

### 1.1 The GPU as an execution engine

- **CuEVM (arXiv 2611.14916)** runs the Ethereum VM **natively on GPU**, over 8M ERC-20 transfers/s, and builds **CuEVM-fuzz** with over 1M end-to-end executions/s — up to **23x** faster than CPU and **4.8x** faster than prior GPU baselines. Lesson: for *massively parallel, semantically bounded* targets, GPU-native execution is a step change.
- **IcFuzz (ASE 2026, arXiv 2608.06088)** fuzzes NVIDIA **Isaac Sim** (GPU physics/rendering) using LLM-based semantic stage segmentation and multi-level mutation with a bandit scheduler; 11 bugs, 9 confirmed/fixed.
- **MLFuzz / Revisiting Neural Program Smoothing (FSE 2023, arXiv 2309.16618)** provides a **GPU-access platform** for reproducible ML-fuzzing evaluation — and a sobering result: neural program smoothing fuzzers were **outperformed by standard greybox** fuzzers once implementation limits were fixed. Lesson: GPU alone does not beat good search.

### 1.2 The GPU as a *target* (GPU-native fuzzing)

- **cuFuzz (OOPSLA 2026, arXiv 2603.12485)** — first practical CUDA fuzzer: whole-program fuzzing, **NVBit** device-side coverage merged with host coverage, sanitizers decoupled from coverage in separate processes; **43 unknown bugs (19 in commercial libraries)** across 14 CUDA programs.
- **GPU-Fuzz (arXiv 2602.10478)** models operator parameters as **formal constraints** and uses a constraint solver to probe boundary conditions; **13 unknown bugs** in PyTorch, TensorFlow, PaddlePaddle.
- **CuFuzz (arXiv 2601.01048)** transforms CUDA to CPU programs with compiler IR-level transforms (PREX, AXIPrune) for ASan-based fuzzing; up to **224x** speedup; **122 vulnerabilities**.
- **GPU-native fuzzing challenges (HotEthics 2026, arXiv 2603.05725)** argues that CPU translation is **unfaithful** (misses architectural differences) and lays out requirements for a faithful GPU-native pipeline.
- **WarpDRF (arXiv 2610.07541)** formalizes a warp data-race-freedom contract in **Rocq**, fuzzes 10K conformance tests across CUDA/HIP/HLSL/Metal/SPIR-V, and found **3 previously unknown races in llama.cpp**.
- **Memory DisOrder (arXiv 2601.08770)** runs a fuzzing campaign showing memory re-orderings leak activity across processes on CPUs **and GPUs** — a timerless side channel.
- **DarthShader (CCS 2024, arXiv 2409.01824)** fuzzes **WebGPU shader translators/compilers** with IR-based and AST-based mutators across the full pipeline; **39 faults** in Chrome/Firefox/Safari, **15 CVEs**.
- **RTL GPU fuzzing**: a SIMT lockstep methodology paper (arXiv 2609.13311) cites **FuzzGPU (USENIX Security 2026)** as a concurrent RTL GPU fuzzer on the same DUT.

### 1.3 The GPU as an *oracle*

- **Differential oracles at scale**: NNSmith (arXiv 2207.13066, 72 DL-compiler bugs), HirGen (ISSTA 2023, 21 bugs), TorchProbe (arXiv 2310.20078, 20 bugs), DESIL (arXiv 2504.01379, 23 silent + 19 crash bugs) all rely on **differential/metamorphic** comparison — an oracle that is naturally data-parallel.
- **Xamt / XAMT (arXiv 2608.11886, 2508.12546)** do **cross-framework differential fuzzing** of DL library APIs, catching bugs that intra-framework CPU-vs-GPU oracles miss.
- **`GraphicsFuzz` / `GPUVerify`** (reviewed by Donaldson, arXiv 2611.12338) are the canonical examples of **randomized testing of GPU compilers** and formal GPU verification — and the paper explicitly links fuzzing and formal methods through **the oracle problem**.
- **Oracle weakness is measurable**: "The Correctness Illusion in LLM-Generated GPU Kernels" (arXiv 2606.20128) shows `allclose` on one shape certifies buggy kernels as correct (10/10 illusions caught by seeded differential fuzzing); "Measuring the Checker" (arXiv 2609.22220) uses **mutation analysis** to show the official KernelBench oracle misses **16.9%** of witnessed faults (78.6% of precision faults). Lesson: at GPU scale, **oracle quality dominates throughput**.

### 1.4 The LLM as a semantic layer

- **Spec/harness generation**: **KernelGPT** (ASPLOS 2025, arXiv 2401.00563) synthesizes syscall specs (24 kernel bugs, 12 fixed), **SeedMind** (arXiv 2411.18143) generates test-case *generators*, **HGFuzzer** (arXiv 2505.03425) turns path constraints into code (17/20 CVEs triggered, 9 new CVEs).
- **Semantic mutation**: **Fuzz4All** (ICSE 2024, 98 bugs), **TitanFuzz** (ISSTA 2023, 65 bugs), **FuzzCoder** (arXiv 2409.01944), **LLAMAFUZZ** (arXiv 2406.07714, +41 bugs), **ReFuzzer** (arXiv 2508.03603) raises test validity from ~48% to ~97%.
- **Constraint solving / concolic**: **HyllFuzz** (arXiv 2412.15931) uses the LLM as a solver inside hybrid fuzzing (31–59% more branches; 7 new bugs); **AutoBug** (OOPSLA 2025, arXiv 2505.13452) does LLM-based symbolic execution via path decomposition.
- **Directed fuzzing**: **RandLuzz** (arXiv 2507.22065) generates reachable seeds and bug-specific mutators; **ProphetFuzz** (arXiv 2409.00922) predicts high-risk option combinations.
- **Domain agents**: **SaFliTe** (arXiv 2412.18727) predicts safety-relevant test cases for autonomous systems; **LLM4Fuzz** (arXiv 2401.11108) guides smart-contract fuzzing; **ChatHTTPFuzz** (arXiv 2411.11929) targets IoT HTTP.

### 1.5 Where they already meet (but not in one loop)

- **AutoBug / HyllFuzz** use LLMs as solvers — LLM *is* the compute, not the director.
- **MLFuzz** gives GPU access for evaluating ML fuzzers — but for benchmarking, not for a live LLM-in-the-loop.
- **IcFuzz / 5uGUE** use LLMs as directors — but on GPU simulators or hardware, without a GPU batch-oracle loop.
- **CPU-translation CUDA fuzzers (CuFuzz)** and **GPU-native fuzzers (cuFuzz)** use the GPU as both target and executor — with no semantic LLM layer.

**No published system closes the loop: LLM proposes → GPU executes at batch scale → GPU/CPU oracles evaluate → LLM diagnoses and re-proposes.**

---

## 2. The gap

Three unaddressed problems:

1. **Oracle bottleneck at scale.** GPU fuzzing produces findings faster than humans or LLMs can judge them. The LLM must be used to *compile* oracles into fast checks, not to judge every output.
2. **Faithfulness vs throughput.** CPU translation is fast but can miss GPU-specific bugs (HotEthics 2026); GPU-native is faithful but complex. An LLM can bridge this by generating harnesses and interpreting device-side signals.
3. **Search direction.** GPU gives breadth; it does not give *aim*. LLM supplies aim (specs, seeds, mutators, invariants) — but current LLM fuzzers run on CPUs at low throughput.

The synthesis is obvious but unbuilt: **LLM-directed, GPU-executed, oracle-distilled fuzzing.**

---

## 3. Proposed architecture: the Triad

Three planes plus a memory:

```
            +---------------------------------------------------+
            |   SEMANTIC PLANE (LLM + RAG)                      |
            |   spec mining | harness/gen | oracle design |      |
            |   triage | root cause | reward design | planner    |
            +----------------------+----------------------------+
                                   |  artifacts (code, grammar, oracle kernels, seeds)
                                   v
   +-------------------------------+-------------------------------+
   |   EXECUTION PLANE (GPU + CPU hybrid)                          |
   |   GPU: batch target execution, EVM/kernels/shaders/sim        |
   |   CPU: stateful/protocol/syscall targets, glue, drivers       |
   |   scheduler: bandit / RL policy over mutation operators       |
   +-------------------------------+-------------------------------+
                                   |  observations (outputs, coverage, sanitizer, timing)
                                   v
   +-------------------------------+-------------------------------+
   |   ORACLE PLANE (multi-signal)                                  |
   |   differential (fp64/CPU/cross-framework) | sanitizer         |
   |   metamorphic | authz/state invariants | timing | telemetry    |
   |   distilled classifiers (GPU/CPU) for cheap mass judgement     |
   +-------------------------------+-------------------------------+
                                   |
                                   v
                      +----------------------------+
                      |  MEMORY: corpus, crashtree, |
                      |  traces, learned oracles,   |
                      |  spec index (RAG)           |
                      +----------------------------+
```

**Division of labor**

| Role | Best done by |
|---|---|
| Infer spec / invariants | LLM |
| Generate grammar/harness/seeds | LLM |
| Design and *compile* oracles | LLM, then GPU/CPU |
| Mass execution | GPU (CPU when stateful) |
| Mass oracle evaluation | GPU kernels / classifiers |
| Triage, root cause, dedup | LLM + crashtree |
| Scheduling / aim | LLM + bandit |

**Core principle: "LLM thinks once, GPU judges millions."** Expensive semantic reasoning is amortized into artifacts (generators, oracle kernels, mutators) that run at GPU speed.

---

## 4. Twelve concrete ideas (each is a proposal)

Legend: **Grounded in** = existing work it extends; **First experiment** = the cheapest falsification.

### Idea 1 — Compile semantic oracles to GPU kernels
- **What:** The LLM reads the spec and emits a **Triton/CUDA oracle kernel** (e.g., "output must equal the fp64 reference within per-dtype tolerance", "response must not reveal another user's data") rather than judging outputs one by one. Only mismatches escalate to the LLM.
- **Why it could work:** Oracle quality dominates at scale (Measuring the Checker); LLMs can write kernel code (KernelGPT, TitanFuzz).
- **Grounded in:** Measuring the Checker (2609.22220), Correctness Illusion (2606.20128), Xamt.
- **First experiment:** For 20 DL ops, compare per-output LLM judging vs LLM-compiled GPU oracle on cost, latency, and precision/recall.
- **Risk:** LLM-written oracle bugs. Mitigate with mutation analysis of the oracle itself.

### Idea 2 — GPU-parallel differential testing with LLM diagnosis
- **What:** Run the target on GPU in massive batches against a **CPU/fp64/cross-framework reference**; the LLM gets only the mismatch clusters and proposes root causes and minimizations.
- **Why:** Differential oracles are inherently data-parallel; LLMs are good at explaining clusters.
- **Grounded in:** NNSmith, HirGen, DESIL, Xamt, TorchProbe.
- **First experiment:** Re-run NNSmith/Xamt-style campaigns with a GPU batched comparator and measure bugs/hour and diagnosis time.

### Idea 3 — Spec → GPU structured generator
- **What:** LLM derives grammar/constraints (ASN.1, protobuf, 3GPP, WebGPU IR, SMT2) and **emits a GPU-parallel instance generator**; GPU produces billions of valid structured inputs; CPU/GPU executes; LLM triages.
- **Why:** Structure is the gate (GRIMOIRE); GPU gives mass validity-preserving generation.
- **Grounded in:** SeedMind, Fuzz4All, mGPTFuzz, LLAMAFUZZ, DarthShader.
- **First experiment:** Pick one binary format (e.g., ELF or a protobuf schema), compare GPU generator vs AFL++ grammar throughput at equal validity.

### Idea 4 — GPU-accelerated concolic with LLM constraint pruning
- **What:** LLM supplies semantics/assumptions and rewrites hard constraints; a GPU bit-vector/SAT solver evaluates candidate assignments in parallel; LLM handles only the residual hard branches.
- **Why:** LLM-as-solver is slow but semantic; GPU solving is fast but syntactic.
- **Grounded in:** HyllFuzz (LLM concolic), AutoBug (LLM symbolic), cuKLEE in M2K (GPU symbolic).
- **First experiment:** On a set of branchy parsers, measure branches covered vs HyllFuzz at equal wall-clock.

### Idea 5 — LLM-designed reward, GPU-trained scheduler
- **What:** LLM proposes the **reward function and action space** (which mutation, which seed, which region); a lightweight bandit/RL policy is trained on GPU over coverage/oracle rewards.
- **Why:** GPU provides the sample volume RL needs; LLM provides the objective.
- **Grounded in:** IcFuzz (bandit scheduler), NPS/Neuzz++ caution (2309.16618).
- **First experiment:** A/B against AFL++ and a fixed bandit baseline on Magma; report coverage and bugs.
- **Risk:** The NPS result warns that learned mutation can lose to greybox. Frame as *scheduler*, not replacement.

### Idea 6 — GPU-native sanitizer fuzzing with LLM harnesses
- **What:** LLM writes the CUDA harness and the memory-safety assertions; GPU-native fuzzing with **compute-sanitizer/NVBit**; LLM interprets device-side reports and minimizes.
- **Why:** Faithfulness matters (HotEthics); harness writing is a known LLM strength.
- **Grounded in:** cuFuzz, GPU-Fuzz, CuFuzz, WarpDRF.
- **First experiment:** Reproduce cuFuzz's 43-bug campaign with an LLM-generated harness set; measure coverage and false positives.

### Idea 7 — Cross-stage shader/WebGPU mutators with semantic invariants
- **What:** LLM proposes semantics-preserving transformations for each stage of the shader pipeline; GPU executes compiled shaders; differential oracle across backends/browsers.
- **Why:** Different pipeline stages need different mutation strategies (DarthShader); LLMs can generate stage-aware transforms.
- **Grounded in:** DarthShader, GraphicsFuzz lineage.
- **First experiment:** Add LLM-generated transforms to DarthShader and measure new coverage/faults on a browser build.

### Idea 8 — Oracle hardening via mutation analysis + LLM fault taxonomy
- **What:** Use **mutation analysis** to measure oracle blindness; LLM reads the surviving mutants and proposes new oracle checks and input distributions; iterate.
- **Why:** Measuring the Checker shows the oracle is the weak link; LLMs can read the fault taxonomy.
- **Grounded in:** Measuring the Checker (mutation analysis), Correctness Illusion.
- **First experiment:** On KernelBench-M, close the 16.9% miss rate with LLM-proposed checks; report residual.

### Idea 9 — GPU-simulated 5G/6G hardware-in-the-loop
- **What:** GPU RF/channel simulation (e.g., Sionna) generates adversarial conditions at scale; LLM agent orchestrates scenario search; hardware-in-the-loop validates the winners.
- **Why:** Telecom HIL is slow; GPU simulation is fast; LLM agent adapts (5uGUE).
- **Grounded in:** AI-driven traffic-steering fuzzing (2601.18690), 5uGUE (2610.29155), Chapter 07.
- **First experiment:** Traffic-steering campaign: GPU sim finds candidates, then run top-k on a testbed; measure transfer rate.

### Idea 10 — GPU EVM fuzzing with LLM invariants
- **What:** LLM extracts protocol invariants (e.g., "total supply conserved", "only owner can withdraw") from contracts/specs; CuEVM-fuzz executes millions of tx/s; oracle = invariant violation.
- **Why:** CuEVM already gives 1M+ exec/s; the missing piece is semantic invariants.
- **Grounded in:** CuEVM/CuEVM-fuzz (2611.14916), LLM4Fuzz.
- **First experiment:** Re-run a DeFi corpus with LLM invariants; compare to LLM4Fuzz on CPU at equal budget.

### Idea 11 — Oracle distillation
- **What:** LLM labels a small, high-quality sample of oracle decisions; distill into a fast GPU/CPU classifier; LLM re-engages on distribution shift or low confidence.
- **Why:** LLM-as-judge is too slow/costly for millions of outputs; a distilled model is cheap.
- **Grounded in:** NLP cross-layer telemetry oracle (2305.08226), SaFliTe.
- **First experiment:** Distill a "does this response violate policy" oracle; measure agreement, cost, and false-negative rate at 1M samples.

### Idea 12 — Multi-agent fabric with RAG over specs and CPG
- **What:** Planner, Mutator, Verifier, Critic, Historian agents; RAG over specs, CPGs, prior crashes, and oracle results; GPU tools exposed to agents.
- **Why:** Agentic orchestration already produces CVEs (5uGUE); adding a GPU toolset multiplies throughput.
- **Grounded in:** 5uGUE, LLMxCPG (CPG slicing), HyllFuzz.
- **First experiment:** Build a 3-agent prototype on one target class; measure confirmed bugs per GPU-hour vs a single-agent baseline.

---

## 5. The path (phased roadmap)

### Phase 0 — Target selection (1 week)
Score candidate targets on **GPU-amplifiability**: is the oracle data-parallel? is the target stateless or batchable? is the input space structured? Use the matrix in Section 7. Pick one "easy" (DL kernel/compiler) and one "hard" (protocol/stateful) target.

### Phase 1 — Semantic baseline on CPU (2–3 weeks)
LLM spec mining + harness/grammar + oracle design; run the existing CPU fuzzer (AFL++/NNSmith/TitanFuzz-style). Establish bug/coverage baselines and cost per bug.

### Phase 2 — GPU execution plane (3–4 weeks)
Port the target/oracle to GPU batch execution (Triton/CUDA or CuEVM-style). Measure executions/s, validity rate, and oracle latency. Keep the CPU reference/fp64 oracle as ground truth.

### Phase 3 — Close the loop (3–4 weeks)
LLM triages GPU-found mismatches, proposes minimizations and new seeds; feed back into Phase 2. Add oracle distillation (Idea 11) when LLM cost dominates.

### Phase 4 — Faithful GPU-native fuzzing (4–6 weeks)
Add sanitizers/coverage on device (cuFuzz/NVBit) and LLM-generated harnesses (Idea 6). Compare against CPU-translation to quantify faithfulness gains.

### Phase 5 — Aim and scheduling (3–4 weeks)
Add the LLM-designed reward + GPU bandit scheduler (Idea 5) and directed seeds/mutators (RandLuzz-style). Test the NPS caution: is it better than greybox?

### Phase 6 — Hard targets and publication (ongoing)
Apply the split architecture (GPU for batch/oracle, CPU for stateful/protocol) to 5G/6G, WebGPU, EVM, and OS kernels. Report CVEs and false-positive rates.

---

## 6. Experiment plan, metrics, research questions

### Metrics
- Executions/second (and **valid** executions/second).
- Coverage delta per GPU-hour.
- Unique bugs / CVEs; **confirmed** vs reported.
- **Cost per bug** (GPU-hours + LLM tokens).
- Oracle precision/recall; false-positive rate.
- Time-to-confirm; reproduction rate.
- Faithfulness delta: GPU-native vs CPU-translation bug yield.

### Falsifiable research questions
- **RQ1:** Does an LLM-compiled GPU oracle beat per-output LLM judging on cost/precision at 1M+ samples? (Idea 1)
- **RQ2:** Does GPU mass structured generation exceed CPU grammar fuzzing on *valid* inputs/second and unique bugs? (Idea 3)
- **RQ3:** Which target classes are GPU-amplifiable — is there a predictor (oracle parallelism × statelessness)? (Section 7)
- **RQ4:** Does LLM triage of GPU mismatches increase unique-bug yield per GPU-hour? (Phase 3)
- **RQ5:** Does GPU-native fuzzing find bugs that CPU-translation misses, and how many? (Idea 6)
- **RQ6:** Does oracle distillation preserve precision/recall at 100x lower cost? (Idea 11)
- **RQ7:** Does the LLM-designed reward + bandit scheduler beat AFL++ on Magma? (Idea 5; expect a nuanced answer given NPS results.)

### Negative-result guardrails
- Always include a **strong CPU baseline** (AFL++, NNSmith, cuFuzz) and report equal-budget comparisons.
- Report **cost**, not just throughput.
- Treat GPU-native vs CPU-translation as a **faithfulness experiment**, not an assumption.

---

## 7. Target-selection matrix (is this GPU-amplifiable?)

| Target class | Oracle parallelism | Statefulness | GPU fit | Examples |
|---|---|---|---|---|
| DL kernels / compilers | very high (differential) | low | **excellent** | TVM, Triton, XLA, PyTorch |
| GPU shaders / WebGPU | high (differential) | low | **excellent** | Chrome/Firefox/Safari, Dawn |
| Smart contracts (EVM) | very high (per-tx) | medium | **excellent** | CuEVM, DeFi |
| GPU simulators (physics/RF) | high | medium | **good** | Isaac Sim, Sionna |
| Binary format parsers | high | low | **good** | ELF/images/media |
| Baseband firmware | medium | low | medium | BaseSAFE-style rehosting |
| Telecom protocol (RRC/NAS/SBI) | medium | **high** | medium (CPU glue) | srsRAN, free5GC |
| OS kernels / syscalls | low | high | poor | Syzkaller targets |
| Stateful web apps | low | very high | poor | DAST targets |

Rule of thumb: **GPU pays off when the oracle is data-parallel and the target is batchable.** For stateful/protocol targets, keep GPU for the *oracle* (e.g., batch authz differentials) and CPU for execution.

---

## 8. Risks, failure modes, ethics

| Risk | Why it matters | Mitigation |
|---|---|---|
| Divergent control flow | GPU stalls on branchy code | use GPU only for data-parallel phases |
| Unfaithful translation | CPU translation misses GPU bugs | GPU-native or validated translation (HotEthics) |
| Weak oracle | more throughput = more noise | mutation-analysis measurement; oracle hardening |
| LLM hallucinated oracle | false confidence | oracle testing; differential cross-check |
| Non-determinism | irreproducible findings | record seeds/model/version; replay harness |
| Cost inversion | GPU + LLM can be expensive | cost-per-bug metric; oracle distillation |
| Benchmark overfitting | inflated results | held-out targets; adversarial evaluation |
| Prompt injection | target content attacks the agent | sandbox, data/instruction separation |
| Legal/ethical | telecom/device testing is regulated | authorized, isolated testbeds only |

---

## 9. Summary

- The GPU is a **throughput and oracle** engine; the LLM is a **semantics and aim** engine.
- Existing work uses one without the other. The unbuilt system is **LLM-directed, GPU-executed, oracle-distilled fuzzing**.
- The winning pattern is **"LLM thinks once, GPU judges millions"**: amortize semantics into generators, mutators, and oracle kernels.
- The most promising near-term wins are **DL kernels/compilers, WebGPU shaders, EVM contracts, and GPU simulators** — all data-parallel oracles.
- The most defensible novelty is **oracle compilation + mathematical oracle-quality measurement**, not raw speed.

---

## References (retrieved via arXiv; venue per paper)

**GPU as execution engine**
- CuEVM: Scaling Smart Contract Execution on GPUs — arXiv 2611.14916 (ICDE 2027).
- IcFuzz: Fuzzing Isaac Sim with Semantic Stage Guidance and Multi-level Mutation — arXiv 2608.06088 (ASE 2026).
- Revisiting Neural Program Smoothing for Fuzzing / MLFuzz — arXiv 2309.16618 (FSE 2023).

**GPU as target**
- cuFuzz: Hunting CUDA Bugs at Scale — arXiv 2603.12485 (OOPSLA 2026).
- GPU-Fuzz: Finding Memory Errors in Deep Learning Frameworks — arXiv 2602.10478.
- CuFuzz: Hardening CUDA Programs through Transformation and Fuzzing — arXiv 2601.01048.
- Challenges and Design Considerations for Finding CUDA Bugs Through GPU-Native Fuzzing — arXiv 2603.05725 (HotEthics 2026).
- WarpDRF: The Unwritten Contract of GPU Warp Programming — arXiv 2610.07541.
- Memory DisOrder: Memory Re-orderings as a Timerless Side-channel — arXiv 2601.08770.
- DarthShader: Fuzzing WebGPU Shader Translators & Compilers — arXiv 2409.01824 (CCS 2024).
- SIMT-Aware Lockstep Verification ... RISC-V GPGPU — arXiv 2609.13311 (cites FuzzGPU, USENIX Security 2026).

**GPU as oracle**
- NNSmith — arXiv 2207.13066 (ASPLOS 2023).
- HirGen — arXiv 2208.02193 (ISSTA 2023).
- TorchProbe — arXiv 2310.20078.
- DESIL — arXiv 2504.01379.
- Xamt / XAMT — arXiv 2608.11886, arXiv 2508.12546.
- The Correctness Illusion in LLM-Generated GPU Kernels — arXiv 2606.20128.
- Measuring the Checker: Mutation Analysis for GPU-Kernel Benchmark Oracles — arXiv 2609.22220.
- From Formal Methods to Fuzzing and Back (GPUVerify, GraphicsFuzz, oracle problem) — arXiv 2611.12338.

**LLM semantic layer**
- KernelGPT — arXiv 2401.00563 (ASPLOS 2025).
- SeedMind — arXiv 2411.18143.
- HGFuzzer — arXiv 2505.03425.
- Fuzz4All — arXiv 2308.04748 (ICSE 2024).
- TitanFuzz — arXiv 2212.14834 (ISSTA 2023).
- FuzzCoder — arXiv 2409.01944.
- LLAMAFUZZ — arXiv 2406.07714.
- ReFuzzer — arXiv 2508.03603.
- HyllFuzz: LLM assisted Hybrid Fuzzing — arXiv 2412.15931.
- AutoBug: LLM-based Symbolic Execution — arXiv 2505.13452 (OOPSLA 2025).
- M2K / cuKLEE — arXiv 2603.24595.
- RandLuzz — arXiv 2507.22065.
- ProphetFuzz — arXiv 2409.00922.
- SaFliTe — arXiv 2412.18727.
- LLM4Fuzz — arXiv 2401.11108.
- ChatHTTPFuzz — arXiv 2411.11929.
- On the Challenges of Fuzzing Techniques via LLMs (survey) — arXiv 2402.00350.
- LLMs for Fuzz Testing in Microservices (SLR) — arXiv 2609.04219.

> All results are paper-reported. Treat Sections 3–6 as proposals. Check the linked PDFs before formal citation.
