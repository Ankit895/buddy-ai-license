# Buddy AI — License Registry

This repository is the activation backend for [Buddy AI](https://github.com/Ankit895/student-id-app).
It uses GitHub Actions as a free serverless function to validate and track license keys.

## How it works

1. The Buddy AI app calls `repository_dispatch` with a key hash + device ID.
2. The `activate.yml` workflow runs, reads `keys.json`, and:
   - Rejects if the key is unknown
   - Rejects if the key is already bound to a different device
   - Accepts (re-activation) if the key is bound to the same device
   - Accepts (fresh) if the key is unused — and binds it
3. The workflow writes the result to `results/<key_hash>.json` and commits.
4. The app polls the result file via `raw.githubusercontent.com` and proceeds.

## Files

- **keys.json** — registry of all 10 valid keys (hashes only — never the plain keys), plus the activation log
- **results/** — per-activation result files written by the workflow
- **.github/workflows/activate.yml** — the validation workflow

## Distributing a key

Look at your private plain-text key list (the file Claude generated alongside this repo).
Pick a row, write the recipient name in `keys.json` next to the matching hash, commit.
Send the plain key to that user via WhatsApp/email.

## Releasing a key (when someone changes phone)

1. Open `keys.json`
2. Find the matching entry in `activations` and remove it
3. Find the matching `keys[i]` and set `status` back to `available`
4. Commit
5. Have the user re-activate on their new phone

## Security

- Plain keys are never stored here — only SHA-256 hashes
- The app's GitHub PAT only has `Actions: Write` scope on this one repo
- Worst case if the PAT leaks: someone can fire dispatch events (eats free Actions minutes)
- They cannot modify `keys.json` directly; only the workflow has commit permission
