# Lab: Artifacts & Outputs — Pass a Build Between Jobs

## Goal

Build a two-job GitHub Actions workflow for the **Harmonic Haven** React app.
You'll compile the app in one job, upload the output as an artifact, then
download it in a second job and inspect it.

This teaches the core artifact pattern: jobs don't share a filesystem — artifacts
are how you move files from one job to another.

---

## Getting Started

```bash
npm install
npm run build      # compiles to dist/
npm run test       # runs tests
```

---

## What you have

A React TypeScript app (Vite). The key script for this lab:

```
npm run build  →  produces dist/
```

---

## Workflow shape

```
build  →  upload dist/  →  inspect (download + print files)
```

Two jobs. The second job can't run until the first finishes.

---

## Acceptance criteria

The workflow you write must:

- [ ] Have a `build` job that runs `npm run build` and uploads the `dist/` folder as an artifact named **`build-output`**
- [ ] Have an `inspect` job that depends on `build`, downloads the `build-output` artifact, and prints its contents to the log
- [ ] Trigger on `push` to any branch

---

## Hints

- Jobs don't share a filesystem — each job starts fresh. Use `actions/upload-artifact@v4`
  in `build` and `actions/download-artifact@v4` in `inspect`.
- The `inspect` job won't start until `build` finishes. Use `needs: build` to express
  that dependency.
- After downloading, run `ls -R` to print what landed on disk. That's your output.

---

## Where the workflow goes

Create your workflow at `.github/workflows/ci.yml`.
