# Using AI in the Capstone

AI assistants are allowed and encouraged as a learning tool. Relying on them to do the work is strongly discouraged.

## Good uses

- **Understanding a tool.** "What does a PodDisruptionBudget protect against, and what does it not protect against?"
- **Learning how enterprises use it.** "How do platform teams usually roll out a new required CI check without blocking merges?"
- **Reading an error.** Paste the error, the command you ran, and what you already tried, then ask what the error means.
- **Checking your understanding.** Explain your design in your own words and ask what you missed.
- **Finding the right documentation page** so you can read the source yourself.

## Poor uses

- Generating your manifests, scripts, report, or risk register and committing them without understanding each line.
- Pasting AI output into evidence files. Evidence must be real output from your own runs.
- Asking it to make the decisions the track asks you to make (your data decision, which risk to accept, which checks to require).

## Ground rules

1. **Verify before you apply.** AI answers are often confidently wrong about flags, versions, and defaults. Check against the tool's documentation or `--help` on the version you have installed.
2. **Understand every line you commit.** If you cannot explain it in your presentation, it does not belong in your repo.
3. **Never paste secrets, tokens, or credentials** into an AI tool.
4. **Stay inside the rules.** An AI suggestion to use `sudo`, disable firewalld, or edit `/etc/docker/daemon.json` is still against the Rules of the Road.

## A useful prompt shape

```
I am working on <track> for a course. I ran:
<command>
and got:
<output>
I expected <what you expected> because <why>.
I already tried <what you tried>.
Explain what this error means and what I should check next. Do not write the fix for me.
```

The last line keeps you doing the work while still getting unstuck.
