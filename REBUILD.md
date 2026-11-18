# Rebuild

In Week 13 you prove that your work lives in your repository and not only on the VM where you built it. In Week 14 you show it.

## The contract

Start from a clean VM. Clone your repository. Follow **only** your README, RUNBOOK, and the scripts in your repo. No commands from memory, no files copied over from the old VM, no "oh, I also had to run this one thing."

If the system comes up, your documentation is complete. If it does not, you have found a gap, and finding gaps is the point of the exercise.

A link counts as documentation when it points at the exact section to follow (for example a specific part of a lab wiki). "Install the tools like before" does not.

Every change you made to the app (see [MAKE-IT-YOURS.md](MAKE-IT-YOURS.md)) has to come back too: new columns, new services, new routes, new images.

## How it runs in each section

### Local VM (Ubuntu, individual)

Your setup is a single-node k3s cluster managed by OpenTofu and your deploy script (see [LOCAL-VM.md](LOCAL-VM.md)). Much of the rebuild is already written down in the "Rebuild on a fresh VM" sections of the READMEs you wrote in Weeks 7, 8, and 9. Start from those, then add what the capstone changed.

**Before you start**

- **Check your laptop's disk.** You will have two VMs at once, and the new one needs as much space as the first. If your laptop does not have room, ask the TA before you delete anything.
- **Keep your original VM until the rebuild works.** You may need to compare the two. Shut it down while you work on the new one, so you always know which VM you are looking at.

**Week 13: build the new VM, in this order**

1. **The VM itself.** Create a new VM in VirtualBox with the same steps you used for your original VM: the same Ubuntu Server release, CPU, memory, and disk size, attached to NAT, with both port forwarding rules (SSH and App on 8080). If VirtualBox will not save a rule because the original VM already uses that host port, keep the original VM shut down, or give the new VM a different host port and write that down.
2. **Remote access.** Connect over SSH and VS Code Remote-SSH. The new VM has a new host key on the same address, so your laptop may refuse to connect until you remove the old entry with `ssh-keygen -R "[127.0.0.1]:2222"` (use your SSH port).
3. **Git and GitHub.** Install `git` and `gh`, run `gh auth login`, set `git config --global user.name` and `user.email`, then clone your repository.
4. **Core tools, in lab order.** Docker (Week 3), k3s and your `~/.kube/config` setup (Week 4), OpenTofu (Week 5), Helm (Week 6), cron (Week 7), and k6 (Week 10). Then your track's tools from the Tools Available table in your track README, including your Python virtualenv from `requirements.txt`. Your README must list each install step or link to the exact wiki part for it.
5. **The stack.** Follow your Week 7, 8, and 9 "Rebuild on a fresh VM" sections: create `week-5/terraform.tfvars` from the example file, build and import every image your `.tf` files point at (the `web` tag, plus any image you added in the capstone), run `./week-7/deploy.sh`, put the deploy cron line back, fix file modes, confirm the `protect-master` ruleset still exists on GitHub, recreate `~/backups`, restore a backup, and put the backup cron line back.
6. **Your track's work.** Everything under `capstone/<track>/` that has to run: data generation, training, new services, HPA and PDB, evidence scripts, pre-commit hooks.
7. **Proof of life** for your track (table below).

You have sudo on your own VM, so steps that need it are allowed. Every one of them must appear in your README.

Your GitHub settings (the ruleset, workflows, and repository secrets) are not affected by a new VM. Your old VM will also keep deploying from `master` whenever it is running, into its own cluster. That does no harm, but it is another reason to keep it shut down while you test.

**Week 14:** record a presentation of your rebuild.

### Shared VM (RHEL, teams)

- **Week 13:** Teams take turns in a queue to test their rebuild on a separate VM. The VM is wiped between teams so every team starts from the same clean state. Use your slot to find gaps; fix them afterward by pull request.
- **Week 14:** Live presentation. The tools are pre-installed on the presentation VM. Your team clones the repository live and brings the app up using your README and setup instructions.

**Rules for the Shared VM rebuild:**

- **No sudo.** The rebuild VM and the presentation VM work like the lab VM: every step in your README has to run as your normal user.
- **Ansible and [PREDICT ONLY] code stay reference-only.** Do not run your playbooks or `tofu` during the rebuild. Bringing the app up must not depend on them, and your README should label them clearly as reference-only.
- **Same cluster subnet.** Recreate your k3d cluster with the same `--subnet 100.97.<team>.0/24` flag (your team number) that you used during the semester, so pods can pull images and teams never overlap.
- **Bring a way to clone.** If your team repository is private, the rebuild VM needs a GitHub sign-in. Decide in advance who signs in (`gh auth login`) and run `gh auth logout` at the end of the slot.

## What lives outside git

These are the things students most often forget, because they existed on the old VM but were never committed. Check each one before your rebuild.

### Both sections

| Item | Why it is a gap | What to do |
|---|---|---|
| Git and GitHub credentials | A new VM is not signed in to GitHub | Note in the README how to sign in (`gh auth login`) before cloning |
| Tool installs | The new VM (Local) has none of your tools | List the install steps, or link to the exact wiki part and your track README's Tools Available section |
| Python virtualenv | Lives in your home directory, not the repo | `requirements.txt` committed; README shows how to create the venv from it |
| Prometheus and other helm releases | Installed by hand (Shared VM) | The helm commands and any values file in the repo. Local VM: `deploy.sh` installs Prometheus for you from `week-6/prometheus-values.yaml` |
| Docker images | Built and imported locally | Build and import commands in the RUNBOOK, for every image tag your manifests or `.tf` files use |
| Database rows and schema changes | Seeded data lives in a volume, and columns added by hand are not in git | Seed or generate command, migration or startup code for every schema change, and whether each is safe to run twice |
| Secrets and `.env` or `.tfvars` files | Correctly kept out of git | An example file with placeholder values and instructions for creating the real one |
| Git hooks (`pre-commit install`) | `.git/hooks` is not versioned | Install step in the README |
| GitHub settings (branch protection, rulesets, repo secrets) | Live in GitHub, not git | A record of the settings in your evidence or RUNBOOK |
| Prometheus history | Starts empty on a new cluster | Export what you need, or explain that it restarts |
| Port-forwards | Started in a terminal and never written down | The exact command and port in the RUNBOOK |

### Shared VM only

| Item | Why it is a gap | What to do |
|---|---|---|
| k3d cluster | Created by a command you ran weeks ago | Exact `k3d cluster create` command with every flag, including `--subnet 100.97.<team>.0/24`, in a script or the RUNBOOK |
| kubeconfig and kube context | Generated by k3d on the old VM | Note which context the commands expect |
| File permissions | `umask 077` makes edited files mode 600 | `COPY --chmod=644` in Dockerfiles, or a `chmod` step |

### Local VM only

| Item | Why it is a gap | What to do |
|---|---|---|
| VirtualBox settings | CPU, memory, disk, and port forwarding rules live in VirtualBox | Write them in your README |
| k3s and `~/.kube/config` | Installed and copied by hand in Week 4 | The install command and the kubeconfig steps, including `export KUBECONFIG=$HOME/.kube/config` |
| `week-5/terraform.tfvars` and the OpenTofu state | Live only on the VM | Created from `terraform.tfvars.example`; the first deploy on a new VM creates fresh state |
| Images inside k3s | k3s has its own image store, separate from Docker | `docker save` and `sudo k3s ctr images import` for every tag before the first deploy |
| Cron lines (deploy and backup) | `crontab` is not in git | The exact `crontab` commands from your Week 7 and Week 9 READMEs |
| `~/.deployed-commit`, `~/deploy.log` | Live in your home directory | Nothing to restore: a new VM deploys the current `master` on the first run |
| Backups in `~/backups` | Live on the VM, plus the copy on your laptop | Copy one in with `scp` and restore it, as in your Week 9 README |

## Proof of life

When your rebuild reaches the end of your README, run your track's proof of life. If these pass, the system is back.

| Track | Shared VM | Local VM |
|---|---|---|
| Networking and Cloud | Three nodes `Ready`; your HPA and PodDisruptionBudgets exist; `/health` returns 200 on your Kubernetes port; a 60-second k6 run completes | One node `Ready`; your HPA and PodDisruptionBudget exist; `/health` returns 200 on the `web` Service; a 60-second k6 run completes |
| Machine Learning and AI | The generator reproduces the same dataset from the seed (`sha256sum` matches); the model loads; one request returns a category, a severity, and a confidence | Same |
| Applied Data Science | One command regenerates every CSV and PNG; row counts match `data_quality.csv` | Same |
| Cybersecurity and Governance | The evidence script produces a new timestamped folder; the statuses in `controls.csv` still match what the new evidence shows | Same |
| DevSecOps and PM | `pre-commit run --all-files` exits 0; both annotated tags exist; the rollback command from your RUNBOOK works | Same, and `gh api repos/{owner}/{repo}/rulesets` still lists `protect-master` |

## The gap log

Every time the rebuild stops or needs a step your README did not mention, write it down in `capstone/<track>/REBUILD-GAPS.md`:

```markdown
# Rebuild Gap Log

Rebuild date: YYYY-MM-DD
VM: (new VirtualBox VM / Shared VM queue slot)
Started from commit: <git rev-parse --short HEAD>

| # | Step in README | What happened | Root cause | Fix (PR link) |
|---|---|---|---|---|
| 1 | Step 4: create cluster | `k3d cluster create` failed: port already allocated | README did not say to delete the old cluster first | #42 |
| 2 | | | | |

## What we would do differently
(two or three sentences)
```

A short honest log beats an empty one. Engineers who can say "here are the five things our docs were missing, and here is how we fixed each one" are the ones teams trust with production.

## Rebuild checklist

- [ ] Repo cloned into a fresh location on a clean VM
- [ ] Every step taken from the README or RUNBOOK, nothing from memory
- [ ] Every app change came back (schema, routes, new services, images)
- [ ] Gap log filled in as you go, not afterward
- [ ] Every gap fixed by pull request, and the README updated
- [ ] Proof of life passed
- [ ] Rehearsed once more from the fixed README before Week 14
- [ ] Local VM: original VM kept until this checklist is complete
