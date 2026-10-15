# Root Cause Analysis

Fill this in as you go. Bullet points are fine — be specific, not long.

---

## Defect 1: Proxy image fails to build

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix

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

---

## Anything you would do differently with more time
