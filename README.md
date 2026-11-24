# Security Research with LLMs

### From static/dynamic analysis to the oracle problem, USENIX methods, and the full vulnerability-discovery path

This repository is a research synthesis on **how vulnerabilities are actually found** — by SAST, DAST, fuzzing, formal verification, and LLM-driven analysis — and **how to run that process end to end**. It combines:

- a theoretical model (why "all states" is the wrong frame; existential vs. universal search),
- a deep treatment of the **oracle problem** (the real bottleneck of security testing),
- a survey of relevant **USENIX Security** methods (2020–2026) and what each contributes,
- a concrete **LLM discovery pipeline** with prompt and command templates,
- worked examples (login, calculator, deserialization gadget chain).

> Scope note: this is a literature-informed synthesis for security research and authorized testing. Every technique here must be used only against systems you own or have written permission to test.

---

## TL;DR — the one-page answer

Finding a bug is an **existential search** (find *one* reachable path to impact), not a **universal proof** (show *all* states are safe). That distinction explains everything:

1. You never need all states to *find* a bug; you need **one reachable witness**.
2. You *do* need all states (over a model) to *prove absence* — that is formal verification, and it is always relative to a spec.
3. Fuzzers and DAST **sample** executions; SAST **approximates** states; LLMs **reason over a compressed source/graph model**.
4. The bottleneck is not input generation but the **oracle** — the procedure that decides "is this behavior a bug?"
5. Most "new" bugs are old code meeting a **new oracle**, a **new attack class**, **new code/config**, or a **new reachable path**.
6. LLMs make discovery cheaper by supplying **semantics**: spec mining, oracle synthesis, and exploit-chain reasoning. Truth still comes from **executing the oracle** against a target.

The pipeline (detailed in [`docs/06-playbook.md`](docs/06-playbook.md)):

```
scope -> recon -> spec mining -> reachability (SAST-like)
      -> oracle shaping -> hypothesis/payload synthesis
      -> verification (DAST-like) -> triage -> feedback loop
```

---

## Repository map

| Document | What it covers |
|---|---|
| [`docs/01-foundations.md`](docs/01-foundations.md) | SAST vs. DAST, soundness/completeness, state-space reality, why new bugs appear |
| [`docs/02-oracles.md`](docs/02-oracles.md) | The oracle problem, oracle taxonomy, how new oracles are shaped |
| [`docs/03-usenix-methods.md`](docs/03-usenix-methods.md) | Survey of USENIX Security methods: oracles, structure, state, LLM-assisted fuzzing, agents |
| [`docs/04-llm-discovery.md`](docs/04-llm-discovery.md) | LLM = semantic SAST + oracle synthesis + exploit reasoning; limits and controls |
| [`docs/05-worked-examples.md`](docs/05-worked-examples.md) | Three end-to-end cases: login, calculator, PHP deserialization to RCE |
| [`docs/06-playbook.md`](docs/06-playbook.md) | The full 8-stage path, prompts, commands, checklist, metrics |
| [`docs/07-5g-6g-fuzzing.md`](docs/07-5g-6g-fuzzing.md) | 5G/6G fuzzing: attack surfaces, methods, results, and why they succeeded |
| [`docs/08-5g-6g-method-deep-dive.md`](docs/08-5g-6g-method-deep-dive.md) | Per-method deep dive: what each does, its novelty, and a novelty tier ranking |
| [`references.md`](references.md) | Annotated bibliography with USENIX and arXiv links |

---

## Core thesis in five lines

- **Static** analysis approximates the state space; it is coverage-broad but imprecise.
- **Dynamic** analysis samples real states; it is precise but partial.
- **Fuzzing** raises finding probability under an oracle; it never proves absence.
- **Formal verification** proves properties about a *model under assumptions*; bugs survive in the gaps (spec, model/TCB, side channels, environment).
- **LLMs** do not escape these limits — they lower the cost of *specification, oracle shaping, and exploit reasoning*, and must be paired with execution to be trustworthy.

---

## How to use this repo

- New to the topic: read `01` then `02`.
- Want the research grounding: read `03` with `references.md` open.
- Want to run it: go straight to `06-playbook.md`, then skim `05`.
- Want the LLM-specific argument: read `04`.
- Want the telecom (5G/6G) case study and results: read `07`.
- Want the per-method novelty analysis: read `08`.
