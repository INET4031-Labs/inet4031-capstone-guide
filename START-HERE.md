# Start Here

Read this on the first day of your track. It takes about ten minutes.

## Reading order

1. **Your track README: Overview and The Business Problem.** Read these twice. Then write, in two or three sentences of your own, who is asking, what hurts today, and what "done" means. If you cannot write that, reread before you touch a terminal.
2. **Your track README: Success criteria.** These are the contract. Every criterion needs a file in your repo that proves it. Open [EVIDENCE-CHECKLIST.md](EVIDENCE-CHECKLIST.md) and find your track's table.
3. **Your track README: Rules of the Road.** These rules exist because the environment is shared or because something broke in an earlier semester. Most "mystery" failures trace back to one of them.
4. **Your track README: Tools Available.** You do not need to master every tool. Look each one up in [TOOL-GUIDE.md](TOOL-GUIDE.md) and read the one-line "question it answers."
5. **Your track README: Example Approaches.** Pick one to start from, or combine them. You can change direction later.
6. **[MAKE-IT-YOURS.md](MAKE-IT-YOURS.md).** The app is yours to change. Read what you may change, what must keep working, and how to propose your own criterion by the end of Week 11.
7. **[WEEKLY-PLAN.md](WEEKLY-PLAN.md).** Find the Week 11 row for your track.

**Local VM students:** read [LOCAL-VM.md](LOCAL-VM.md) before step 1. Your cluster, app names, branch, and week numbers differ from the Shared VM text.

## Your first working session

1. Run the **Environment Checkpoint** from your track README. Save the output to a file and commit it. If anything fails, fix that first; see [GETTING-UNSTUCK.md](GETTING-UNSTUCK.md).
2. Run the **Verify** command for every tool in your track's table and save the output to `TOOL-VERSIONS.txt`.
3. Turn each success criterion into a GitHub issue on your project board. Split big ones.
4. Start a file called `capstone/<your track>/NOTES.md` (or a section in your RUNBOOK) and paste every command that worked, in order. This file becomes your RUNBOOK and your rebuild script later. Teams that skip this step struggle most in Week 13.
5. Pick the smallest piece of work that produces evidence (a baseline measurement, a first scan, a first dataset) and do it today.

## Habits that pay off in Week 13

- **If you typed it, write it down.** Any command you ran by hand that changed the system belongs in your RUNBOOK or a script in git.
- **Save output to files, not memory.** `command | tee capstone/<track>/evidence/<name>.txt` keeps the output on screen and in a file.
- **Commit small and often** through pull requests (on `main` for the Shared VM, on `master` for the Local VM).
- **Ask "could a classmate run this from my README?"** at the end of every session.

## What "comfortable with being uncomfortable" means here

You will read documentation for tools you have never used, hit errors you have never seen, and make design decisions without a right answer in the back of the book. That is the skill this capstone builds. Expect the first two or three days to feel slow. Progress usually arrives all at once after the environment is healthy and the first piece of evidence exists.
