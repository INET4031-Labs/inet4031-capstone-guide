# Evidence Checklist

Each success criterion in your track README needs a committed file that proves it. Use this as a checklist and tick items as they land in `main`. File names are suggestions unless your track README names them exactly.

**Save output with `tee`** so you see it and keep it:

```bash
<command> 2>&1 | tee capstone/<track>/evidence/<name>.txt
```

Add a timestamp line at the top of long captures: `date | tee capstone/<track>/evidence/<name>.txt` then append with `tee -a`.

## Every track

- [ ] Environment Checkpoint output
- [ ] `TOOL-VERSIONS.txt` (and `requirements.txt` for Python packages)
- [ ] `RUNBOOK.md`
- [ ] `ARCHITECTURE.md` with a diagram and a "What we changed and why" section ([MAKE-IT-YOURS.md](MAKE-IT-YOURS.md))
- [ ] `DESIGN-BRIEF.md`, only if you replaced a success criterion, with the approval noted
- [ ] "How to run this" section in the top-level README
- [ ] `docs/qa-report-final.md`
- [ ] Retrospective (Shared VM) or Individual Reflection (Local VM)
- [ ] `REBUILD-GAPS.md` from Week 13

## Networking and Cloud Infrastructure

| Criterion | Evidence file | Produced by |
|---|---|---|
| Baseline | `evidence/baseline.txt` | k6 summary plus `kubectl get pods -o wide` and `kubectl top pods` before changes |
| 1. Scales to 2+ replicas under 3x load | `evidence/hpa-over-time.txt`, `evidence/k6-load.txt` | `kubectl get hpa -w` (or a timestamped loop) while k6 runs |
| 2. Under 1% failures in a 3-minute drill | `evidence/drill-k6.txt`, `evidence/drill-times.txt` | k6 summary showing `http_req_failed`; `date` before `docker stop` and after `docker start` |
| 3. PDB per scaled tier and what happens on node loss | `evidence/pdb.txt`, `evidence/drill-pods.txt` | `kubectl get pdb`; `kubectl get pods -o wide` during the drill |
| 4. Request trace and tightened policy | `ARCHITECTURE.md`, `evidence/netpol-before-after.txt` | Your analysis; allowed and denied test output before and after |
| 5. Memory budget | `evidence/memory-budget.txt` | Your arithmetic, plus `kubectl describe` showing limits |

**Local VM variants** (see the Local VM criteria in the track README):

| Criterion | Evidence file | Produced by |
|---|---|---|
| 2. Under 1% failures through a pod failure and a rollout | `evidence/drill-k6.txt`, `evidence/drill-times.txt`, `evidence/drill-pods.txt` | k6 summary; `date` at each pod deletion and at the start and end of the rollout; `kubectl get pods -o wide` during the drill |
| 3. PDB for `web` and what it does on one node | `evidence/pdb.txt` | `kubectl get pdb` and `kubectl describe pdb` at the starting replica count and when scaled, plus your written explanation |
| 4. Request trace and a new or tightened policy | `ARCHITECTURE.md`, `evidence/netpol-before-after.txt` | Your trace of k6 to `web` to `db`; allowed and denied tests from debug pods before and after |

## Machine Learning and AI

| Criterion | Evidence file | Produced by |
|---|---|---|
| 1. Labeled dataset, reproducible | the dataset file, `evidence/dataset-hash.txt` | Run the generator twice with the same seed; `sha256sum` both outputs |
| 2. Metrics with baseline | `metrics.json` | Your evaluation script on the held-out test set |
| 3. Service in cluster under 500 ms | `evidence/describe-deploy.txt`, `evidence/timing.txt` | `kubectl describe deploy`; `curl -w "%{time_total}"` |
| 4. Only intended traffic reaches it | `evidence/netpol-blocked.txt` | A request from a pod that is not allowed, timing out |
| 5. Under 512Mi and image under 600 MB | `evidence/memory.txt`, `evidence/image-size.txt` | `kubectl describe`; `docker image ls` |

## Applied Data Science

| Criterion | Evidence file | Produced by |
|---|---|---|
| 1. 10,000+ rows, 90+ days, categories and resolution times | `output/data_quality.csv`, `DATA.md` | Your pipeline's quality step |
| 2. Three analysis CSVs from one command | `output/*.csv` | Your one-command pipeline |
| 3. Three labeled charts | `output/*.png` | matplotlib with title, axis labels, units |
| 4. Statistical check with interpretation | `report.md` or `output/stats.txt` | scipy, plus one plain-English sentence |
| 5. Staffing recommendation | `report.md` | Your analysis and stated assumptions |

## Cybersecurity and Governance

| Criterion | Evidence file | Produced by |
|---|---|---|
| 1. Control matrix, 15+ rows, all with evidence | `controls.csv` | Your controls, each pointing to a file in `evidence/` |
| 2. Image and manifest scans, before and after | `evidence/trivy-image-before.json`, `evidence/trivy-image-after.json`, manifest scans | Trivy, kube-score, checkov |
| 3. Full-history secrets scan, 0 unresolved | `evidence/gitleaks-report.json` | gitleaks on the repo root, with redaction |
| 4. 8+ RBAC checks and 2+ blocked NetworkPolicy tests | `evidence/rbac-checks.txt`, `evidence/netpol-tests.txt` | `kubectl auth can-i --as=...`; blocked requests from a test pod, with predictions written first. Local VM: identities and the policy under test are listed in the track README. |
| 5. Risk register, 3 policies, evidence script | `risk-register.md`, `policies/`, the script and one timestamped output folder | Your documents; `time ./collect-evidence.sh` |

## DevSecOps and IT Project Management

| Criterion | Evidence file | Produced by |
|---|---|---|
| 1. Four new gates on five real PRs | `evidence/gate-runs.txt` | `gh pr checks <number>` per PR |
| 2. Three bad PRs red, one good PR merged, protection intact | `evidence/gate-runs.txt`, `evidence/branch-protection.txt` | `gh pr checks`; Shared VM: `gh api .../branches/main/protection` before and after; Local VM: `gh api repos/{owner}/{repo}/rulesets/<id>` before and after |
| 3. Pre-commit rejects a bad commit, passes clean | `evidence/pre-commit-reject.txt`, `evidence/pre-commit-clean.txt` | `git commit` output; `pre-commit run --all-files` |
| 4. Two tags, changelog, timed rollback | `CHANGELOG.md`, `evidence/rollback-rehearsal.txt` | `git tag -n`; `git-cliff`; `time` on your rollback command |
| 5. Roadmap, RACI, risk register, status report | `ROADMAP.md`, `RACI.md`, `RISK-REGISTER.md`, `STATUS-REPORT.md` | Your documents |
