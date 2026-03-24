# Study Results and Findings
## Source: "The Impact of SBOM Generators on Vulnerability Assessment in Python: A Comparison and a Novel Approach"

---

## Main Research Questions

### RQ1: How much does the SBOM generation process impact the detection of vulnerabilities in the dependency network of an SSC?

### RQ2: How can we improve the SBOM generation approach to achieve better performance on the security assessment of the dependency network in an SSC?

---

## Key Findings

### Finding 1: Current SBOM Generation Tools Are Not Suitable for Vulnerability Assessment

**Result:** State-of-the-art SBOM generation tools cannot provide SBOMs accurate enough for vulnerability scanners to identify more than 20% of the vulnerabilities actually present in the software.

**Evidence:**
- All analyzed SBOM generation tools failed to correctly identify vulnerabilities in more than 80% of cases
- Only exception: cdxgen achieved correct identification in almost 40% of cases
- Average precision across all tools: 5.57% - 17.08%
- Average recall across all tools: 8.10% - 21.42%

---

### Finding 2: Extremely High False Positive Rates

**Result:** SBOM generation tools produce an overwhelming number of false positives, creating significant burden during vulnerability investigation.

**Evidence:**

| Tool | False Positives | False Negatives | FP Percentage |
|------|-----------------|-----------------|---------------|
| cdxgen | 978 | 5 | 99.5% |
| Syft | 926 | 21 | 97.8% |
| Trivy | 893 | 21 | 97.7% |
| ORT | 449 | 10 | 97.8% |
| GH-sbom | 2793 | 29 | 99.0% |

**Root Cause:** On average, **75% of dependencies listed in SBOMs are not actually installed**. This misalignment occurs because:
- SBOM tools read metadata files that list dependencies
- During actual installation, package managers only collect files specified in the build metadata
- This creates a gap between "declared" and "actually installed" dependencies

---

### Finding 3: Two Main Causes of SBOM Tool Ineffectiveness

**Cause 1:** SBOM generation tools do not properly support multiple package managers

- Python has 6+ package managers (setuptools, poetry, hatch, pipenv, pdm, conda)
- 76.44% of packages use setuptools
- Most tools only support a subset of these managers

**Cause 2:** SBOM generation tools do not consider how dependencies are actually collected by Python's package managers

- Tools rely on static metadata analysis
- They don't replicate the actual dependency resolution process
- This leads to incorrect dependency trees

---

### Finding 4: Tool Performance Comparison by Generation Method

**Environment-Based (Best Performance):**
- cdxgen: Jaccard 49.77%, Precision 17.08%, Recall 21.42%
- Installs dependencies in a virtual environment
- Supports most package managers

**Metadata-Based (Poor Performance):**
- Syft: Jaccard 26.33%, Precision 12.39%, Recall 15.61%
- Trivy: Jaccard 23.63%, Precision 12.17%, Recall 14.01%
- ORT: Jaccard 36.50%, Precision 16.31%, Recall 19.93%
- Do not install dependencies, only parse metadata files

**Dependency Graph-Based (Worst Performance):**
- GH-sbom: Jaccard 23.98%, Precision 5.57%, Recall 8.10%
- Depends on repository owner settings
- Only works when dependency graph feature is enabled
- Cannot acquire SBOM for specific commits or tags (only latest main branch)

---

### Finding 5: PIP-sbom (Proposed Solution) Dramatically Outperforms All Existing Tools

**Result:** By extending PIP to generate SBOMs natively using the package manager's own dependency resolution logic, performance improved drastically.

**Performance Comparison:**

| Metric | PIP-sbom | Best Competing (cdxgen) | Improvement |
|--------|----------|------------------------|-------------|
| Jaccard Similarity | 78.39% | 49.77% | +28.62 percentage points |
| Average Precision | 80.95% | 17.08% | **+64%** |
| Average Recall | 80.26% | 21.42% | **+59%** |
| False Positives | 47 | 978 | **~20x reduction** |
| False Negatives | 3 | 5 | Comparable |

**Key Advantages of PIP-sbom:**
1. Uses the same dependency resolution algorithm as PIP (resolvelib)
2. Mimics the actual installation process
3. Correctly identifies component names and versions
4. Correctly reports all software dependencies
5. Only 47 false positives vs. 978+ for other tools
6. Manageable number of vulnerabilities for developers to review

---

### Finding 6: The Solution Is Simple - Use Native Package Manager Generation

**Result:** The problems affecting SBOM generation tools can be solved by adding changes directly to package managers.

**Implications:**
- Developers using tools like ShiftLeft Scan or Grype are missing most actual vulnerabilities in their dependencies
- With minimal changes to package managers, SBOMs can be shipped with projects by default
- This approach is transferable to other programming language ecosystems

---

## Conclusions

### Primary Conclusion
**SBOM generation heavily affects vulnerability scan accuracy.** Current state-of-the-art tools fail to identify more than 80% of actual vulnerabilities due to:
1. Lack of support for Python's multiple package managers
2. Incorrect dependency resolution techniques

### Proposed Solution
**Native SBOM generation within package managers** (demonstrated via PIP-sbom) can:
- Increase precision by 64%
- Increase recall by 59%
- Reduce false positives by 10x
- Enable practical vulnerability assessment workflows

### Broader Impact
- Most software ecosystems lack provenance and SBOM generation (only npm ships SBOMs, only npm and homebrew provide provenance)
- Embedding SBOM generation in package managers is achievable without breaking build processes
- This should be adopted as a standard practice across all programming language ecosystems

---

## Recommendations for Software Communities

1. **Include SBOMs as part of projects by default** - easily achievable with native package manager support

2. **Use centralized SBOM** - provide a common, accessible knowledge base for all developers (GH-sbom offers this but with limitations)

3. **Standardize dependency listing files** - multiple data sources cause confusion and unexpected dependencies (npm uses package.json, Cargo uses Cargo.toml)

4. **Implement SBOM generation inside ecosystem package managers** - native generation methods largely improve SBOM quality and ecosystem-specific transparency

---

## Study Limitations

1. **Ground Truth:** Reliance on pip-audit may not detect all relevant vulnerabilities
2. **Tool Selection:** Manual selection process may have excluded potentially effective tools
3. **Scope:** Results tailored to Python; transferability to other languages cannot be guaranteed (though authors argue extensibility)
4. **Vulnerability Databases:** Some false positives/negatives due to missing vulnerability identifier mappings across databases

---

## Data and Reproducibility

All study materials available at: https://osf.io/9agz7/?view_only=1c4704de735b46de8595c40dfa4fb1ad

Includes:
- List of 1000 projects analyzed
- Ground truth vulnerability data
- Generated SBOMs from all tools
- Security reports
- Scripts to replicate results

---

## Funding

- European Commission Horizon Europe Programme, project LAZARUS (Grant Agreement no. 101070303)
- Project SERICS (PE00000014) under MUR National Recovery and Resilience Plan funded by EU - NextGenerationEU
