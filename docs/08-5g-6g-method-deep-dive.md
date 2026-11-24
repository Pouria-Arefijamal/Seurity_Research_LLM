# 08 — 5G/6G fuzzing: method-by-method deep dive and novelty analysis

This chapter expands Chapter 07. For each method it answers three questions: **what the method actually is**, **what was genuinely new**, and **how novel it is**. Ratings are an analytic assessment based on the papers' published abstracts and the surrounding literature; they are not the authors' claims, and novelty is not the same as impact.

---

## How novelty was assessed

Each method was judged on four axes:

1. **New target** — did it reach a surface previously untested (closed baseband, Apple ARI, core SBA, hardware-in-the-loop)?
2. **New mechanism** — a genuinely new algorithm or a recombination of known ideas?
3. **New oracle** — did it invent a way to *decide* a bug exists where no crash signal was available?
4. **New enabler** — did it lower a cost/access barrier so more work becomes possible?

A method can be high on one axis and low on another. "First-of-kind" usually means a **new target + new enabler**, not a new algorithm.

| Tier | Meaning |
|---|---|
| **True first / new capability** | opens a target that was previously unreachable or unobservable |
| **Notable methodological novelty** | new mechanism, or a genuinely new combination with a strong effect |
| **Solid engineering / measurement** | known ideas applied well; improves access or signal |
| **Application / early signal** | known ML/search applied to a new domain |

---

## 1. Berserker (2021) — ASN.1-based RRC/NAS fuzzer

- **What it is:** a fuzzer that reads the 4G/5G **ASN.1 message definitions** from the 3GPP specification and generates structured RRC messages, plus NAS messages tunneled inside RRC.
- **How it works:** build valid messages from ASN.1, drive a concrete protocol stack (srsLTE/openLTE), deliberately escape size/type constraints to probe parser edges. Spec-version-driven, so it is backward/forward compatible across RRC releases.
- **Novelty:** structure generation *from the standard itself*, NAS-in-RRC coverage, and constraint evasion that stays spec-aware.
- **Novelty level: Medium.** ASN.1 fuzzing existed; the contribution is faithful, version-agnostic telecom structure plus NAS coverage.
- **Result:** 2 previously unknown serious vulnerabilities in srsLTE (one also in openLTE).

## 2. FivGeeFuzz (2025) — 3GPP-API grammar fuzzing of the 5G core

- **What it is:** a grammar-based fuzzer for the 5G **Service-Based Interfaces** (HTTP/2 APIs between Network Functions), including OAuth access control.
- **How it works:** auto-derive grammars from **3GPP API specifications**, emit malformed, unexpected, and **semantically inconsistent** requests, then combine automated detection with manual root-cause analysis.
- **Novelty:** moving fuzzing from the RAN to the **core SBA**, deriving grammars from API specs, and targeting **authorization semantics** rather than crashes ("Cross-Service Token").
- **Novelty level: High (for the domain).** Core-SBI and OAuth-logic fuzzing is far less explored than RAN fuzzing.
- **Result:** 8 previously unknown vulnerabilities in free5GC, including the Cross-Service Token Attack; 7 patched.

## 3. CovFUZZ (2024) — coverage-guided 4G/5G attach fuzzing

- **What it is:** a fuzzer for the **attach procedure**, assigning probabilities to packet fields and adapting them over time.
- **How it works:** mutate fields with learned probabilities; use DUT **coverage feedback**; when coverage is unavailable (black-box device), *estimate* it.
- **Novelty:** field-level probability tuning plus **coverage estimation for closed devices**.
- **Novelty level: Medium.** Coverage-guided mutation is well known; the novel parts are field-level probability control and the estimation trick for black-box UEs.
- **Result:** vulnerabilities in 10 of 12 COTS devices and all srsRAN 4G instances.

## 4. MulCovFuzz (2026) — multi-component coverage greybox

- **What it is:** a greybox fuzzer that collects coverage from **several components** of a 5G system simultaneously.
- **How it works:** aggregate coverage across components, score test cases by a mix of coverage reward and efficiency, and guide test-case generation.
- **Novelty:** **multi-component coverage aggregation** instead of a single-component signal.
- **Novelty level: Medium.** A sensible extension of greybox fuzzing; the insight that one component's coverage under-rewards deep paths is real.
- **Result:** +5.85% branch coverage, +7.17% line coverage, +16% unique crash discovery; 3 zero-days on OpenAirInterface, 2 unique to this tool.

## 5. Formal-Guided Fuzz / Formal-and-Fuzzing Amplification (2023)

- **What they are:** hybrid frameworks that connect **formal verification and fuzzing** in both directions.
- **How they work:** formal analysis finds **attack traces** in a protocol model; those traces seed fuzz cases; fuzz feedback widens the formal search. The approach is hierarchical: protocol logic to implementation stack.
- **Novelty:** **bidirectional feedback** ("amplification") and the exponential-to-linear reduction of search cost.
- **Novelty level: Medium-High.** Formal+fuzzing hybrids exist, but the mutual amplification and telecom protocol-to-stack hierarchy are notable.
- **Result:** 61 vulnerabilities on srsRAN (2307.11247); 53 in the amplification paper (2307.05758); identifier leakage, DoS, eavesdropping, and MITM despite TLS.

## 6. LAL / SyAL / SoAL — "Systematic Meets Unintended" (2023)

- **What it is:** a **knowledge-adaptive multi-fuzzing** framework.
- **How it works:** choose a strategy based on attacker knowledge:
  - **LAL (Listen-and-Learn)** — black-box, no protocol knowledge.
  - **SyAL (Synchronize-and-Learn)** — black-box plus attacker-accessible user information and **probability-based** fuzzing.
  - **SoAL (Source-and-Learn)** — white-box, fuzz **significant bits** to explain vulnerabilities.
  It also serves as a **digital twin** for defense simulation.
- **Novelty:** the explicit **strategy taxonomy by knowledge level**, plus the digital-twin framing.
- **Novelty level: Medium-High (conceptual).**
- **Result:** 129 RRC connection vulnerabilities at 0.072s average detection; 3 MITM vulnerabilities via white-box bit fuzzing.

## 7. BaseSAFE (2020) — emulated baseband fuzzing

- **What it is:** a framework that **rehosts closed-source baseband firmware** and fuzzes its parsers off-device.
- **How it works:** partial rehosting plus a **sanitizing drop-in allocator** to catch heap bugs; heavy optimization reaching roughly 15k test cases/s in total.
- **Novelty:** per the paper, the **first emulation-based security fuzzing of commercial cellular basebands**; the sanitizing allocator is the enabling trick.
- **Novelty level: High (first-of-kind per authors).**
- **Result:** memory corruptions including heap out-of-bounds writes in the MediaTek baseband.

## 8. ARIstoteles (ESORICS 2021) — Apple baseband interface

- **What it is:** reverse-engineering plus fuzzing of Apple's undocumented **ARI** interface on iOS.
- **How it works:** Ghidra scripts parse closed-source iOS libraries and **auto-generate a Wireshark dissector**, then fuzz the ARI interface based on the RE results.
- **Novelty:** first RE+fuzz of ARI; **automated dissector generation** from closed binaries.
- **Novelty level: High (for the Apple ecosystem gap).** Fills a platform blind spot; the tooling is the contribution.

## 9. Soft Tester UE (2024) — software-defined test UE

- **What it is:** an open-source, software-defined **UE test instrument** for 5G/O-RAN over the **Uu air interface**.
- **How it works:** send fuzzed data during connection setup; run automated security tests without proprietary hardware.
- **Novelty:** cheap, extensible, open **test equipment** (access democratization).
- **Novelty level: Medium (engineering).** Not a new algorithm; removes a cost barrier so more labs can test.

## 10. SIMulator (WiSec 2025) — cheap SIM tracing

- **What it is:** SIM-modem **APDU-level tracing** on a ~$4 microcontroller.
- **How it works:** electrically decouple SIM and modem, transfer only APDU frames; enables fuzzing communication interfaces and monitoring keys.
- **Novelty:** full SIM tracing with commodity parts instead of specialized hardware.
- **Novelty level: Medium (tooling).** Impact comes from accessibility, not from a new technique.

## 11. 5uGUE (2026) — LLM agentic hardware-in-the-loop fuzzer

- **What it is:** an **LLM agent** that fuzzes real 5G devices (and Bluetooth) in a hardware loop.
- **How it works:** multi-step reasoning plus tools (internet search, 5G protocol analysis, memory management, test-case execution); iteratively generates cases, runs them, collects runtime feedback, and refines strategy.
- **Novelty:** LLM agent operating **beyond software-in-the-loop**, on proprietary hardware, with adaptive strategy.
- **Novelty level: High (2026 frontier).**
- **Result:** 167 crashes (141 novel, 31 unique) vs 13 for the 5GHOUL baseline; about 12.8x total and 5.2x unique improvement; **CVE-2026-20525** confirmed by MediaTek; extended to Bluetooth.

## 12. Smart Fuzzing of OAI5G (2023) — AFL++ plus LLM

- **What it is:** fuzz OpenAirInterface **configuration files** with AFL++, and use an LLM (Google Bard) to explain parameters.
- **How it works:** AFL++ on configs; the LLM annotates the meaning of parameters used in fuzzing.
- **Novelty:** an **early LLM-for-parameter-semantics** use; technically simple.
- **Novelty level: Low-Medium.** Interesting as an early signal of the LLM-in-fuzzing direction, not a deep method.

## 13. NLP-based cross-layer detection (2023) — learned oracle

- **What it is:** detect 5G security and performance problems from **fuzzing-generated runtime logs**.
- **How it works:** map LogInfo to a high-dimensional metric space, build timestamped features, and classify impacts with Logistic Regression, k-NN, and Random Forest.
- **Novelty:** a **learned oracle from runtime telemetry** when no crash signal exists.
- **Novelty level: Medium (application).** 93.4%-95.9% accuracy; depends heavily on log quality.

## 14. AI-driven traffic-steering fuzzing (2026) — evolutionary adversarial search

- **What it is:** fuzz 5G **traffic-steering algorithms** with adversarial conditions (interference spikes, handover storms, outages).
- **How it works:** **NSGA-II** multi-objective search in NVIDIA **Sionna**; five algorithms, six scenarios.
- **Novelty:** reframing "fuzzing" as **multi-objective adversarial search** over algorithmic behavior, framed for 6G.
- **Novelty level: Medium.**
- **Result:** 34.2% more total vulnerabilities and 5.8% more critical failures than traditional testing.

---

## Novelty at a glance

| Tier | Methods | Why |
|---|---|---|
| **True first / new capability** | BaseSAFE, ARIstoteles, 5uGUE | opened previously unreachable targets (closed baseband, Apple ARI, hardware-in-the-loop) |
| **Notable methodological novelty** | Formal-guiding/Amplification, LAL/SyAL/SoAL, FivGeeFuzz | new or bidirectional methods; core-SBA logic |
| **Solid engineering / measurement** | Berserker, CovFUZZ, MulCovFuzz, Soft Tester UE, SIMulator | known ideas applied well; improved access or signal |
| **Application / early signal** | NLP telemetry, Bard-on-config, NSGA-II traffic steering | known ML/search applied to a new domain |

## Conclusions

1. **Most novelty is not a new algorithm.** It is **new access** (rehosting, SDR, cheap tracing), **new oracles** (authorization semantics, telemetry, robustness), and **new structure sources** (ASN.1, 3GPP APIs, reverse engineering).
2. **The genuinely new mechanisms** are the **formal-to-fuzz bidirectional amplification** and the **knowledge-adaptive multi-fuzzing** taxonomy.
3. **The genuine firsts** are the hardware/baseband ones, where the contribution is making an invisible target observable.
4. **LLM agentic fuzzing** is the current frontier. It is promising and already produced a confirmed CVE, but it is not yet a settled, reproducible method.
5. **Novelty gradients follow observability.** The more of the system you can see or reach, the more real novelty becomes possible.

> All results are paper-reported. Numbers depend on testbeds, baselines, and target versions; check the linked PDFs before formal citation.
