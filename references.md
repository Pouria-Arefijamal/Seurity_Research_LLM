# References

Annotated bibliography, focused on USENIX Security venues. Entries marked "preprint" are USENIX pre-publication pages. Summaries reflect the published abstracts.

## Fuzzing, oracles, and structure

1. **FuzzJIT: Oracle-Enhanced Fuzzing for JavaScript Engine JIT Compilers.** Junjie Wang, Zhiyi Zhang, Shuang Liu, Xiaoning Du, Junjie Chen. USENIX Security 2023.
   <https://www.usenix.org/conference/usenixsecurity23/presentation/wang-junjie>
   *Differential oracle between interpreter and JIT to detect wrong-result bugs.*

2. **GRIMOIRE: Synthesizing Structure while Fuzzing.** Tim Blazytko, Cornelius Aschermann, Moritz Schlögel, Ali Abbasi, Sergej Schumilo, Simon Wörner, Thorsten Holz. USENIX Security 2019.
   <https://www.usenix.org/conference/usenixsecurity19/presentation/blazytko>
   *Infers input grammar from coverage to synthesize structured inputs.*

3. **Fuzztruction: Using Fault Injection-based Fuzzing to Leverage Implicit Domain Knowledge.** Nils Bars, Moritz Schloegel, Tobias Scharnowski, Nico Schiller, Thorsten Holz. USENIX Security 2023 (Distinguished Paper; Internet Defense Prize runner-up).
   <https://www.usenix.org/conference/usenixsecurity23/presentation/bars>
   *Instrument the producer and inject faults to reach consumer states.*

4. **Fuzzification: Anti-Fuzzing Techniques.** USENIX Security 2019.
   <https://www.usenix.org/conference/usenixsecurity19/presentation/jung>
   *Coverage feedback is an attackable signal; treat coverage as a heuristic.*

5. **MUZZ: Thread-aware Grey-box Fuzzing for Effective Bug Hunting in Multithreaded Programs.** Hongxu Chen et al. USENIX Security 2020.
   <https://files.usenix.org/s3fs-public/cdn/sec20-chen-hongxu.pdf>
   *Concurrency interleavings are a separate input dimension; thread-aware instrumentation.*

## State and web application testing

6. **Atropos: Effective Fuzzing of Web Applications for Server-Side Vulnerabilities.** USENIX Security 2024.
   <https://www.usenix.org/system/files/usenixsecurity24-guler.pdf>
   *Stateful web fuzzing using snapshots instead of fragile request replay.*

## LLM-assisted fuzzing and specification

7. **HLPFUZZ: Hybrid Language Processor Fuzzing via LLM-Based Constraint Solving.** Yupeng Yang, Shenglong Yao, Jizhou Chen, Wenke Lee. USENIX Security 2025.
   <https://www.usenix.org/conference/usenixsecurity25/presentation/yang-yupeng>
   *LLM solves input constraints to explore deeper program states; divergence analysis refines context.*

8. **mGPTFuzz: From One Thousand Pages of Specification to Unveiling Hidden Bugs.** Xiaoyue Ma et al. USENIX Security 2024.
   <https://www.usenix.org/system/files/usenixsecurity24-ma-xiaoyue.pdf>
   *LLM extracts specification to drive a black-box fuzzer and target non-crash application-layer bugs.*

9. **Low-Cost and Comprehensive Non-textual Input Fuzzing with LLM-Synthesized Input Generators.** Kunpeng Zhang et al. USENIX Security 2025.
   <https://www.usenix.org/system/files/conference/usenixsecurity25/sec25cycle1-prepub-1291-zhang-kunpeng.pdf>
   *LLM writes and mutates grammar-conforming input generators; traditional fuzzer explores the outputs.*

10. **Fuzzing BusyBox: Leveraging LLM and Crash Reuse for Embedded Bug Unearthing.** Asmita et al. USENIX Security 2024.
    <https://www.usenix.org/conference/usenixsecurity24/presentation/asmita>
    *LLM assistance plus crash reuse finds crashes without full traditional fuzzing.*

11. **PANGOLIN: Fuzzing Multilingual IoT Firmware with LLM-Driven Code Analysis.** Zhipeng Jia, Xiaokang Yin, Shuitao Gan, Chao Zhang, et al. USENIX Security 2026 (preprint).
    <https://www.usenix.org/conference/usenixsecurity26/presentation/jia-zhipeng>
    *LLM agent performs cross-language analysis, generates and corrects parameter specifications.*

## LLM vulnerability detection and autonomous systems

12. **LLMxCPG: Context-Aware Vulnerability Detection Through Code Property Graphs.** Lekssays et al. USENIX Security 2025.
    <https://www.usenix.org/conference/usenixsecurity25/presentation/lekssays>
    *CPG-based slicing reduces code 67.84–90.93% while preserving vulnerability-relevant context before LLM analysis.*

13. **SoK: DARPA's AI Cyber Challenge (AIxCC): Competition Design, Results, and Lessons Learned.** Cen Zhang et al. USENIX Security 2026 (preprint).
    <https://www.usenix.org/system/files/usenixsecurity26-zhang-cen.pdf>
    *Systematization of autonomous find/patch/validate systems; reliance on fuzzing harnesses and validated findings.*

## Evaluation, risk, and LLM-as-attack-surface

14. **Lost at C: A User Study on the Security Implications of Large Language Model Code Assistants.** Sandoval et al. USENIX Security 2023.
    <https://www.usenix.org/conference/usenixsecurity23/presentation/sandoval>
    *User study (N=58): LLM-assisted code can be less secure; LLMs also introduce vulnerabilities.*

15. **We Have a Package for You! A Comprehensive Analysis of Package Hallucinations by Code Generating LLMs.** USENIX ;login:.
    <https://www.usenix.org/publications/loginonline/we-have-package-you-comprehensive-analysis-package-hallucinations-code>
    *LLMs invent package names, creating a supply-chain (slopsquatting) risk.*

16. **Exploiting Task-Level Vulnerabilities: An Automatic Jailbreak Attack.** Lan Zhang et al. USENIX Security 2025.
    <https://www.usenix.org/system/files/usenixsecurity25-zhang-lan.pdf>
    *Automatic jailbreak construction; the LLM itself is an attack surface.*

17. **Quantifying Large Language Model Attacks Through the Lens of Model Internals.** Xiuming Liu et al. USENIX Security 2026 (preprint).
    <https://www.usenix.org/conference/usenixsecurity26/presentation/liu-xiuming>
    *Layer-wise probing of hidden states to detect adversarial/harmful prompting before generation.*

## Foundational background (non-USENIX, for context)

- The halting problem and Rice's theorem — basis for undecidability of full static analysis.
- The oracle problem in software testing — basis for why finding is not proving.
- OWASP Top 10 and OWASP ASVS — common vulnerability classes and verification requirements.
- DARPA Cyber Grand Challenge and AIxCC — autonomous cyber reasoning systems.
- Google Project Zero "Big Sleep" and "Naptime" — LLM-assisted vulnerability research and fuzzing harness generation (industry reports).

> Verification note: this document was assembled from USENIX presentation and proceedings pages retrieved during research. Bibliographic details (author lists, exact numbers) should be checked against the linked PDFs before formal citation. Preprint entries may change before final publication.
