# ASIX content

Packs for the ASIX SIEM, one YAML file each in `packs/`. ASIX reads them through a signed catalog (a Marketplace **feed**), so they do not have to ship inside the ASIX image. Started from the 10 packs shipped with ASIX on 2026-11-19. Full description: `docs/marketplace-feeds.md` in the ASIX repository.

## One-time set-up
1. In the ASIX repository: `node scripts/pack-tool.js keygen ./keys asix-content-1`. Paste the printed line into `KEYS` in `src/packs/catalog-keys.js`, commit, rebuild and deploy ASIX. **Keep `keys/asix-content-1.private.pem` out of Git.**
2. In this repository: secret `PACKS_SIGNING_KEY` = the text of the private key; variable `PACKS_KEY_ID` = `asix-content-1`; if the ASIX repository is private, secret `ASIX_REPO_TOKEN` (read-only). In `.github/workflows/publish.yml` set `ahsan520/ash-app-builder`.
3. Push to `main`. The workflow checks every pack, builds and signs `index.json`, and publishes the branch `feed`.
4. In ASIX, Marketplace > Feeds > Add feed: `https://raw.githubusercontent.com/ahsan520/asix-content/feed/`, then Refresh, then Download.

## Adding or changing a pack
Edit or add `packs/<id>.pack.yaml` (the file name is the pack id) and open a pull request. The workflow runs `check`: the pack is validated with the same rules ASIX uses and every parser sample must give the fields it lists. Raise `version` and add a `history` entry on the top when a pack changes. A parser can be edited as YAML in ASIX (Parsing rules > Pack parsers) and the text copied back here.

Packs are data only: rules, parsers and collection notes. A pack never carries code.

Locally, with the ASIX repository checked out next to this one: `node ../ash-app-builder/scripts/pack-tool.js check .`
