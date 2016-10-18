---
layout: post
title:  "What can you test in Software Design Phase? Part II"
date:   2016-09-06 18:43:59
author: Eing Ong
categories: Testing
tags: shift-left
---
As a continuation of [Part 1 of Software Design Testing](/testing/2016/08/03/WhatCanYouTestAtDesignPhase-Part1/), I'll share what I would bring up during design phase for security, and tooling.

<h2>Security</h2>

For security, I'd typically explore trust boundaries, data flows, data entry & exit points, threats and vulnerabilities. You can refer to [OWASP guide](https://www.owasp.org/index.php/Application_Threat_Modeling) when defining the security threats for your application. Another good reference is from [Microsoft](https://msdn.microsoft.com/en-us/library/ff648006.aspx), although it's no longer maintained, it contains timeless and valuable content and even has a template for threat modelling. For simplicity, here's the table to list your threats and to ensure you cover all aspects of STRIDE.

| Security threat type (STRIDE) | Security threat | Mitigations | Result | Recovery | 
| -------- | ------- | -------- | -------- | -------------------------------- |
| **Spoofing** <br>Illegal access and use another user's credentials,<br>such as username and password. | E.g. Admin credentials are compromised | | e.g. Alerts will be sent out<br>Audit tools will detect and log these requests| |
| **Tampering** <br>Maliciously change/modify persistent data | | |
| **Repudiation** <br>Perform illegal operations in a system that lacks the ability to trace the prohibited operations | | |
| **Information disclosure**<br>Read a file that one was not granted access to, or to read data in transit | | |
| **Denial-of-Service**<br>Deny access to valid users| | |
| **Elevation of privilege**<br>Gain privileged access to resources for gaining unauthorized access to information or to compromise a system | | |

A good book to read is [Threat Modeling: Designing for Security by Adam Shostack](https://www.safaribooksonline.com/library/view/threat-modeling-designing/9781118810057/). The end goal for security should be no unmitigated high priority threats to leak to production. Tools such as Checkmarx or Fortify (static code analysis), OWASP dependency scan, AppScan (runtime analysis), and Threat Modeling should help you achieve these goals. This brings us to the next topic - Tooling.

<h2>Tooling </h2> 

["Only fools use no tools"](https://drive.google.com/file/d/0B0QztbuDlKs_M2ZiMWU2NjQtNjExYS00M2ZmLThmNWUtYmU5MGYwNjRhNjYz/view?hl=en&pref=2&pli=1)

It helps to explore tooling early so your team will think about testing early and familiarize with the means at their disposal to measure how they are doing as a team even before a first line of code is checked in. Some questions you can ask
 
* What are the static and dynamic analysis tools we can use?
* What will be used for unit testing and code coverage?
* What do we use for component testing, performance, and resiliency testing?
* Do we need to minimize time to create new testsuites?
* Do we need to replicate/generate/enrich/versioned existing data for testing?


| Purpose | Tools to consider |
| ------- | ----------------- |
| Static analysis for code quality | E.g. CheckStyle, FindBugs, Checkmarx, [Mutation Testing](http://zeroturnaround.com/rebellabs/mutation-testing-in-java-with-pitest-by-henry-coles/) |
| Dynamic analysis for code quality | E.g. [DriftDetector](/testing/2016/07/06/DetectingDrifts/) or [Hystrix Network Auditor Agent](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-network-auditor-agent) |
| Code reviews | E.g. Git's pull request (PR) |
| Unit testing | E.g. JMockit, TestNG |
| Component testing (including dependencies stubbing and resiliency testing capability) | E.g. RestAssured, WireMock |
| Performance | E.g. Gatling, JMeter |
| Data generation/enrichment/replication/versioning | E.g. Delphix |
| Minimizing time to create new tests | E.g. [RestAssured CLI](https://github.com/eing/restassured_cli) |

Two additional areas that is part of building tools and capabilities are testability & productivity enablement. While testability is owned by the scrum team, productivity enablement ownership can be a combination of scrum teams, DevOps and center of excellence.

<h4>Testability</h4>

The ability to Unit Test is a form of testabilty. Sometimes you have to build in capabilities such as HTTP headers, messages so that they can trigger application behaviors that are otherwise hard to simulate. These capabilities are only enabled in pre-production. To shed some light on how testable your application is, explore the following questions.  

* Is my code testable?
* Can all tests be automated?
* Are there any scenarios that are difficult to test?
* How can we simulate other services we depend on?
* How can we enable other teams to exercise our services reliably in pre-production?

<h4>Productivity enablement</h4>

With the transition of SCM/Ops to DevOps role, DevOps now develop capabilities to enable all scrum teams to deliver releases fast. Scrum teams are empowered to own production deployment, feature toggles, monitoring and alerts. This may vary on the company, however, you would still want to have the following questions answered for a low-risk, automated and frequent releases.

| Capability | Is plan in place to meet SLA on failure? | Supported in Pre-Prod | Supported in Production? |
| Blue-Green deployments | | | |
| System health monitors and alerts | | | |
| Logging monitors and alerts | | | |
| Performance and Hystrix Dashboards and alerts | | | |
| Production failover clusters & availability zones | | | |


<h3>Conclusion</h3>
This concludes Software Design Testing Part I & II blogs. I hope this is useful. Please add any feedback or comments you may have.

