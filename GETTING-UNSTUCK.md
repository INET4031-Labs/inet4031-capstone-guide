# Getting Unstuck

## A routine for any problem

1. **Read the whole error.** The useful line is often the last one, or the first line after "caused by."
2. **Check you are where you think you are.** Which VM, which directory, which kube context (`kubectl config current-context`), which port, which branch.
3. **Look at the object, then its events, then its logs.**
   ```bash
   kubectl get pods -o wide
   kubectl describe pod <name>        # Events at the bottom
   kubectl logs <name>                # add --previous after a crash
   ```
4. **Change one thing at a time** and re-run the same check.
5. **Write down what you tried** in your NOTES file. You will need it when you ask for help, and again in your retrospective.

## When to ask

Give a problem about 30 minutes of honest effort (steps above, documentation, the table below). If you are still stuck, ask. Waiting two days to ask costs more than the question.

On the Shared VM, anything that needs `sudo`, a missing tool, or a VM-wide problem goes to the TA or professor. Do not work around it.

When you ask, include: the command, the full output, what you expected, and what you already tried.

## Common symptoms in this course

| Symptom | Usual cause | Check |
|---|---|---|
| Request hangs with no error, then times out | Default-deny NetworkPolicy has no allow for the new pod | `kubectl get networkpolicy`; does an allow select your pod's labels and port? |
| Pod stuck in `Pending` | Requests too large for the node, or scheduling rules cannot be met | `kubectl describe pod` Events |
| `ErrImagePull` or `ImagePullBackOff` on your own image | Image was not imported into your cluster, or the tag differs | Shared VM: `k3d image import <image>:<tag> -c <your cluster>`. Local VM: `sudo k3s crictl images`, then `docker save` and `sudo k3s ctr images import`. Compare the tag in the manifest or `.tf` file. |
| `CrashLoopBackOff` | The app starts and exits | `kubectl logs <pod> --previous` |
| `Permission denied` inside a container | `umask 077` made files mode 600 and the container runs as non-root | `COPY --chmod=644` or `chmod 644` before building |
| `docker build` cannot download packages | Containers on the campus network cannot reach the internet | Build with `docker build --network host` |
| `address already in use` on a port-forward | A stale port-forward from an earlier session | `pkill -u $USER -f "kubectl port-forward"` |
| kubectl shows someone else's resources, or none of yours | Wrong kube context | `kubectl config current-context` |
| HPA target shows `<unknown>` | No CPU request on the pods, or metrics not ready yet | `kubectl describe hpa`; `kubectl top pods` |
| `pip` compiles for minutes | A package has no prebuilt wheel for your Python | Stop and ask the TA |
| Required check never reports on a PR | Job renamed, or a path filter skipped the workflow | Compare the required check name to the job name in the PR's checks list |
| Local VM: `kubectl` errors about `/etc/rancher/k3s/k3s.yaml` | `KUBECONFIG` is not set in this terminal | `export KUBECONFIG=$HOME/.kube/config` |
| Local VM: a change you made with `kubectl` disappeared | OpenTofu manages that object and the next deploy put it back | Make the change in `week-5/*.tf` and ship it through a pull request ([LOCAL-VM.md](LOCAL-VM.md)) |
| Local VM: merged, but nothing deployed | The gate failed for that commit, or `deploy.sh` stopped at a deletion | `tail -20 ~/deploy.log`; look for `BLOCKED` or `FAILED` |
| Local VM: `security scan` is red on your pull request | A container in `week-5/` is missing CPU or memory requests or limits, or allows privilege escalation | Read the Checkov output in the job log; add the missing settings |

## Section-specific help

- **Shared VM (RHEL, teams):** only the professor and TA have sudo. Escalate missing tools, permission problems that need sudo, and anything that affects the whole VM to them. Never work around it.
- **Local VM (Ubuntu, individual):** you own your VM and have sudo, but ask before making system-wide changes the track README does not mention. They become rebuild gaps later. Your setup differs from the Shared VM text in several places; check [LOCAL-VM.md](LOCAL-VM.md) before assuming a command is broken.
