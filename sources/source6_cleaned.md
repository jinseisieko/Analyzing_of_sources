# An Empirical Study on Software Bill of Materials: Where We Stand and the Road Ahead

**Authors:** Boming Xia, Tingting Bi, Zhenchang Xing, Qinghua Lu, Liming Zhu

**Affiliations:**
- CSIRO's Data61, Sydney, Australia
- University of New South Wales, Sydney, Australia
- Monash University, Melbourne, Australia
- Australian National University, Canberra, Australia

## Abstract

The rapid growth of software supply chain attacks has attracted considerable attention to software bill of materials (SBOM). SBOMs are a crucial building block to ensure the transparency of software supply chains that helps improve software supply chain security. Although there are significant efforts from academia and industry to facilitate SBOM development, it is still unclear how practitioners perceive SBOMs and what are the challenges of adopting SBOMs in practice. Furthermore, existing SBOM-related studies tend to be ad-hoc and lack software engineering focuses. To bridge this gap, we conducted the first empirical study to interview and survey SBOM practitioners. We applied a mixed qualitative and quantitative method for gathering data from 17 interviewees and 65 survey respondents from 15 countries across five continents to understand how practitioners perceive the SBOM field. We summarized 26 statements and grouped them into three topics on SBOM's states of practice. Based on the study results, we derived a goal model and highlighted future directions where practitioners can put in their effort.

**Index Terms:** software bill of materials, SBOM, bill of materials, responsible AI, empirical study

---

## 1 Introduction

Modern software products are assembled through intricate and dynamic supply chains [1], while recent attacks against software supply chains (SSC) have increased significantly (e.g., SolarWinds attack [2]). According to Sonatype's report [3], there was a 650% year-over-year increase in SSC attacks in 2021, and the number was 430% in 2020. SSC attacks mainly aim at the upstream open source software/components (OSS) [4], yet OSS is heavily relied upon in software development [5]. The reliance on OSS leads to additional risks, such as the lack of reliable maintenance and support compared to proprietary software/components [6]. The security risks of software and its supply chain call for improved visibility into the SSC, with which timely and accurate identification of the impacted software/components could be carried out in case of a vulnerability or an SSC attack.

A software bill of materials (SBOM) is a formal machine-readable inventory of the components (and their dependency relationships) used for producing a software product [7]. SBOMs enhance the security of both the proprietary and open source components in SSCs [8] through improved transparency. According to Linux Foundation's SBOM and Cybersecurity Readiness report (SBOM readiness report for short) [5], SBOMs are critical for enhancing SSC security. 90% of the surveyed organizations have started or are planning their SBOM journey, with 54% already addressing SBOMs. The report also estimated 66% and 13% growth in SBOM production or consumption in 2022 and 2023, respectively. Nonetheless, some organizations are still concerned about how SBOM adoption and application will evolve (e.g., 40% are uncertain about industrial SBOM commitment and 39% seek consensus on SBOM data fields).

Despite SBOMs' essentiality for software and SSC security, there remain questions to answer and problems to solve. Motivated by the value of SBOMs and the existing gaps, with the overarching goal of investigating the SBOM status quo from practitioners' perspectives, this paper aims to answer the following research questions (RQs).

**RQ1: What is the current state of SBOM practice?**

Despite the benefits of SBOMs and the SBOM readiness report showing an overall 90% of SBOM readiness, how practitioners perceive SBOMs and how SBOMs are being addressed in practice needs further investigation. To answer this question, we analyzed the SBOM practice status from SBOM generation, distribution and sharing, validation and verification, and vulnerability and exploitability management. We summarized the current SBOM practices and what practitioners expect.

**RQ2: What is the current state of SBOM tooling support?**

Despite the proliferation of the SBOM tooling market, this RQ focuses on the SBOM tooling status from the practitioners' perspective. We investigated the practitioners' attitudes towards existing tools from the following aspects: the necessity/availability/usability/integrity of SBOM tools. While exploring the current SBOM tooling state, we also looked into practitioners' expectations of SBOM tools.

**RQ3: What are the main concerns for SBOM?**

This RQ investigates the most outstanding concerns SBOM practitioners have. Although the prospect of SBOMs is promising, there are still challenges to resolve. With this RQ, we aim to provide a reference for the most imminent issues for future research and development on SBOMs.

Our research aims to unveil the state of the SBOM field, investigating what practitioners have and how they are addressing SBOMs, versus what they expect. Our work on SBOMs has four main distinctions and contributions compared to existing work represented by the SBOM readiness report:

1. **Timeliness:** The SBOM readiness report was published in January 2022, with the survey launched in June 2021. Google Trends shows a significant increase in SBOM interest since July 2021. This study provides a more updated view of SBOMs.

2. **Software engineering (SE) angles:** The SBOM readiness report is industry-oriented and security-focused, whereas this research broadens and deepens the report and supplements SE angles, evidenced by considerations such as SBOM generation/update throughout software development lifecycle (Finding 5) and AIBOM (Section IV-A8).

3. **Different objectives:** The SBOM readiness report aims to investigate whether and to what extent organizations are prepared for SBOM production and consumption (i.e., readiness). In contrast, this study focuses on current SBOM practices and expectations from practitioners' perspectives. We provide a set of implications, including a goal model, for future endeavors towards further operationalizing SBOMs.

4. **Systematic methodology:** We conducted the first empirical study on the SBOM status from practitioners' perspectives, using a mixed methodology. Instead of using predefined questions as in SBOM readiness report, we qualitatively and quantitatively coded in-depth opinions from 17 interviewees into 26 representative statements, which were validated in a survey with 65 valid respondents from 15 countries.

The remainder of this paper is organized as follows. Section II describes the context of the SBOM field. Section III presents the methodology of our study. In section IV we present the study results. Section V discusses the implications of this study. Section VI discusses related work, and section VII draws conclusions and outlines avenues for future work.

## 2 Background: What is SBOM?

A bill of materials (BOM) was initially used in the manufacturing industry as an inventory list of all the sub-assemblies and components in a parent assembly [9]. Sharing the same origin, an SBOM as the building block to enhanced software supply chain security is a software "BOM".

There are three main SBOM standard formats: a) Software Package Data eXchange (SPDX), b) CycloneDX, and c) Software Identification (SWID) Tagging, while the first two are most adopted [11]. SPDX is an open-source international standard hosted by the Linux Foundation, emphasizing licence compliance. CycloneDX was designed by OWASP in 2017 whose primary focus is security. SWID Tagging is also an international standard maintained by the US National Institute of Standards and Technology, focusing on providing a transparent software/components identification mechanism.

The above three formats all have the corresponding tooling to help operationalize the formats into practice that are listed on their respective websites. It is worth mentioning that the SBOM formats and tooling working group under the US National Telecommunications and Information Administration (NTIA) had an effort to summarize all tools supporting different standards (i.e., SPDX list, CycloneDX list, SWID list).

In terms of government-side SBOM efforts, with incidents such as the SolarWinds attack ringing the alarm of SSC security, the US government issued an executive order on enhancing cybersecurity [12] in May 2021, explicitly mandating all companies trading with the US government to provide SBOMs. NTIA has published a series of documents and guidelines (e.g., SBOM minimum elements [7]) to facilitate SBOM development. The Cybersecurity and Infrastructure Security Agency (CISA) is also actively working on SBOM facilitation by regularly hosting listening sessions with the SBOM industrial community.

Notably, the Linux Foundation published the SBOM readiness report [5] in January 2022, which surveyed 412 organizations across the globe. According to the report, 98% of the surveyed organizations are concerned about software security, over 80% are aware of the US executive order, and 90% have started their SBOM journey. Although the evolvement of SBOM adoption and application remains a concern, the report predicts that the SBOM tool market is expected to explode in 2022 and 2023. As of January 2022, there were about 20 SBOM tool vendors in the market, and some were from adjacent markets like Software Composition Analysis (SCA) which dates back to 2002 [13]. Meanwhile, various open source tools are available with a focus on SBOM generation.

However, while the SBOM readiness report comprehensively covers a broad category, their reports were based on multi-choice questions with limited choices, which constrained the possible answers. The guidelines and documents published by NTIA provide a reference for understanding SBOM and SBOM practice but lack the actual perception from the SBOM practitioners. To fill the gaps, we interviewed 17 SBOM practitioners with open-ended questions on how they perceive current SBOM practice and SBOM tooling. We then organized an online survey containing statements from the interviews, allowing more practitioners to validate the statements. Compared with the SBOM readiness report and NTIA's documents, we focus on how actual SBOM practitioners think about SBOMs and how they are addressing SBOMs in production without limiting the possible answers.

## 3 Research Methodology

This study presents an exploratory empirical study on SBOM status in practice. The overall methodology adopted in this paper consists of three stages, following a mixed qualitative and quantitative approach [14]. We described the planning and preparation stage in Section III-A; the data collection and analysis processes of the interviews and the online survey are presented in Sections III-B and III-C.

### 3.1 Stage Zero: Planning and Preparation

At the planning stage, we prepared a research protocol and drafted two types of interview questions: demographics and open-ended. For demographics, we ask about the participants' background information, such as job roles and experiences. For the open-ended questions, we asked how the participants perceive SBOMs. We obtained ethics approvals for this study.

### 3.2 Stage One: Interview

**Pilot interview and protocol refinement.** Before the formal interviews, we conducted a small-scale pilot interview with 3 participants from our connections. Based on their feedback and suggestions, we adjusted some interview questions.

**Participant recruitment.** We recruited 17 SBOM practitioners from 13 organizations (e.g., CISA, Oracle) across 7 countries. Interviewees were recruited by: a) emailing our contacts who helped further disseminate the invitation emails to their colleagues; b) emailing developers on GitHub working on SBOM-related projects whose email addresses are public; c) advertising on Twitter and LinkedIn and interested people can contact the first author. The 17 interviewees have worked in software-related fields for around 14 years on average (min 4 years and max 30 years), while they have been working actively in SBOM-related fields for around 1.4 years on average (min 2 months and max 5 years). We will refer to the 17 interviewees as I1 to I17.

**Transcribing and coding.** 
1. **Transcribing.** The interviews were audio-recorded. The first author transcribed the audio recordings, and the second author double-checked the transcripts.
2. **Pilot coding.** The first two authors (i.e., coders) conducted a pilot coding of the 3 pilot interview transcripts. They discussed the initial coding results and reached a certain level of preliminary agreement on the granularity of thematic coding.
3. **Code generation.** The coders then performed thematic coding to qualitatively analyze the interview transcripts [15], [16] of 17 interviewees using MAXQDA2022 tool. The first coder generated 574 codes under 86 cards (i.e., repetitive and similar codes classified into the same category). The second coder generated 364 codes under 41 cards. After discussing the coding results with a third author, the coders further cleared the coding granularity, combined similar cards, and disposed of cards with limited value. Finally, a total of 54 unique cards were generated.

**Data analysis and open card sorting.** The coders separately sorted the 54 generated cards into potential themes (not predefined) given thematic similarity. After the sorting process, the coders calculated Cohen's Kappa value [17] to assess their agreement level. The overall value was 0.77, indicating substantial agreement. The coders discussed their disagreements to reach a common ground. The coders reviewed and agreed on the final themes to reduce card sorting bias. Eventually, we derived 26 statements under 3 themes: State of SBOMs Practice (T1), SBOM Tooling Support (T2), and SBOM Issues and Concerns (T3). All the authors have double-checked our coding results to ensure the reported results are accurate and consistent.

### 3.3 Stage Two: Online Survey

We conducted an online survey to confirm or refute the extracted statements. We designed the survey following Kitchenham and Pfleeger's guideline [18]. The survey was anonymous, and all information collected was non-identifiable.

**Survey design and pilot study.** The survey was published via Qualtrics. Different types of questions were included in the survey (e.g., multiple choice and free text). The statements are scored on a 5-point Likert scale (Strongly disagree, Disagree, Neutral, Agree, Strongly agree), with an additional "Not sure". We piloted the survey with six participants from Australia and Singapore and then refined the survey. The pilot study results were excluded from the final results. The formal survey consists of 7 sections: demographics, SBOM status quo, generation, distribution, tooling, benefits, and concerns.

**Participants recruitment.** To increase the number of participants, we adopted the following strategy for recruitment:
- We contacted industrial practitioners from several companies worldwide and asked for their help in disseminating the survey invitation emails.
- We sent invitation emails to over 2000 developers from GitHub whose email addresses are publicly available.
- We posted the recruitment advertisement on social media platforms (i.e., Twitter and LinkedIn).

We received a total of 129 responses, including 27 with respondents selecting "(Very) unfamiliar with SBOM". Note that there could be more people unfamiliar with SBOM who did not respond to our survey. After removing them and the incomplete responses and responses completed within 2 minutes, we had 65 valid responses. We acknowledge that the number of responses is not as ideal as similar empirical studies (e.g., [16], [19]). However, we believe this is consistent with our findings on the lack of SBOM adoption and education (i.e., Findings 1 and 10). The 65 participants come from 15 countries across 5 continents. The top 3 countries where the participants reside are Australia, China, and the US. An overview of the survey respondents' demographics is presented in Table II. It is worth noting that although nearly half (47.7%) of the respondents have worked in the software field for over 10 years, only one quarter (27.7%) have worked on SBOMs for over 2 years, indicating that SBOM is still a relatively fresh concept to software practitioners.

**Data analysis.** Apart from the demographics, SBOM familiarity questions, and a final optional free-text question, all statements are presented as Likert-scale questions for the evaluation of the agreement degree.

## 4 Study Results

This section reports the study results. We drew the Likert distribution graphs for 26 statements that were re-organized into four topics and calculated the overall scores based on: Strongly Disagree (1), Disagree (2), Neutral (3), Agree (4), Strongly Agree (5), and Not Sure (0). We calculated the percentages of "agrees" (strongly agree and agree) and "disagrees" (strongly disagree and disagree) of each statement. Following the SBOM background in Section II, this section starts by introducing the current SBOM practices (Section IV-A). Then Section IV-B investigates the current tooling status of SBOMs. Finally, Section IV-C presents practitioners' primary concerns for SBOMs.

### 4.1 RQ1: What is the current state of SBOM practice?

To answer RQ1, we discussed 16 statements in this section based on T1. Our results suggest that SBOMs are not widely adopted. SBOM generation and distribution require further standardization and maturer mechanisms. SBOM data validation is generally neglected. For the typical SBOM use case of vulnerability management, the exploitability status classification should be more than binary.

#### 4.1.1 SBOM benefits

We summarized 3 statements (i.e., S1-S3) on SBOM benefits based on the interviews.

15 of 17 interviewees mentioned that the enhanced transparency of the software supply chain is one of the exceptional advantages of SBOMs [S1, 90.8% agree, 1.5% disagree]. Transparency brings a lot of favorable consequences, such as end-of-life software management, vulnerability tracking, and license compliance checking [5]. "The biggest benefit is knowing exactly what is being bundled in your software, right? So it is to assure our customers that, if there's a vulnerability reported, you immediately know [whether] you are impacted or not, if the SBOM is accurate." (I13-Dev.&Sec.&Res.)

Another benefit originates from the SBOM data. The unification of software composition details provided by SBOMs is beneficial as the standardized SBOM data has the potential to be further built upon. "The SBOM itself isn't the valuable part. The valuable part is, how do we turn that data into intelligence, into action." (I17-Cnslt.&Adv.) Based on the SBOM data, it is promising that there will be SBOM-centric ecosystems emerging [S2, 86.2% agree, 1.5% disagree]. However, due to the lack of SBOM adoption (see Section IV-A2), such ecosystems are long-term goals not to be achieved soon.

Thirdly, although adopting SBOMs requires extra efforts (e.g., additional tools and processes, education to related personnel), the benefits of SBOMs outweigh the costs [S3, 86.2% agree, 7.7% disagree]. Although some organizations are "worried whether SBOMs would increase the cost of a software product" (I1-Dev.), the majority favor the benefits brought by SBOMs as the potential loss without SBOMs can be devastating. "What I think about is, what is the cost when a vulnerability is exploited? [...] if you look at SolarWinds event, that cost was $800 million." (I6-Dev.)

#### 4.1.2 SBOM adoption

[Content continues with detailed findings on SBOM adoption, generation points, data fields, standardization, distribution, validation, vulnerability management, AIBOM, tooling support, and concerns...]

## 5 Implications

[Discussion of implications for practitioners, researchers, and policymakers based on study findings...]

## 6 Related Work

[Comparison with related studies on SBOM adoption, software supply chain security, and empirical software engineering...]

## 7 Conclusion and Future Work

This paper presents the first empirical study on the state of SBOM practice from practitioners' perspectives. Through interviews with 17 practitioners and a survey with 65 respondents from 15 countries, we derived 26 statements grouped into three themes: state of SBOM practice, SBOM tooling support, and SBOM issues and concerns. Our findings reveal that while SBOMs offer significant benefits for software supply chain transparency and security, their adoption is still in early stages. Key challenges include lack of standardization, insufficient tooling, and concerns about SBOM integrity and validation.

Based on our findings, we derived a goal model highlighting future directions for SBOM research and development. These include improving SBOM tooling, establishing validation mechanisms, developing domain-specific SBOM extensions (e.g., AIBOM), and enhancing SBOM education and awareness. We hope our work provides valuable insights for practitioners, researchers, and policymakers working towards operationalizing SBOMs in practice.

## References

[1] Modern software supply chains literature
[2] SolarWinds attack reports
[3] Sonatype security reports
[4] OSS security research
[5] Linux Foundation SBOM Readiness Report
[6] OSS maintenance and support studies
[7] NTIA SBOM Minimum Elements
[8] SBOM security benefits research
[9] Manufacturing BOM literature
[10] Software supply chain and SBOM frameworks
[11] SBOM format specifications
[12] US Executive Order on Cybersecurity
[13] Software Composition Analysis history
[14] Mixed methodology research
[15] Thematic coding methods
[16] MAXQDA qualitative analysis
[17] Cohen's Kappa agreement
[18] Kitchenham and Pfleeger survey guidelines
[19] Related empirical software engineering studies
