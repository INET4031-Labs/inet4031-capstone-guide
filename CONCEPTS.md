# Concepts

Short explanations of the ideas behind each track. Each one ends with where to read more. When a tool's own documentation disagrees with this page, trust the documentation for the version you installed.

## Shared ideas (every track)

**Baseline.** A measurement of the system before you change it. Every claim like "faster," "safer," or "better" needs a baseline to compare against. Take it first; you cannot go back for it later.

**Evidence.** Saved output from a real run, committed to the repo, that proves a claim. A screenshot is weaker than a text file because it cannot be searched, diffed, or re-run.

**Reproducible.** Someone else can run the same command or script and get the same result. Fixed random seeds, committed scripts, and recorded tool versions make work reproducible.

**Runbook.** Step-by-step instructions for operating, checking, and recovering a system, written so another engineer can follow them without asking you.

**Infrastructure as code.** The system is described in files in git (manifests, scripts, playbooks) instead of in someone's memory. The rebuild in Week 13 tests how much of your system is really in code.

**Default-deny NetworkPolicy.** Your `default` namespace blocks all incoming traffic to pods unless an allow policy selects the pod and port. New kinds of pods are unreachable until you write their allow. (Shared VM. The Local VM has no default-deny: only the `db` pods are protected, by the `db-clients` policy, and any pod you add accepts traffic from anywhere until a policy selects it.)

**Requests and limits.** A memory or CPU *request* is what the scheduler reserves for a pod. A *limit* is the most it may use; a pod over its memory limit is killed. Every pod in this course needs memory requests and limits: on the Shared VM because several teams share one VM, and on the Local VM because your VM is small and your Deploy gate's security scan requires them.

## Networking and Cloud Infrastructure

**Service and endpoints.** A Service is a stable name and virtual IP in front of a changing set of pods. Its endpoints are the IPs of the pods that match its label selector *and* are passing their readiness probe.

**Cluster DNS (CoreDNS).** Pods find Services by name, such as `flask` or `flask.default.svc.cluster.local`. CoreDNS answers those lookups inside the cluster.

**kube-proxy.** Programs each node so that traffic to a Service's virtual IP is forwarded to one of its endpoints.

**Readiness vs liveness probes.** Readiness decides whether a pod receives traffic. Liveness decides whether the container gets restarted. A failing readiness probe removes a pod from the Service without killing it.

**HorizontalPodAutoscaler (HPA).** Changes a Deployment's replica count based on a metric, usually CPU use as a percentage of the CPU *request*. It reacts after load arrives, checks on an interval, and waits before scaling down, so there is always a gap.

**PodDisruptionBudget (PDB).** Limits how many pods can be taken down at once by *voluntary* disruptions such as a node drain. It does not prevent *involuntary* disruptions, like a node container being stopped.

**Scheduling spread.** Rules (topology spread constraints or pod anti-affinity) that place replicas on different nodes, so one node failure does not take all of them.

Read more: [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/), [HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/), [Disruptions and PDBs](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/), [Probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/).

## Machine Learning and AI

**Label.** The answer you want the model to learn, such as a category or severity. In this track you create the labels, which is the hardest and most important part.

**Train/test split.** Hold back part of the data the model never sees during training, then measure on it. Measuring on training data tells you nothing.

**Baseline model.** The simplest possible guess, such as always predicting the most common label. If your model does not beat it clearly, it is not useful.

**Accuracy, precision, recall, F1.** Accuracy is the share of correct predictions. Precision asks "when the model says Critical, how often is it right?" Recall asks "of all real Critical incidents, how many did it catch?" F1 combines the two. **Macro F1** averages F1 across classes equally, so a rare class counts as much as a common one.

**Confusion matrix.** A table of true label against predicted label. It shows *which* mistakes the model makes.

**TF-IDF.** Turns text into numbers by weighting words that are common in one title but rare across all titles.

**Unsupervised anomaly detection.** Finding unusual points without labels. IsolationForest flags values that are easy to isolate from the rest.

**Drift.** Real data changes over time (new systems, new wording) and model accuracy drops even though the code did not change.

Read more: [scikit-learn user guide](https://scikit-learn.org/stable/user_guide.html), [model evaluation](https://scikit-learn.org/stable/modules/model_evaluation.html).

## Applied Data Science

**Data quality check.** Counts of rows, nulls, and date ranges, saved before any analysis. It tells the reader what the data can and cannot support.

**Median vs 90th percentile.** The median is the typical case. The 90th percentile is the slow case that one in ten incidents reaches. Staffing for the median leaves you short on bad days.

**Skewed distribution.** Resolution times usually have a long tail of slow cases, so the mean is pulled upward and comparisons should use medians or rank-based tests.

**Trend slope and p-value.** A regression slope says how much weekly volume changes per week. The p-value says how surprising that slope would be if there were no real trend. A small p-value is evidence of a trend, not proof of its cause.

**Synthetic data.** Data you generated. Any pattern you find was put there by your generator, so say what you planted and treat the analysis as a demonstration of method.

Read more: [pandas user guide](https://pandas.pydata.org/docs/user_guide/index.html), [scipy.stats](https://docs.scipy.org/doc/scipy/reference/stats.html).

## Cybersecurity and Governance

**Control, test, evidence.** A control is a rule ("containers do not run as root"). A test is a yes/no check of the rule. Evidence is the saved output of the test. Auditors want all three, linked.

**CIS-style controls.** Practical security baselines in the style of the CIS Benchmarks. You do not need the full benchmark; you need a short list you can test.

**Vulnerability severity.** Scanners rate known vulnerabilities (CVEs) from LOW to CRITICAL. A HIGH finding with no available fix may be accepted as a risk; a CRITICAL one with an easy fix should be fixed.

**Triage.** Every finding gets one of three outcomes: fix it, accept it as a documented risk, or mark it a false positive with a reason.

**Risk register.** A list of risks, each with likelihood (1 to 5), impact (1 to 5), score (likelihood times impact), an owner, and a treatment (reduce, accept, transfer, avoid).

**Least privilege (RBAC).** Each identity gets only the permissions it needs. `kubectl auth can-i` lets you test this directly.

Read more: [Trivy docs](https://trivy.dev/docs/latest/guide/), [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/), [CIS Benchmarks overview](https://www.cisecurity.org/cis-benchmarks).

## DevSecOps and IT Project Management

**Quality gate.** An automated check that runs on every pull request. *Required* gates block merging when they fail; *non-required* gates report without blocking.

**Shift left.** Catch problems earlier in the process. A pre-commit hook on your machine costs seconds; the same defect found after merge costs a regression.

**Semantic versioning.** `MAJOR.MINOR.PATCH`. Breaking changes raise MAJOR, new features raise MINOR, fixes raise PATCH.

**Annotated tag.** A git tag with its own message, author, and date, created with `git tag -a`. Releases use annotated tags.

**Rollback.** Returning to a previous known-good release. If it is not rehearsed and timed, you do not know whether it works.

**RACI.** For each activity: Responsible (does the work), Accountable (owns the outcome; exactly one per activity), Consulted, Informed.

**Status report.** A short update for a sponsor: what is done, what is next, what is at risk, and what you need from them. Written in plain language.

Read more: [GitHub protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches), [pre-commit](https://pre-commit.com/), [Semantic Versioning](https://semver.org/), [Conventional Commits](https://www.conventionalcommits.org/).
