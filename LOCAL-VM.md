# Local VM Reference

This page is for the **Local VM (Ubuntu, individual)** section. Your labs (Weeks 3 to 10) built a different setup from the Shared VM section: one k3s node instead of a three-node k3d cluster, an app called `web` instead of Flask behind Nginx, and a cluster that changes through OpenTofu and your deploy script instead of `kubectl apply`. Read this page once before you start your track, and come back to it whenever a command in your track README mentions something your VM does not have.

Your own repository is the final word. If you changed something in an earlier lab (a file name, a tag, a port), use what your repo says and note the difference in your RUNBOOK.

## Your environment at a glance

| Topic | Shared VM (what most README text assumes) | Your Local VM |
|---|---|---|
| Cluster | Three-node k3d cluster per team | Single-node k3s, installed as a system service in Week 4. `kubectl get nodes` shows one node: your VM. |
| kubectl access | k3d context | `~/.kube/config` copied from k3s. If kubectl mentions `/etc/rancher/k3s/k3s.yaml`, run `export KUBECONFIG=$HOME/.kube/config`. |
| App objects | `flask`, `nginx`, and database Deployments | `web` (Flask, label `app=web`, container port 5000, Service `web` on port 8080) and `db` (PostgreSQL, label `app=db`, port 5432). There is no Nginx tier. |
| How the cluster changes | `kubectl apply` by the team | OpenTofu files in `week-5/`, applied by `week-7/deploy.sh` (cron, every two minutes) after the Deploy gate passes on `master`. See "The one rule" below. |
| Default branch and protection | `main` with branch protection, `build-and-push` required | `master` with the `protect-master` ruleset (`week-8/ruleset.json`). Required checks: `tofu fmt and validate`, `render the Prometheus chart`, `security scan`. Pull requests only, no bypass. |
| Workflows | `build-and-push` | `Deploy gate` (`.github/workflows/deploy-gate.yml`, three jobs), `OpenTofu checks` (Week 6, not required), and Week 10's `load-test.yml` (k6 on pull requests that touch the app, not required, has a `paths` filter). |
| Images | `<name>:team<N>`, `k3d image import` | Build, give it a new tag, then `docker save` and `sudo k3s ctr images import` (Weeks 4 and 10). Point the OpenTofu file at the new tag. |
| NetworkPolicy | Default-deny ingress plus allows | No default-deny. One policy, `db-clients` (Week 9), lets only `web` and `db-exporter` pods reach `db` on 5432. Any other pod accepts traffic from anywhere until you add a policy that selects it. |
| Database | `incidents` table, Compose stack in `week-2/` | `tickets` table (`id`, `title`, `description`, `status`, `created_at`) in the `db` pod. Run SQL with `./week-10/sql.sh`. Your Compose stack (`week-4/`) has been stopped since Week 4. |
| App routes | `/health`, JSON `/incidents` | `/health`, `/` (the HTML list), `POST /tickets`, `POST /tickets/<id>/status`, plus anything you changed in Week 10. There is no JSON list endpoint and no `/metrics` unless you add one. |
| Seed data | `seed.py` | Week 10's SQL insert of 10,000 `Seeded ticket` rows. There is no `seed.py` in your repo. |
| Prometheus | Port 9090 + team number, `week-5/pq.sh` | Helm release `prometheus` in `monitoring`, values in `week-6/prometheus-values.yaml`, kept up to date by `deploy.sh`. `kubectl port-forward --namespace monitoring service/prometheus-server 9090:80`, then `./week-6/pq.sh 'up'`. |
| Reaching the app | Ports 8080 + N, 8090 + N | From your laptop: `kubectl port-forward --address 0.0.0.0 service/web 8080:8080`, then `http://localhost:8080` (your VirtualBox rule). From inside the VM, scripts can use the Service address: `http://$(kubectl get service web -o jsonpath='{.spec.clusterIP}'):8080`, which is what `week-10/run-k6.sh` does. |
| Load testing | k6 against 8090 + N | k6 is installed (Week 10). Use `./week-10/run-k6.sh <test-file> <label>`. |
| Backups | Week 8 lab | `week-9/backup.sh` on cron into `~/backups`, restore with `week-9/restore.sh`. |
| Team number | Ports, tags, subnets | None. Wherever a track says `<N>`, `team<N>` or `8090 + N`, use the Local VM value from this page. Tag images with a version instead, such as `week-4-web:v3` or `ml-ai-infer:v1`. |
| sudo | Never | You have sudo on your own VM. Use it for package installs, `sudo install` of single binaries, and `sudo k3s ctr`. Do not use it for `kubectl`, `helm`, `tofu`, or `k6`. |
| Ansible | Committed, never run | Not part of this section. |
| metrics-server | Ships with k3s | Same. `kubectl top` works and a HorizontalPodAutoscaler can read CPU. |
| Firewall and `--network host` | Needed on campus | Not needed. Your VM uses VirtualBox NAT and containers can reach the internet. |
| `umask 077` and `COPY --chmod=644` | Needed on RHEL | Not needed unless you set `umask 077` yourself. Your `web` container runs as UID 10001, so files it reads must stay readable by other users. |
| Memory budgets | Shared by six teams | Still apply. Treat each track's budget as a design constraint: your VM is small, and Prometheus already uses a real share of it. Check `free -h` before and after you add anything. |

## The one rule: change the cluster through OpenTofu

Since Week 5, OpenTofu owns your `web` and `db` objects, and since Week 7 your deploy script applies `master` on its own. Week 10's CPU limit incident showed what happens when you change a managed object by hand: OpenTofu puts it back on the next plan.

- **Anything that should last** (an HPA, a PodDisruptionBudget, a NetworkPolicy, a new Deployment and Service, a changed image tag) goes in a `.tf` file under `week-5/`, through a pull request, the Deploy gate, and the deploy script. The hashicorp/kubernetes provider has resources for all of these, for example `kubernetes_horizontal_pod_autoscaler_v2`, `kubernetes_pod_disruption_budget_v1`, `kubernetes_network_policy_v1`, `kubernetes_deployment_v1`, and `kubernetes_service_v1`. Check the provider documentation for the version in your lock file.
- **Throwaway test pods** (a DNS debug pod, a blocked client) can be applied with `kubectl apply -f` and must be deleted afterward.
- **The gate checks what you add.** The `security scan` job runs Checkov on `week-5/` with rules that require CPU and memory requests and limits, no privileged containers, and no privilege escalation. A new container without them turns the gate red, and the ruleset will not let you merge.
- **The deploy script stops at deletions.** If a plan would delete or replace something, `deploy.sh` logs `BLOCKED` and waits for you. Read the plan with `cd week-5 && tofu plan`, and run `tofu apply` there yourself only if the deletion is intended.
- **Plan freely, apply through the pipeline.** Run `tofu plan` on your branch as often as you like. Do not `tofu apply` from a branch: the state on your VM would stop matching `master`.
- **Waiting for cron is optional.** After the gate passes on `master`, you may run `./week-7/deploy.sh` by hand instead of waiting up to two minutes. Use this when you are timing something.

## Same topics, different week numbers

When a track README names a week, it means the Shared VM week. Use this table to find your lab.

| Topic | Shared VM week | Local VM week |
|---|---|---|
| Git and repository setup | 1 | 3 (with your first container) |
| Compose stack (app and database) | 2 | 4 |
| First Kubernetes cluster | 3 (k3d) | 4 (k3s and kompose) |
| Infrastructure as code with OpenTofu | 4 | 5 |
| Prometheus, Helm, and `pq.sh` | 5 | 6 |
| GitHub Actions CI | 6 | 6 (checks) and 7 (deploy gate and continuous delivery) |
| Security hardening, NetworkPolicy, scanning, merge rules | 7 | 8 |
| Data observability and backup | 8 | 9 |
| End-to-end performance and k6 | 9 | 10 |

## Where things live in your repo

| Path | What it holds |
|---|---|
| `week-4/` | The app (`week-4/app/`) and `docker-compose.yml`. Build a new `web` image with `docker compose -f week-4/docker-compose.yml build web`. |
| `week-5/` | Every OpenTofu file for the cluster. `terraform.tfvars` and the state file live only on your VM, never in git. |
| `week-6/` | `pq.sh` and `prometheus-values.yaml` |
| `week-7/` | `deploy.sh` |
| `week-8/` | `ruleset.json` |
| `week-9/` | `backup.sh` and `restore.sh` |
| `week-10/` | `sql.sh`, `run-k6.sh`, `k6-stat.sh`, `throttle.sh`, and your k6 test files |
| `capstone/<track>/` | Everything you add in Weeks 11 to 14 |

## Translating common commands

| Track README says | On your Local VM |
|---|---|
| `k3d cluster list`, `docker ps --filter name=k3d-...` | `kubectl get nodes -o wide` |
| `k3d image import <image>:team<N> -c <cluster>` | `docker save <image>:<tag> -o /tmp/img.tar && sudo k3s ctr images import /tmp/img.tar && rm /tmp/img.tar`, then check with `sudo k3s crictl images` |
| `docker build --network host -t <image>:team<N> <dir>` | `docker build -t <image>:<tag> <dir>` |
| `kubectl apply -f <manifest>` for something lasting | Add it to `week-5/*.tf`, open a pull request, merge after the gate passes |
| `curl http://localhost:$((8090 + N))/health` | `curl -s http://$(kubectl get service web -o jsonpath='{.spec.clusterIP}'):8080/health` |
| `curl http://localhost:$((8080 + N))/health` (Compose) | The same Service address as above. Compose is stopped. |
| `docker compose -f week-2/docker-compose.yml exec -T db psql ...` | `./week-10/sql.sh 'SELECT ...'`, or for CSV: `kubectl exec -i deploy/db -- sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" --csv -c "SELECT ..."' > out.csv` |
| `./week-5/pq.sh` with `PROM_URL=http://localhost:$((9090 + N))` | `./week-6/pq.sh` (it defaults to `http://localhost:9090`) |
| `gh api repos/<owner>/<repo>/branches/main/protection` | `gh api repos/{owner}/{repo}/rulesets` and `gh api repos/{owner}/{repo}/rules/branches/master` |
| `git push` to `main` is blocked | Same rule on `master`: branch, pull request, gate, merge |
| "write it as [PREDICT ONLY]" | Not needed. Your OpenTofu runs for real through the deploy script. |

## If you run out of disk

Week 9's wiki has a full "If Your VM Runs Out of Disk Space" section. Images add up quickly in this capstone, because every new tag is a new copy in both Docker and k3s. Remove tags you no longer deploy with `docker image rm` and `sudo k3s crictl rmi`, and never remove the tag your `web.tf` points at.
