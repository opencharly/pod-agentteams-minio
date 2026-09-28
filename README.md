# agentteams-minio

MinIO object storage for AgentTeams, as an OpenCharly candy.

MinIO is the shared object store the AgentTeams Manager and Workers use to
exchange agent config and artifacts. This candy ships the `minio` server and the
`mc` client, with the server (S3 API `:9000`, console `:9001`) and a storage
initializer as supervisord services.

The `mc-mirror` service creates the default bucket and placeholder directories,
then mirrors the `agentteams-config` control-plane state from MinIO to a local
directory every 10 seconds (the controller watches it via fsnotify). Both
services run rootless as the image user (uid 1000) against the
`~/.agentteams/minio` volume. The start scripts are charly-owned.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `agentteams-minio` |
| Binaries | `minio` server + `mc` client |
| Services / ports | `minio` (50) on `9000` (S3) / `9001` (console); `mc-mirror` (700) |
| Volume | `~/.agentteams/minio` |

Deploy-overridable `env_accept` vars set the MinIO root credentials
(`AGENTTEAMS_MINIO_USER` / `AGENTTEAMS_MINIO_PASSWORD`, with the AgentTeams
admin credentials as fallback). When no password is supplied, a random one is
generated and persisted on the volume so the controller and `mc-mirror` read the
same value.

## How to use it

Compose the candy into a box (or the full AgentTeams stack, whose top
composition already includes it):

```yaml
my-agentteams:
  candy:
    base: cachyos.cachyos
    candy:
      - '@github.com/opencharly/pod-agentteams-minio:<tag>'
```

Then build and deploy with the charly CLI:

```bash
charly box build my-agentteams
charly start my-agentteams
```

See the owning skill for the composition, ports, and both deploy substrates.

## Layout

- `charly.yml` — the `agentteams-minio:` candy entity: the two `extract` entries,
  `env_accept`, the volume, the two ports, the two services, and the plan that
  writes the rootless start scripts.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-agentteams:agentteams` — the full stack composition.
- Sibling services: `/charly-agentteams:agentteams` (matrix, element, higress, controller).
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
