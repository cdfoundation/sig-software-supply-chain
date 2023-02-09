# Supply Chain Maturity Metrics

* **Author:** [David Bendory (Google/Tekton)](https://github.com/bendory)
* **Contributors:** Justin Abrahms (eBay)
* **This public document is part of the [CDF’s Supply Chain Maturity Model workstream](https://github.com/cdfoundation/sig-software-supply-chain/blob/main/workstreams/scmm/README.md).**
* **History:** This file was imported from the original [source doc](https://docs.google.com/open?id=1CDSbQezqauwL2BaFob7o2ztLk6dTQGZqZCMZ_szNhW8&resourcekey=0-ooiOpNu2gyR-KOlMNOCcDA).


## Objective

The goal of this document is to describe Supply Chain Maturity Metrics. For each metric, we seek a definition and description that clarifies its purpose and reason for inclusion.

The metrics herein are lifted from a [previous brainstorming doc](https://docs.google.com/document/u/0/d/1rGvQv2GH8HYbQUZxg89LmWsYe8-MyeTEcj08nwULitw/view).


### Background

The most common question asked about Supply Chain Maturity Metrics is: “How is this different from [Dora Metrics](https://www.devops-research.com/research.html)?”

Dora identifies five key metrics, four of which focus on software delivery. These metrics “provide a high-level systems view of software delivery and performance and predict an organization’s ability to achieve its goals.” The Dora metrics are:

*   Software Delivery Performance metrics
    *   Lead time for changes
    *   Deployment frequency
    *   Time to restore service (when there is an outage)
    *   Change failure rate
*   Operational Performance metric
    *   Reliability

Supply Chain Maturity metrics are supplemental to the many details Dora elides and are helpful in identifying gaps in the supply chain that will improve Dora metrics. For example:

*   Dora is silent about how to improve change failure rate. Supply Chain Maturity metrics provide numerous milestone metrics around everything from linting and static analysis to code coverage to pre- and post-commit events and integrations -- all of which contribute toward preventing failures and thus reducing the change failure rate.
*   Dora is silent about how to lower the time to restore service in the event of an outage. Supply Chain Maturity metrics provide milestones like automated rollout, automated rollback, and deployment tracking that provide the tooling and insights needed to minimize recovery time.

One way to think about this is: “Good Dora metrics result from Supply Chain Maturity (and other practices).” If your Supply Chain is more mature, your Dora metrics will improve. When it is unclear how to improve Dora metrics, Supply Chain Maturity provides areas of focus and best practices that will contribute to improvements in Dora metrics.


## Source Code Management

A supply chain without robust source code management quickly degenerates into an ongoing series of non-repeatable, untraceable deployments of unidentifiable software. Thus the first measures of maturity are around Source Code Management.

Once Source Code Management exists -- and existence is agnostic to the specific system chosen for use -- the following metrics measure maturity of use:


### Source Code Management Required

While developers must be free to build arbitrary software, the ability to deploy software built from outside the Source Code Management system must be disallowed in a mature supply chain. Assets deployed into production must be identifiably built from code in the Source Code Management system. If an emergency break-glass procedure exists, it must track the deployment of any software that has bypassed this control and a procedure must be followed to restore integrity between managed source code and software deployed in production.


### Review Requirements

The organization has defined review requirements for submitted code. Ideally, no code can be submitted for inclusion in a deployable artifact without an independent review from another software engineer, manager or other appropriate party. Highly sensitive code _may_ require more than one approval. More reviews is not necessarily more mature -- that is an organizational decision that must be aligned with organizational goals and requirements.


### Linting and Static Analysis

Source code must be regularly analyzed for bugs, errors, and best-practices that are detectable from static software analysis. Standard tools like `pylint` and `go vet` can be used and augmented by custom, per-organization standards and requirements. Static analyzers include auto-formatters that ensure that all code adheres to organizational style guides and coding standards.

Linters and Static Analyzers are best run both nightly and via triggered events.


#### Nightly Analysis

A nightly analysis will pick up any changes to code standards -- thus existing code in the repository that was committed before the standard existed (or before it was updated) can be flagged for update.


#### Pre-Commit Triggers

A pre-commit trigger will flag violations before code is committed to the repository and _may_ be configured to prevent submission. Organizations will have different standards for blocking and non-blocking issues.


### Code Management [cdevents](https://cdevents.dev/docs/source-code-version-control/)

Source Code Manager events can be used to trigger pipelines that verify the integrity of software.


#### Pre-Commit Events

Pre-commit triggers run on proposed changes (pull requests, merge requests, etc.). In increasing levels of maturity, pre-commit pipelines will confirm that proposed changes pass static analysis, merge cleanly, build, pass unit tests, and pass integration tests.


#### Post-Commit Events

Post-commit triggers launch CICD pipelines on each commit. In addition to confirmations run by pre-commit triggers, post-commit triggers may additionally trigger automatic deployment of software to an integrated testing environment and beyond. Note that different pipelines may be triggered by different branches, tags, etc.


##### Detection of Bad Commits

A mature testing framework will identify bad commits and emit events that may notify the developer who committed the code, the build team, or adjacent teams. A mature system may auto-generate a request for rollback or automatically roll back the bad commit.


## Build Maturity

Build maturity seeks to measure the maturity of build processes. For the purposes of this document, a “build” is a CICD process -- which may be a single task or a pipeline of tasks -- that either results in an artifact that will be tested or deployed or verifies the readiness of an artifact for deployment.


### Build Automation

An automated build process is typically the first step toward CICD automation, and its stable existence unblocks all further automation. An automated build is repeatable.


#### Pre-Merge CI

Pre-merge continuous integration is used to verify software integrity of an individual proposed change and can prevent a single change from breaking the shared integration environment. Pre-merge CI minimally ensures that proposed changes build properly; ideally, some set of integrity tests (at least unit tests) are performed as well.


#### Post-Merge CI

Post-merge integration and testing verifies that committed software remains deployable. Artifacts from post-merge integration may proceed to deployment to a shared integration environment.


#### Nightly Build

A nightly build-from-scratch ensures that software can be bootstrapped at any time. A nightly build is typically the first step toward a nightly automated deployment to a shared QA environment.


#### Build Metrics

Collecting build metrics in a time series enables tracking of build latency and the rate of acceptance of breaking changes. Time-to-repair for a broken build can provide a Dora insight for the health of the development and/or integration environment.


### [SLSA Level](https://slsa.dev/spec/v0.1/levels)

SLSA Levels measure the integrity of built assets, demonstrating traceability of build inputs and output artifacts. Higher SLSA levels enforce more mature dependency management. A more mature CICD will feature greater automation including the audit trail enabled by SLSA provenance generation and reproducible builds. Security features such as hermetic builds further ensure the integrity of deployed assets.


### Dependency Management

“Dependencies” here means “anything needed to recreate our production environment.” In most organizations, this will mean source code in the Source Code Management system, Open Source, and potentially built artifacts in a repository such as a Docker registry or Python repo.

More mature organizations will include lower-level details as “dependencies” -- such as the build tool versions and even the OS version and installed OS packages.

Highly sensitive organizations may go even further to specify hardware requirements and dependencies.

This document assumes that “dependencies” has been defined appropriately for the organization’s desired goals.


#### Dependency Inventory

A mature supply chain has an exhaustive list of its build dependencies, hence the existence of a Dependency Inventory and the completeness thereof both demonstrate and contribute to supply chain maturity. A dependency inventory is a pre-requisite for higher SLSA levels that have requirements like “reproducible builds” and “hermetic builds.”


#### Package Managers

As dependencies grow in number, package managers become essential to dependency management. A mature supply chain will not arbitrarily take in dependencies but rather will track dependencies, verify them, scan for vulnerabilities and take steps to avoid malicious software.


#### License Management

Dependencies must be monitored to align license requirements with intended use and avoid license violations.


#### EOL Management

Every dependency must be monitored for ongoing support and maintenance and sunset as end-of-life approaches.


#### Security Updates

Each dependency should be monitored for known CVEs and remediation should be pursued. Counts of vulnerabilities and severity levels should be tracked over time as well as time to remediation -- which could take the form of taking a newer version, removing the dependency, or verifying that the vulnerability does not apply given the use of the dependency.


## Testing Maturity

Engineering confidence in built artifacts is highly dependent on test automation in CICD pipelines. We break testing into three realms with different metrics for each.


### Unit Tests

Unit tests are typically run in response to an event (like the addition of new code to the repository or the upgrade of a dependency) or a schedule (like a nightly build).


#### Code coverage metrics

Code coverage is indicative of the quantitative reach of unit tests, but by itself may not be indicative of the quality of unit tests. Good unit tests will have high code coverage metrics, but high coverage in itself may not be indicative of good testing. Also note that there is a law of diminishing returns with code coverage; not all organizations should make high-cost investments in improving code coverage beyond some “good enough” point that supports the reliability of deployed software in alignment with desired goals for Dora metrics and organizational outcomes.


#### Pre-merge testing and gating

Pre-merge unit tests (passing tests as a gate prior to acceptance of proposed code changes) help keep mainline development in an “always green” state. Pre-merge coverage metrics help maintain coverage by giving an indication of the coverage of newly added code and can be used to prevent addition of untested code.


#### Post-merge testing

Post-merge tests can detect breakage or regressions early in the CICD cycle and enable identification of breaking changes.


#### Nightly Testing

Nightly test runs commonly verify main line integrity; in some implementations they are used to gate the creation of nightly deployment candidates.


#### Unit Test Metrics

Unit test metrics create a time series that tracks:

*   Changes in latency
*   Changes in code coverage
*   Historic pass/fail rate -- enabling flake detection


### Integration Testing

Integration testing is typically “larger” than unit testing, either because it requires interaction with an external dependency like a database or microservice or requires the existence of multiple deployable artifacts (like components of a data processing pipeline).


#### Event-driven Integration Tests

Integration tests are typically triggered by the creation of a new deployable artifact. Integration tests are used to verify artifact functionality in relation to other deployable artifacts and to gate deployments.


#### Forwards / Backwards Compatibility Testing

An integration environment is often the first point where different components can run against each other in a version compatibility matrix to ensure forward and backward compatibility, essential requirements for smooth rollouts.


#### Integration Test Metrics

As with unit tests, integration test metrics create a time series that tracks:

*   Changes in latency
*   Historic pass/fail rate -- enabling flake detection as well as failure rate for integrated changes


### Deployment Testing and Probes

A mature supply chain includes constant operational monitoring of production. Deployment tests and probes may in fact be identical to integration tests (or a subset thereof). Their existence enables monitoring of production integrity, deployment health checks, Canary testing, automated rollbacks, blue/green rollouts, and additional signals all of which contribute to higher levels of rollout automation and rollout confidence.


### Security Testing

Software integrity testing may include penetration testing and fuzz testing.


### Load Testing

A mature supply chain provides signal on how deployed artifacts perform under bursts of traffic, including measures of maximum load, verification of graceful degradation prior to complete failure, and testing of load-handling strategies such as traffic shedding and auto-scaling.

Proper load testing contributes to reliability in production.


## Deployment Maturity

Measures of deployment maturity are likely to be highly correlated with Dora metrics as higher levels of deployment automation and management enable more frequent rollouts and faster time to repair when outages occur.

Some organizations will prefer a fully-automated deployment process with no human interaction except in failure scenarios, while others will prefer manual gating whereby a human decides when to launch each stage of the rollout. Neither answer is inherently more correct; the choice is dependent on organizational needs.


### Automated Rollout

Even with manual gating, rollouts themselves are automated such that the manual gate consists of a human hitting some variation of an “OK to proceed” button. Automation ensures reproducible rollouts that are verifiable via deployment health checks (see Deployment Testing above). Automation should include automated rollback -- which may be achieved by simply rolling out a previous candidate or may involve some other strategy such as traffic splitting.


### Rollout Strategies

Each organization will choose a rollout strategy suitable to its stability and velocity needs. Possibilities include gradual rollouts, blue/green deployments, canary testing, and traffic-splitting. A similar strategy should be in place for automated rollback.


### Break-Glass Process

The existence of a defined robust “break-glass” process can allow more rapid emergency fixes (potentially including manual repair), but a mature process will track break-glass use for later post-incident analysis. This ensures both later integration of the production reparation intervention with the existing CICD as well as operational analysis of the need for and security around break-glass.


### Feature Gating

Mature organizations often separate software deployment from feature release, gating features behind feature flags that enable and disable new functionality. This often makes rollout/rollback of the feature as simple as restarting the deployed production process with a changed flag value (--enable-my-new-feature / --disable-my-new-feature). This more fine-grained control over feature enablement can change software deployments from “all-or-none” rollouts that are rolled back on failure to smoother software no-op (or “small change”) software deployments followed by a series of incremental flag-flips to enable new features -- keeping enabled only the features that are found to work successfully in production. This approach can improve stability, enable very rapid rollback when needed, and drastically reduce change failure rates.


### Deployment Tracking

A mature deployment system enables the creation of both a live state of production as well as a historical audit trail of deployed artifacts. This in turn enables security tracing, an audit of bugs and malicious code, and analysis of historic existence of newly-discovered but long-existing bugs and security vulnerabilities in production.


### Deployment Metrics

Deployment metrics include:

*   Latency from merge to candidate creation
*   Latency from merge to QA deployment
*   Latency from merge to production deployment (Dora metric)
*   Release frequency (Dora metric)

Note that the latencies from merge to candidate creation and QA deployment are pieces of the Dora metric for latency from merge to production deployment.


## Configuration Management

A mature Supply Chain treats configuration as code -- whether that configuration controls the build system (that creates production artifacts) or controls the running of production aritfacts themselves (like feature-gating flags above).

Treating configuration as code demonstrates robust change management over all production controls, from artifact creation to runtime flags and configuration parameters. These may include hardware requirements like CPU, memory, available free disk, or the availability of specific hardware such as GPUs.

Configuration management provides more granular traceability and an audit trail for answering questions like “what was running in production at this time” or “was feature X enabled or disabled when the failure occurred?”

The treatment of configuration as code means that configuration is included in (nearly?) all of the above maturity measures.


## Production Security

Organizationally, the “Supply Chain” _may_ be considered complete once the artifact is in production. Thus Production Security maturity and subsequent metrics might be considered Operational Maturity rather than Supply Chain Maturity. That said, so long as the artifact lives in production, it’s the final link in the Supply Chain, and thus this document includes Production Security and subsequent measures of maturity as well.


### Environment Isolation

A mature supply chain includes controls segregating developer environments from production deployments. Some organizations may place restrictions on the ability of non-production software to access production assets and vice-versa, while for others a less strict boundary may be sufficient. These restrictions ideally isolate every environment from every other environment in accordance with legal, business, and regulatory requirements. Technical considerations include the dangers of code in a non-production environment reading or changing production information as well as the possibility of production software accessing non-production data.


### Production Lockdown

Production lockdown ensures the integrity of deployed artifacts by preventing unauthorized changes in production. Ideally, only the automated rollout system can change production, though in the real world a break-glass procedure is often required. A mature supply chain will have controls over human access to production -- whether read or write access.

Proper production lockdown both enables and requires deployment automation.

Workload identity and policy enforcement restrict inappropriate artifacts from running in production at all.


### Access Tracking

Tracking of production access -- even read access -- creates an essential audit trail in maintaining supply chain integrity. Post-incident analysis of production access ensures that manual fixes in production are properly reflected in the software supply chain itself. Read access tracking enables assessment of the extent of compromise in a breach as well as compliance with regulatory requirements around data privacy.


### Vulnerability Scanning

A secure production environment requires vulnerability identification in all artifacts. Scans ideally occur not only prior to deployment to prevent release of vulnerable software but also on an ongoing basis to identify newly-identified vulnerabilities in currently-running software.

The first step is the existence and integration of artifact scanning for deployable artifacts. The ability to identify new vulnerabilities in existing production assets depends on accurate [Deployment Tracking](#bookmark=id.qfm3s8qrc324).

Vulnerability metrics include both the number and severity of vulnerabilities as well as latency of time to mitigate newly-discovered vulnerabilities. For critical vulnerabilities, time to mitigation of a severe vulnerability might be incorporated into Dora measures of time-to-mitigation of an outage.


### Policy Enforcement

Ongoing policy enforcement in production can be used to ensure that only verified assets in compliance with organizational policies are permitted to run in production. This can prevent execution of malicious software that has bypassed normal rollout procedures, block deployment of inappropriate assets, and provide continuous verification of existing assets vs. new policies.


## Lifecycle and Audit Trail

Lifecycle management of production assets can be incorporated into a complete audit trail, easing compliance with regulatory and internal audits. Auditable assets include:

*   CICD log lifecycle
*   Deployment event logging lifecycle
*   License inventory for all dependencies
*   Production access log lifecycle
*   Artifact freshness, potentially including build horizon and sunsetting of assets by age
*   Artifact history and life cycle tracking

Appropriate time horizons must be established for all audit trails based on organizational needs.


## Out of Scope

This document completes its scope with the deployment of software assets produced and managed by an organization; thus the below areas -- which may be considered part of the Supply Chain -- are considered out of scope for this document.


### Platform Management

This document is limited to the software delivered but does not include the platform where it runs. All of the metrics herein can be applied to the underlying platform down to the OS itself.

Note that if the platform itself is the artifact produced and deployed by an organization, then all the metrics herein can be applied to said asset.

[Additional information around this topic](https://docs.google.com/document/d/1UDL3E5BqPyDdzV1wek9o8xR-nLJQXEEMU1qIMCgq5cI/view#heading=h.g2ibuwx8cji2) is available from the CNCF Platforms Working Group.


### Hardware Verification / Hardware Management

Hardware supply goes a level deeper than Platform Management and is similarly considered out-of-scope for this document.

