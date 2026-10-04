# Gitea 28.0.0 with Fluxer webhooks

This fork adds **Fluxer** as a webhook destination to the upstream Gitea **28.0.0**
release. Select Fluxer in repository, user, organization or administrator webhook
settings, enter your webhook URL, and optionally override the username and icon URL.
The REST API also supports `type: "fluxer"` and preserves these overrides.

Fluxer-compatible embeds reuse Gitea's event formatting, apply Unicode length limits,
omit invalid optional fields and disable automatic mentions.

[Download the Docker release](https://github.com/JaRamOxO/gitea/releases/tag/v28.0.0-fluxer.1).

## Run the Docker image

The prebuilt image is for **Linux amd64**, using Gitea's standard Docker layout:
`/data`, HTTP port 3000 and SSH port 22. Gitea reports `28.0.0+fluxer.1`.

Download the Docker archive and its `.sha256` file from the release, then run:

```sh
sha256sum -c gitea-28.0.0-fluxer.1-docker-linux-amd64.tar.gz.sha256
docker load --input gitea-28.0.0-fluxer.1-docker-linux-amd64.tar.gz
docker run --rm --entrypoint /app/gitea/gitea jaramoxo/gitea:28.0.0-fluxer.1 --version
```

The image is distributed as a GitHub release asset and imported with `docker load`.
In your existing Compose service, use:

```yaml
image: jaramoxo/gitea:28.0.0-fluxer.1
pull_policy: never
```

Use your existing volumes, networking, ports and database configuration. Recreate
the Gitea service, replacing `gitea` below with your Compose service name:

```sh
docker compose up -d --no-deps --pull never gitea
```

For installations on 1.27.x, complete the ordinary upstream upgrade to 28.0.0 first,
including backups and configuration changes described in the
[28.0.0 release notes](https://blog.gitea.com/release-of-28.0.0/) and
[upgrade guide](https://docs.gitea.com/installation/upgrade-from-gitea/).

## Build from source

```sh
git clone --branch stable/28.0-fluxer https://github.com/JaRamOxO/gitea.git
cd gitea
docker build --build-arg GITEA_VERSION=28.0.0+fluxer.1 \
  --tag jaramoxo/gitea:28.0.0-fluxer.1 .
```

The repository's Dockerfile builds the frontend and backend from this branch.
For a native build, follow [Gitea's build setup](docs/build-setup.md), then run:

```sh
GITEA_VERSION=28.0.0+fluxer.1 TAGS='bindata timetzdata' make build
```

## Provenance and validation

The release binary was built from commit
`fd40f495bb4b7f6a5a9ab8f502fb98562c1e0239`, based on upstream tag `v28.0.0`
(`15b8a5805adf57c5189602008d38cccfd3c795e0`). Subsequent release documentation
commits are on the same branch. The Docker runtime base is pinned to:

```text
docker.gitea.com/gitea@sha256:a9dc25679ebdd33356242d2e0667cc220043745e8339922879f187c20329a42e
```

The static binary uses Go 1.27.1 with `bindata timetzdata`; frontend assets,
the official Fluxer symbol and time zone data are embedded. `build-info.json`,
checksums, the source patch and the exact built-source bundle accompany the release.

Webhook/API unit tests, Fluxer API/UI integration tests with SQLite, Go/template/
Swagger lint and API schema validation passed. Docker export/import, Chrome UI,
an isolated SQLite upgrade from 1.27.3 and switching from stock 28.0.0 were tested.
The latter checks preserved repository data, login and Fluxer metadata.

The Fluxer implementation was AI-assisted with Codex and reviewed by the repository owner.
