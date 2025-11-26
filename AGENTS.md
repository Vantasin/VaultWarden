# Repository Guidelines

## Project Structure & Module Organization
- Root holds `README.md`, `LICENSE`, and the compose stack files (`docker-compose.yml`, `.env`, `env.example`); keep operational docs close to the stack.
- Persistent data stays outside the repo (`~/docker/data/vaultwarden` or `tank/docker/data/vaultwarden`); never commit secrets, backups, or stateful volumes.
- Reverse-proxy settings live in Nginx Proxy Manager; note required proxy tweaks in `README.md` to keep deployments reproducible.

## Build, Test, and Development Commands
- `docker compose config -q` — lint/validate the compose file before committing.
- `docker compose up -d` — start the stack; `docker compose logs -f vaultwarden` to follow service output.
- `docker compose pull && docker compose up -d` — upgrade to newer images.
- `docker compose down` — stop services while preserving volumes; avoid `--volumes` unless intentionally wiping data.
- `docker compose exec vaultwarden sh` — short-lived debugging inside the app container; prefer config changes via files, not interactive edits.

## Coding Style & Naming Conventions
- YAML: 2-space indentation, lowercase service names, and short keys; keep environment blocks sorted.
- Env files: uppercase `SNAKE_CASE` keys; example minimum: `DOMAIN=https://vault.example.com`, `ADMIN_TOKEN=...`, `DATA_DIR=/tank/docker/data/vaultwarden`.
- Comments stay concise and operational (why a setting exists), not descriptive restatements.

## Testing Guidelines
- Run `docker compose config -q` locally on every change; mention it in the PR.
- For config edits, bring the stack up on a test host (`docker compose up`) and verify the main UI and `/admin` via the proxy hostname you set.
- Share smoke steps taken (login, create item, restart stack) and note platform tested (Raspberry Pi 5 / ARM64 recommended).

## Commit & Pull Request Guidelines
- Commit messages: imperative mood, <72 chars; prefix with a scope when helpful (`docs:`, `config:`, `ops:`).
- PRs need a short summary, linked issue (if any), config changes, and test notes; include screenshots of Nginx Proxy Manager entries when relevant.
- Highlight security-impacting changes (ports, tokens, backup paths) so reviewers can double-check the model.

## Security & Configuration Tips
- Keep `.env` owned by you and `chmod 600`; rotate `ADMIN_TOKEN` if shared.
- Do not expose container ports directly; all ingress should stay behind HTTPS via the reverse proxy.
- For ZFS setups, place `DATA_DIR` on the snapshot-backed dataset to preserve recovery points; confirm backups before upgrades.
