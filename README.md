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
- **dev-mode.json** — allowlist of device IDs that bypass the license screen
- **results/** — per-activation result files written by the workflow
- **.github/workflows/activate.yml** — the validation workflow

## Dev mode (toggle without rebuilding the APK)

The app fetches `dev-mode.json` from this repo at every cold start. If the
running device's Device ID appears in `enabledDevices`, the license screen is
skipped and the orange DEV badge appears — useful for testing without burning
real keys.

### Enable dev mode on a device

1. Install the production APK on the device.
2. Open the app. On the License screen (or Settings → This device after
   activation) you'll see a Device ID. Long-press to copy.
3. Edit `dev-mode.json` here, paste the ID into `enabledDevices`, commit, push.
4. On the device, force-stop and reopen the app — or open Settings →
   This device → Refresh dev mode. Done.

### Disable dev mode on a device

Remove the ID from `enabledDevices`, commit, push. On the device, tap
Refresh dev mode (or wait up to 24h for the cache to expire and refetch).

### Notes

- Results are cached locally for 24h to keep startup fast and survive offline.
- A stale cache (up to 7 days old) is reused if the network is down.
- If the file is missing or the device isn't on the list, the app behaves
  exactly like a production install — license screen, real activation.
- **Never add a school's device ID to this list.** Their installs should
  always require an activation key.

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
