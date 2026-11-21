# ASIX content

Packs for the ASIX SIEM, one YAML file each in `packs/`, and optionally the SigmaHQ detection rules. ASIX reads them through a signed catalog (a Marketplace **feed**), so they do not have to ship inside the ASIX image. Started from the 10 packs shipped with ASIX. Full description: `docs/marketplace-feeds.md` and `docs/sigma-import.md` in the ASIX repository.

## One-time set-up
1. In the ASIX repository: `node scripts/pack-tool.js keygen ./keys asix-content-1`. Paste the printed line into `KEYS` in `src/packs/catalog-keys.js`, commit, rebuild and deploy ASIX. **Keep `keys/asix-content-1.private.pem` out of Git.**
2. In this repository (Settings, Secrets and variables, Actions): secret `PACKS_SIGNING_KEY` = the text of the private key; variable `PACKS_KEY_ID` = `asix-content-1`; if the ASIX repository is private, secret `ASIX_REPO_TOKEN` (read-only). In `.github/workflows/publish.yml` set `YOUR-ORG/ash-app-builder`. Under Settings, Actions, General, allow workflows to read and write.
3. Push to `main`. The workflow checks every pack, builds and signs `index.json`, and publishes the branch `feed`.
4. In ASIX, Marketplace > Feeds > Add feed: `https://raw.githubusercontent.com/<owner>/<repo>/feed/`, then Refresh, then Download.

## Offering the SigmaHQ rules (optional)
Set the repository variable `SIGMA_COMMIT` to the 40 character id of a commit of https://github.com/SigmaHQ/sigma (use a recent one from the SigmaHQ commit list) and push, or run the workflow again. The workflow clones the rules at that commit, writes `sigma/index.json` (the rules ASIX can convert, with each file's checksum), and the signed index points to it. In ASIX, refresh the feed: the Feeds tab then has a **Sigma rules** section to search, preview and download rules as a pack. Rules are fetched from SigmaHQ only when someone previews or downloads them, and they install switched off. To move to newer rules, change `SIGMA_COMMIT` and run the workflow again. Keep the Detection Rule License notice of SigmaHQ in mind if you copy rules (`docs/sigma-import.md`).

## Adding or changing a pack
Edit or add `packs/<id>.pack.yaml` (the file name is the pack id) and open a pull request. The workflow runs `check`: the pack is validated with the same rules ASIX uses and every parser sample must give the fields it lists. Raise `version` and add a `history` entry on the top when a pack changes. A parser can be edited as YAML in ASIX (Parsing rules > Pack parsers) and the text copied back here.

Packs are data only: rules, parsers and collection notes. A pack never carries code.

Locally, with the ASIX repository checked out next to this one: `node ../ash-app-builder/scripts/pack-tool.js check .`
