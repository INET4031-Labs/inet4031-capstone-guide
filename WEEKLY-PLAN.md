# Weekly Plan: Weeks 11 to 14

Both sections follow the same four weeks. Shared VM teams split the work across roles; Local VM students do all of it alone, so keep scope tight.

## Week 11: Kickoff

**Priorities, in order**

1. Environment Checkpoint passes. Output saved and committed.
2. Tool versions recorded in `TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages).
3. The business problem restated in your own words (top of your ARCHITECTURE or NOTES file).
4. Success criteria turned into issues on the project board, with owners on the Shared VM.
5. QA test cases drafted in `docs/qa-report-final.md` before the implementation exists.
6. A baseline or first decision: the "before" picture your later evidence compares against (see your track below).
7. A `NOTES.md` or RUNBOOK draft that records every command that worked.

**By the end of Week 11:** your environment is healthy, the before picture is committed, and one thin slice of the work runs end to end, even if it is rough.

## Week 12: Build

**Priorities**

1. Implement every success criterion to "works once" quality.
2. Mid-sprint check-in with the TA or professor. Bring your evidence files, not a description of them.
3. Write RUNBOOK steps as you discover them, not at the end.
4. Re-run your QA test cases against what exists.

**By the end of Week 12:** about three of your five success criteria have evidence files committed, and the rest are in progress. Stop adding features at the end of this week. Week 13 is for consolidation.

## Week 13: Consolidate and rebuild

**Priorities**

1. Finish the documents: RUNBOOK, ARCHITECTURE (with diagram), the "How to run this" section in your top-level README, the QA report, and the retrospective or individual reflection.
2. Self-check: clone your repo into a fresh folder and read only the README. Does it tell you every step?
3. Run the rebuild. See [REBUILD.md](REBUILD.md).
   - **Local VM:** create a new VM in VirtualBox with the same steps you used for your original VM, clone your repository, and follow your own README.
   - **Shared VM:** your team takes a turn in the rebuild queue on a separate VM. The VM is wiped between teams.
4. Record every gap you hit in `REBUILD-GAPS.md`, fix each one by pull request, and update the README so the next run does not hit it.

**By the end of Week 13:** documents match reality, the gap log is committed, and your README brings the system up without help.

## Week 14: Present

- **Shared VM:** live presentation. The tools are pre-installed; your team clones the repository live and brings the app up from your README and setup instructions, then shows your track's evidence.
- **Local VM:** recorded presentation of your rebuild.

Rehearse at least once from a clean clone.

## Track-specific targets

### Networking and Cloud Infrastructure

| Week | Target |
|---|---|
| 11 | Baseline recorded (request rate, p95, replica count, memory). Request path sketched. `docker ps --filter name=k3d-<cluster>-` output saved. |
| 12 | Requests and limits, probes, replicas spread across nodes, PodDisruptionBudgets, capped HPA. First scale-up captured. One practice drill. Memory arithmetic ready for the check-in. |
| 13 | Final 3-minute drill evidence, NetworkPolicy before and after test, request trace in ARCHITECTURE, RUNBOOK, rebuild and gap log. |

### Machine Learning and AI

| Week | Target |
|---|---|
| 11 | Generator with labels and a fixed seed, at least 5,000 rows, a first metrics file with the baseline (even if the model is weak). |
| 12 | Trained models, `metrics.json` against the baseline, service in a container, Deployment with memory limits, NetworkPolicy, first timing. |
| 13 | Blocked-traffic evidence, image size evidence, one documented wrong prediction, docs, rebuild and gap log. |

### Applied Data Science

| Week | Target |
|---|---|
| 11 | Data decision written in `DATA.md`, dataset meets criterion 1, `data_quality.csv` committed. |
| 12 | Three analysis CSVs, three charts, one statistical check, one command regenerates all of it. |
| 13 | `report.md` with the staffing recommendation and assumptions, docs, rebuild and gap log. |

### Cybersecurity and Governance

| Week | Target |
|---|---|
| 11 | Tools verified, first scans of image, repo, and manifests saved, control matrix drafted with at least 8 rows. |
| 12 | Remediation and rescan (before and after), RBAC and NetworkPolicy tests, risk register, matrix at 15 rows. |
| 13 | Three policies, evidence script running in under 5 minutes, docs, rebuild and gap log. |

### DevSecOps and IT Project Management

| Week | Target |
|---|---|
| 11 | Branch protection "before" record, commit message convention agreed, new checks added as non-required, pre-commit config, roadmap draft. |
| 12 | Checks reporting on real PRs, deliberately bad PRs shown red, first tag `v1.0.0`, RACI and risk register, decisions on which checks to promote. |
| 13 | Second tag and changelog, timed rollback, status report, docs, rebuild and gap log. |
