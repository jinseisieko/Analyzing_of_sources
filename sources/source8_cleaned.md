# AI-Assisted Verification of SBOM Accuracy and Drift in Software Supply Chains

**Author:** Karthikeyan Thirumalaisamy

**Affiliation:** Independent Researcher, Washington, USA

**Corresponding Author:** kathiru11@gmail.com

**Received:** 23 October 2025 | **Revised:** 28 November 2025 | **Accepted:** 08 December 2025 | **Published:** 27 December 2025

## Abstract

Modern supply chain security depends on Software Bills of Materials (SBOMs), which provide transparency and compliance verification and component origin tracking for complex software systems. The fast-paced nature of DevSecOps operations causes SBOM accuracy to deteriorate rapidly. The actual software artifact contents differ from the declared SBOM because dependencies change, build environments transform, and automatic updates occur. The software supply chain becomes vulnerable to version spoofing, dependency confusion, and unintentional untrusted component usage because of this metadata accuracy decline. The paper presents an AI-based system that detects SBOM drift and measures the extent of discrepancies between declared and actual software content. The system uses machine learning to compare software build artifacts with dependency graphs and component metadata to detect any discrepancies between declared SBOM information and actual software components. The system evaluates each discrepancy through a four-level severity assessment, which ranges from Critical to Low. The system blocks deployment of builds that show High or Critical issues until developers fix the underlying problems. The system performs continuous automated checks to enhance supply-chain security while minimizing human inspection requirements and maintaining development pipeline compliance standards. The system enables organizations to maintain a dependable software supply chain that supports fast-paced development methods.

**Keywords:** SBOM, Software Bills of Materials, Vulnerability Management, Supply Chain Security, AI Security, DevSecOps

---

## 1 Introduction

The software supply chain has become more complex, which makes it difficult to track dependencies and build artifacts through their entire lifecycle. The Software Bill of Materials (SBOM) functions as a solution that addresses third-party library and external dependency problems in software systems. An SBOM contains complete information about software components, including their licensing details and origin points. Industry-recognized standards, including SPDX and CycloneDX, enable organizations to create and distribute SBOMs through standardized formats. Organizations can detect vulnerabilities more effectively through these standards, which also support their legal requirements.

The generation of SBOMs occurs at particular development stages, which restricts their ability to maintain accuracy because source code, dependencies, and container images change. The SBOM document loses its accuracy when SBOM drift occurs because it no longer reflects the actual software components. Security threats emerge from the combination of low SBOM accuracy, confidence, and SBOM drift because it enables dependency confusion, version mismatches, and hidden unknown components. The current review methods, based on rules and manual checks, do not identify major inconsistencies that occur in DevSecOps environments that undergo fast-paced changes.

The paper creates an Artificial Intelligence (AI) system that operates across the CI/CD pipeline to check SBOMs and identify drift events for achieving complete SBOM accuracy. The system employs artificial intelligence to analyze SBOM statements against build results and dependency lists and digital authentication evidence to detect any discrepancies. The system evaluates detected inconsistencies through a severity rating system, which classifies them as Critical, High, Medium, or Low. The system will prevent build and release operations from executing when it identifies any High or Critical severity discrepancies. The system allows organizations to verify software product authenticity and regulatory compliance while maintaining developer freedom. Organizations can achieve instant software product composition verification through intelligent SBOM validation, which reduces manual verification work and strengthens their software supply chain defenses.

## 2 Background and Motivation

This section provides an overview of the foundational concepts related to SBOMs, outlines their significance within contemporary software development workflows, and identifies key challenges in ensuring their ongoing accuracy.

### 2.1 Definition and Purpose of SBOMs

An SBOM is essentially a complete, accurate, machine-readable listing of every component, dependency, library, and other item included in software. The typical details include the component name, the version number, the name of the supplier, and additional information about the component, including licensing information and the results of the cryptographic hash. This information is useful to companies because it identifies the precise items of software being used and where, in those items, potential weaknesses are located.

The main goal of SBOM is to improve the transparency and traceability in the software supply chain. With an accurate and current inventory of the components of the software, developers and security professionals can easily find and assess known vulnerabilities, check on licensing requirements, and quickly react to new threats. In addition, SBOMs are also a key part of regulatory and compliance environments, in which companies need to prove they have knowledge and control over third-party and open-source software used in their products.

In summary, the SBOM provides a digital footprint for software composition, so there is visibility into the process from the time the developer begins building through the time the software is deployed. A correct SBOM enables the basis for managing vulnerabilities, assessing risk, and establishing good practices for the governance of secure software.

### 2.2 SBOM Generation Tools Across Languages

Tools to generate a Software Bill of Materials (SBOM) are available in all ecosystems, but the way these tools obtain dependency data differs by the specific build system and package manager for each language ecosystem. The overall objective remains the same: to determine the software composition of your application, though the tools you will use, the formats they produce, and the degree of support they have for integrating with your development workflow depend upon which ecosystem you work in, how mature that ecosystem is, and what metadata is available within that ecosystem.

**Table 1: SBOM Generation Tools Across Languages**

| Language | Tools | Formats | Notes |
|----------|-------|---------|-------|
| .NET / C# | Microsoft SBOM Tool, CycloneDX .NET, Syft | SPDX, CycloneDX | Integrated with Azure DevOps, supports NuGet and DLL metadata |
| Java / JVM | CycloneDX Maven/Gradle Plugin, Syft, ORT | SPDX, CycloneDX | Reads POM and Gradle metadata, supports JAR and WAR packaging |
| Python | Syft, CycloneDX Python, pip-audit, FOSSA CLI | SPDX, CycloneDX | Extracts dependency trees from requirements.txt and virtual environments |
| JavaScript / Node.js | CycloneDX Node, Syft, npm audit | SPDX, CycloneDX | Integrates with package-lock.json and yarn.lock |
| C/C++ | Syft, FOSSology, SCANCODE Toolkit | SPDX | Parses Make/CMake files and library manifests |
| Go (Golang) | Syft, CycloneDX Go, Go SBOM Generator | SPDX, CycloneDX | Supports Go modules; analyzes static binaries |
| Container Images | Syft, Anchore, Tern, Docker SBOM | SPDX, CycloneDX | Analyzes OS packages and embedded language layers |

## 3 The Challenge of SBOM Accuracy and Drift

Modern SBOM generation tools deliver excellent software component visibility, yet organizations face major difficulties when it comes to keeping their SBOMs up to date. The process of generating SBOMs happens during build or release operations to record dependency information at that particular time. The rapid changes in development environments create SBOM drift because the declared SBOM no longer matches the actual software components.

### 3.1 Automatic Dependency Updates

The current default mode of operation in modern software package managers is to obtain the most recent stable and secure versions of packages and libraries. Therefore, most package managers will automate the resolution of dependencies, especially when it comes to patch level and minor updates. Although there are some benefits to automating the dependency resolution with regard to security and performance, this automation can lead to unintentional SBOM drift.

SBOMs are typically created at a single point in time, typically during the initial build or release process of an application. When the SBOM captures a dependency version such as Library X v1.2.0, and the package manager then subsequently resolves Library X v1.2.3, the SBOM does not reflect the components that make up the final artifact. The difference between the SBOM and the actual components used in the final artifact may remain undetected due to several reasons:

- Automatic updates of patch levels occur without developers making conscious decisions
- Many build processes have "floating" versions specified in dependency declarations (e.g., ^1.2 or 1.2.x)
- Many lock files are not consistently updated or committed
- Different build agents may resolve dependencies differently due to different cache states or repository states

The continued occurrence of these silent and frequent updates results in the declared SBOM being out of sync with the dependency tree that was actually built. Although the updates were minor, they may introduce new vulnerabilities, change how components interact, or create licensing issues that the outdated SBOM does not identify.

### 3.2 Transitive Dependency Changes

The problem with transitive dependencies (i.e., packages that are pulled in as part of another package) is that while most applications rely upon both direct and indirect library packages, many of these indirect packages do not remain static—they tend to evolve based on updates made to the direct package. Since the primary application is typically unaware of the transitive dependencies that were established during build time, this means there will likely be times when transitive dependencies are updated, yet the SBOM has not been refreshed to reflect those changes.

There are several common ways in which SBOM drift occurs as a result of this dynamic:
- An upstream package developer updates the version of a library being used
- Security patches are applied to lower-level libraries
- A component is replaced or removed from a third-party package
- Build-time resolution of different transitive versions differs due to caching or environmental conditions

Since these updates to transitive dependencies occur independently of the application's own source code, developers are often not aware that these updates have occurred and therefore may not manually refresh the SBOM. As such, SBOM drift can lead to vulnerabilities or outdated sub-components being masked, thereby hindering the ability to gain a comprehensive view of the entire dependency tree.

### 3.3 Environment-Specific Builds

The software components included in a build may be influenced by the environment in which the build is performed. The operating system, the container base image, the build agent, and the system packages installed on the build server are all examples of how the environment can influence the software components included in a build, with varying results in terms of the final artifact created.

However, the source code for the application has not been changed. Differences in the build environment commonly cause what is referred to as environment-specific drift. Environment-specific drift typically occurs because of one or more of the following factors:
- OS-level packages differ (e.g., Alpine-based vs. Debian-based container images)
- Differing compilers or runtimes result in additional native dependencies
- Tools, SDKs, or cached packages are locally installed by the build agent and differ between build servers
- Resolution of platform-dependent packages may result in different binaries being selected

As a result of using SBOM generation in a singular environment while executing builds across multiple environments and/or stages, the SBOM may not be entirely representative of the dependencies contained within the final output.

### 3.4 Cached and Layered Builds

While many build tools today use caching and layering to accelerate build times and streamline CI/CD pipelines, both of which offer significant advantages in terms of efficiency, they can contribute to an accumulation of non-declared, but still present, components in the build, which ultimately contributes to drift.

The way many container-based builds operate is by using layers that are built upon other layers (previous builds), and will reuse layers from previous builds if there has been no detection of change to those layers. Layers reused from previous builds will contain all libraries and OS packages that existed at the time those layers were created, regardless of how old they may be.

Build tools such as Docker, Gradle, Maven, and the .NET incremental compiler are also designed to utilize cached artifacts, and in doing so, do not re-resolve nor verify dependencies against the original source(s). Caching of this nature results in several common examples of drift, including:
- Layers from previous container-based builds are reused, with older packages/libraries included
- Previously compiled modules that are cached are rebuilt with newer binary images while the cached image remains unchanged
- Cached local packages (e.g., NuGet, npm, pip) resolve to dependencies that have not changed since last installation
- CI/CD cache restore mechanisms restore older versions of artifacts during pipeline execution

Since the SBOM is typically generated from the declared dependency files (not the actual cached artifact), it will likely exclude any component that has accumulated through the caching process.

### 3.5 Manual or Ad-Hoc Changes

In many cases, especially in rapidly evolving projects, an organization will make updates outside of its normal build or dependency management process. When they do so, it is common for them to cause SBOM Drift as the component has been added, updated, or removed from the system, but there was no trigger to have the entire SBOM rebuild. Examples of such practices include:
- Directly patching or hotfixing deployed binaries
- Replacing DLLs or Libraries manually during debugging or troubleshooting
- Adding temporary dependencies into test environments that do not get added to project files
- Using local development overrides that utilize untracked local packages or custom libraries
- Making emergency fixes within a Production Environment by bypassing the standard pipeline

These types of manual changes are typically done without creating an automated record of those changes. Therefore, security and compliance teams do not know about undocumented components that are included in the final product.

## 4 AI-Assisted Verification Framework

As SBOM Drift occurs increasingly in modern pipeline environments, we are at a point where static generation alone will be insufficient to provide for the required reliability and trust. Organizations must therefore have mechanisms that continually verify that the stated SBOM matches the actual software being built by comparing it against the actual build output at every build step. In this Section, an Artificial Intelligence-assisted verification framework will be introduced. The Framework utilizes Artificial Intelligence (AI) to automatically determine build artifacts, dependency manifests, and component metadata to identify any differences between them; classify the differences by severity and ensure that the SBOM remains aligned to the current state of the software through the entire development lifecycle.

### 4.1 Architecture Overview

The AI-assisted verification framework is a modular platform to be integrated into current CI/CD frameworks, and will use an automated method (AI) to compare the actual build outputs of a software application to its stated SBOM. The Framework has four primary modules with AI incorporated into both the verification module and the risk assessment module.

### 4.2 Compile Source Code

The pipeline process starts with the compilation of the application's source code to create binary objects, libraries, or container images. The build process will include the entire dependency graph necessary to generate the final product through the retrieval of all dependencies declared in the application's project files by the build tool/package manager; and also including any other transitive dependencies; and/or platform specific objects/components and/or any runtime libraries that were automatically included during the build process.

Depending on how many different environments are involved (local builds, CI servers, containerized builds, etc.), it may cause different dependency resolution due to differences in available dependency caches, base images, or runtime versions. Due to these differences, the build/compile phase is typically the first place where minor differences between the final set of components can be introduced.

### 4.3 SBOM Generation

Once the application's source code was successfully compiled by the system, the system produces a Software Bill of Materials (SBOM), which details the components the build process believes make up the application. Typically, the SBOM contains all of the direct dependencies associated with the application; the internal libraries used by the application; and metadata pulled from project configuration files, package manifest files, or other files used in conjunction with the build process.

Normally, SBOM tools capture detailed information about each component included in the application, including, but not limited to, name, version, license, and supplier information. However, because these SBOMs are created prior to post-build operations such as building containers, creating binaries, or injecting run-time components into an application, the SBOM may represent the intended dependency list rather than the actual dependency list. Therefore, the SBOM produced at this time is referred to as the **declared SBOM** and is used as the primary documentation of what the developer declares the application contains.

### 4.4 SBOM Ingestion Layer

The SBOM Ingestion Layer serves as the standardization and preparation phase for verification. Since SBOMs are created using various formats (i.e., SPDX, CycloneDX, etc.) and/or language-specific metadata files, the ingestion layer creates a standardized version of the SBOM. As a result, the various analyses that occur after ingestion can operate on a common model, regardless of the environment/tools used to produce the SBOM.

As part of the ingestion process, the system verifies the SBOM structure, extracts key elements (i.e., component name, version, hash, license) from the SBOM, and resolves reference(s) to the dependency graph. In addition, the ingestion layer identifies any potential errors (i.e., incomplete metadata, missing field(s), inconsistent formatting) within the SBOM that may affect the accuracy of verification.

### 4.5 Build Artifact Analyzer (AI-Assisted)

The Build Artifact Analyzer examines build process outputs to determine which components actually exist in the final software product. The analyzer detects actual content within compiled binaries, packages, and container images, whereas the SBOM shows what the build system planned to include.

The analyzer checks different types of artifacts based on the technology stack used in the application:
- .NET assemblies together with their associated NuGet packages
- Java applications through their JAR and WAR file formats
- Python applications through their wheel packages and distribution folders
- Node.js applications through their npm package directories
- Native binary files and compiled modules

### 4.6 Discrepancy Detection and Severity Assessment

The system compares the declared SBOM against the actual build artifacts to identify discrepancies. Each discrepancy is evaluated through a four-level severity assessment:

1. **Critical:** Security vulnerabilities or malicious components detected
2. **High:** Significant version mismatches or missing critical components
3. **Medium:** Minor version differences or license compliance issues
4. **Low:** Metadata inconsistencies or documentation gaps

The system blocks deployment of builds that show High or Critical issues until developers fix the underlying problems.

### 4.7 Continuous Monitoring and Feedback Loop

The system performs continuous automated checks to enhance supply-chain security while minimizing human inspection requirements and maintaining development pipeline compliance standards. The feedback loop ensures that:
- Developers receive immediate notification of discrepancies
- Security teams can track SBOM accuracy trends over time
- Compliance requirements are continuously met
- The SBOM remains synchronized with actual software composition

## 5 Conclusion

This paper presents an AI-assisted verification framework for detecting and managing SBOM drift in modern software supply chains. The framework addresses the critical challenge of maintaining SBOM accuracy in fast-paced DevSecOps environments where automatic dependency updates, transitive dependency changes, environment-specific builds, cached and layered builds, and manual changes can cause significant discrepancies between declared and actual software components.

The proposed system uses machine learning to compare software build artifacts with dependency graphs and component metadata, evaluating discrepancies through a four-level severity assessment. By blocking deployments with High or Critical issues and providing continuous automated monitoring, the framework enables organizations to maintain a dependable software supply chain that supports fast-paced development methods while ensuring security and compliance.

Future work will focus on expanding the AI model's detection capabilities, integrating with additional CI/CD platforms, and developing industry-standard metrics for SBOM accuracy and drift measurement.

## References

[1] SBOM fundamentals and standards (SPDX, CycloneDX)
[2] Software supply chain security research
[3] DevSecOps and CI/CD pipeline security
[4] Dependency management and vulnerability scanning
[5] AI/ML in software security
[6] SBOM generation tools documentation
[7] SBOM drift and accuracy research
[8] Container security and layered builds
[9] Automated vulnerability detection
[10] Software composition analysis
