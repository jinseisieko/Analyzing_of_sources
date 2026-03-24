# Beyond Metadata: Code-centric and Usage-based Analysis of Known Vulnerabilities in Open-source Software

**Authors:** Serena Elisa Ponta, Henrik Plate, Antonino Sabetta

**Affiliation:** SAP Security Research, Mougins, France

**Emails:** {serena.ponta,henrik.plate,antonino.sabetta}@sap.com

## Abstract

The use of open-source software (OSS) is ever-increasing, and so is the number of open-source vulnerabilities being discovered and publicly disclosed. The gains obtained from the reuse of community-developed libraries may be offset by the cost of detecting, assessing, and mitigating their vulnerabilities in a timely manner.

In this paper we present a novel method to detect, assess and mitigate OSS vulnerabilities that improves on state-of-the-art approaches, which commonly depend on metadata to identify vulnerable OSS dependencies. Our solution instead is code-centric and combines static and dynamic analysis to determine the reachability of the vulnerable portion of libraries used (directly or transitively) by an application. Taking this usage into account, our approach then supports developers in choosing among the existing non-vulnerable library versions.

Vulas, the tool implementing our code-centric and usage-based approach, is officially recommended by SAP to scan its Java software, and has been successfully used to perform more than 250000 scans of about 500 applications since December 2016. We report on our experience and on the lessons we learned when maturing the tool from a research prototype to an industrial-grade solution.

**Keywords:** open source software; known vulnerabilities; code-centric vulnerability analysis; metric-based update support

---

## 1 Introduction

Open-source software (OSS) libraries are widely used in the software industry: by some estimates, as much as 80% to 90% of the software products on the market include some OSS component [1], and each of them contains, on average, 100 distinct open source components, whose code weighs as much as 35% of the overall application size [2]. The same study reports that for applications developed for internal use, the proportion is as high as 75%.

At the same time, the number of vulnerabilities disclosed for OSS libraries has been steadily increasing since 2009 [1]. While using OSS components with known vulnerabilities is included in the OWASP Top 10 Application Security Risks since 2013 [3], [4], still today the problem is far from being solved. On the contrary, OSS vulnerabilities have been hitting the headlines of mainstream media many times over the past few years [5], [6]. As reported by [7], OSS vulnerabilities were the root cause of the majority of the data breaches that happened in 2016. In 2017, the Equifax incident [8], caused by a missed update of a widely used OSS component, compromised the personal data of over 140 millions of U.S. citizens.

Establishing effective OSS vulnerability management practices, supported by adequate tools, is broadly understood as a priority in the software industry, and tools helping to detect known vulnerable libraries are available nowadays, either as OSS or as commercial products (e.g., [9], [10]). These tools differ in terms of detection capabilities, but (to the best of our knowledge) the approaches they use rely on the assumption that the metadata associated to OSS libraries (e.g., name, version), and to vulnerability descriptions (e.g., technical details, list of affected components) are always available and accurate. Unfortunately, these metadata, which are used to map each library onto a list of known vulnerabilities that affect it, are often incomplete, inconsistent, or missing altogether. Therefore, the tools that rely on them may fail to detect vulnerabilities (false negatives), or they may report as vulnerable artifacts that do not contain the code that is the actual cause of the vulnerability (false positives).

Furthermore, merely detecting the inclusion of vulnerable libraries does not cater for the needs of the entire software development life-cycle. In the early phases of development, updating a library to a more recent release is relatively unproblematic, because the necessary adaptations in the application code can be performed as part of the normal development activities. On the other hand, as soon as a project gets closer to the date of release to customers, and during the entire operational lifetime, all updates need to be carefully pondered, because they can impact the release schedule, require additional effort, cause system downtime, or introduce new defects.

To evaluate precisely the need and the urgency of a library update, it is necessary to answer the key question: "is the vulnerability exploitable, given the particular way the library is used within the application?". Answering this question is extremely difficult: vulnerabilities are typically described in advisories that consist of short, high-level, textual descriptions in natural language, whereas a reliable assessment of the exploitability and the potential impact of a vulnerability demands much lower-level, detailed, technical information.

In our previous work [11], we introduced a method to analyze the code changes introduced by security fixes and to assess the impact of the vulnerability for a given application using dynamic analysis. In this paper we build on that work, proposing a more comprehensive code-centric and usage-based approach to detect, analyze, and mitigate OSS vulnerabilities: A) We generalize the vulnerability detection approach of [11] by considering fixes independently of the vulnerable libraries; B) We use static analysis to determine whether vulnerable code is reachable and through which call paths; C) We combine static and dynamic analysis to overcome their mutual limitations; D) We define metrics which support the choice of alternative library versions that are not vulnerable, highlighting which options are more likely to minimize the update effort and the risk of incompatibility.

Our approach is implemented as a tool, Vulas, which is adopted at SAP as the officially recommended solution to scan Java software. The tool has been successfully used to scan about 500 applications. Vulnerable code was found reachable for 131 of them and we found that in 7.9% of the cases this was only determined by the combination of static and dynamic analysis. We report on our experience and on the lessons we learned when maturing Vulas from a research prototype to an industrial-grade solution that has been used to perform over 250000 scans since December 2016.

The remainder of the paper is organized as follows: Sec. II describes the technical approach, Sec. III defines the update metrics, and Sec. IV illustrates our approach in practice. In Sec. V, we report on our experience, lessons learned and the challenges we identified. Sec. VI discusses related literature and Sec. VII concludes the paper.

## 2 Technical Description of the Approach

In [11], we introduced the idea of shifting the problem of establishing whether an application is exploitable because of a known vulnerability in an OSS library, to the problem of assessing whether the vulnerable code is reachable.

Sections II-A, II-B, and II-C generalize [11], whereas Sections II-D,II-E extend it with unique novel contributions, that are the basis of the update metrics presented in Section III.

### 2.1 Background

The core of [11] lies on the assumption that a vulnerability can be detected and analyzed considering the set of program constructs (such as methods), that were modified, added, or deleted to fix that vulnerability.

We define a program construct (or simply construct) as a structural element of the source code characterized by a type (e.g., package, class, constructor, method), a language (e.g., Java, Python), and a unique identifier (e.g., the fully-qualified name).

Changes to program constructs are performed through commits in a source code repository; therefore, the set of changes that fix a vulnerability can be obtained from the analysis of the corresponding fix commit. We define a construct change as the tuple (c, t, AST(c)_f, AST(c)_v) where c is a construct, t is a change operation (i.e., addition, deletion or modification) on the construct c, and AST(c)_f, AST(c)_v are, respectively, the abstract syntax trees of c at commit n and at commit n − 1, i.e., the AST of the fixed and vulnerable construct. Notice that for deleted (added) constructs only AST(c)_v (AST(c)_f) exists.

When a fix is implemented over multiple commits, we rely on commit timestamps to compute the set of changes by comparing the source code of the first and the last commit. If the vulnerability fix includes changes in a nested construct (e.g., a method of a class), two distinct entries are included in the set of construct changes, one for the outer construct (the class), one for the nested construct (the method).

The fix commits of a vulnerability are not always included in its disclosure. Some OSS projects (e.g., Apache Tomcat) provide such information via security advisories; others reference issue tracking systems which in turn describe the vulnerability being solved; some other OSS projects do not explicitly refer to vulnerabilities being fixed. Thus reconciling the information based on the textual description and code changes requires considerable manual effort (see Sec. V-B3). A broader discussion of the data integration problem can be found in [11].

Differently from [11], we provide a definition of construct change and consider the ASTs of the modified program constructs. This is used in Sec. II-B to establish whether libraries include the changes introduced by the fix.

### 2.2 Vulnerability Detection

Figure 1 illustrates how a vulnerability j is associated to an application a. Cj is the set of the constructs obtained as described above by analyzing the fix commits of j. The set Si is the set of all constructs of the OSS library i bundled in the application a whereas Sa is the set of all constructs belonging to the application itself.

If Cj ∩ Si ≠ ∅ and ∀c ∈ Cj ∩ Si, AST(c) = AST(c)_v, then we conclude that the application includes a library i with code vulnerable to j (referred to as vulnerable constructs), i.e., constructs that have been changed in the fix commits of j.

If ∃ c ∈ Cj ∩Si | AST(c) ≠ AST(c)_v, then we relax the equality constraint and use both AST(c)_v and AST(c)_f to establish to which of the two AST(c) is "closest". To this end, we use tree differencing algorithms [12], [13] and a library comparison method that we devised (whose description is omitted from this paper because of space constraints). Manual inspection is still required whenever no automated conclusion can be taken, e.g., when ∃ c1, c2 ∈ Cj ∩ Si | AST(c1) = AST(c1)_v and AST(c2) = AST(c2)_f.

Note that, even when a vulnerability is fixed by adding new methods to an existing class, the intersection Cj ∩ Si is not empty because, as explained in the previous section, it would contain the construct for the class. If the fix includes the addition of a class, we assume that existing code is modified to invoke the new construct.

Differently from [11], we define the set of constructs Cj as independent of any library i, and a vulnerability in a library is then detected through the intersection of its constructs with Cj. This approach has several advantages: First, it makes it explicit that the vulnerable constructs responsible for a vulnerability j can be contained in any library i, hence, the approach is robust against the prominent practice of repackaging the code of OSS libraries. Second, it is sufficient that a library includes a subset of the vulnerable constructs for the vulnerability to be detected. Last, it improves the accuracy compared to approaches based on metadata, which typically flag entire open-source projects as affected, even if projects release functionalities as part of different libraries. Apache POI [14], for instance, while developed in a single source code repository, is released as a set of distinct, independent libraries. Because [11] focuses on newly-disclosed, it assumes that, at the time of disclosure, every library that includes constructs changed in the fix commit must be vulnerable. While this assumption is valid at that moment in time, it is not valid for old vulnerabilities, which require that one establishes whether a given library contains the fixed code. We achieve this by comparing the AST of constructs in use with those of the affected and fixed versions.

### 2.3 Dynamic Assessment of Vulnerable Code

After having determined that an application depends on a library that includes vulnerable constructs, it is important to establish whether these constructs are reachable. In this paper, we use the term reachable to denote both the case where dynamic analysis shows that a construct is actually executed and the case where static analysis shows potential execution paths. The underlying idea is that if an application executes (or may execute) vulnerable constructs, there exists a significant risk that the vulnerability can be exploited. The dynamic assessment described here is borrowed from [11].

Figure 2(a) illustrates the use of dynamic analysis to assess whether the vulnerable constructs are reachable by observing actual executions. Tai represents the set of constructs, either part of application a or its bundled library i, that were executed at least once during some execution of the application. The collection of actual executions of constructs can be done at different times: during unit tests, integration tests, and even during live system operation (if possible). The intersection Cj ∩ Tai comprises all those constructs that are both changed in the fix commits of j and executed in the context of application a because of its use of library i.

### 2.4 Static Assessment of Vulnerable Code

In addition to the analysis of actual executions (dynamic analysis), our approach uses static analysis to determine whether the vulnerable constructs are potentially executable. Specifically, we use static analysis in two different flavors. First, we use it to complement the results of the dynamic analysis, by identifying the library constructs reachable from the application. Second, (Sec. II-E) we combine the two techniques by using the results of the dynamic analysis as input for the static analysis, thereby overcoming limitations of both techniques: static analyzers are known to struggle with dynamic code (such as, in Java, code loaded through reflection [15]); on the other hand, dynamic (test-based) methods suffer from inadequate coverage of the possible execution paths.

Figure 2(b) illustrates how we use static analysis to complement the results of dynamic analysis. Rai represents the set of all constructs, either part of application a or its bundled library i, that are found reachable starting from the application a and thus can be potentially executed. Static analysis is performed by using a static analyzer (e.g., [16]) to compute a graph of all library constructs reachable from the application constructs. The intersection Cj ∩ Rai comprises all constructs that are both changed in the fix commit of j and can be potentially executed.

### 2.5 Combination of Dynamic and Static Assessment

Figure 2(c) illustrates how we combine static and dynamic analysis. In this case the set of constructs actually executed, Tai, is used as starting point for the static analysis. The result is the set RTai of constructs reachable starting from the ones executed during the dynamic analysis. The intersection Cj ∩ RTai comprises all constructs that are both changed in the fix commit of j and can be potentially executed.

We explain the benefits of the combinations of the two techniques through the example in Figure 3.

In the following, we denote a library bundled within a software program with the term dependency. Let Sa be a Java application having two direct dependencies S1 and Sf where S1 has a direct dependency S2 that in turn has a direct dependency S3 (thereby S2 and S3 are transitive dependencies for the application Sa). S1 is a library offering a set of functionalities to be used by the application (e.g., Apache Commons FileUpload [17]). Moreover the construct γ of S1 calls the construct δ of S2 dynamically, e..g, by using Java reflection, which means the construct to be called is not known at compile time. Sf is what we call a "framework" providing a skeleton whose functionalities are meant to call the application defining the specific operations (e.g., Apache Struts [18], Spring Framework [19]). The key difference is the so-called inversion of control as frameworks call the application instead of the other way round.

With the vulnerability detection step of Sec. II-B, our approach determines that Sa includes vulnerable constructs for vulnerabilities j1 and j2 via the dependencies Sf and S3, respectively. Note that even if S3 only contains two out of the three constructs of Cj2, our approach is still able to detect the vulnerability.

We start the vulnerability analysis by running the static analysis of Sec. II-D that looks for all constructs potentially reachable from the constructs of Sa. The result is the set Ra1 including all constructs of Sa and all constructs of S1 reachable from Sa. As expected, Sf is not reachable in this case as frameworks are not called by the application. Moreover, it is well known that static analysis cannot always identify dynamic calls like those performed using Java reflection. As the call from γ to δ uses Java reflection, in this example only S1 is statically reachable from the application. As shown in Figure 3 Ra1 does not intersect with any of the vulnerable constructs.

The dynamic analysis of Figure 2(a) produces the set Ta (omitted from the figure) of constructs that are actually executed. Though no intersection with the vulnerable constructs is found, the dynamic analysis increases the set of reachable constructs (Ra1 ∪ Ta in Figure 3). In particular it complements static analysis revealing paths that static analysis missed. First, it contains construct of framework Sf that calls construct α of the application. Second, it follows the dynamic call from γ to δ.

Combining static and dynamic analysis, as shown in Figure 2(c), we can use static analysis with the constructs in Ta as starting point. The result is the set RTa (omitted in the figure) of all constructs that can be potentially executed starting from those actually executed Ta.

After running all the analyses, we obtain the overall set Ra1 ∪ Ta ∪ RTa (shown with solid fill in Figure 3) of all constructs found reachable by at least one technique. Its intersection with Cj1 and Cj2 reveals that both vulnerabilities j1 and j2 are reachable, since one vulnerable construct for each of them is found in the intersection and is thus reachable (η ∈ Cj1 is reachable from and ω ∈ Cj2 is reachable from δ).

## 3 Vulnerability Mitigation

The analysis presented in Sections II-C to II-E provides in-depth information about the control-flow between the code of the application and its dependencies. In the following, this information is leveraged to support application developers in mitigating vulnerable dependencies.

As long as non-vulnerable library versions are available, updating to one of those is the preferred solution to fix vulnerable application dependencies. And since the approach described in Sec. II-A depends on the presence of at least one fix commit, a non-vulnerable library version becomes available when the respective open-source project releases a version including this commit. However, it is well known that developers are reluctant to update dependencies because of the risk of breaking changes, the difficulties in understanding the implications of changes, and the overall migration effort [20], [21]. Such risk and effort depend on the usage the application makes of the library, and on the amount of changes between the library version currently in use and the respective non-vulnerable version. As a result of the analysis described in Sections II-C to II-E, the reachable share of each library is known. Whether a construct with a given identifier is also available in other versions of a library can be determined, for instance, by comparing compiled code with tools such as Dependency Finder [22]. Among all the reachable constructs, of particular importance in the scope of mitigation are those which are called directly from the application, as they provide a measure of how many times the application developer explicitly used the library.

We define a touch point as a pair of constructs (c1, c2) such that c1 ∈ Sa is an application construct, c2 ∈ Si is a library construct, and there exists a call from c1 to c2. We define callee the library construct called directly from the application, i.e., c2. In the example of Figure 3 there are two touch points: (α, β) and (λ, ψ), with β and ψ being the callees. Given a library in use Si and its candidate replacement Sj, we define the following update metrics.

**Callee Stability (CS).** Let c(Si)_k with k = 1, . . . , n be the callees of Si, and c(Sj)_k = 1 if c(Si)_k ∈ Sj, 0 otherwise. Then the callee stability is the number of callees of Si that exist in Sj over the number of callees of Si:

CS = Σ(k=1 to n) c(Sj)_k / |{c(Si)_1, . . . , c(Si)_n}| = Σ(k=1 to n) c(Sj)_k / n

If Sj contains all the callees of Si, then the callee stability is 1, to indicate that the constructs of Si called by the application exist also in library Sj. In case Sj does not contain all the callees of Si, then the callee stability is smaller than 1 and reaches 0 when none of the callees of Si is present in Sj.

**Development Effort (DE).** Let a(Si)_k with k = 1, . . . , n be the calls from the application to the callees of Si, and a(Sj)_k = 1 if a(Si)_k ∉ Sj, 0 otherwise. The development effort for updating from library Si in use by the application to library Sj is defined as the number of application calls that require a modification due to callees of Si that do not exist in Sj.

DE = Σ(k=1 to n) a(Sj)_k

Compared to the callee stability, the development effort keeps into account the fact that each callee can be called multiple times within an application.

In Figure 3, for instance, each callee is called only once by α and λ respectively. However, assuming that β is called by two application constructs in addition to α, and that it is not contained in the new library Sj, CS = 1/2 whereas the DE = 3. This reflects the fact that multiple calls need to be modified as a result of a change in a single callee. The notion of development effort we use does not take into account the complexity of each modification; rather, it focuses on the number of modifications required by the application, considering that each modification comes at the cost not only of updating the code (which could be automated to some extent) but also of testing it.

**Reachable Body Stability (RBS).** The reachable body stability is calculated in the same way as the callee stability, but instead of callees, it considers the reachable share of a library, i.e., the set of dynamically and/or statically reachable library constructs. Given the total number of constructs of Si reachable from the application, it measures the ratio of those that are contained as-is, i.e., with identical identifier and byte code, also in Sj. By quantifying the share of modified reachable constructs, this metric provides the likelihood that the behavior of the library changes from Si to Sj. In case all reachable constructs of Si exist in Sj, then RBS = 1 and thus there is a higher likelihood that the library change does not break the application.

**Overall Body Stability (OBS).** The overall body stability is calculated similarly to RBS but now considers all the constructs of Si. This metric provides the same indication as the one above but, by considering the entire library rather than only its reachable share, it is independent of the application-specific usage.

The above metrics support the application developer in estimating the effort and risks of updating a library. When several non-vulnerable library versions exist that are newer than the one in use, they are all candidate replacements. By quantifying the changes to be performed on the application and the changes that the library underwent, our update metrics allow the developer to take an informed decision.

Note that the callee stability and development effort metrics only apply for dependencies that are called directly from the application, whereas the reachable and overall body stability also apply for transitive dependencies and frameworks.

## 4 Evaluation

The implementation of our approach for Java, has been successfully used at SAP to perform over 250000 scans of about 500 applications since December 2016. In the following, we illustrate how our approach works in a typical scan, applying it to a SAP-internal web application that we adapted, for illustrative purposes, to include vulnerable OSS. The application allows users to upload files, such as documents or compressed archives, through an HTML form, inspects the file content and displays a summary to the user. It is developed using Maven [23], and depends on popular open-source libraries from the Apache Software Foundation, such as Struts 2.3.24 (released on 3 May 2015), Commons FileUpload 1.3.1 (6 February 2014), POI 3.14 (6 March 2016) and HttpClient 4.5.2 (21 February 2016). Overall, the application has 12 direct and 15 transitive compile-time dependencies.

The analysis is performed using an implementation of the approach described previously: Sections II-B, II-D, and II-E are implemented as goals of a Maven plugin; the collection of traces during the dynamic analysis (Sec. II-C) is performed by instrumenting all classes of both the application and all its dependencies as described in [11]. This happens either at runtime, when classes are loaded, or by modifying the byte code of the application before deploying it in a runtime environment such as Apache Tomcat.

### 4.1 Detection and Analysis

To illustrate the benefits of our approach, we go through the analysis steps and highlight selected findings. To demonstrate the added value of static analysis compared to [11], we perform it after dynamic analysis. However, our implementation also supports changing their order (or executing only a subset of the steps).

**(1) Vulnerability Detection.** The first step is to create a bill of materials (BOM), consisting of the constructs of the application and all its dependencies, as explained in Sec. II-B.

Vulnerabilities in a library are detected by intersecting the set of constructs found in (the BOM of) that library with the vulnerable constructs of all the vulnerabilities known to our knowledge base. As an example, the bottom part of Figure 6 shows a table listing the vulnerable constructs for CVE-2017-5638 (columns Type and Qualified Construct Name) together with the respective change operation (column Change), as well as the information that those constructs are actually present in the Java archive corresponding to struts-core:2.3.24 (column Contained).

The vulnerability detection step reveals that our sample application includes vulnerable code related to 25 different vulnerabilities, affecting nine different compile-time dependencies: seven are direct, while the remaining two (ognl:3.0.6 and xwork-core:2.3.24, pulled in through struts2-core:2.3.24) are transitive.

**(2) Dynamic Assessment (Unit tests).** The execution of unit tests reveals that vulnerable constructs related to three vulnerabilities are executed, e.g., the method URIBuilder.normalizePath(String), which is part of httpclient:4.5.2 and subject to vulnerability HTTPCLIENT-1803 [24]. Another example is shown in Figure 4: method SharedStringsTable.readFrom(InputStream), which is part of poi-ooxml:3.14 and subject to CVE-2017-5644, is invoked in the context of unit test openSpreadsheetTest. The fact that reflection is heavily used inside the poi-ooxml method XSSFFactory.createDocumentPart(Class,Class[],Object[]) (as visible from a sequence of four invocations of newInstance, see figure) makes it difficult for static analysis to determine the reachability of the vulnerable method.

**(3) Dynamic Assessment (Integration tests).** The execution of integration tests is done using an instrumented version of the application deployed in a runtime container. They reveal the execution of vulnerable code related to eight additional vulnerabilities, all affecting struts2-core, or its dependencies ognl and xwork-core. As an example, the last line of the table in Figure 6 shows a vulnerable construct of CVE-2017-5638, FileUploadInterceptor.intercept(ActionInvocation), whose actual execution is traced (column Traced) at the reported time. This method is included in struts2-core 2.3.24 which is part of the Struts2 framework and exemplifies the inversion of control (IoC) happening when frameworks invoke application code.

**(4) Static Assessment.** The static analysis starting from application constructs reveals that the constructor MultipartStream(InputStream,byte[],int,ProgressNotifier), part of commons-fileupload:1.3.1 and subject to CVE-2016-3092, is reachable from the application. Dynamic analysis was not able to trace its execution due to the limited test coverage.

On the other hand, static analysis starting from the application constructs falls short in the presence of IoC. As application methods are called by the framework, there is no path on the call graph starting from application and reaching framework constructs that are involved in the IoC mechanism.

**(5) Combination of Static and Dynamic Assessment.** The static analysis starting from constructs traced with dynamic analysis provides additional evidence regarding the relevance of CVE-2017-5638 (the vulnerability that was exploited in the Equifax breach [8]). In addition to the execution of method FileUploadInterceptor.intercept(ActionInvocation) during step 3, the combination of static and dynamic analysis reveals that method MultiPartRequestWrapper.buildErrorMessage(Throwable, Object[]), included in struts2-core 2.3.24, is reachable with two calls from the traced method Dispatcher.wrapRequest(HttpServletRequest), as shown in Figure 5. Its reachability is indicated with the red paw icons in the table containing the construct changes for CVE-2017-5638 (cf. the two right-most columns of the table in Figure 6).

The value of combining the two analysis techniques becomes more evident when considering all applications scanned with our approach: vulnerable constructs are reachable, statically or dynamically, in 131 out of 496 applications. In particular we observed 390 pairs of applications and vulnerabilities whose constructs were reachable. In 32 cases, the reachability could only be determined through the combination of techniques, which represents a 7.9% increase of evidence that vulnerable code is potentially executable.

### 4.2 Mitigation

During the execution of dynamic analysis (steps 2 and 3) and static analysis (steps 4 and 5), touch points and reachable constructs are collected. They are the basis for the computation of the update metrics for the application at hand.

Figure 7 shows that for httpclient:4.5.2, one of the direct dependencies of the application, there are nine touch points between the application and the library. The application method ArchivePrinter.httpRequest2(String), for instance, calls the constructor HttpGet(String) (cf. first table in the figure). This invocation was observed during dynamic analysis, and was also found by static analysis (cf. rightmost columns in the first table in the figure). The second table of the figure shows the number of constructs of httpclient:4.5.2 by type. For example, of the 608 constructors (CONS), 199 were found reachable by static analysis, and 117 were actually executed during tests.

The table at the bottom of Figure 7 shows the update metrics that can guide the developer in the selection of a non-vulnerable replacement for httpclient:4.5.2. Each table row corresponds to a release of Apache HttpClient that is not subject to any vulnerability known to our knowledge base, hence, the developer is advised to choose among the three versions: 4.5.3, 4.5.4 and 4.5.5. The callees of all touch points exist in all of those versions, hence, the update to any of those would not result in signature incompatibilities (cf. columns 3 and 4). The RBS metric indicates that 872 out of 876 reachable constructs of type method and constructor are also present in release 4.5.3 (870 out of 876 in 4.5.4 and 4.5.5). The OBS metric is also relatively high for all three non-vulnerable releases, thus, the developer would likely choose httpclient:4.5.5 in order to update the vulnerable library.

While the update decision is relatively straightforward for httpclient, it is more difficult for strut2-core, since there are non-vulnerable replacements from both the 2.3 and the 2.5 branch (cf. Figure 8). Here, the RBS and OBS metrics indicate a more significant change of constructs between the current version struts-core:2.3.24 and the latest version of the 2.5 branch (RBS =862/887 and OBS =2781/3101) than between the current version and the latest version of the 2.3 branch (RBS =885/887 and OBS =3095/3101). Hence, the developer may be more inclined to stick to the 2.3 branch, thus updating to struts-core:2.3.34 rather than to struts2-core:2.5.16.

## 5 Experience Report

### 5.1 From Research Prototype to Industrial-Grade Solution

The approach presented in this paper is implemented as a SAP-internal tool called Vulas. Initially, a research prototype was used to run pilots with a small number of development units, to clarify the needs of real-life development projects and to evaluate the viability of our approach.

From the feedback gathered from the users of the prototype, we could make two clear observations:

**Precision:** Developers feared that using yet another tool (general static code analyzers as well as dynamic security testing tools were widely available to scan the code of open-source dependencies used in SAP's products) would mean being flooded with more findings referring to potential issues, many of which irrelevant (not exploitable) in practice. Especially the more security-aware among developers were reluctant to use metadata-based tools, because of their excessive rate of false-positives and false-negatives. As a matter of fact, frequent false-positives can challenge the adoption of a tool as much as false-negatives do. This observation confirmed the importance of our decision to strive for a reliable, precise method to detect the actual presence of vulnerable code in a given library.

**Inobtrusiveness and Automation:** A tool whose adoption requires changes in the development practices is extremely hard to promote. We made the choice to integrate with the de-facto standard build processes and tools, making Vulas a pluggable element of the automated build tool-chain that could be enabled with minimal configuration effort. Automating vulnerability scans has the additional benefit that issues are detected in a more timely manner, allowing better prioritization and effort allocation to identify and apply cost-effective fixes.

After the prototyping phase, as the demand for Vulas started to increase, the tool underwent a major reimplementation in order to make it more flexible and scalable, by adopting a micro-services architecture and deploying it onto SAP's internal cloud infrastructure.

With the growth of the user base, we could observe that the tool is used differently depending on the phase of the development life-cycle.

Projects in the earliest phases of development, and particularly those that have not yet been released to customers, are mainly concerned with identifying as early as possible the dependency on a vulnerable library. The large majority of the projects that adopted Vulas (roughly 90%) are scanned routinely, as part of an automated build pipeline, in which vulnerability detection (based purely on the analysis of the constructs of the application and its libraries as presented in Sec. II-B) is performed at each commit; these projects perform a deeper analysis during nightly (or weekly) jobs. The performance offered by the vulnerability detection implemented in Vulas is adequate for frequent scans, since its average execution time is about 70 seconds (static and dynamic analysis can take hours to complete, depending on the size of the application).

We observed that in this phase, the resistance to updates (especially, to minor releases) is quite low, and developers tend to update their OSS dependencies in a short time-frame.

Deeper analysis of the vulnerability becomes more and more critical the closer the project gets to the date of release to customers, and stays so throughout the operational lifetime of the application because new vulnerabilities impacting the application could be discovered at any point in time after the release. After the release date, Vulas is used mostly in manual scans, as a program comprehension tool, to achieve a deeper understanding of whether and how vulnerable code is reachable in a target application, what its concrete impact is, and what remediation options are available.

To deal also with legacy applications, built without modern tools such as Maven [23] or Gradle [25], a dedicated command-line version of Vulas is often used.

We found that both in new and legacy applications, library artifacts are very often renamed in an ad-hoc, often inconsistent manner, and the content of one or more original libraries might be extracted and repackaged as a single self-contained archive. In this context, metadata is most often incomplete or missing, which makes our code-centric approach critical. The method implemented in Vulas, whereby vulnerable code is identified in the (byte)code of libraries, does not suffer from the limitations of the approaches based on metadata (such as OWASP Dependency Check [9], which relies on Maven artifact identifiers, file names or the content of manifest files to work correctly).

At the time of writing, Vulas is the officially recommended tool at SAP to scan Java projects. It is used by over 500 distinct development projects, and its adoption has been growing at a rate of 15 to 20 new applications per week since the beginning of 2018. The sizes of the applications scanned as of today are plotted in Figure 9. During these scans, we detected about 30000 pairs of vulnerabilities and libraries. More than a half of these pairs concern libraries that are not published to Maven central (because they have been recompiled, repackaged, or otherwise manipulated).

Despite its success, several challenges, pertaining to either organizational or technical aspects, remain to be addressed. In the remainder of this section we briefly discuss the most important ones.

### 5.2 Challenges

**1) Developer Opt-in vs. Central Scans:** To foster the uptake by developers, one has to minimize the required changes of development artifacts and processes. Plug-ins for common build tools support this goal, and the provision of templates for continuous integration pipelines goes as far as enabling in-depth Vulas scans by means of boolean flags. Still, the tool adoption ultimately depends on the developer's initiative, even at that degree of integration and automation. Awareness campaigns, trainings and other organizational measures are one means to this end, however, they require significant resources in large, international and heterogeneous development organizations. Future work aims at overcoming such issues by running fully-automated scans at a few central elements of an organization's development infrastructure, e.g., its source code or artifact repositories.

**2) Decision-Support vs. Decision-Making:** Vulas is essentially a fact-finding tool, whose goal is to provide comprehensive evidence that vulnerable code is included and reachable in a given application. Clearly, it is cannot prove (decide) whether vulnerable code can or cannot be exploited. However, we occasionally observe this expectation, particularly when library updates are difficult and expensive. Again, trainings and security-awareness campaigns are essential to ensure that developers internalize that they are responsible for drawing the final conclusion in regards to the exploitability of each vulnerability, especially before sticking to a vulnerable open-source version. Also, while such conclusions and the decision to upgrade (or not) a library have to be documented, we decided to keep this functionality out of the scope of Vulas and to stick to its fact-finding mission.

**3) Vulnerability Knowledge-Base:** The creation and maintenance of a comprehensive vulnerability database is key for our approach. However, the required vulnerability information is scattered across many different sources, e.g., public vulnerability databases, issue trackers and security advisories of individual projects, or source code repositories. Moreover, as discussed in [11], [26], different sources like the National Vulnerability Database and code repositories are difficult to integrate using the existing data. Based on a study that we performed on the NVD, we estimate our knowledge base to cover about 90% of all vulnerabilities affecting Java open-source projects and that are part of the NVD. At the time of writing, experimenting with machine-learning methods to automate the identification of fix commits and to ease the maintenance of a rich knowledge base where vulnerabilities are linked to the corresponding source code changes. The maintenance of such a detailed knowledge base, which is done manually as of today, would greatly benefit from a coordinated approach to vulnerability disclosure and patch release across the open-source community, and through the governance exercised by established institutions, such as the Apache Software Foundation.

**4) Shallow vs. Deep Updates:** The metrics of Sec. III support the update to non-vulnerable library versions. In the case of transitive dependencies, however, developers are required to interfere with the transparency and automation of the dependency resolution mechanism. In fact, one would need to add the updated non-vulnerable version of the library as a direct dependency, thereby taking the risk of future incompatibilities. Therefore, the mitigation strategy could be optimized to recommend the update of the library that is "nearest" to the application. As an example, to mitigate the vulnerable library S3 of Figure 3, a newer version of S1 can be recommended if it pulls in a fixed version of S3.

**5) Problematic Types of Vulnerabilities:** By identifying a vulnerable dependency through the presence of vulnerable code, our approach is robust against false-positives and false-negatives, as typical for solutions based on the mapping of library and vulnerability metadata. As a drawback, the fraction of vulnerabilities whose fix does not involve any code change, e.g., those that are fixed by modifying a default configuration, cannot be covered by our code-centric approach described in Sec. II-A. Nevertheless, Vulas is able to cover such cases, which are relatively rare compared to code-related vulnerabilities, by flagging entire libraries as affected.

By assessing the exploitability of a vulnerability in terms of potential or actual code execution, our approach provides evidence about the application-specific usage of vulnerable code. However, this approach does not work with vulnerabilities that are due to the deserialization of untrusted data, where the mere presence of so-called deserialization gadgets in the application classpath can cause the application to be vulnerable (regardless of whether they can be reached during normal program execution). Attackers exploit the behavior of the Java serialization mechanism, which creates objects (through deserialization) as long as the definition of the respective class is known, regardless of whether the application actively uses it or not.

**6) Shortcomings of Name-based Construct Identification:** While the approach relies on the generic concept of construct identifiers, Vulas uses Java fully-qualified names in its implementation. This implementation choice supports other programming languages as long as they have a comparable naming scheme, and its development community follows consistent naming conventions (that is, construct names can be assumed to be globally unique, as in Java). However, this property is not satisfied for certain languages, hence, the construct identification cannot be done just by using their fully-qualified names, but must consider other information, such as their bodies.

## 6 Related Work

There exist several free [9] and commercial tools [10], [27], [28], [29] for detecting vulnerabilities in OSS components. In [11] we compared the vulnerability detection capabilities of our approach with state-of-the-art tools. To the best of our knowledge, the combination of static and dynamic techniques and the metric-based support to mitigation offered by our approach are unique. OWASP Dependency Check [9] is used in [30] to create a vulnerability alert service and to perform an empirical investigation about the usage of vulnerable components in proprietary software. The results showed that 54 out of 75 of the projects analyzed have at least one vulnerable library. However the results had to be manually reviewed, as the matching of vulnerabilities to libraries showed low precision. Alqahtani et al. proposed an ontology-based approach to establish a link between vulnerability databases and software repositories [26]. The mapping resulting from their approach yields a precision that is 5% lower than OWASP Dependency Check. All these approaches and tools differ from ours in that they focus on vulnerability detection based on metadata, and do not provide application-specific reachability assessment nor mitigation proposals.

In our previous work [11], we used reachability analysis (and in particular, dynamic analysis) to establish whether vulnerable code is actually executed and thus to make a first step beyond the mere detection of vulnerabilities. In this work, we extend [11] by including static analysis and presenting a novel combination of static and dynamic analysis. To the best of our knowledge none of the existing works and tools combine static and dynamic analysis, nor provide application-specific mitigation proposals.

The empirical study conducted by Kula et al. on library migrations of 4600 GitHub projects showed that 81.5% of them do not update their direct library dependencies, not even when they are affected by publicly known vulnerabilities [31]. In particular, that study highlights the lack of awareness about security vulnerabilities. Considering 147 Apache software projects, [32] studied the evolution of dependencies and found that applications tend to update their dependencies to newer releases containing substantial changes. Tools based on reward or incentives to trigger the update of out-of-date dependencies exist (e.g., [33], [34]), however they are mostly concerned about breaking changes and mechanisms are needed to provide–next to transparency–a motivation for the update and confidence measures to estimate the risk of performing the update. By automatically detecting vulnerabilities, providing evidence about the reachability of the vulnerable code, and supporting mitigation via update metrics, our work addresses the need of motivating updates and estimating effort and risk.

In [36] four metrics to measure the stability of libraries through time are proposed. In particular it considers the removal of units (constructs in our context), the amount of change in existing constructs, the ratio of change in new and old constructs, and the percentage of new constructs. Similar to our work, the metrics are meant to be representative for the amount of work required to update a certain library and thus they also consider usages of library methods in other projects. However the main focus of [36] is the library, and the metrics are used to measure its stability over time given a set of projects. Though some of their metrics ingredients can also be considered in our work, the metrics we propose are about the application-specific library usage. Moreover, our metrics benefit of our in-depth analysis of the application (e.g., some usages of the libraries that can only be observed with a combination of static and dynamic analysis).

Raemaekers et al. studied breaking changes in library releases over seven years and showed that they occur with the same frequency in major and minor releases [37]. This shows that the rules of semantic versioning, according to which breaking changes are only allowed in major releases, are not followed in practice. It also shows that top 3 most frequent breaking changes involve a deletion of methods, classes, fields, respectively. This result reinforces our belief that our update metrics based on measuring the removal of program constructs provides a critical information.

Mileva et al. studied the usage of different library versions and provided a tool to suggest which one to use based on the choice of the majority of similar users [38]. Our work differs from theirs, in that we provide quantitative measures to support the user in selecting a non-vulnerable library.

Existing works on library migration [39], [40], [41] are complementary to our approach in that they support developers in evolving their code to adapt to new libraries or library versions. [39] proposes a tool that keeps track of API popularity and migration of major frameworks/libraries, amounting to 650 Github projects resulting in 320000 APIs at the time of publication.

## 7 Conclusion

The unique contribution of this paper is the use of static analysis and its combination with dynamic analysis to support the application-specific assessment and mitigation of open-source vulnerabilities. This approach further advances the code-centric detection and dynamic analysis of vulnerable dependencies we originally proposed in [11]. The accuracy and application-specific nature of our method improves over state-of-the-art approaches, which commonly depend on metadata. Vulas, the implementation of our approach for Java, was chosen by SAP among several candidates as the recommended OSS vulnerability scanner. Since December 2016, it has been used for over 250000 scans of about 500 applications, which demonstrates the viability and scalability of the approach.

The variety of programming languages used in today's software systems pushes us to extend Vulas to support languages other than Java. However, fully-qualified names can be inadequate to uniquely identify the relevant program constructs in certain languages, so we are considering the use of information extracted from the construct bodies.

Finally, the problem of systematically linking open-source vulnerability information to the corresponding source code changes (the fix) remains open. Maintaining a comprehensive knowledge base of rich, detailed vulnerability data is critical to all vulnerability management approaches and requires considerable effort. While this effort could be substantially reduced creating specialized tools, we strongly believe that the maintenance of this knowledge base should become an industry-wide, coordinated effort, whose outcome would benefit the whole software industry.

## Acknowledgment

We are grateful to the anonymous reviewers who contributed to improve significantly this paper, and to our colleagues Michele Bezzi, Luca Compagna, Cédric Dangremont, and Brian Duffy for their insightful comments on early drafts of this work.

## References

[1] "The state of open source security," Snyk, 2017, accessed: 2018-03-30. [Online]. Available: https://snyk.io/stateofossecurity/pdf/The%20State%20of%20Open%20Source.pdf

[2] M. Pittenger, "Open source security analysis: The state of open source security in commercial applications," Black Duck Software, Tech. Rep., 2016. [Online]. Available: https://info.blackducksoftware.com/rs/872-OLS-526/images/OSSAReportFINAL.pdf

[3] OWASP Foundation, "OWASP Top 10 - 2013," [Online]. Available: https://www.owasp.org/index.php/Top_10_2013-Top_10

[4] ——, "OWASP Top 10 - 2017," [Online]. Available: https://www.owasp.org/index.php/Top_10_2017-Top_10

[5] http://heartbleed.com/, accessed: 2018-03-30.

[6] N. Perlroth, "Security experts expect "Shellshock" software bug in Bash to be significant," The New York Times, [Online]. Available: https://nyti.ms/1n26UDQ

[7] "Which of the owasp top 10 caused the world's biggest data breaches?," Snyk, 2017, accessed: 2018-03-30. [Online]. Available: https://snyk.io/blog/owasp-top-10-breaches/

[8] "Equifax releases details on cybersecurity incident, announces personnel changes," accessed: 2018-03-30. [Online]. Available: https://investor.equifax.com/news-and-events/news/2017/09-15-2017-224018832

[9] "OWASP Dependency Check," accessed: 2018-03-30. [Online]. Available: https://www.owasp.org/index.php/OWASP_Dependency_Check

[10] https://www.whitesourcesoftware.com/oss_security_vulnerabilities/, accessed: 2018-03-30.

[11] H. Plate, S. E. Ponta, and A. Sabetta, "Impact assessment for vulnerabilities in open-source software libraries," in Proc. of IEEE Int. Conf. on Software Maintenance and Evolution, ICSME, Bremen, Germany, 2015, pp. 411–420.

[12] J. Falleri, F. Morandat, X. Blanc, M. Martinez, and M. Monperrus, "Fine-grained and accurate source code differencing," in Proc. of ACM/IEEE Int. Conf. on Automated Software Engineering, ASE, Vasteras, Sweden, 2014, pp. 313–324.

[13] B. Fluri, M. Wuersch, M. PInzger, and H. Gall, "Change distilling:tree differencing for fine-grained source code change extraction," IEEE Trans. on Software Engineering, vol. 33, no. 11, pp. 725–743, Nov 2007.

[14] https://poi.apache.org/, accessed: 2018-03-30.

[15] D. Landman, A. Serebrenik, and J. J. Vinju, "Challenges for static analysis of Java reflection: literature review and empirical study," in Proc. of the Int. Conf. on Software Engineering, ICSE, Buenos Aires, Argentina, 2017, pp. 507–518.

[16] "T. J. Watson libraries for analysis (Wala)," accessed: 2018-03-30. [Online]. Available: http://wala.sourceforge.net/wiki/index.php/Main_Page

[17] https://commons.apache.org/proper/commons-fileupload/, accessed: 2018-03-30.

[18] https://struts.apache.org/, accessed: 2018-03-30.

[19] https://projects.spring.io/spring-framework/, accessed: 2018-03-30.

[20] R. G. Kula, D. M. Germán, A. Ouni, T. Ishio, and K. Inoue, "Do developers update their library dependencies? - an empirical study on the impact of security advisories on library migration," Empirical Software Engineering, vol. 23, no. 1, pp. 384–417, 2018.

[21] S. Mostafa, R. Rodriguez, and X. Wang, "Experience paper: a study on behavioral backward incompatibilities of Java software libraries," in Proc. of the 26th ACM SIGSOFT Int. Symposium on Software Testing and Analysis, Santa Barbara, CA, USA, 2017, pp. 215–225.

[22] "The Dependency Finder toolkit," accessed: 2018-03-30. [Online]. Available: https://github.com/jeantessier/dependency-finder

[23] https://maven.apache.org/, accessed: 2018-03-30.

[24] https://issues.apache.org/jira/browse/HTTPCLIENT-1803, accessed: 2018-03-30.

[25] https://gradle.org/, accessed: 2018-03-30.

[26] S. S. Alqahtani, E. E. Eghan, and J. Rilling, "Tracing known security vulnerabilities in software repositories–a semantic web enabled modeling approach," Science of Computer Programming, vol. 121, pp. 153–175, 2016.

[27] https://snyk.io/, accessed: 2018-03-30.

[28] https://www.blackducksoftware.com/technology/vulnerability-reporting, accessed: 2018-03-30.

[29] "The busy managers' guide to open source security," SourceClear, 2017. [Online]. Available: https://www.sourceclear.com/resources/TheBusyManagersGuideToOpenSourceSecurity.pdf

[30] M. Cadariu, E. Bouwers, J. Visser, and A. van Deursen, "Tracking known security vulnerabilities in proprietary software systems," in Proc. of 22nd IEEE Int. Conf. on Software Analysis, Evolution and Reengineering (SANER), 2015, pp. 516–519.

[31] R. G. Kula, D. M. German, A. Ouni, T. Ishio, and K. Inoue, "Do developers update their library dependencies?" Empirical Software Engineering, May 2017.

[32] G. Bavota, G. Canfora, M. Di Penta, R. Oliveto, and S. Panichella, "How the Apache community upgrades dependencies: an evolutionary study," Empirical Software Engineering, vol. 20, no. 5, pp. 1275–1317, Oct 2015.

[33] https://david-dm.org/, accessed: 2018-03-30.

[34] https://greenkeeper.io/, accessed: 2018-03-30.

[35] S. Mirhosseini and C. Parnin, "Can automated pull requests encourage software developers to upgrade out-of-date dependencies?" in Proc. of the 32Nd IEEE/ACM Int. Conf. on Automated Software Engineering, ser. ASE 2017, pp. 84–94.

[36] S. Raemaekers, A. van Deursen, and J. Visser, "Measuring software library stability through historical version analysis," in Proc. of 28th IEEE Int. Conf. on Software Maintenance (ICSM), Sept 2012, pp. 378–387.

[37] ——, "Semantic versioning versus breaking changes: A study of the Maven repository," in Proc. of IEEE 14th Int. Working Conference on Source Code Analysis and Manipulation, Sept 2014, pp. 215–224.

[38] Y. M. Mileva, V. Dallmeier, M. Burger, and A. Zeller, "Mining trends of library usage," in Proc. of the Joint Int. and Annual ERCIM Workshops on Principles of Software Evolution (IWPSE) and Software Evolution (Evol) Workshops, ser. IWPSE-Evol, 2009, pp. 57–62.

[39] A. Hora and M. T. Valente, "Apiwave: Keeping track of API popularity and migration," in Proc. of 2015 IEEE International Conference on Software Maintenance and Evolution (ICSME), Sept 2015, pp. 321–323.

[40] B. Dagenais and M. P. Robillard, "SemDiff: Analysis and recommendation support for API evolution," in Proc. of the 31st Int. Conf. on Software Engineering, ser. ICSE, 2009, pp. 599–602.

[41] H. A. Nguyen, T. T. Nguyen, G. Wilson, Jr., A. T. Nguyen, M. Kim, and T. N. Nguyen, "A graph-based approach to API usage adaptation," in Proc. of the ACM Int. Conf. on Object Oriented Programming Systems Languages and Applications, ser. OOPSLA, 2010, pp. 302–321.
