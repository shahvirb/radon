# Hermes Agent Integration Plan

## Scope

Integrate [`shahvirb/hermes-agent`](https://github.com/shahvirb/hermes-agent) into this stack while preserving the existing Radon environment and stack-management conventions.


## Files

- Add `docker-compose.yaml` based on the upstream deployment.
- Add `hermes.env.tpl` containing the full upstream `main` environment template plus the deployment-specific settings and `op://` references.
- Generate `hermes.env` locally from `hermes.env.tpl` with `op-unpack.sh`; never commit it.
- Add a repository-root `.gitignore` entry for `stack-hermes/hermes.env`.
- Add `AGENTS.md` with the upstream deployment guidance adapted to `hermes.env.tpl`, `hermes.env`, and this stack's render command.
- Retain and verify the existing `stack-hermes/.env -> ../.env` symlink so Compose can resolve shared Radon variables from the stack directory.

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
- Let the Hermes image initialize `/opt/data` ownership using `HERMES_UID` and `HERMES_GID`.
- Enable the API server and dashboard.
- Retain `API_SERVER_CORS_ORIGINS=*`.
- Use the configured dashboard basic-auth username, password, and session secret.
- Keep `HERMES_WRITE_SAFE_ROOT` empty as specified by the deployment requirements.
- Keep `TERMINAL_ENV=local`.
- Document that publishing both ports on all host interfaces with wildcard CORS is an intentional trusted-network decision, not a default-safe configuration.
- Document that the Docker socket, SSH-key mount, and writable gitsource mount give Hermes broad control over the host environment.
- Document the residual risk that full `docker compose config` output may expose rendered secret values.

## Deployment Workflow

1. Run `/home/shahvirb/gitsource/utils/op-unpack.sh` from the Radon root to render `stack-hermes/hermes.env`.
2. Verify that `stack-hermes/.env` points to the shared Radon `.env`.
3. Run the existing stack controller's Compose config operation from the Radon root before pulling or starting the service. Inspect the full rendered configuration as required, accepting the documented secret-output risk.
4. Pull `nousresearch/hermes-agent:latest`.
5. Start the Hermes service with the existing stack controller.
6. Confirm the container is running and inspect startup logs.
7. Smoke-test `GET http://localhost:8642/health` and the dashboard root on port `9119` for reachability. Expect the API health endpoint to return `200`; an authentication challenge or redirect from the protected dashboard is an expected reachable response.
8. Confirm the mounts explicitly:
   - `docker exec hermes test -d /home/shahvirb/gitsource`
   - `docker exec hermes test -S /var/run/docker.sock`
   - `docker exec hermes docker info`

## Explicit Non-Goals

- Do not implement Docker logging limits from `DOCKERLOGGING_MAXFILE` or `DOCKERLOGGING_MAXSIZE`.
- Do not bind the published ports to localhost.
- Do not add a separate stack-local secret store beyond generated `hermes.env`.
- Do not pin the image to a release tag or digest.
