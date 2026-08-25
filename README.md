# Upwork job monitor — routine state

This repo holds the persistent state for a Claude Code **routine** (a scheduled cloud agent,
see [claude.ai/code/routines](https://claude.ai/code/routines)) that checks my Upwork saved
search every hour, scores new postings, drafts a proposal opening, and sends me a Telegram
message. It never submits anything to Upwork — proposals are always sent by me, manually.

There's no app code here on purpose. The routine's logic lives in its saved prompt (configured
via the Claude Code routines UI/API), not in this repo. This repo exists only so the routine has
somewhere to read and write state between runs, since each run starts from a fresh clone.

## Files

- `seen_jobs.json` — job IDs already alerted on. The routine reads this at the start of each run,
  filters newly-fetched jobs against it, and commits the updated list back to `main` at the end
  of the run so the next run picks up where this one left off.
- `profile.md` — skills, rate, portfolio, and preferences used to score fit and draft proposals.
  Edit this directly to change what the routine looks for and how it writes.

## How it works

1. Routine fires hourly → clones this repo fresh from `main`
2. Reads `seen_jobs.json` and `profile.md`
3. Uses the Upwork MCP connector to check my saved search for new postings
4. Filters out anything already in `seen_jobs.json`
5. For each genuinely new job: scores fit (0-100) and drafts a proposal opening
6. Sends a Telegram message per new job
7. Appends the new job IDs to `seen_jobs.json` and commits directly to `main`
