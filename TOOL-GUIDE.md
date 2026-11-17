# Tool Guide

One line per tool: the question it answers, a safe first command, and the thing that usually trips people up. Install and verify steps are in your track README. Flags change between versions, so when a command here fails, run the tool's `--help` on the version you installed.

Replace `<N>` with your team number, `<cluster>` with your cluster name, and paths with your own.

## Cluster and containers (all tracks)

| Tool | Question it answers | First command to try | Common trip-up |
|---|---|---|---|
| kubectl | What is running in my cluster, and why? | `kubectl get pods -o wide` then `kubectl describe pod <name>` | Wrong context. Check `kubectl config current-context`. |
| k3d | Which clusters exist, and how do I get my image into one? | `k3d cluster list` | Forgetting `-c <cluster>` on `k3d image import`, so the image goes nowhere useful. |
| docker | Which containers and images exist on this host? | `docker ps --filter name=k3d-<cluster>-` | The daemon is shared on the Shared VM. Only touch containers and tags with your name or team number. |
| helm | What did the Prometheus chart install, with which values? | `helm list -A` | Reinstalling over a working release by accident. Read before you change. |
| curl | Does this endpoint answer, and how fast? | `curl -s -o /dev/null -w "%{http_code} %{time_total}\n" http://localhost:<port>/health` | Hitting another team's port. Check the number. |
| jq | How do I pull one field out of JSON? | `kubectl get pods -o json \| jq '.items[].metadata.name'` | Forgetting `-r` when you want raw strings without quotes. |

## Networking and Cloud Infrastructure

| Tool | Question it answers | First command to try | Common trip-up |
|---|---|---|---|
| k6 | How does the app behave under a load I control? | `k6 run --vus 5 --duration 30s script.js` | Starting with high load. Ramp gently; six teams share the CPU. |
| kubectl top | How much CPU and memory are pods using now? | `kubectl top pods` | It needs metrics-server. If it errors, tell the TA. |
| HPA via kubectl | Is my autoscaler seeing a metric and acting on it? | `kubectl get hpa` and `kubectl describe hpa <name>` | `<unknown>` targets mean no CPU request on the pods. |
| dig | What does this name resolve to? | `dig +short <name>` from the VM | Cluster names like `flask.default.svc.cluster.local` only resolve *inside* the cluster. Use a debug pod. |
| OpenTofu | What would this infrastructure definition create? | Local VM: `tofu init` then `tofu plan` | Shared VM: write the code only. Never run `tofu`. |

## Machine Learning and AI

| Tool | Question it answers | First command to try | Common trip-up |
|---|---|---|---|
| scikit-learn | How do I train and evaluate a small model? | `from sklearn.dummy import DummyClassifier` for your baseline first | Evaluating on training data. Split first. |
| joblib | How do I save a trained model to a file? | `joblib.dump(model, "model.joblib")` | Loading with a different scikit-learn version than the one that saved it. |
| Flask | How do I put the model behind an HTTP endpoint? | A single `/predict` route that loads the model once at startup | Loading the model on every request, which ruins your timing. |
| numpy / scipy | Numeric support for scikit-learn | Installed with scikit-learn | Nothing to configure. |

## Applied Data Science

| Tool | Question it answers | First command to try | Common trip-up |
|---|---|---|---|
| psql | What is in the database? | `docker compose -f week-2/docker-compose.yml exec -T db psql -U appuser -d statustracker --csv -c "SELECT COUNT(*) FROM incidents;"` | Using `\copy` inside the container writes the file inside the container. Redirect `--csv` output on the host instead. |
| pandas | How do I group, resample, and summarize this table? | `df = pd.read_csv("file.csv", parse_dates=["created_at"])` | Forgetting `parse_dates`, so time grouping fails. |
| matplotlib | How do I save a labeled chart? | `matplotlib.use("Agg")` before `import matplotlib.pyplot` | No title, axis labels, or units. All three are required. |
| scipy.stats | Is this pattern real or noise? | `scipy.stats.linregress(x, y)` for a trend | Reporting a p-value without saying in plain English what it means. |
| csvkit | What is in this CSV at a glance? | `csvstat file.csv` | Large files are slow. Aggregate first. |
| DuckDB CLI | Can I run SQL directly on a CSV? | `duckdb -c "SELECT COUNT(*) FROM 'file.csv'"` | None worth noting. It is a good way to check pandas results. |

## Cybersecurity and Governance

| Tool | Question it answers | First command to try | Common trip-up |
|---|---|---|---|
| Trivy | Which known vulnerabilities and misconfigurations are in my image and files? | `trivy image --severity HIGH,CRITICAL <image>:<tag>` | Running it inside a container on campus, where it cannot download its database. Run on the host. |
| gitleaks | Has a secret ever been committed, anywhere in history? | `gitleaks --help` first; subcommands differ between versions | Pasting a real secret into the report. Use the redact option. |
| kube-score | Do my manifests follow good practice? | `kube-score score <manifest>.yaml` | Treating every suggestion as a failure. Triage. |
| checkov | Do my Dockerfiles, YAML, workflows, and OpenTofu pass policy checks? | `checkov -d <dir>` | Large output. Use `--compact` and save to a file. |
| kubectl auth can-i | Can this identity do this action? | `kubectl auth can-i get secrets --as=system:serviceaccount:default:flask-app` | Forgetting `--as`, which checks *your* permissions instead. |
| Lynis (Local VM only) | How hardened is this host? | `lynis audit system --quick` as your normal user | Never on the Shared VM. |

## DevSecOps and IT Project Management

| Tool | Question it answers | First command to try | Common trip-up |
|---|---|---|---|
| gh | What happened on this PR, and what are the branch rules? | `gh pr checks <number>` | Reading branch protection needs admin rights. Ask an admin teammate or the TA. |
| pre-commit | Does this commit pass the cheap checks on my machine? | `pre-commit run --all-files` | Every fresh clone needs `pre-commit install`. |
| ShellCheck | Does this shell script have bugs? | `shellcheck script.sh` | Ignoring warnings about unquoted variables. They are usually real. |
| Hadolint | Does this Dockerfile follow good practice? | `hadolint Dockerfile` | Suppressing rules without writing down why. |
| yamllint | Is this YAML valid and consistent? | `yamllint .github/workflows/` | Strict line-length rules on generated files. Configure, do not ignore. |
| git-cliff | What changed between two releases? | `git-cliff --latest` | Flat output when commit messages do not follow a convention. |
| git tag | How do I mark a release? | `git tag -a v1.0.0 -m "First release"` then `git push origin v1.0.0` | Tags are not pushed by a plain `git push`. |
