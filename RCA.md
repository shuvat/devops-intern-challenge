# Root Cause Analysis

Fill this in as you go. Bullet points are fine — be specific, not long.

---

## Defect 1: Proxy image fails to build

### Symptom / error output
- `docker compose build` fails on the proxy: `apt-get install ... did not complete successfully: exit code: 100`
- the important error in the log: 
Configuration file ‘/etc/nginx/nginx.conf’
==> File on system created by you or by a script.
==> File also in package provided by package maintainer.
*** nginx.conf (Y/I/N/O/D/Z) [default=N] ? dpkg: error processing package nginx-common (--configure):
end of file on stdin at conffile prompt

### How I diagnosed it:
- Ran `./scripts/run-tests.sh`.
- Searched the log for `error`: the first dpkg error is the conffile prompt for `/etc/nginx/nginx.conf`.
- Checked `proxy/Dockerfile`: `COPY nginx.conf` runs **before** `apt-get install nginx`.

### Root cause:
- Layer order: `nginx.conf` was already at `/etc/nginx/nginx.conf` when the nginx package was installed.
- dpkg found a conflicting config file and stopped to ask which version to keep.
- `docker build` has no interactive stdin, so dpkg read EOF instead of an answer and failed. apt-get exited with 100, and the build failed.

### Fix
- Moved `COPY nginx.conf` **after** the `RUN apt-get install` layer: nginx installs cleanly with its default config, then the file overwrites it.
- Verified: the build log shows `[proxy 2/3] RUN apt-get ...` then `[proxy 3/3] COPY nginx.conf`, and the build finishes successfully.
- Side benefit: editing `nginx.conf` now only rebuilds the COPY layer, so the package install stays cached.

---

## Defect 2: Proxy cannot reach the backend

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix

---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix, and why I chose it over the alternatives

---

## Defect 4: CI pipeline fails

> Found first: it blocks running the suite locally too, not just in CI.
### Symptom / error output
- `./scripts/run-tests.sh` → `bash: ./scripts/run-tests.sh: Permission denied`
- CI runs the same command, so it fails the same way.

### How I diagnosed it
- `ls -l scripts/` → `-rw-r--r--`: no execute bit.
- `git ls-files -s scripts/` → `100644`: git stores the script as non-executable, so every fresh clone (including the CI runner) gets it that way.

### Root cause
- `run-tests.sh` was committed with mode `100644` instead of `100755`. Git tracks the execute bit, and checkout restores exactly what was committed.

### Fix
- `git update-index --chmod=+x scripts/run-tests.sh` → mode is now `100755` in the repo.
- Rejected alternatives:
  - `chmod +x` without committing: fixes only my machine.
  - `bash scripts/run-tests.sh` or a `chmod` step in `ci.yml`: CI goes green, but the next person cloning the repo still gets `Permission denied`.

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix

---

## Bonus questions

Optional. Answer only the ones you got to — number them, skip the rest.

>
1. Proxy image size (measured with `docker images`)
   - Before (ubuntu:24.04 + nginx curl htop vim git): 452MB on disk / 124MB compressed
---

## Anything you would do differently with more time
