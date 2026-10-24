# 🩺 Repo Doctor

**Your repos' on-call doctor.** It makes a house call on every GitHub repo you own, diagnoses what's broken,
writes the fix with free AI, ships it only if the tests pass, and texts you a plain-English report before
your coffee's ready.

No dashboards. No yak-shaving. You wake up to:

```
🩺 Repo Doctor · 2026-10-24
9 repos checked: 6 healthy, 2 fixed, 1 needs you

✅ FIXED
• my-api: The test step used a Node version the lockfile doesn't support.
  What I changed: pinned setup-node to 20 in ci.yml.

⚠️ NEEDS ATTENTION
• my-bot: The deploy job fails because the API key secret expired.
  → Renew the key and update the repo secret. (No change made: not a code problem.)
```

---

## What it does

- **Examines** – failing GitHub Actions runs (last 30 days), broken workflow YAML, invalid JSON, Python syntax,
  unresolvable `requirements.txt`, `pytest`, and `npm install / build / lint / test`.
- **Diagnoses** – pulls the failing log lines and asks a free AI model for the *root cause*, in words a
  non-coder understands.
- **Operates** – the smallest possible patch, on its own `doctor/<date>-<what>` branch, with a pull request that
  explains what was wrong and what changed.
- **Checks vitals** – waits for your repo's own CI (or re-runs its local checks) and **squash-merges only when
  green**. Red? It tries once more, then leaves the PR open and tells you.
- **Discharges** – re-runs the workflow that was failing (when that's safe) and confirms the fix.
- **Reports** – one Telegram message plus a Markdown report you can keep in any repo.
- **Knows when not to operate** – expired keys, quotas, billing, an outside service being down: it explains what
  *you* need to do instead of "fixing" code that isn't broken.

## Safety rails (non-negotiable)

- Never force-pushes, never rewrites history, never deletes anything but its own merged `doctor/*` branches.
- Never touches repo settings, visibility or secrets.
- Never sends `.env`, keys or credentials to an AI; every prompt, log and report is scrubbed for secrets, and a
  fix that *looks* like it contains one is thrown away.
- Protected paths (`never_edit`), temporary holds (`hold_paths`) and **report-only repos** you choose.
- Won't re-run workflows that deploy, publish or post (`never_rerun`).
- Hard caps: 10 files / 300 lines per fix, 2 attempts, an AI-call budget and a time budget per run.
- Skips archived repos and forks; quick-looks repos with no changes and nothing failing to save Actions minutes.
- Kill switch: create an `OFF` file or set `DOCTOR_OFF=true`.

## Runs on free AI

Bring any one (or several – it falls through on rate limits):

| Variable | Provider |
|---|---|
| `GEMINI_API_KEY` (+ `DOCTOR_GEMINI_MODEL`) | Google AI Studio |
| `GROQ_API_KEY` | Groq |
| `CEREBRAS_API_KEY` | Cerebras |
| `MISTRAL_API_KEY` | Mistral |
| `OPENROUTER_API_KEY` | OpenRouter free models |
| `LLM_ROUTER_URL` + `LLM_ROUTER_KEY` (+ `LLM_ROUTER_MODELS`, comma list) | any OpenAI-compatible endpoint you run |

Unwanted provider? List it in `blocked_providers`.

## Set up in 5 minutes

1. Create a fine-grained GitHub token with read/write on **Contents, Pull requests, Actions, Workflows** for the
   repos you want looked after. Save it as secret `DOCTOR_GH_TOKEN` in a repo of yours (a private "ops" repo is ideal).
2. Add at least one free AI key as a secret (table above). Optional: `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID`.
3. Copy `config.example.json` to `config.json` in that repo and set your `owner` and rules.
4. Copy [`examples/repo-doctor.yml`](examples/repo-doctor.yml) to `.github/workflows/`. It pulls the engine from this
   repo at a pinned release, so your config, reports and secrets never leave your own repo.
5. Run it once by hand with `dry_run: true` and read the report. Happy? Let the schedule take over.

Run locally:

```bash
pip install -r requirements.txt
export DOCTOR_GH_TOKEN=... GEMINI_API_KEY=...
cp config.example.json config.json   # edit it
DRY_RUN=true python repo_doctor.py
```

### All settings

| Env var | Default | Meaning |
|---|---|---|
| `DOCTOR_CONFIG` | `./config.json` | config file |
| `DOCTOR_REPORTS_DIR` | `./reports` | where daily Markdown reports go |
| `DOCTOR_STATE_FILE` | `./last_run.json` | machine-readable summary of the last run |
| `DOCTOR_OFF_FILE` | `./OFF` | kill-switch file |
| `DOCTOR_OWNER` | config `owner` / token login | user or org to check |
| `DOCTOR_TZ` | config `timezone` | timezone used for report dates |
| `ONLY_REPOS` | all | comma list of repos to check this run |
| `DRY_RUN` | `false` | diagnose and report only |

## License

MIT © 2026 Swanix
