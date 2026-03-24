# Vulnerable Open Source Dependencies: Counting Those That Matter

**Authors:** Ivan Pashchenko, Henrik Plate, Serena Elisa Ponta, Antonino Sabetta, Fabio Massacci

**Affiliations:**
- University of Trento, Trento, TN, Italy
- SAP Security Research, France

**Emails:** ivan.pashchenko@unitn.it, henrik.plate@sap.com, serena.ponta@sap.com, antonino.sabetta@sap.com, fabio.massacci@unitn.it

## Abstract

**Background:** Vulnerable dependencies are a known problem in today's open-source software ecosystems because OSS libraries are highly interconnected and developers do not always update their dependencies.

**Aim:** Our paper addresses the over-inflation problem of academic and industrial approaches for reporting vulnerable dependencies in OSS software, and therefore, caters to the needs of industrial practice for correct allocation of development and audit resources.

**Method:** Careful analysis of deployed dependencies, aggregation of dependencies by their projects, and distinction of halted dependencies allow us to obtain a counting method that avoids over-inflation. To understand the industrial impact of a more precise approach, we considered the 200 most popular OSS Java libraries used by SAP in its own software. Our analysis included 10905 distinct GAVs (group, artifact, version) in Maven when considering all the library versions.

**Results:** We found that about 20% of the dependencies affected by a known vulnerability are not deployed, and therefore, they do not represent a danger to the analyzed library because they cannot be exploited in practice. Developers of the analyzed libraries are able to fix (and actually responsible for) 82% of the deployed vulnerable dependencies. The vast majority (81%) of vulnerable dependencies may be fixed by simply updating to a new version, while 1% of the vulnerable dependencies in our sample are halted, and therefore, potentially require a costly mitigation strategy.

**Conclusions:** Our case study shows that the correct counting allows software development companies to receive actionable information about their library dependencies, and therefore, correctly allocate costly development and audit resources, which is spent inefficiently in case of distorted measurements.

**CCS Concepts:**
- Security and privacy → Software security engineering
- Software and its engineering → Software libraries and repositories; Open source model

**Keywords:** Vulnerable Dependency, Open-Source Software, Mining Software Repositories

---

## 1 Introduction

The inclusion of free open-source software (OSS) components in commercial products is a consolidated practice in the software industry: as much as 80% of the code of the average commercial product comes from OSS [13]. SAP is an active user of and contributor to OSS. In this paper we report our hands-on experience on the industry relevant measurement of vulnerable dependencies in OSS.

Current dependency analysis methodologies are based on assumptions that are not valid in an industrial context. They may not distinguish dependency scopes [8] (which may lead to reporting non-exploitable vulnerabilities), or consider only direct dependencies [4] (although security issues may be introduced transitively [9]). On the other hand, dependency analysis methodologies miss several important factors at all. For example, we could not find studies that distinguish dependencies, whose development had been suspended for unspecified time, although they may still introduce bugs and security vulnerabilities transitively. Additionally, current dependency management practices do not consider the fact that some dependencies are maintained and released simultaneously, and therefore, should be treated as a singular unit, while constructing dependency trees and reporting results of a dependency study.

These issues lead to an inefficient allocation of costly development and audit resources due to the distorted measurements of vulnerable dependencies.

Hence, we make the following contributions:
- A precise approach, that caters to the needs of industrial practice, for reliable measurement of vulnerable dependencies in open-source software;
- A tool to perform large-scale studies of (Maven-based) OSS libraries and to determine whether any of their dependencies are affected by known vulnerabilities
- An empirical study of 10905 library instances of the 200 Java Maven-based open-source libraries that are most frequently used in SAP software.

We found that as many as 20% of the dependencies affected by a known vulnerability are not deployed, and therefore, do not introduce vulnerabilities in the dependent library instances. Also, we found that the developers of the analyzed libraries could directly fix 82% (45% more comparing to a traditional approach) of their vulnerable dependencies. Our study indicates that, under a conservative model to characterize halted dependencies, 14% of the total number of dependencies are halted, and therefore, do not receive updates (including security fixes). Such dependencies should be used with caution, since mitigations of their vulnerabilities are costly.

## 2 Terminology

In this paper we rely on the terminology established among practitioners and used in well-known dependency management tools such as Apache Ivy and Apache Maven:

- **A library** is a separately distributed software component, which typically consists of a logically grouped set of classes (objects) or methods (functions). To avoid any ambiguity, we refer to a specific version of a library as a library instance.
- **A dependency** is a library instance, some functionality of which is used by another library instance (the dependent library instance).
- **A dependency is direct** if it is directly invoked from the dependent library instance.
- **A dependency tree** is a representation of a software library instance and its dependencies where each node is a library instance and edges connect dependent library instances to their direct dependencies.
- **A transitive dependency** is connected to the root library instance of a dependency tree through a path with more than one edge.
- **A project** is a set of libraries developed and/or maintained together by a group of developers. Dependencies belonging to the same project of the dependent library instance are own dependencies, while library instances maintained by other projects are third-party dependencies.
- **A deployed dependency** is actually delivered with the application or system that uses it, while a non-deployed dependency is only needed at the time of development (e.g., for testing) but is not a part of the artifact that is eventually released and operated in a production environment.
- **A library instance is outdated** if there exists a more recent instance of this library at the time of analysis. A halted library is such that the next estimated release time has been passed by far based on the interval of past releases.

## 3 Problem Statement

The construction of the complete bill of materials (BoM) of a project is a necessary preliminary step to determining which dependencies of a project are vulnerable and assessing the risk they represent and the effort needed to mitigate it.

Several approaches exist for analyzing software dependencies. However, they do not provide a reliable measurement of the situation with software dependencies, because they do not consider several key aspects:
- A non-negligible number of dependencies that appear in the BoM could not be possibly exploited because they are only used at development time (e.g., for testing purposes) and are not delivered with the actual software system in operation;
- Libraries from the same project should be treated differently than third-party libraries (the former should be maintained by the same team, which should fix them rather than wait for another project team to release a new non-vulnerable version);
- The mitigation strategy that should be used to deal with each vulnerable library depends on the above two and on the fact that the library might not be maintained any longer (halted).

### RQ1: How many actually vulnerable dependencies does a library have?

A dependency tree for a library may include dependencies that are used only for testing or development purposes and are not deployed in the released version. Since they are not shipped with the product, they cannot possibly be exploited. Hence, allocating resources to fix or mitigate these vulnerabilities is pointless. This is well-known to software developers:

> "In this case, it's a test dependency, so the vulnerability doesn't really apply..."
> "It's only a test scoped dependency which means that it's not a transitive dependency for users of XXX so there is no harm done..."

Several recent works [3, 4, 8] do not mention explicitly that they consider only deployed dependencies. Indeed, the very quotes above show that the paper actually included such dependencies in its study. As a result vulnerable dependency count may become severely over-inflated.

### RQ2: Who is responsible for vulnerable dependencies?

A key question for the user of a vulnerable library is to attribute responsibility for fixing it (or avoiding projects with bad security discipline altogether). Developers of a software project are responsible for own code of their project and its direct dependencies (i.e., to keep them up-to-date). Although the concept seems intuitively simple, the following issues may occur:

- **Own vs third-party dependencies:** Failure to distinguish them may incorrectly present as an insecure ecosystem with several vulnerable dependencies (a "dependency hell" [10]) what in reality is just a project that has broken its components into several libraries and did not fix its own vulnerable code.
- **Direct vs transitive dependencies:** A dependency tree may include several library instances that belong to the same project. Such dependencies should not be considered separately, since an update of one of those dependencies would automatically bring the new versions of all other dependencies from the same project. Hence, some transitive dependencies may actually be controlled directly from the project under analysis.

Proper distinction of these cases is very important for selection of an appropriate mitigation strategy and correct allocation of development resources for fixing security issues introduced by vulnerable dependencies.

### RQ3: How many direct dependencies can be actually fixed?

If an outdated direct dependency is affected by a known vulnerability, the simplest solution to mitigate this vulnerability is to update the dependent library to use the fixed version of the dependency [16]. However, this becomes impossible, if an OSS library becomes inactive [8]:

> "... our project has been inactive and production has been halted for indefinite time"

If a security vulnerability is discovered in a no longer actively developed library, there would be no version of this library that fixes the vulnerability. Hence, being a dependency, this library will introduce the vulnerability to all its dependents. Additionally, a halted dependency may transitively introduce outdated dependencies and expose the root library instance to bugs and security vulnerabilities.

Clearly, the presence of halted dependencies has a major impact on a company maintenance strategy. Indeed, any user of library would not obtain any benefit from switching to its latest version. The vulnerable version would always be present. A different mitigation strategy might be needed: (i) contribute to the halted library, i.e., to develop its new release; or (ii) fork the halted library and continue its maintenance as part of the dependent library.

## 4 Counting Dependencies in Maven

Considering the popularity and industrial relevance of Java, in the following we demonstrate our approach on Java projects.

Over the past decade, Apache Maven established itself as a standard solution in the Java ecosystem for dependency management and other tasks related to build processes. Other solutions exist, such as Apache Ivy and Gradle (which is gaining popularity), however Maven still has the largest share of users. Hence, we use it to demonstrate the proposed mitigations for each problem described in Section 3, although the concepts presented below can be easily extended to other dependency management systems.

In Maven the name of a component is standardized and represented as groupId:artifactId:version. Hence:
- A "project" may be referenced as Maven groupId
- A "library" corresponds to groupId:artifactId (GA)
- A "library instance" corresponds to the name of Maven component groupId:artifactId:version (GAV)

### 4.1 Dependency Resolution

For each of the library instances in our sample, we use Maven to determine the complete set of dependencies. Before doing so, Maven requires that the Project Object Model (POM) files be installed in the local repository. Once a POM is installed locally, we use the standard Maven goals dependency:tree and dependency:resolve to construct the dependency tree of each POM and to resolve conflicts and duplications.

### 4.2 Post-processing of the Results

The next step of our data collection process is a post-processing of the data obtained after the dependency resolution step to address the problems discussed in Section 3.

**Filter non-deployed dependencies.** To control whether a dependency is deployed with an artifact, Maven provides a possibility for a software developer to specify the scope. The dependencies with scopes provided and test are used only for development purposes and are not shipped with a released artifact, hence, we do not consider them for the further analysis as non-deployed dependencies.

**Dependency grouping.** The Maven dependency resolution process starts from the POMs under analysis as the major source of the necessary information to build the dependency trees. However, at the final step of our analysis, the vulnerable dependency represents the most valuable asset. Hence, we perform the final aggregation of the results in the opposite direction, i.e., considering the paths from vulnerable dependencies to libraries under analysis:
- We group all GAVs with the same groupId within one path and substitute them in the path with the GAV, closest to the vulnerable GAV

**Identification of halted dependencies.** Public software package repositories keep all published library instances, since there is a possibility to break a build of a library in case a library dependency is removed. So, even if a certain library is no longer maintained, it is still available for download from a software package repository. At the same time, when selecting a mitigation strategy, software developers need to know that a fixed version of a vulnerable dependency is going to appear (otherwise, a costly mitigation strategy is required).

We propose to consider the amount of time library developers require to release a new version for determining whether a library development becomes halted. Some libraries may have varying time intervals between releases due to different release strategies adopted within development teams, as well as the maturity of a certain library: at earlier stages of development it needs to have more updates than an established library with a long development history.

Since the time difference between recent releases should have bigger impact on the Last release interval comparing to the time difference between older releases, the typical statistical model that describes such a process is a simple Exponential Smoothing model [2]:

```
Last release interval = α Σ(1 − α)^i * Release time^(n−i)
Expected release date = Last release + Last release interval
```

where Release time_i is the time needed to release the i-th version of a library, 0 < α < 1 is the smoothing parameter that shows how fast the influence of previous time intervals decreases. We estimate the Expected release date for a library by adding the Last release interval to the release date of the latest available version of the library. Then we determine the status of the library as follows:
- next release date < TIME: the library is halted
- next release date ≥ TIME: the library is alive

## 5 Study Design

To evaluate our approach, we conducted an empirical study on the 200 most popular OSS Java libraries used by SAP in its own software. Our analysis included 10905 distinct GAVs (group, artifact, version) in Maven when considering all the library versions.

### 5.1 Sample Selection

We selected the 200 most popular Java Maven-based open-source libraries that are most frequently used in SAP software. The popularity was determined based on usage frequency within SAP's software portfolio.

### 5.2 Data Collection

For each library instance, we:
1. Retrieved the POM file from Maven Central repository
2. Used Maven to resolve the complete dependency tree
3. Filtered out non-deployed dependencies (test and provided scopes)
4. Grouped dependencies by project
5. Identified halted dependencies using our exponential smoothing model
6. Cross-referenced dependencies with known vulnerability databases

### 5.3 Vulnerability Data

We used publicly available vulnerability databases to identify known vulnerabilities affecting the dependencies in our sample. Each vulnerability was mapped to the affected library versions.

## 6 Results

### 6.1 RQ1: Actually Vulnerable Dependencies

We found that about 20% of the dependencies affected by a known vulnerability are not deployed, and therefore, they do not represent a danger to the analyzed library because they cannot be exploited in practice. This confirms that traditional approaches that do not distinguish between deployed and non-deployed dependencies significantly over-inflate the count of vulnerable dependencies.

### 6.2 RQ2: Responsibility for Vulnerable Dependencies

Developers of the analyzed libraries are able to fix (and actually responsible for) 82% of the deployed vulnerable dependencies. This is 45% more compared to a traditional approach that does not distinguish between own and third-party dependencies. The key insight is that many dependencies belong to the same project and can be fixed together with the main library.

### 6.3 RQ3: Fixable Dependencies

The vast majority (81%) of vulnerable dependencies may be fixed by simply updating to a new version. However, 1% of the vulnerable dependencies in our sample are halted, and therefore, potentially require a costly mitigation strategy. Under a conservative model to characterize halted dependencies, 14% of the total number of dependencies are halted, and therefore, do not receive updates (including security fixes).

## 7 Discussion

### 7.1 Implications for Industry

Our case study shows that the correct counting allows software development companies to receive actionable information about their library dependencies, and therefore, correctly allocate costly development and audit resources, which is spent inefficiently in case of distorted measurements.

The distinction between deployed and non-deployed dependencies is crucial for avoiding wasted effort on vulnerabilities that cannot be exploited in practice. Similarly, recognizing own dependencies enables development teams to take direct responsibility for fixing vulnerabilities rather than waiting for external updates.

### 7.2 Implications for Research

Our work highlights the need for more precise measurement approaches in software dependency analysis. Traditional metrics that do not account for deployment status, project boundaries, and library maintenance status may lead to misleading conclusions about the state of software supply chain security.

### 7.3 Threats to Validity

**Construct validity:** Our study relies on publicly available vulnerability databases, which may not be complete. However, we used multiple sources to maximize coverage.

**Internal validity:** The exponential smoothing model for identifying halted dependencies uses parameters based on our hands-on experience. Different parameters might yield different results.

**External validity:** Our study focuses on Java Maven-based projects. While the concepts are applicable to other ecosystems, the specific results may not generalize directly.

## 8 Related Work

Several approaches exist for analyzing software dependencies and identifying vulnerable components. However, they do not provide a reliable measurement of the situation with software dependencies because they do not consider several key aspects addressed in our work.

Recent works on dependency analysis have highlighted various aspects of the dependency management problem, including the prevalence of outdated dependencies, the challenges of transitive dependency management, and the impact of dependency vulnerabilities on software security.

## 9 Conclusion

This paper presents a precise approach for measuring vulnerable dependencies in open-source software that caters to the needs of industrial practice. By carefully analyzing deployed dependencies, aggregating dependencies by their projects, and distinguishing halted dependencies, we obtain a counting method that avoids over-inflation.

Our empirical study of 200 popular Java libraries used at SAP reveals that:
- 20% of dependencies affected by known vulnerabilities are not deployed and therefore not exploitable
- Developers can directly fix 82% of deployed vulnerable dependencies
- 81% of vulnerable dependencies can be fixed by simple updates
- 14% of dependencies are halted and require costly mitigation strategies

These findings demonstrate that correct counting allows software development companies to receive actionable information about their library dependencies and correctly allocate development and audit resources.

## References

[1] Software dependency analysis research
[2] Exponential Smoothing models
[3] Related dependency studies
[4] Direct dependency analysis
[5] OSS vulnerability research
[6] Software supply chain security
[7] Maven dependency management
[8] Dependency scope analysis
[9] Transitive dependency security
[10] Dependency hell research
[11] Vulnerability databases
[12] Software measurement
[13] OSS in commercial products
[14] Maven best practices
[15] Library maintenance studies
[16] Dependency update strategies
