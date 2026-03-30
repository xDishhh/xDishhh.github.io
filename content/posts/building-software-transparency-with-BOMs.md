+++
title = "Building Software Transparency with BOMs"
date = 2026-03-29
draft = true
+++

*How SBOMs, HBOMs, CBOMs, and ML-BOMs are reshaping how organisations understand — and defend — what's actually running in their systems.*

---

## The Wake-Up Call

On 14 March 2025, a security researcher noticed something unusual in a GitHub Actions workflow log: a base64-encoded payload quietly dumping secrets from a CI/CD runner's memory.

The culprit was `tj-actions/changed-files` — a GitHub Action used in over **23,000 repositories** to identify which files changed during a build. An attacker had compromised a bot account's Personal Access Token, injected malicious code into the action's source, and retroactively updated every version tag to point to the malicious commit. Every organisation that ran a workflow referencing this action suddenly found their most sensitive credentials — AWS keys, GitHub PATs, npm tokens, private RSA keys — printed directly into publicly accessible build logs.

> CISA added CVE-2025-30066 (CVSS 8.6) to its Known Exploited Vulnerabilities catalog within days.

When the advisory dropped, every security team faced the same urgent question: *"Do we use `tj-actions/changed-files`, and in which pipelines?"*

**[Diagram 1 — Supply chain attack anatomy: how the tj-actions compromise flowed]**

```bash
ls; whoami
```

Organisations with a Software Bill of Materials covering their CI/CD tooling answered that question in minutes. Those without one spent days manually auditing hundreds of workflow files.

That is the problem Bills of Materials exist to solve.

///

## What Is a Bill of Materials?

Think of the ingredients label on a tin of soup. It tells you everything inside: the components, their origin, what regulations they must meet. If one ingredient is recalled, the manufacturer knows instantly which products are affected.

**A Bill of Materials (BOM) in cybersecurity is that label for technology.** It is a formal, structured inventory of every component inside a piece of software, hardware, or firmware — its name, version, origin, licence, and any known vulnerabilities.

Modern software is never written from scratch. A typical enterprise application is built on hundreds of open-source libraries, each of which depends on dozens more. The code your own developers write might represent as little as 10–20% of what actually runs in production. The rest is a web of third-party components — created by others, maintained by organisations you have never spoken to.

Without a BOM, you do not know what you are running. And if you do not know what you are running, you cannot protect it.

---

## The BOM Family

There is not one type of BOM — there are five, each covering a different layer of your technology stack.

**[Diagram 2 — The five BOM types and what each captures]**

### SBOM — Software Bill of Materials

The most mature and widely adopted type. An SBOM lists every software component in an application: libraries, packages, transitive dependencies (the dependencies *of* your dependencies), and the licences governing each one. It is the most urgently required by regulators and the most immediately actionable when a vulnerability drops.

### HBOM — Hardware Bill of Materials

An HBOM documents the physical components inside a device — chips, circuit boards, processors — and critically, where each was manufactured and sourced. If a chip from a particular supplier turns out to be compromised at the fabrication stage, an HBOM tells you exactly which devices in your fleet are affected. Without one, scoping the problem is a manual, weeks-long exercise.

### FBOM — Firmware Bill of Materials

Firmware sits between hardware and software: the embedded code that tells a router, printer, or industrial controller how to boot. It is historically the least-scrutinised layer — often outdated, rarely patched, and deeply opaque. An FBOM makes every component inside that firmware visible and auditable.

### CBOM — Cryptography Bill of Materials *(emerging)*

A CBOM catalogues every cryptographic asset across your systems: algorithms, key lengths, TLS versions, and certificate authorities. This has become newly critical because NIST finalised its first post-quantum cryptographic standards in 2024. Organisations now need to know which cryptographic assets will become vulnerable as quantum computing matures — and that inventory is only possible with a CBOM. Without one, the migration is an unstructured scramble. With one, it is a planned, prioritised programme.

### ML-BOM — Machine Learning Bill of Materials *(emerging)*

As AI models are embedded into products, a new class of supply chain risk has emerged. In 2025, researchers documented active exploitation of Python Pickle files used in model serialisation — attackers embedding executable payloads inside model files that ran when the model was loaded. An ML-BOM tracks the components of a model: its base architecture, training dataset provenance, fine-tuning details, and inference framework versions. Without this inventory, detecting model poisoning or assessing exposure to a compromised third-party model is nearly impossible.

> All five BOM types are supported by **CycloneDX** (OWASP) — one framework for full-stack component visibility. **SPDX** (Linux Foundation) is the ISO standard and the preferred choice when licence compliance is the primary driver.

---

## Why It Matters

### The supply chain is the new attack surface

The `tj-actions` incident is representative, not exceptional. Sophisticated attackers no longer need to breach your perimeter directly. They compromise a trusted tool used by thousands of organisations and have malicious code run automatically inside those organisations' own pipelines — without touching a single firewall.

The Shai-Hulud npm worm (September 2025) demonstrated the same logic at scale. Attackers phished npm maintainer credentials and injected malicious payloads into widely used JavaScript packages. The worm was self-propagating: once installed, it scanned for npm tokens in the victim's environment and automatically used them to spread to additional packages — reaching over 500 package versions before being disrupted.

Supply chain attacks averaged more than 28 per month throughout 2025 — more than double the rate seen a year earlier. Software Supply Chain Failures debuted at **#3 on the OWASP Top Ten 2025**.

### Speed of response

When a critical vulnerability is disclosed, the first question is always: *are we affected?* Without a BOM, answering it requires manually searching codebases, lock files, container images, and pipeline configs across every product. That takes days.

Exploit code for critical vulnerabilities routinely appears within 24–72 hours of disclosure. The window to act is measured in hours. A BOM collapses that triage from days to minutes.

### The full dependency tree

Your application might have 50 direct dependencies but a transitive dependency footprint of 500 or more. The components your developers consciously chose are only part of the picture. A properly generated BOM captures the entire tree — making the invisible visible.

---

## The Regulatory Wave

For years, SBOMs were a best-practice recommendation implemented by the most sophisticated organisations and ignored by most others. That era is over.

**[Diagram 3 — Regulatory timeline from 2021 to 2026]**

The **US Executive Order 14028** (2021) and the subsequent **OMB directive M-22-18** (2022) made SBOMs a condition of sale for any software sold to the US federal government. Given the government's purchasing scale, this had industry-wide effects well beyond the public sector.

The **EU Cyber Resilience Act**, in force since December 2024 and fully enforced from 2026, is the most sweeping cybersecurity product law ever enacted. It covers all products with digital elements sold in the EU — making it a de facto global standard, since few organisations maintain separate product versions for different markets.

The **FDA's cybersecurity guidance** (2023) requires an SBOM as part of the premarket submission process for every medical device containing software. The **NIS2 Directive** extended supply chain security obligations across critical infrastructure sectors throughout Europe.

If you sell software to any government, sell products into the EU, manufacture medical devices, or operate critical infrastructure — you either have a BOM programme today or you are working against a legal deadline.

---

## How Organisations Use BOMs

A BOM is not a document you produce once and file away. It is a living artefact, regenerated with every build and continuously analysed.

**[Diagram 4 — The four-stage lifecycle: Generate → Share → Analyse → Act]**

**Generate.** Tools like Syft, Trivy, and cdxgen scan codebases, container images, and CI/CD configuration files to produce structured BOM documents automatically on every build. The key principle: automated and continuous, never manual and periodic.

**Share.** BOMs are published to customers, internal teams, procurement, and regulators in standardised formats. The emerging **VEX (Vulnerability Exploitability eXchange)** format lets vendors add context — not just "this CVE exists in our component" but "this CVE is not exploitable in our implementation" — dramatically reducing alert noise.

**Analyse.** BOM contents are automatically matched against vulnerability databases (NVD, OSV, commercial feeds) to surface known CVEs. This must run continuously: a safe component today may have a critical vulnerability disclosed against it tomorrow.

**Act.** When a vulnerable component is found, the BOM tells you where it appears, who owns the affected system, and how critical it is — enabling intelligent triage rather than treating every alert with equal urgency. Options include patching, replacing the component, accepting the risk with documentation, or isolating the system while a fix is prepared.

---

## The Challenges Still Ahead

Adoption is growing, but several gaps remain.

**Tooling inconsistency.** Different tools produce BOMs of varying quality — some miss transitive dependencies, others fail on certain language ecosystems. No single tool handles every stack well.

**The CI/CD and SaaS blind spot.** The `tj-actions` attack targeted CI/CD tooling that most SBOM programmes do not cover. SaaS applications present a similar problem: you have no visibility into a vendor's internal stack. Both remain significant gaps.

**Keeping BOMs current.** A BOM generated three months ago may be dangerously outdated. Dependencies update constantly and new vulnerabilities are disclosed daily. Generation must be tied to every build — not run quarterly.

**HBOM, FBOM, CBOM, and ML-BOM lag.** Software BOM adoption has matured. Hardware, firmware, cryptography, and AI/ML BOM programmes are still rare outside defence and critical infrastructure. Tooling is less mature, supplier participation is inconsistent, and documentation of model provenance is often poor even within the teams building the models.

---

## What's Next

**AI supply chain risk is accelerating.** Model poisoning, malicious serialisation, and compromised training data are active threats. ML-BOMs are the inventory that makes an organised response possible. CycloneDX already supports them as a first-class format.

**Quantum readiness via CBOMs.** Post-quantum cryptographic standards are finalised. The migration clock is running. Organisations that build a CBOM today can plan a prioritised transition. Those that do not will face a scramble when urgency becomes acute.

**Continuous, automated pipelines.** The future is every build artifact shipping with an embedded, cryptographically signed BOM — automatically ingested by vulnerability analysis and made available to customers through standardised APIs.

**Global regulatory convergence.** The UK, Japan, Australia, and India are developing their own frameworks. Within a few years, providing an SBOM will be a baseline expectation of any significant software transaction, anywhere in the world.

---

## The Bottom Line

A threat actor compromised a bot account and quietly touched 23,000 repositories. Organisations with BOM programmes knew their exposure within minutes. Those without them spent days finding out.

Modern software is an assembly of parts — most of them invisible to the organisations that depend on them. A Bill of Materials is how you start to see what's actually there.

It is the foundational layer of visibility on which faster response, better prioritisation, and regulatory compliance all depend. The organisations building serious BOM programmes today — across software, hardware, firmware, cryptography, and AI — are the ones that will move quickly when the next incident drops.

Given that over 130 new CVEs are disclosed every single day, that next incident will not be long in coming.

---

## Further Reading

**Official sources**
- [CISA SBOM Hub](https://www.cisa.gov/sbom) — guidance, minimum element requirements, working group reports
- [CISA Known Exploited Vulnerabilities Catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [NIST Post-Quantum Cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography)

**Standards**
- [CycloneDX](https://cyclonedx.org) — OWASP-backed; covers SBOM, HBOM, CBOM, ML-BOM
- [SPDX](https://spdx.dev) — ISO-approved; strong licence compliance focus

**Threat intelligence**
- [OWASP Top 10 2025 — A03: Software Supply Chain Failures](https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/)
- [Palo Alto Unit 42: GitHub Actions Supply Chain Attack](https://unit42.paloaltonetworks.com/github-actions-supply-chain-attack/)
- [Sonatype State of the Software Supply Chain](https://www.sonatype.com/state-of-the-software-supply-chain)

**Open-source tooling**
- [Syft](https://github.com/anchore/syft) — SBOM generator, multi-ecosystem
- [Trivy](https://github.com/aquasecurity/trivy) — security scanner with SBOM generation
- [cdxgen](https://github.com/CycloneDX/cdxgen) — CycloneDX-native generator
- [OWASP Dependency-Track](https://dependencytrack.org) — continuous SBOM analysis platform

---

*Sources: CISA, OWASP, Palo Alto Networks Unit 42, Sonatype, Sygnia, Infosecurity Magazine, Silobreaker, Trend Micro, StepSecurity.*