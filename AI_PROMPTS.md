# Generative AI Disclosure

Using AI tools for this challenge is allowed. Be honest here — an accurate
"I used AI heavily" is worth more to us than an inaccurate "I did not".

**Did you use AI tools?** (yes / no): yes

**Which ones?** (e.g. ChatGPT, Claude, Copilot, Cursor): claud

> If you answered **no**, write "N/A" in sections 1 and 2 and skip to
> section 3. That is a perfectly good answer.

---

## 1. Key prompts

List the main prompts you used. Paraphrasing is fine.

1. **Setup:** "Before my first commit, walk me through the expected workflow and verify my environment." This led to fixing Docker Desktop on Windows (WSL2 integration, outdated Docker Desktop) and authenticating git via `gh`.
2. **Defect 4:** "What do `ls -l` and `git ls-files -s` show, and what's the difference?" After that I compared three fixes (local `chmod`, editing `ci.yml`, changing the mode in git) and asked which one survives a fresh clone.
3. **Defect 1:** "Help me understand this dpkg conffile error. Is the existing `nginx.conf` the problem?"
4. **Defect 2:** "Walk me through the test output line by line. Which part shows why the test fails?"
5. **Defect 2:** "Isn't `127.0.0.1` my own machine's address? Why can't the proxy reach the backend on it?"
6. **Defect 3:** "The proxy `depends_on` the backend and the test-runner `depends_on` the proxy. Why isn't that enough?"
7. **Docs:** "Turn my own notes into a concise RCA section." I wrote the analysis in my own words first, and used AI to tighten the wording.

---

## 2. Where the AI got it wrong

Describe **one concrete case** where the AI's answer was wrong, outdated, or
sub-optimal. Cover: what it suggested, how you noticed it was wrong, and what
you did instead. 3–5 sentences.

> ...
> After committing the file-mode fix for Defect 4, the AI told me to verify it with `git show --stat HEAD` and said I should see `mode change 100644 => 100755`. The actual output only showed `scripts/run-tests.sh | 0` with no mode line, so its claim didn't match what I saw. The reason: `--stat` only counts added and removed lines, and a mode-only change has zero of both, so it shows up as `| 0`. I verified with `git show --summary`, which printed `mode change 100644 => 100755 scripts/run-tests.sh`, and cross-checked with `git ls-files -s scripts/`, which reads the mode straight from the index. Takeaway: a "you should see X" claim is a hypothesis, so when the output disagrees, trust the output and find a command that checks the fact directly.
---

## 3. Anything in your submission you could not fully explain?

We would rather know. Write "nothing" if that is the case.

> ...
## 3. Anything in your submission you could not fully explain?

> I can explain every change in this PR: what was broken, why, and why I chose each fix over the alternatives. Within the 4-hour budget, though, I focused on understanding each specific problem rather than the wider topic behind it. Areas I'd like to study further:
> - **Docker networking:** how Compose's internal DNS and network namespaces work under the hood, and the difference between bridge, host and custom networks.
> - **Healthchecks and readiness:** liveness vs. readiness, and how the same "wait until ready" problem is handled in Kubernetes (readiness probes).
> - **Debian packaging:** how dpkg handles conffiles, and the `DEBIAN_FRONTEND=noninteractive` / `--force-confold` options that I chose not to use here.
