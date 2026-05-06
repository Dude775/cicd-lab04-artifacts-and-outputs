# Lab: Artifacts & Outputs — Capture and Share CI Results

## Goal

Build a multi-job GitHub Actions workflow for the **Harmonic Haven** React app.
You'll learn how to produce build outputs and test coverage reports, upload them as
artifacts, and download them in a downstream job to generate a pipeline summary.

By the end of this lab you'll know how to pass data between jobs using
`actions/upload-artifact` and `actions/download-artifact` — a core skill for
real-world CI pipelines.

---

## Getting Started

This is a React TypeScript app built with Vite. To run it locally:

```bash
npm install
npm run dev        # start the dev server at http://localhost:5173
npm run build      # compile to dist/
npm run test       # run tests with coverage (produces coverage/)
npm run typecheck  # TypeScript type check
npm run lint       # ESLint
```

All four check commands (`build`, `test`, `typecheck`, `lint`) should pass with
zero errors before you start writing the workflow.

---

## What you have

```
harmonic-haven/
├── src/
│   ├── types.ts                    # Track and Playlist interfaces
│   ├── App.tsx                     # Root component — manages playlist state
│   ├── App.test.tsx                # App-level tests
│   └── components/
│       ├── PlaylistItem.tsx        # Renders a single track with a Remove button
│       └── PlaylistItem.test.tsx   # Component tests
├── index.html
├── package.json
├── vite.config.ts
└── tsconfig.json
```

---

## Workflow shape

Your workflow should have **5 jobs** arranged like this:

```
typecheck ──┐
            ├──→ build → upload-artifact: dist/
lint    ────┘

test ──────────→ upload-artifact: coverage/
                         │
                         ▼ (both build and test must finish first)
                      report → download both artifacts → print summary
```

---

## Acceptance criteria

The workflow you write must:

- [ ] Have a `typecheck` job that runs `npm run typecheck` on `ubuntu-latest` using Node 20
- [ ] Have a `lint` job that runs `npm run lint` on `ubuntu-latest` using Node 20
- [ ] Have a `build` job that depends on both `typecheck` and `lint`, runs `npm run build`, and uploads the `dist/` folder as an artifact named **`build-output`** with a retention of **7 days**
- [ ] Have a `test` job that runs `npm run test` and uploads the `coverage/` folder as an artifact named **`coverage-report`** with a retention of **7 days**
- [ ] Have a `report` job that depends on both `build` and `test`, downloads both artifacts, and prints a summary to the workflow log

Triggers: `push` (any branch), `pull_request`, `workflow_dispatch`.

---

## Hints

- Each job runs in a fresh VM — jobs don't share a filesystem. Every job that
  needs the source code must have its own `actions/checkout@v4` step, and every
  job that needs Node must have its own `actions/setup-node@v4` step.

- `actions/upload-artifact@v4` accepts three key inputs: `name` (the artifact
  label), `path` (what to upload), and `retention-days`. The `path` must match
  exactly what the script produces — `dist/` for `vite build` and `coverage/`
  for `vitest --coverage`.

- To make a job depend on multiple other jobs, `needs` accepts a list:
  ```yaml
  needs: [job-a, job-b]
  ```

- In the `report` job, use `actions/download-artifact@v4` once per artifact.
  Each artifact downloads into a folder named after it by default. Add a
  `run: ls -R` step to confirm the files landed where you expect.

---

## Where the workflow goes

Create your workflow file at:

```
.github/workflows/ci.yml
```
