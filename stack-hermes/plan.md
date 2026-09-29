# Hermes Agent Integration Plan

## Scope

Integrate [`shahvirb/hermes-agent`](https://github.com/shahvirb/hermes-agent) into this stack while preserving the existing Radon environment and stack-management conventions.


## Files

- Add `docker-compose.yaml` based on the upstream deployment.
- Add `hermes.env.tpl` containing the full upstream environment template.
- Generate `hermes.env` locally from `hermes.env.tpl` with `op-unpack.sh`; never commit it.
- Add `.gitignore` rules for `hermes.env`.
- Retain the upstream `AGENTS.md` deployment guidance.
- Add `.env -> ../.env` so Compose can resolve shared Radon variables from the stack directory.

## Compose Configuration

- Use image `nousresearch/hermes-agent:latest`.
- Use fixed container name `hermes`.
- Use `restart: unless-stopped`.
- Run `gateway run`.
- Publish `8642:8642` for the API server.
- Publish `9119:9119` for the dashboard.
- Load Hermes secrets and settings from `hermes.env`.
- Forward `TZ=${TZ}`, `HERMES_UID=${PUID}`, and `HERMES_GID=${PGID}` from the shared root environment.
- Persist Hermes data at `${DOCKERDATADIR}/hermes:/opt/data`.
- Mount `/var/run/docker.sock:/var/run/docker.sock`.
- Mount `/home/shahvirb/.ssh:/ssh:ro`.
- Mount `/home/shahvirb/gitsource:/home/shahvirb/gitsource` read-write.
- Keep `/workspace` as the container working directory.
- Keep resource limits at 8 GB memory and 4 CPUs.
- Do not add a Compose `logging` section; use Docker daemon defaults.
- Do not add a healthcheck.

## Environment and Security

- Preserve the upstream 1Password `op://` references in `hermes.env.tpl`.
- Keep generated `hermes.env` out of version control.
- Enable the API server and dashboard.
- Retain `API_SERVER_CORS_ORIGINS=*`.
- Use the configured dashboard basic-auth username, password, and session secret.
- Keep `HERMES_WRITE_SAFE_ROOT` empty as specified by the deployment requirements.
- Keep `TERMINAL_ENV=local`.

## Deployment Workflow

1. Run `/home/shahvirb/gitsource/utils/op-unpack.sh` from the Radon root to render `stack-hermes/hermes.env`.
2. Run the existing stack controller so Compose executes from `stack-hermes` and resolves the shared `.env` symlink.
3. Pull `nousresearch/hermes-agent:latest`.
4. Start the Hermes service.
5. Validate the rendered Compose configuration with `docker compose config`.
6. Confirm the container is running and inspect startup logs.
7. Smoke-test the API server and dashboard endpoints using their configured authentication.
8. Confirm Hermes can access the mounted workspace and Docker socket.

## Explicit Non-Goals

- Do not implement Docker logging limits from `DOCKERLOGGING_MAXFILE` or `DOCKERLOGGING_MAXSIZE`.
- Do not bind the published ports to localhost.
- Do not add a separate stack-local secret store beyond generated `hermes.env`.
- Do not pin the image to a release tag or digest.
