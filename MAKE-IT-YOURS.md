# Make It Yours

The incident tracker you have run since the early weeks is now yours to change. You can add columns, routes, pages, services, and features, as long as the platform underneath keeps its promises. The success criteria in your track README are the floor. How you meet them, and what you build into the app along the way, is where your own design shows.

Read this page in Week 11, before you plan your board.

## What you may change

Almost everything in the application is fair game:

- **Routes and pages.** New API endpoints, a JSON version of a page, a search or filter, a ticket detail view, a dashboard page.
- **The data model.** New columns (category, severity, assignee, `resolved_at`), new tables (comments, audit log, users), indexes.
- **The generator and seed data.** Write your own generator, give it realistic patterns, change what it produces.
- **New small services.** A second container that does one job (a classifier, a summary endpoint, an exporter), with its own Service and NetworkPolicy.
- **Behavior.** Validation, authentication, rate limiting, caching, a `/ready` endpoint that differs from `/health`, a `/metrics` endpoint.
- **The UI.** Templates and styles, if your section's app has them.

## What must keep working (the platform contract)

These are the promises other people, scripts, and the Week 14 rebuild depend on. Change anything you like around them, not through them.

### Both sections

- `/health` returns HTTP 200 when the app is up. Probes, rebuild checks, and several tracks depend on it.
- Every pod has memory requests and limits, and your additions stay inside your track's memory budget.
- Secrets stay out of git. New settings get a placeholder in the example file (`.env.example` or `terraform.tfvars.example`) and real values only on the VM.
- All data is synthetic. No real names, emails, or customer details.
- Every change reaches the default branch through a pull request.
- **Everything you change must come back in the rebuild.** If the rebuild cannot recreate it from your README, it does not count.

### Shared VM (RHEL, teams)

- Your ports stay 8080 + N, 8090 + N, and 9090 + N. New services use the ports your track README assigns.
- Images keep the `:team<N>` tag and are imported with `-c <your cluster name>`.
- The default-deny NetworkPolicy stays. A new pod gets its own allow policy.
- `build-and-push` stays required on `main`.
- No sudo, no Ansible or `tofu` runs, and nothing that touches another team's containers, ports, or images.

### Local VM (Ubuntu, individual)

- OpenTofu in `week-5/` stays the way lasting changes reach the cluster, through the Deploy gate and `week-7/deploy.sh`. See [LOCAL-VM.md](LOCAL-VM.md).
- The `protect-master` ruleset and its three required checks stay in place. Do not delete it or add a bypass to get your own work merged.
- Your Week 9 backup and restore keep working. If you change the schema, run a backup and a restore against the new schema and save the output.

## Changing the database safely

Schema changes are where most rebuilds break. Follow these rules:

1. **Make every change safe to run twice.** Use `ALTER TABLE ... ADD COLUMN IF NOT EXISTS` and `CREATE TABLE IF NOT EXISTS`. The app already creates its table at startup this way, so the startup code is a natural place for small additions.
2. **Commit every change as a script or as app startup code.** A column you added by hand in `psql` is a rebuild gap.
3. **Give new columns a default or allow nulls**, so rows that already exist keep working.
4. **Write down which data your track relies on.** If you add a `category` column, your generator, analysis, or model has to agree with it.

## Ideas by track

These are starting points, not requirements. Each one connects an app change to your track's business problem.

| Track | App change that fits the business problem |
|---|---|
| Networking and Cloud | Add a `/ready` endpoint that checks the database, and use it for the readiness probe so a pod that lost its database leaves the Service. Or add a heavier read endpoint (search, report) and see how it changes your scaling numbers. |
| Machine Learning and AI | Add `category` and `severity` columns, have the app call your inference service when a ticket or incident is created, and show the suggestion, its confidence, and an "advisory only" label next to it. |
| Applied Data Science | Make the app record `resolved_at` when a ticket or incident is closed, so resolution times come from the app instead of only from a generator. Add a small JSON summary endpoint that your pipeline or Dana's team can read. |
| Cybersecurity and Governance | Add an audit log table that records who changed what and when, or add simple authentication to the write routes. Then make it a control in your matrix and test it. |
| DevSecOps and IT PM | Ship real features as your releases. If `v1.1.0` adds a column, your rollback rehearsal has to show what happens to data written after the upgrade, which is the hard part of rolling back in real jobs. |

## Swapping a success criterion

Sometimes a better idea does not fit a criterion as written. You may propose replacing one success criterion with one of your own.

- Write `capstone/<track>/DESIGN-BRIEF.md` (template below) and bring it to the TA or professor **by the end of Week 11**. Nothing is swapped until they agree.
- Your replacement must be at least as measurable as the one it replaces, and it needs an evidence file just like the others.
- Creative work replaces work. It does not stack on top of all five criteria. The Week 12 feature freeze in [WEEKLY-PLAN.md](WEEKLY-PLAN.md) still applies.
- Usually one criterion is swapped. Ask if you want to change more.

```markdown
# Design Brief

Track: <track>
Section: Shared VM (team <N>) / Local VM
Date:

## What we want to change in the app
(two to four sentences)

## Which business problem it serves
(name the person from the Harborlight cast and what it fixes for them)

## Criterion we propose to replace (if any)
Original: (copy the criterion text)
Replacement: (your criterion, written so a stranger could check it)
Evidence file: capstone/<track>/evidence/<name>

## What could break
(schema, other tracks' assumptions, rebuild, memory budget, and how you will check)

## Approved by / date
```

## Shipping an app change

### Shared VM

1. Change the code on a branch.
2. Build with `docker build --network host -t <image>:team<N> <dir>` and import with `k3d image import <image>:team<N> -c <your cluster name>`.
3. Roll it out with `kubectl` (for example `kubectl rollout restart deploy/<name>`) and check `kubectl rollout status`.
4. Open a pull request, wait for `build-and-push`, merge.
5. Add the build, import, and rollout commands to your RUNBOOK.

### Local VM

1. Change the code on a branch.
2. Build a **new tag** (never overwrite the one that is running): `docker compose -f week-4/docker-compose.yml build web`, then `docker tag week-4-web:latest week-4-web:v3`.
3. Import it into k3s **before** you open the pull request: `docker save week-4-web:v3 -o /tmp/web.tar && sudo k3s ctr images import /tmp/web.tar`.
4. Point `week-5/web.tf` at the new tag and run `tofu plan` to see the change.
5. Open a pull request, let the Deploy gate pass, merge, and let `deploy.sh` roll it out.
6. Add every step to your RUNBOOK. Week 10 Part 5 walks through the same pattern.

## Write it down

Add a section called **"What we changed and why"** to your `ARCHITECTURE.md`. List each app change, the problem it serves, and the evidence file that shows it works. This is how instructors and future employers see the decisions you made.
