# Gitea 28.4.0 with Fluxer webhooks

This fork adds **Fluxer** as a webhook destination to the upstream Gitea **28.4.0**
release. It includes upstream security fixes, an updated queue library and
refreshed Alpine runtime packages. Fluxer retains release asset counts and
existing webhook settings.
Select Fluxer in repository, user,
organization or administrator webhook settings, enter your webhook URL, and
optionally override the username and icon URL.
The REST API also supports `type: "fluxer"` and preserves these overrides.

Fluxer-compatible embeds reuse Gitea's event formatting, apply Unicode length limits,
omit invalid optional fields and disable automatic mentions.

[Download the Docker release](https://github.com/JaRamOxO/gitea/releases/tag/v28.4.0-fluxer.1).

## Run the Docker image

The prebuilt image is for **Linux amd64**, using Gitea's standard Docker layout:
`/data`, HTTP port 3000 and SSH port 22. Gitea reports `28.4.0+fluxer.1`.

Download the Docker archive and its `.sha256` file from the release, then run:

```sh
sha256sum -c gitea-28.4.0-fluxer.1-docker-linux-amd64.tar.gz.sha256
docker load --input gitea-28.4.0-fluxer.1-docker-linux-amd64.tar.gz
docker run --rm --entrypoint /app/gitea/gitea jaramoxo/gitea:28.4.0-fluxer.1 --version
```

The image is distributed as a GitHub release asset and imported with `docker load`.
In your existing Compose service, use:

```yaml
image: jaramoxo/gitea:28.4.0-fluxer.1
pull_policy: never
```

Use your existing volumes, networking, ports and database configuration. Recreate
the Gitea service, replacing `gitea` below with your Compose service name:

```sh
docker compose up -d --no-deps --pull never gitea
```

Back up your database, configuration and data before replacing the image. Updating
from `28.3.0+fluxer.1` uses the same Compose procedure and retains existing Fluxer
webhook settings. See the upstream [28.4.0 release notes](https://github.com/go-gitea/gitea/releases/tag/v28.4.0) and
[upgrade guide](https://docs.gitea.com/installation/upgrade-from-gitea/).
For installations on 1.27.x, first complete the ordinary upstream 28.x upgrade,
including the breaking configuration changes in the
[28.0.0 release notes](https://blog.gitea.com/release-of-28.0.0/).

Keep existing egress allow entries when replacing the image. An HTTPS webhook
resolving to a private IP needs an explicit IP allow entry (such as
`192.168.1.25:443`) in `GITEA__security__ALLOWED_HOST_LIST`; a hostname entry alone
does not grant access to private addresses. Append the actual destination IP to
your existing list.

## Build from source

```sh
git clone --branch stable/28.4-fluxer https://github.com/JaRamOxO/gitea.git
cd gitea
docker build --build-arg GITEA_VERSION=28.4.0+fluxer.1 \
  --tag jaramoxo/gitea:28.4.0-fluxer.1 .
```

The repository's Dockerfile builds the frontend and backend from this branch.
For a native build, follow [Gitea's build setup](docs/build-setup.md), then run:

```sh
GITEA_VERSION=28.4.0+fluxer.1 TAGS='bindata timetzdata' make build
```

## Provenance and validation

The release binary was built from commit
`e6c1f45bf2ec0d6637aa99e3836d02a8debeb8d6`, based on upstream tag `v28.4.0`
(`0b5dfdeeeaadf6350314246c5936b53cce38170c`). Subsequent release documentation
commits are on the same branch. The Docker runtime base is pinned to:

```text
docker.gitea.com/gitea@sha256:aa90e8f00b755b0d665214ab9c14447ea44d3190dbe6b6d0cb46cd9c4f55d309
```

The static binary uses Go 1.27.4 with `bindata timetzdata`; frontend assets,
the official Fluxer symbol and time zone data are embedded. `build-info.json`,
checksums, the source patch and the exact built-source bundle accompany the release.

Webhook/API/queue unit tests, Fluxer API/UI integration tests with SQLite, Go/template/
Swagger lint and API schema validation passed. Docker export/import, Chrome UI and
an isolated SQLite upgrade from `28.3.0+fluxer.1` were tested. The upgrade preserved
repository data, issues, login and Fluxer metadata, including API PATCH behavior.

The Fluxer implementation was AI-assisted with Codex and reviewed by the repository owner.
