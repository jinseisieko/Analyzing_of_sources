# Extracted Facts with Citations
## Source: "The Impact of SBOM Generators on Vulnerability Assessment in Python: A Comparison and a Novel Approach"

---

## Key Statistics and Facts

### Software Supply Chain Security

| Fact | Citation |
|------|----------|
| The security of the Software Supply Chain (SSC) is an increasing concern for users and developers as reported by ENISA | [34] |
| The UE Executive Order on Improving the Nation's Cybersecurity addresses SSC security | [31] |
| The SolarWinds Orion compromise demonstrated the far-reaching impact of compromised software distribution | [37] |
| SSC security depends on multiple factors, including the use of open-source software as dependencies | [32] |
| 96% of 1,703 commercial codebases across 17 industry sectors leverage open-source code (Synopsys 2023 study) | [45] |
| 76% of total application code was open-source (Synopsys 2023 study) | [45] |
| Targeting software components like libraries allows attackers to affect wide-range software using a single entry point | [27, 32, 33, 48] |

### SBOM (Software Bill of Materials)

| Fact | Citation |
|------|----------|
| CISA defines SBOM as "a formal record containing the details and supply chain relationships of various components used in building software" | [17] |
| Most common SBOM formats: SWID tagging, SPDX, and CycloneDx | [17] |
| Tools developed to receive SBOM as input and provide security information | [1, 6, 8, 18] |
| SBOM tools rely on open-source vulnerability databases (e.g., NVD) to map components to vulnerabilities | [25] |
| SBOMs can be complemented with Vulnerability Exploitability eXchange (VEX) | - |
| Lack of standardized SBOM format leads to limitations in accuracy of existing SBOM generation tools | [20] |
| Wrongly identifying components' names, versions, or dependencies impairs SBOM representation capabilities | [20] |
| SBOM generation tools are prone to parsing errors or inability to correctly gather all dependencies | [19, 22, 23, 42] |
| Code-centric call graph analysis and behavioral analysis are highly resource intensive | [30, 38–40] |

### Programming Languages and SBOM Research

| Fact | Citation |
|------|----------|
| Python is the most popular programming language in 2024 according to IEEE | [21] |
| Existing work evaluated SBOM generation tools for Java | [19] |
| Existing work evaluated SBOM generation tools for JavaScript | [42] |
| Existing work evaluated SBOM generation tools for Python | [22] |
| No existing work evaluated SBOM tools' impact on detection of security issues (prior to this study) | - |

### SBOM Generation Tools

| Fact | Citation |
|------|----------|
| 169 open-source SBOM tools listed on CycloneDX tool center | - |
| Five selected tools for evaluation: cdxgen, GH-sbom, ORT, Syft, and Trivy | - |
| cdxgen uses Environment Based generation method | - |
| cdxgen is used by Shiftleft Scan and Macaron | [15] |
| Syft uses Metadata Based generation method | - |
| Syft is used by Grype and KubeClarity | - |
| Trivy uses Metadata Based generation method | - |
| Trivy is used by KubeClarity | - |
| GH-sbom uses Dependency Graph Based generation method | - |
| GH-sbom cannot acquire an SBOM for a specific commit or tag | [16] |
| Previous research reported SBOMs have scarce accuracy with static metadata-based generation | [42, 49] |
| NTIA established minimum required elements for an SBOM | [35] |
| sbom-scorecard can quantify SBOM compliance level | [9] |

### Python Package Management

| Fact | Citation |
|------|----------|
| Python has multiple package managers with different dependency management approaches | - |
| Python projects should contain either setup.py or pyproject.toml | - |
| PIP uses the resolvelib package for dependency resolution | [24] |
| PyPI allows multiple versioning schemas: semantic versioning, calendar versioning, and combinations | [11, 29, 41] |
| jq is a JSON parser used for parsing SBOMs | [5] |

### Vulnerability Scanners and Databases

| Fact | Citation |
|------|----------|
| Vulnerability databases: NVD and OSV | - |
| ShiftLeftScan, Grype, KubeClarity, and Bomber use SBOM as source for dependency analysis | [1, 6, 8, 18] |
| Grype is a vastly used tool for security analysis of projects | [3, 12, 13, 47] |
| Grype covers multiple languages (Python, Go, Rust) and artifacts (Docker images, filesystems, SBOMs) | - |
| pip-audit is largely used to detect vulnerabilities in dependencies | - |
| NVD had trouble collecting CVEs for a long time, causing issues for tools relying on it | [36] |

### Package Manager Distribution (from 590 packages)

| Package Manager | Packages | Percentage |
|-----------------|----------|------------|
| setuptools | 451 | 76.44% |
| hatch | 85 | 14.41% |
| poetry | 38 | 6.44% |
| pdm | 9 | 1.53% |
| pipenv | 7 | 1.19% |
| conda | 0 | 0.00% |

### Study Results

| Fact | Value |
|------|-------|
| Sample size: 1000 packages from PyPI | - |
| Margin of error: 3.04% at 95% confidence level | - |
| Cross-validation error: ±2% | - |
| All analyzed SBOM tools cannot correctly identify vulnerabilities in more than 20% of cases (except cdxgen) | - |
| cdxgen achieves correct identification in almost 40% of cases | - |
| On average 75% of dependencies listed in SBOMs are not actually installed | - |
| 99.5% of misclassified vulnerabilities are false positives for cdxgen (978 FP/5 FN) | - |
| 97.8% of misclassified vulnerabilities are false positives for Syft (926 FP/21 FN) | - |

### PIP-sbom Performance

| Metric | PIP-sbom | Best Competing (cdxgen) | Improvement |
|--------|----------|------------------------|-------------|
| Jaccard Similarity | 78.39% | 49.77% | +28.62% |
| Average Precision | 80.95% | 17.08% | +64% |
| Average Recall | 80.26% | 21.42% | +59% |
| False Positives/False Negatives | 47/3 | 978/5 | ~10x reduction |

### Related Works

| Fact | Citation |
|------|----------|
| Yu et al. conducted differential analysis on SBOM correctness across seven programming languages | [49] |
| Torres-Arias et al. studied NTIA minimum required elements for SBOMs using SPDX standard | [46] |
| Halbritter and Merli studied CycloneDX SBOMs | [28] |
| Balliu et al. focused on Java ecosystem SBOM challenges | [19] |
| Rabbi et al. focused on npm ecosystem | [42] |
| Cofano et al. studied Python ecosystem and SBOM generation tools | [22] |
| Sharma et al. proposed technology using SBOM to mitigate vulnerabilities in Java | [43] |
| Enck et al. reported SBOM security use is debated among practitioners | [26] |
| Zahan et al. reported S3C2 Industry Summit attendees were skeptical about SBOM adoption | [50] |
| According to OpenSSF, only npm ships SBOM generation for hosted packages | [44] |
| Only npm and homebrew provide provenance information | [44] |

### Security Concepts

| Fact | Citation |
|------|----------|
| False negatives in vulnerability detection | [2] |
| Intrusion detection concepts | [4] |
| False positives and false negatives in information security | [14] |
| Macaron: Logic-based Framework for Software Supply Chain Security Assurance | [15] |

---

## Reference URLs (from bibliography)

| # | URL/Source |
|---|------------|
| [1] | bomber: SBOM vulnerability scanner |
| [2] | https://www.contrastsecurity.com/glossary/false-negative |
| [3] | https://www.cisa.gov/resources-tools/services/grype |
| [4] | https://owasp.org/www-community/controls/Intrusion_Detection |
| [5] | https://github.com/jqlang/jq |
| [7] | https://pypi.org/project/resolvelib/ |
| [9] | SBOM-scorecard |
| [10] | https://s3c2.org/ |
| [11] | https://packaging.python.org/en/latest/discussions/versioning/ |
| [12] | https://www.chainguard.dev/unchained/why-chainguard-uses-grype-as-its-first-line-of-defense-for-cves |
| [13] | https://edu.chainguard.dev/chainguard/chainguard-images/working-with-images/scanners/grype-tutorial/ |
| [14] | https://www.guardrails.io/blog/false-positives-and-false-negatives-in-information-security/ |
| [15] | https://doi.org/10.1145/3605770.3625213 |
| [16] | https://github.com/orgs/community/discussions/118612 |
| [17] | https://www.cisa.gov/resources-tools/resources/sbom-faq |
| [18] | https://github.com/anchore/grype/ |
| [21] | https://spectrum.ieee.org/ibm-quantum-computer-2668978269 |
| [22] | https://arxiv.org/abs/2409.01214 |
| [24] | https://pip.pypa.io/en/stable/topics/more-dependency-resolution/ |
| [29] | https://calver.org/ |
| [30] | https://doi.org/10.1007/s10664-021-10071-9 |
| [31] | https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/ |
| [34] | https://www.enisa.europa.eu/publications/enisa-threat-landscape-2021 |
| [36] | https://securityscorecard.com/blog/national-vulnerability-database-nvd-leaves-thousands-of-vulnerabilities-without-analysis-data/ |
| [38] | https://doi.org/10.1109/ICSM.2015.7332492 |
| [39] | https://doi.org/10.1109/ICSME.2018.00054 |
| [40] | https://doi.org/10.1007/s10664-020-09830-x |
| [41] | https://semver.org/ |
| [43] | https://arxiv.org/abs/2407.00246 |
| [44] | https://openssf.org/blog/2024/07/31/how-to-make-programming-language-package-repositories-more-secure/ |
| [45] | https://www.synopsys.com/content/dam/synopsys/sig-assets/reports/rep-ossra-2023.pdf |
| [49] | https://doi.org/10.1109/DSN58291.2024.00018 |

---

## Data Availability

The list of projects, ground truth, generated SBOMs, security reports, and scripts to replicate results: https://osf.io/9agz7/?view_only=1c4704de735b46de8595c40dfa4fb1ad

---

## Funding

- European Commission Horizon Europe Programme, project LAZARUS (Grant Agreement no. 101070303)
- Project SERICS (PE00000014) under MUR National Recovery and Resilience Plan funded by EU - NextGenerationEU
