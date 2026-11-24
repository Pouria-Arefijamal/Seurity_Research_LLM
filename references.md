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

## 5G / 6G fuzzing and security

Sources retrieved via the arXiv API. Venue labels follow each paper's own statement. arXiv IDs are given for direct lookup; many are preprints.

### Protocol and RAN fuzzing

18. **Berserker: ASN.1-based Fuzzing of Radio Resource Control Protocol for 4G and 5G.** Srinath Potnuru, Prajwol Kumar Nakarmi. arXiv 2107.01912 (WiMob 2021).
    <https://arxiv.org/abs/2107.01912>
    *ASN.1-derived RRC/NAS fuzzing across spec versions; found 2 unknown vulnerabilities in srsLTE (one also in openLTE).*

19. **CovFUZZ: Coverage-based fuzzer for 4G&5G protocols.** Ilja Siroš, Dave Singelée, Bart Preneel. arXiv 2410.20958 (2024).
    <https://arxiv.org/abs/2410.20958>
    *Field-probability mutation guided by (or estimating) DUT coverage; vulnerabilities in 10 of 12 COTS devices and all srsRAN 4G instances.*

20. **MulCovFuzz: A Multi-Component Coverage-Guided Greybox Fuzzer for 5G Protocol Testing.** Yu Wang, Yang Xiang, Chandra Thapa, Hajime Suzuki. arXiv 2602.21794 (2026).
    <https://arxiv.org/abs/2602.21794>
    *Multi-component coverage + scoring on OpenAirInterface; +5.85% branch, +7.17% line, +16% unique crashes; 3 zero-days (2 unique to it).*

21. **Systematic Meets Unintended: Prior Knowledge Adaptive 5G Vulnerability Detection via Multi-Fuzzing.** Jingda Yang, Ying Wang, Yanjun Pan, Tuyen X. Tran. arXiv 2305.08039 (2023).
    <https://arxiv.org/abs/2305.08039>
    *LAL/SyAL/SoAL strategies by attacker knowledge; 129 RRC vulnerabilities at 0.072s average; 3 MITM vulnerabilities.*

22. **Formal and Fuzzing Amplification: Targeting Vulnerability Detection in 5G and Beyond.** Jingda Yang, Ying Wang. arXiv 2307.05758 (2023).
    <https://arxiv.org/abs/2307.05758>
    *Formal traces seed fuzzing and vice versa; 53 vulnerabilities on 5G NSA RRC.*

23. **Formal-Guided Fuzz Testing: Targeting Security Assurance from Specification to Implementation for 5G and Beyond.** Jingda Yang, Sudhanshu Arya, Ying Wang. arXiv 2307.11247 (2023).
    <https://arxiv.org/abs/2307.11247>
    *Hierarchical formal + fuzzing; 61 vulnerabilities on srsRAN; identifier leakage, DoS, eavesdropping, MITM despite TLS; exponential to linear search.*

24. **NLP-based Cross-Layer 5G Vulnerabilities Detection via Fuzzing Generated Run-Time Profiling.** Zhuzhu Wang, Ying Wang. arXiv 2305.08226 (2023).
    <https://arxiv.org/abs/2305.08226>
    *Classifies fuzzing impact from LogInfo telemetry; 93.4%-95.9% accuracy; a learned oracle.*

### Core network (SBA) fuzzing

25. **Cross-Service Token: Finding Attacks in 5G Core Networks (FivGeeFuzz).** Anqi Chen, Riccardo Preatoni, Alessandro Brighente, Mauro Conti, Cristina Nita-Rotaru. arXiv 2509.08992 (2025).
    <https://arxiv.org/abs/2509.08992>
    *Grammar-based fuzzing of 3GPP SBIs on free5GC; 8 unknown vulnerabilities including the Cross-Service Token Attack; 7 patched.*

### Baseband, devices, and hardware-in-the-loop

26. **BaseSAFE: Baseband SAnitized Fuzzing through Emulation.** Dominik Maier, Lukas Seidel, Shinjo Park. arXiv 2005.07797 (WiSec '20).
    <https://arxiv.org/abs/2005.07797>
    *Rehosting + sanitizing allocator; heap OOB in MediaTek baseband; ~15k test cases/s total.*

27. **ARIstoteles — Dissecting Apple's Baseband Interface.** Tobias Kröll, Stephan Kleber, Frank Kargl, Matthias Hollick, Jiska Classen. arXiv 2604.23457 (ESORICS 2021).
    <https://arxiv.org/abs/2604.23457>
    *Reverse-engineering + auto-generated dissector + fuzzing of the iOS ARI interface.*

28. **Soft Tester UE: A Novel Approach for Open RAN Security Testing.** Joshua Moore, Aly Sabri Abdalla, Charles Ueltschey, Vuk Marojevic. arXiv 2410.09641 (IEEE VTC Fall 2024, RitiRAN Workshop).
    <https://arxiv.org/abs/2410.09641>
    *Software-defined test UE sending fuzzed data over Uu; lowers testbed cost.*

29. **SIMulator: SIM Tracing on a (Pico-)Budget.** Gabriel K. Gegenhuber, Philipp É. Frenzel, Adrian Dabrowski. arXiv 2506.20800 (WiSec 2025 poster).
    <https://arxiv.org/abs/2506.20800>
    *Low-cost SIM-modem APDU tracing enabling fuzzing and key monitoring.*

30. **5uGUE: Agentic Hardware-in-the-Loop Fuzzing for 5G Devices and beyond.** Guoqiang Hua, Sakshi Udeshi, Matheus Eduardo Garbelini, Daisuke Mashima, Sudipta Chattopadhyay. arXiv 2610.29155 (2026).
    <https://arxiv.org/abs/2610.29155>
    *LLM HIL agent on COTS 5G devices; 167 crashes vs 13 for 5GHOUL; CVE-2026-20525; extended to Bluetooth.*

31. **Smart Fuzzing of 5G Wireless Software Implementation.** Huan Wu, Brian Fang, Fei Xie. arXiv 2309.12994 (2023).
    <https://arxiv.org/abs/2309.12994>
    *AFL++ on OpenAirInterface5G config files; LLM (Bard) interprets parameters.*

### 6G and algorithmic components

32. **AI-Driven Fuzzing for Vulnerability Assessment of 5G Traffic Steering Algorithms.** Seyed Bagher Hashemi Natanzi, Hossein Mohammadi, Bo Tang, Vuk Marojevic. arXiv 2601.18690 (2026).
    <https://arxiv.org/abs/2601.18690>
    *NSGA-II adversarial search in NVIDIA Sionna; 34.2% more vulnerabilities, 5.8% more critical failures; 6G-ready.*

33. **Security and privacy for 6G: A survey on prospective technologies and challenges.** Van-Linh Nguyen, Po-Ching Lin, Bo-Chao Cheng, Ren-Hung Hwang, Ying-Dar Lin. arXiv 2108.11861 (IEEE COMST 2021).
    <https://arxiv.org/abs/2108.11861>

34. **6G White paper: Research challenges for Trust, Security and Privacy.** Mika Ylianttila et al. arXiv 2004.11665 (2020).
    <https://arxiv.org/abs/2004.11665>

35. **Zero Trust Architecture for 6G Security.** Xu Chen, Wei Feng, Ning Ge, Yan Zhang. arXiv 2203.07716 (2022).
    <https://arxiv.org/abs/2203.07716>

36. **Bridging the Security Gap: Lessons from 5G and What 6G Should Do Better.** Isabella D. Lutz, Matthew C. Valenti. arXiv 2501.11045 (2025).
    <https://arxiv.org/abs/2501.11045>

37. **Toward a Unified Security and Privacy Framework for AI-Native 6G Networks.** Bidushi Barua et al. arXiv 2607.01019 (2026).
    <https://arxiv.org/abs/2607.01019>

38. **Towards a Security Plane for 6G Ecosystems.** Xavi Masip-Bruin, Eva Rodríguez, Admela Jukan, Panos Trakadas. arXiv 2512.20733 (2025).
    <https://arxiv.org/abs/2512.20733>

## GPU-accelerated and LLM-assisted fuzzing

Retrieved via arXiv; venues per paper's own statement. See `docs/09-gpu-llm-fuzzing.md` for analysis.

### GPU execution, targets, and oracles

39. **CuEVM: Scaling Smart Contract Execution on GPUs.** Nhut-Minh Ho et al. arXiv 2611.14916 (ICDE 2027).
    <https://arxiv.org/abs/2611.14916>
    *GPU-native EVM; >8M ERC-20 transfers/s; CuEVM-fuzz >1M exec/s, up to 23x/4.8x over baselines.*

40. **cuFuzz: Hunting CUDA Bugs at Scale.** Mohamed Tarek Ibn ziad, Christos Kozyrakis. arXiv 2603.12485 (OOPSLA 2026).
    <https://arxiv.org/abs/2603.12485>
    *Whole-program CUDA fuzzing, NVBit coverage, decoupled sanitizers; 43 unknown bugs (19 commercial).*

41. **GPU-Fuzz: Finding Memory Errors in Deep Learning Frameworks.** Zihao Li et al. arXiv 2602.10478.
    <https://arxiv.org/abs/2602.10478>
    *Operator parameters as formal constraints; 13 unknown bugs across PyTorch/TensorFlow/PaddlePaddle.*

42. **CuFuzz: Hardening CUDA Programs through Transformation and Fuzzing.** Saurabh Singh et al. arXiv 2601.01048.
    <https://arxiv.org/abs/2601.01048>
    *Compiler-runtime transforms (PREX, AXIPrune); up to 224x; 122 vulnerabilities.*

43. **Challenges and Design Considerations for Finding CUDA Bugs Through GPU-Native Fuzzing.** Mingkai Li et al. arXiv 2603.05725 (HotEthics 2026).
    <https://arxiv.org/abs/2603.05725>
    *Faithfulness argument against CPU translation for GPU fuzzing.*

44. **WarpDRF: The Unwritten Contract of GPU Warp Programming.** Zheyuan Chen et al. arXiv 2610.07541.
    <https://arxiv.org/abs/2610.07541>
    *Formal (Rocq) warp model, 10K conformance tests across CUDA/HIP/HLSL/Metal/SPIR-V; 3 races in llama.cpp.*

45. **DarthShader: Fuzzing WebGPU Shader Translators & Compilers.** Lukas Bernhard et al. arXiv 2409.01824 (CCS 2024).
    <https://arxiv.org/abs/2409.01824>
    *IR + AST mutators across the shader pipeline; 39 faults, 15 CVEs in Chrome/Firefox/Safari.*

46. **IcFuzz: Fuzzing Isaac Sim with Semantic Stage Guidance and Multi-level Mutation.** Zhixiang Chen et al. arXiv 2608.06088 (ASE 2026).
    <https://arxiv.org/abs/2608.06088>
    *LLM stage segmentation + bandit scheduler; 11 bugs, 9 confirmed/fixed.*

47. **Revisiting Neural Program Smoothing for Fuzzing.** Maria-Irina Nicolae, Max Eisele, Andreas Zeller. arXiv 2309.16618 (FSE 2023).
    <https://arxiv.org/abs/2309.16618>
    *NPS fuzzers underperform greybox; MLFuzz GPU platform for ML-fuzzing evaluation.*

48. **The Correctness Illusion in LLM-Generated GPU Kernels.** Dipankar Sarkar. arXiv 2606.20128.
    <https://arxiv.org/abs/2606.20128>
    *Allclose-on-one-shape oracle certifies buggy kernels; seeded differential fuzzing catches 10/10.*

49. **Measuring the Checker: Mutation Analysis for GPU-Kernel Benchmark Oracles.** Mingzhe Du et al. arXiv 2609.22220.
    <https://arxiv.org/abs/2609.22220>
    *Official KernelBench oracle misses 16.9% of witnessed faults (78.6% of precision faults); KernelBench-M released.*

50. **From Formal Methods to Fuzzing and Back.** Alastair F. Donaldson. arXiv 2611.12338.
    <https://arxiv.org/abs/2611.12338>
    *GPUVerify/GraphicsFuzz lineage; oracle problem links fuzzing and formal methods; Keystone.*

### GPU symbolic execution

51. **Large Language Model Powered Symbolic Execution (AutoBug).** Yihe Li, Ruijie Meng, Gregory J. Duck. arXiv 2505.13452 (OOPSLA 2025).
    <https://arxiv.org/abs/2505.13452>
    *Path-decomposed LLM symbolic execution; language-agnostic; consumer hardware.*

52. **M2K: Making the Model-Kernel Interface Explicit for Reliable CUDA Kernel Verification.** Mengting He et al. arXiv 2603.24595.
    <https://arxiv.org/abs/2603.24595>
    *cuKLEE symbolic execution on CUDA; 181 unknown bugs, 9 false positives.*

### LLM-assisted fuzzing

53. **KernelGPT.** Chenyuan Yang, Zijie Zhao, Lingming Zhang. arXiv 2401.00563 (ASPLOS 2025).
    <https://arxiv.org/abs/2401.00563>
    *LLM synthesizes syscall specs; 24 kernel bugs; specs merged into Syzkaller.*

54. **SeedMind.** Wenxuan Shi et al. arXiv 2411.18143.
    <https://arxiv.org/abs/2411.18143>
    *LLM generates test-case generators, not just cases.*

55. **HGFuzzer.** Hanxiang Xu, Yanjie Zhao, Haoyu Wang. arXiv 2505.03425.
    <https://arxiv.org/abs/2505.03425>
    *LLM-directed greybox; 17/20 triggered, 9 new CVEs.*

56. **Fuzz4All.** Chunqiu Steven Xia et al. arXiv 2308.04748 (ICSE 2024).
    <https://arxiv.org/abs/2308.04748>
    *Universal LLM fuzzing across six languages; 98 bugs, 64 confirmed.*

57. **TitanFuzz.** Yinlin Deng et al. arXiv 2212.14834 (ISSTA 2023).
    <https://arxiv.org/abs/2212.14834>
    *Zero-shot LLM fuzzing of DL libraries; 65 bugs, 41 confirmed.*

58. **HyllFuzz: Large Language Model assisted Hybrid Fuzzing.** Ruijie Meng, Gregory J. Duck, Abhik Roychoudhury. arXiv 2412.15931.
    <https://arxiv.org/abs/2412.15931>
    *LLM as solver in hybrid concolic; 31-59% more branches; 7 new bugs.*

59. **LLAMAFUZZ.** Hongxiang Zhang et al. arXiv 2406.07714.
    <https://arxiv.org/abs/2406.07714>
    *LLM-enhanced greybox; +41 bugs over top competitor.*

60. **FuzzCoder.** Liqun Yang et al. arXiv 2409.01944.
    <https://arxiv.org/abs/2409.01944>
    *Fine-tuned LLM byte-level mutation.*

61. **ReFuzzer.** Iti Shree, Karine Even-Mendoza, Tomasz Radzik. arXiv 2508.03603.
    <https://arxiv.org/abs/2508.03603>
    *Feedback loop raises LLVM test validity from ~48% to ~97%.*

62. **RandLuzz: Fuzzing: Randomness? Reasoning!** Xiaotao Feng et al. arXiv 2507.22065.
    <https://arxiv.org/abs/2507.22065>
    *LLM-generated reachable seeds and bug-specific mutators; up to 4.8x seed speedup.*

63. **ProphetFuzz.** Dawei Wang et al. arXiv 2409.00922.
    <https://arxiv.org/abs/2409.00922>
    *LLM predicts high-risk option combinations; 21 CVEs.*

64. **SaFliTe: Fuzzing Autonomous Systems via Large Language Models.** Taohong Zhu et al. arXiv 2412.18727.
    <https://arxiv.org/abs/2412.18727>
    *LLM predictor for safety-relevant test cases in UAV fuzzing.*

65. **LLM4Fuzz.** Chaofan Shou et al. arXiv 2401.11108.
    <https://arxiv.org/abs/2401.11108>
    *LLM-guided smart-contract fuzzing; 5 critical vulnerabilities.*

66. **On the Challenges of Fuzzing Techniques via Large Language Models.** Linghan Huang et al. arXiv 2402.00350.
    <https://arxiv.org/abs/2402.00350>
    *Survey of LLM + fuzzing.*

### Deep-learning compiler fuzzing (oracle background)

67. **NNSmith.** Jiawei Liu et al. arXiv 2207.13066 (ASPLOS 2023).
    <https://arxiv.org/abs/2207.13066>
    *Valid DNN test models + gradient-based input search + differential testing; 72 bugs.*

68. **HirGen.** Haoyang Ma et al. arXiv 2208.02193 (ISSTA 2023).
    <https://arxiv.org/abs/2208.02193>
    *High-level IR testing; 21 bugs at TVM.*

69. **TorchProbe.** Qidong Su et al. arXiv 2310.20078.
    <https://arxiv.org/abs/2310.20078>
    *Dynamic-feature transformations for PyTorch/Triton; 20 bugs.*

70. **DESIL.** Chenyao Suo et al. arXiv 2504.01379.
    <https://arxiv.org/abs/2504.01379>
    *Silent MLIR bugs; 23 silent + 19 crash bugs.*

## Foundational background (non-USENIX, for context)

- The halting problem and Rice's theorem — basis for undecidability of full static analysis.
- The oracle problem in software testing — basis for why finding is not proving.
- OWASP Top 10 and OWASP ASVS — common vulnerability classes and verification requirements.
- DARPA Cyber Grand Challenge and AIxCC — autonomous cyber reasoning systems.
- Google Project Zero "Big Sleep" and "Naptime" — LLM-assisted vulnerability research and fuzzing harness generation (industry reports).

> Verification note: this document was assembled from USENIX presentation and proceedings pages retrieved during research. Bibliographic details (author lists, exact numbers) should be checked against the linked PDFs before formal citation. Preprint entries may change before final publication.
