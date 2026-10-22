# Gitea 28.2.0 with Fluxer webhooks

This fork adds **Fluxer** as a webhook destination to the upstream Gitea **28.2.0**
release. It includes the upstream 28.1.0 and 28.2.0 fixes; the Fluxer implementation
is unchanged from the previous custom release. Select Fluxer in repository, user,
organization or administrator webhook settings, enter your webhook URL, and
optionally override the username and icon URL.
The REST API also supports `type: "fluxer"` and preserves these overrides.

Fluxer-compatible embeds reuse Gitea's event formatting, apply Unicode length limits,
omit invalid optional fields and disable automatic mentions.

[Download the Docker release](https://github.com/JaRamOxO/gitea/releases/tag/v28.2.0-fluxer.1).

## Run the Docker image

The prebuilt image is for **Linux amd64**, using Gitea's standard Docker layout:
`/data`, HTTP port 3000 and SSH port 22. Gitea reports `28.2.0+fluxer.1`.

Download the Docker archive and its `.sha256` file from the release, then run:

```sh
sha256sum -c gitea-28.2.0-fluxer.1-docker-linux-amd64.tar.gz.sha256
docker load --input gitea-28.2.0-fluxer.1-docker-linux-amd64.tar.gz
docker run --rm --entrypoint /app/gitea/gitea jaramoxo/gitea:28.2.0-fluxer.1 --version
```

The image is distributed as a GitHub release asset and imported with `docker load`.
In your existing Compose service, use:

```yaml
image: jaramoxo/gitea:28.2.0-fluxer.1
pull_policy: never
```

Use your existing volumes, networking, ports and database configuration. Recreate
the Gitea service, replacing `gitea` below with your Compose service name:

```sh
docker compose up -d --no-deps --pull never gitea
```

Back up your database, configuration and data before replacing the image. Updating
from `28.0.0+fluxer.1` uses the same Compose procedure and retains existing Fluxer
webhook settings. See the upstream [28.1.0 release notes](https://github.com/go-gitea/gitea/releases/tag/v28.1.0),
[28.2.0 release notes](https://github.com/go-gitea/gitea/releases/tag/v28.2.0) and
[upgrade guide](https://docs.gitea.com/installation/upgrade-from-gitea/).
For installations on 1.27.x, first complete the ordinary upstream 28.x upgrade,
including the breaking configuration changes in the
[28.0.0 release notes](https://blog.gitea.com/release-of-28.0.0/).

## Build from source

```sh
git clone --branch stable/28.2-fluxer https://github.com/JaRamOxO/gitea.git
cd gitea
docker build --build-arg GITEA_VERSION=28.2.0+fluxer.1 \
  --tag jaramoxo/gitea:28.2.0-fluxer.1 .
```

The repository's Dockerfile builds the frontend and backend from this branch.
For a native build, follow [Gitea's build setup](docs/build-setup.md), then run:

```sh
GITEA_VERSION=28.2.0+fluxer.1 TAGS='bindata timetzdata' make build
```

## Provenance and validation

The release binary was built from commit
`de270ed94d375e96ad5a82312fa65ab85762989f`, based on upstream tag `v28.2.0`
(`829b49c1ae96015e345f9d2a0eb1ea7acd5f31e1`). Subsequent release documentation
commits are on the same branch. The Docker runtime base is pinned to:

```text
docker.gitea.com/gitea@sha256:30777c91280d973fa02be48b1ade735bd6859e8f6f7af3b4444d519437852504
```

The static binary uses Go 1.27.3 with `bindata timetzdata`; frontend assets,
the official Fluxer symbol and time zone data are embedded. `build-info.json`,
checksums, the source patch and the exact built-source bundle accompany the release.

Webhook/API unit tests, Fluxer API/UI integration tests with SQLite, Go/template/
Swagger lint and API schema validation passed. Docker export/import, Chrome UI and
an isolated SQLite upgrade from `28.0.0+fluxer.1` were tested. The upgrade preserved
repository data, issues, login and Fluxer metadata, including API PATCH behavior.

The Fluxer implementation was AI-assisted with Codex and reviewed by the repository owner.
