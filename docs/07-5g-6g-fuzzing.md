# 07 — Fuzzing 5G and 6G: how the works did it, and how they succeeded

## Lead

5G and 6G are not "an app with a login form." They are **stateful, layered, often encrypted, partly closed-source telecom systems** spanning radio (RRC/NAS), core (HTTP/2 service-based interfaces), baseband firmware, SIM, and — for 6G — AI-native and non-terrestrial components. Fuzzing them forced researchers to solve three problems the web world takes for granted: **produce valid protocol structure**, **model or reach protocol state**, and **build an oracle for non-crash bugs**. The successful works all did the same thing: they moved the difficulty from "generate bytes" to "generate *semantically valid* messages, guided by spec/coverage/formal models, with a richer oracle."

> Source note: the 5G/6G literature is spread across arXiv, WiSec, ESORICS, IEEE surveys, and vendor/standards work. Citations below use arXiv IDs and any venue the paper states. Numbers are quoted from the papers' abstracts.

---

## 1. Why telecom fuzzing is structurally harder

| Difficulty | Consequence |
|---|---|
| ASN.1 / binary encodings with length and type constraints | random mutation breaks the message; structure must be generated |
| Stateful protocols (attach, RRC setup, authentication, handover) | single-message fuzzing misses state bugs; sequences are required |
| Encryption and integrity (NAS/RRC protection) | fuzzer cannot reach protected fields without keys or rehosting |
| Closed-source baseband firmware | need emulation/rehosting to instrument and sanitize |
| Hardware-in-the-loop (real UEs, real gNBs) | slow, flaky, expensive; needs agents/oracles |
| Non-crash failures (downgrade, MITM, identifier leakage, DoS) | crash oracle is blind; need protocol/state/semantic oracles |
| 6G: AI-native, NTN, O-RAN, THz | new attack surfaces (models, controllers, physical layer) |

This is exactly the oracle/state problem from Chapters 01–02, amplified by telecom constraints.

---

## 2. Attack surfaces that get fuzzed

- **RAN control plane:** RRC, NAS (tunneled), paging, handover, attach procedure.
- **5G core (SBA):** HTTP/2 service-based interfaces (SBIs), OAuth token handling, Network Functions (free5GC, Open5GS).
- **Baseband / modem firmware:** parsers reachable from rogue base stations (MediaTek, Apple ARI interface).
- **UE / devices:** COTS phones and modules via hardware-in-the-loop.
- **O-RAN:** near-real-time RIC, E2, open fronthaul.
- **SIM / APDU:** SIM-modem interface tracing and fuzzing.
- **Physical layer / RF:** jamming, spoofing, waveform-level tests (more 6G).
- **6G-specific:** AI/ML components, NTN/UAV base stations, network slicing, zero-trust control planes.

---

## 3. The methods, by family

| Family | Representative work | Lever |
|---|---|---|
| ASN.1 / grammar-based | Berserker; FivGeeFuzz | valid structure from specs |
| Coverage-guided greybox | CovFUZZ; MulCovFuzz | coverage feedback across components |
| Formal-guided fuzzing | Formal-Guided Fuzz Testing; Formal and Fuzzing Amplification | formal traces prune the search |
| Prior-knowledge adaptive multi-fuzzing | Systematic Meets Unintended (LAL/SyAL/SoAL) | choose strategy by knowledge level |
| Emulation / rehosting + sanitizers | BaseSAFE; ARIstoteles | instrument closed firmware |
| SDR / hardware test equipment | Soft Tester UE | test the real air interface |
| ML/NLP on runtime profiling | NLP-based Cross-Layer detection | classify fuzzing impact |
| LLM-driven / agentic | 5uGUE; Smart Fuzzing (Bard) | semantic test generation + parameter understanding |
| AI/evolutionary search | AI-Driven Fuzzing for Traffic Steering | search adversarial conditions |

---

## 4. The works, and how they succeeded

### 4.1 Berserker — ASN.1-based RRC/NAS fuzzing (2021; WiMob)
- **Target:** 4G/5G RRC (and NAS tunneled in RRC), srsLTE/openLTE stacks.
- **How:** derive message formats from **ASN.1 definitions** in the 3GPP spec; stay compatible across spec versions; bypass size/type constraints; use a concrete protocol stack so encryption/segmentation do not block it.
- **Success:** found **2 previously unknown serious vulnerabilities in srsLTE** (one also affected openLTE).
- **Lesson:** structure from the spec is the gate. This is GRIMOIRE's lesson applied to telecom.

### 4.2 FivGeeFuzz — Cross-Service Token in the 5G core (2025; arXiv 2509.08992)
- **Target:** 5G core Service-Based Interfaces, free5GC (Release 17 SBIs + OAuth access control).
- **How:** automatically **derive grammars from 3GPP API specifications**; generate malformed, unexpected, and *semantically inconsistent* inputs; combine automated detection with manual validation and root-cause analysis.
- **Success:** **8 previously unknown vulnerabilities**; runtime crashes, improper error handling, and unauthorized resource access; a severe **Cross-Service Token Attack**; all confirmed by the free5GC team, **7 patched** and 1 patch in progress.
- **Lesson:** in the core, the interesting bugs are **authorization semantics**, not crashes. The oracle must be "did access control fail," not "did it segfault."

### 4.3 CovFUZZ — coverage-guided 4G/5G attach fuzzing (2024; arXiv 2410.20958)
- **Target:** attach procedure; uplink+downlink in 4G, downlink in 5G; all layers except PHY.
- **How:** assign **probabilities to packet fields** and adjust them using **coverage feedback** from the device under test; when coverage is unavailable, *estimate* it. Compare against an optimized random fuzzer.
- **Success:** vulnerabilities in **10 of 12 COTS devices** and **all srsRAN 4G instances**; coverage-guided beating random on DUT code coverage.
- **Lesson:** coverage is the steering signal, and "estimate coverage when the device is a black box" is a practical oracle-shaping trick.

### 4.4 MulCovFuzz — multi-component greybox for 5G (2026; arXiv 2602.21794)
- **Target:** OpenAirInterface (OAI) 5G.
- **How:** **multi-component coverage collection** across 5G system components, plus a scoring function that combines coverage rewards with efficiency to guide test-case generation; addresses the fact that black-box fuzzers cannot see internal state.
- **Success:** **+5.85% branch coverage, +7.17% line coverage, +16% unique crash discovery** over 24h; **3 zero-day vulnerabilities**, **2 not found by any other fuzzing technique**.
- **Lesson:** when the system has multiple components, aggregate coverage across them. A single-component signal under-rewards deep paths.

### 4.5 Formal-Guided Fuzz Testing / Formal and Fuzzing Amplification (2023; arXiv 2307.11247 and 2307.05758)
- **Target:** 5G NSA security processes, RRC connection; srsRAN.
- **How:** formal verification finds **attack traces** in the protocol model; those traces **seed** fuzz-test case generation; fuzz feedback **broadens** the formal search. Hierarchical: protocol logic → implementation stack.
- **Success:** **61 vulnerabilities** exploited on srsRAN (2307.11247); **53 vulnerabilities** in the amplification paper (2307.05758). Found identifier leakage, DoS, and two eavesdrop attack models, including MITM despite a decade-old TLS remedy. Turned exponential search into **linear growth**.
- **Lesson:** formal methods are not a competitor to fuzzing — they are a **search-space pruner** and an oracle for "reachable violation." This is the strongest confirmation of the repo's existential-search framing.

### 4.6 Systematic Meets Unintended — prior-knowledge adaptive multi-fuzzing (2023; arXiv 2305.08039)
- **Target:** srsRAN 5G RRC.
- **How:** three strategies by attacker knowledge:
  - **LAL (Listen-and-Learn):** black-box, no protocol knowledge.
  - **SyAL (Synchronize-and-Learn):** black-box with attacker-accessible user info + **probability-based fuzzing**.
  - **SoAL (Source-and-Learn):** white-box, fuzz **significant bits** to explain vulnerabilities.
  It also serves as a **digital twin** for defense simulation.
- **Success:** LAL found **129 RRC connection vulnerabilities** with **0.072s average detection**; SyAL beat existing models in precision/recall with far fewer cases; SoAL found **3 MITM vulnerabilities**.
- **Lesson:** there is no single best strategy; **choose the fuzzing strategy by the attacker's knowledge**. This is a meta-oracle decision.

### 4.7 BaseSAFE — emulation-based baseband fuzzing (2020; WiSec '20; arXiv 2005.07797)
- **Target:** MediaTek cellular baseband (Nucleus RTOS), parsers reachable from rogue base stations.
- **How:** **partial rehosting** of closed firmware off-device; a **sanitizing drop-in allocator** to catch heap bugs; optimized emulation reaching ~15k test cases/s in total (hundreds/s/core on complex cases).
- **Success:** found **memory corruptions including heap out-of-bounds writes** in the MediaTek baseband.
- **Lesson:** when you cannot see source, **rehost + instrument + sanitize**. The sanitizer *is* the oracle.

### 4.8 ARIstoteles — Apple baseband interface (ESORICS 2021; arXiv 2604.23457)
- **Target:** Apple Remote Invocation (ARI) interface, iOS.
- **How:** reverse-engineer undocumented iOS libraries with Ghidra scripts to auto-generate a **Wireshark dissector**, then **fuzz the ARI interface** based on those RE results.
- **Success:** first RE + fuzzing of ARI; demonstrated that ARI is under-tested.
- **Lesson:** **understanding the protocol is the fuzzing enabler**. Reverse engineering produces the grammar and the state model.

### 4.9 Soft Tester UE — Open RAN security testing (2024; arXiv 2410.09641)
- **Target:** 5G / O-RAN over the **Uu air interface**.
- **How:** a software-defined **test UE** (free, open-source, extensible) that sends **fuzzed data** during connection setup; addresses the hardware barrier for smaller labs.
- **Success:** demonstrated automated security testing of a RAN under test.
- **Lesson:** lowering the **testbed cost** is itself a research contribution; more labs → more findings.

### 4.10 SIMulator — SIM tracing on a Pico budget (2025; WiSec poster; arXiv 2506.20800)
- **Target:** SIM–modem interface (APDU level).
- **How:** electrically decouple SIM and modem, transfer only APDU with a ~4 USD Raspberry Pi Pico; enables fuzzing SIM communication and key monitoring.
- **Lesson:** instrumentation access is a prerequisite; cheap tracing democratizes fuzzing.

### 4.11 NLP-based cross-layer detection from fuzzing profiles (2023; arXiv 2305.08226)
- **Target:** srsRAN; fuzzing-generated **LogInfo** runtime profiling.
- **How:** map logs to a high-dimensional metric space, build feature spaces by timestamp, classify impacts (performance/security) with Logistic Regression, k-NN, Random Forest.
- **Success:** **93.4%–95.9% accuracy** in detecting fuzzing impacts.
- **Lesson:** when the crash oracle is absent, **learn an oracle from runtime telemetry**.

### 4.12 Smart Fuzzing of OAI5G with LLM parameter interpretation (2023; arXiv 2309.12994)
- **Target:** OpenAirInterface5G **configuration files**.
- **How:** **AFL++** fuzzes the config files; **Google Bard** automatically explains the meaning of the codebase parameters used in fuzzing.
- **Success:** identified errors/defects/vulnerabilities in config handling; improved comprehensibility.
- **Lesson:** an early, simple LLM use: **semantic documentation of inputs** to make fuzzing interpretable. This is "spec mining" in miniature.

### 4.13 5uGUE — agentic hardware-in-the-loop fuzzing (2026; arXiv 2610.29155)
- **Target:** COTS 5G devices (phones), then Bluetooth Classic.
- **How:** an **LLM-driven HIL agent** with multi-step reasoning and tool use: internet search, 5G protocol analysis, memory management, test-case execution. It iteratively generates cases, runs them on targets, collects runtime feedback, and refines strategy.
- **Success:** on five COTS 5G devices across six LLMs, best runs triggered **167 crashes (141 novel, 31 unique)** in 60 turns (49–133 min) vs the state-of-the-art baseline **5GHOUL with 13 crashes (10 novel, 6 unique)** in 150 min — about **12.8x total and 5.2x unique** improvement. **46 unique crashes, 44 vendor-confirmed but previously reported**; MediaTek confirmed **two previously unknown high-severity vulnerabilities**, one assigned **CVE-2026-20525**. Extended to Bluetooth with minimal changes.
- **Lesson:** LLM agents can operate **beyond software-in-the-loop**, on proprietary hardware. This is the clearest 2026 example of the repo's "LLM + tool execution + oracle" pattern applied to telecom.

### 4.14 AI-driven fuzzing for 5G traffic steering (2026; arXiv 2601.18690)
- **Target:** 5G Traffic Steering algorithms under adversarial conditions (interference spikes, handover storms, outages).
- **How:** **NSGA-II** evolutionary search over a simulated environment (NVIDIA Sionna); five algorithms, six scenarios.
- **Success:** **34.2% more total vulnerabilities**, **5.8% more critical failures** than traditional testing, with better diversity and edge-case discovery; framed as **6G-ready**.
- **Lesson:** for algorithmic/optimization components, "fuzzing" becomes **adversarial search** and the oracle is a **performance/robustness invariant**.

---

## 5. Synthesis: why these works succeeded

Seven recurring levers, each mapping to the repo's framework:

1. **Structure from the spec** (Berserker ASN.1, FivGeeFuzz 3GPP grammars, ARIstoteles RE). Valid structure is the gate to deep execution. (Chapter 01, GRIMOIRE.)
2. **State and sequence modeling** (attach procedure, RRC setup, handover). Single-message fuzzing misses state bugs. (Chapter 01, Atropos.)
3. **Oracle enrichment** — crash + sanitizer (BaseSAFE) + authorization semantics (FivGeeFuzz) + timing/coverage estimation (CovFUZZ) + telemetry classifiers (NLP paper) + robustness invariants (traffic steering). (Chapter 02.)
4. **Search-space pruning by formal methods** (Formal-Guided, Amplification). Formal traces convert exponential search to linear. This is the clearest validation of "formal = universal checks on a model; fuzzing = existential search on the implementation." (Chapters 01, 04.)
5. **Coverage aggregation across components** (MulCovFuzz, CovFUZZ). A single signal under-rewards deep paths in multi-component systems.
6. **Access and instrumentation** (BaseSAFE rehosting, Soft Tester UE, SIMulator, ARIstoteles). You cannot fuzz what you cannot reach or observe.
7. **LLM/agentic semantics** (5uGUE, Bard parameter interpretation, and the broader LLM-fuzzing literature in Chapter 03). LLMs supply protocol understanding, test-case synthesis, and adaptation — especially on hardware-in-the-loop where symbolic methods fail.

### The pattern in one line
**Telecom fuzzing succeeded when it stopped generating bytes and started generating *valid, stateful, spec-grounded messages judged by a rich oracle*, with formal or coverage signals pruning the search.**

---

## 6. How this maps to the generic pipeline

| Generic stage (Ch. 06) | 5G/6G instantiation |
|---|---|
| Recon | map RRC/NAS/SBI/baseband/PHY surfaces; identify stacks (srsRAN, OAI, free5GC, Open5GS) |
| Spec mining | 3GPP specs, ASN.1, API grammars, RE of closed interfaces |
| Reachability | protocol state machine; which messages reach which parsers |
| Oracle shaping | crash/sanitizer + authz diff + protocol invariant + timing/coverage + telemetry |
| Synthesis | ASN.1/grammar messages, state sequences, malformed-but-valid fields |
| Verification | run on emulator/rehosted firmware/testbed/COTS device |
| Triage | vendor confirmation, CVE, patch status |
| Loop | reuse traces/crashes; update grammar and oracle |

---

## 7. The 6G outlook

6G is largely pre-standard, so the literature is mostly **surveys and position papers** plus 5G testbeds used as proxies. The security themes that will shape 6G fuzzing:

- **AI-native networks** — fuzzing moves to *models* and *controllers* (RIC, ML pipelines), not just parsers; oracles become behavioral/robustness properties, and adversarial ML meets protocol fuzzing (cf. traffic-steering work).
- **Non-terrestrial networks / UAV base stations** — mobility and wireless backhaul create new state and attack surfaces (handover manipulation, jamming/spoofing).
- **Open RAN** — disaggregated, multi-vendor interfaces widen the trust boundary; software-defined test UEs lower the cost of testing.
- **Massive MIMO / THz / IRS** — physical-layer and beam-management state become test targets; physical-layer security and side channels gain weight.
- **Network slicing and multi-stakeholder identity** — authorization across domains; zero-trust and decentralized identity are proposed defenses, and *slicing isolation* is a natural fuzzing invariant.
- **Quantum-safe and privacy** — new cryptographic assumptions and privacy properties become oracles.

Representative 6G security sources retrieved: "Security and privacy for 6G: A survey" (arXiv 2108.11861, IEEE COMST 2021), "6G White paper: Research challenges for Trust, Security and Privacy" (arXiv 2004.11665), "Zero Trust Architecture for 6G Security" (arXiv 2203.07716), "Bridging the Security Gap: Lessons from 5G and What 6G Should Do Better" (arXiv 2501.11045), "Toward a Unified Security and Privacy Framework for AI-Native 6G Networks" (arXiv 2607.01019), "Towards a Security Plane for 6G Ecosystems" (arXiv 2512.20733).

### What carries over from 5G
1. Spec-grounded structure (now including AI/ML API schemas).
2. State modeling (slices, handovers, NTN links).
3. Rich oracles (isolation, robustness, authorization, timing).
4. Formal pruning + coverage aggregation.
5. LLM/agentic orchestration, especially for hardware and cross-language/cross-domain specs.

---

## 8. Practical playbook for a 5G/6G fuzzing engagement

**Scope and ethics first.** Only test networks/devices you own or are authorized to test; use isolated testbeds and licensed spectrum; never transmit over commercial networks without permission.

1. **Pick a testbed target:** srsRAN (RAN), OpenAirInterface (RAN/CN), free5GC / Open5GS (core), emulated baseband (BaseSAFE-style), or a lab COTS device.
2. **Obtain the grammar:** 3GPP ASN.1/API specs, or reverse-engineer closed interfaces (Ghidra + dissector, ARIstoteles-style).
3. **Model state:** enumerate the attach / RRC / authentication / handover state machine; script sequences.
4. **Choose the strategy by knowledge:** black-box (LAL), informed black-box (SyAL), white-box (SoAL).
5. **Instrument:** coverage across components (MulCovFuzz), sanitizers on rehosted firmware (BaseSAFE), telemetry (NLP profiler).
6. **Enrich the oracle:** crash + sanitizer + authorization differential + protocol invariant + timing/robustness.
7. **Add formal pruning** where a protocol model exists: extract attack traces, seed the fuzzer, feed results back.
8. **Add an LLM agent** for protocol understanding, parameter interpretation, and hardware-in-the-loop adaptation (5uGUE-style), with sandboxing and human review.
9. **Verify and report:** reproduce, confirm with the vendor/upstream, track CVE/patch status.
10. **Loop:** reuse traces and crashes; update grammar, state model, and oracle.

### Tool/target quick map

| Layer | Open-source targets | Techniques |
|---|---|---|
| RAN (RRC/NAS) | srsRAN, OpenAirInterface, srsLTE | ASN.1 fuzzing (Berserker), coverage-guided (CovFUZZ), multi-fuzzing (LAL/SyAL/SoAL) |
| Core (SBI) | free5GC, Open5GS | 3GPP-grammar fuzzing (FivGeeFuzz), authz differential |
| Baseband | MediaTek (emulated) | rehosting + sanitizers (BaseSAFE) |
| UE / device | COTS phones, modules | hardware-in-the-loop + LLM agent (5uGUE) |
| O-RAN / air interface | software-defined test UE | SDR fuzzing at Uu (Soft Tester UE) |
| SIM | SIM–modem APDU | tracing + fuzzing (SIMulator) |
| 6G algorithmic | traffic steering, RIC | evolutionary/adversarial search (NSGA-II, Sionna) |

---

## 9. Honest limits

- **Numbers are paper-reported**; they depend on testbeds, baselines, and target versions.
- **COTS/hardware results are hard to reproduce**; vendor confirmation is the strongest evidence.
- **6G work is mostly pre-standard**, so many "6G" claims are projections from 5G testbeds.
- **Coverage does not equal correctness**; multi-component coverage is a heuristic.
- **LLM agents add non-determinism and cost**, and the target itself can be an attack surface (prompt injection from device output).
- **Ethics and law** constrain what can be tested; always stay within authorized, isolated environments.
