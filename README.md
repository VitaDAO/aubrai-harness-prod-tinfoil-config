# Aubrai harness: production Tinfoil deployment

Files here are the source of truth for the public config repo `VitaDAO/aubrai-harness-prod-tinfoil-config`,
the attestation anchor of the production enclave `aubrai-harness-prod` (API https://api.aubr.ai, UI
https://chat.aubr.ai). Staging is `deploy/tinfoil/` and `VitaDAO/aubrai-harness-tinfoil-config`.

Production runs an image staging has already run: never build for production. It uses the production
database (`aubrai-prod`, schema `harness`) and the previous Aubrai server's own enclave keys
(`HPKE_PRIVATE_KEY`, `HPKE_PUBLIC_KEY`, `ENCRYPTION_MASTER_KEY`), so users' sealed requests and recovery
keys keep working. Staging must not use either.

## Promote a staging release

1. Take the image line (`ghcr.io/vitadao/aubrai-harness:tinfoil-<version>@sha256:<digest>`) from
   `deploy/tinfoil/tinfoil-config.yml` of a release that passed on staging, and set it in
   `deploy/tinfoil-prod/tinfoil-config.yml`. `tests/deploy/tinfoil-config.test.ts` checks the rest.
2. Copy `tinfoil-config.yml` to the config repo, commit, and tag it `<version>`. Its workflow
   (`.github/workflows/tinfoil-build.yml`) publishes the attestation release for that tag.
3. `tinfoil container relaunch aubrai-harness-prod --tag <version> --secret …`, listing every secret in the
   config (the flag replaces the whole list). Rollback: relaunch with the previous tag.

First time: `tinfoil container create aubrai-harness-prod --repo VitaDAO/aubrai-harness-prod-tinfoil-config
--tag <version> --custom-domain api.aubr.ai --secret …`. `api.aubr.ai` is verified in Tinfoil; its DNS
(Cloudflare) needs the CNAME `api.aubr.ai → c37a9fd823d6518329d7a46b059ff564.tf-dns.com`, DNS only.
