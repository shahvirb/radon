# Hermes Stack

This stack is managed from the Radon repository root. Generated secrets stay
local in `stack-hermes/hermes.env`; only `hermes.env.tpl` is tracked.

## Render and Inspect

From `/home/shahvirb/gitsource/radon`, render the environment template:

```bash
/home/shahvirb/gitsource/utils/op-unpack.sh
```

Verify the shared environment symlink before using Compose:

```bash
test -L stack-hermes/.env
test "$(readlink stack-hermes/.env)" = "../.env"
```

Inspect the full rendered Compose configuration from the Radon root before
pulling or starting the service. Use the existing stack controller's Compose
config operation, or inspect this stack directly with:

```bash
docker compose -f stack-hermes/docker-compose.yaml config
```

The full config output can contain rendered secret values. Treat it as
sensitive and do not persist or share it.

## Deployment

From the Radon repository root:

1. Render `hermes.env` with `op-unpack.sh`.
2. Verify `stack-hermes/.env` points to `../.env`.
3. Inspect the full Compose config.
4. Pull `nousresearch/hermes-agent:latest` with the existing stack controller.
5. Start the Hermes service with the existing stack controller.
6. Confirm the container is running and inspect its startup logs.
7. Smoke-test `GET http://localhost:8642/health` and the dashboard root on
   port `9119`. The API health endpoint should return `200`; an
   authentication challenge or redirect from the protected dashboard is an
   expected reachable response.
8. Confirm the host mounts from the running container:

```bash
docker exec hermes test -d /home/shahvirb/gitsource
docker exec hermes test -S /var/run/docker.sock
docker exec hermes docker info
```

The stack controller can be invoked from the repository root with commands
such as `/home/shahvirb/gitsource/utils/stackcontrol.sh config`,
`/home/shahvirb/gitsource/utils/stackcontrol.sh pull`, and
`/home/shahvirb/gitsource/utils/stackcontrol.sh up` when those operations
are appropriate for the deployment.

## Risks

- Ports `8642` and `9119` publish on all host interfaces, and
  `API_SERVER_CORS_ORIGINS=*` permits wildcard cross-origin access. This is an
  intentional trusted-network decision, not a safe default for untrusted
  networks.
- The Docker socket mount gives Hermes broad control over the Docker host.
- The SSH mount exposes `/home/shahvirb/.ssh` to Hermes.
- The writable `gitsource` mount allows Hermes to modify repositories on the
  host.
- Full `docker compose config` output renders environment values and may
  expose secrets.
