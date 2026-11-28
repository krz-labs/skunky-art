# SkunkyArt

SkunkyArt is an alternative frontend for DeviantArt. It works without
JavaScript, runs as a single binary with one config file, and needs no
database.

This is a maintained fork of [SkunkyArt](https://git.macaw.me/skunky/SkunkyArt).
The upstream project has not been updated in over a year, has open bugs, and
its listed instances are no longer reachable.

If you find it useful, you can support the project through
[GitHub Sponsors](https://github.com/sponsors/ccleberg).

## Contents

- [Features](#features)
- [Public instances](#public-instances)
- [Installation](#installation)
- [Configuration](#configuration)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## Features

| Path | Description |
| --- | --- |
| `/` | Search box, with links to daily deviations and the about page. |
| `/dd` | Daily deviations. Add `?atom=true` for an Atom feed. |
| `/search?q=<query>&type=all\|tag\|r` | Search art, tags or groups. |
| `/post/<author>/<name-id>` | A single deviation. Comments load on request. |
| `/group_user?q=<name>&type=about\|gallery\|favourites` | A user or group profile. Add `&atom=true` to a gallery for an Atom feed. |
| `/api/...` | JSON API, documented in [API.md](API.md). |

[REDIRECTS.md](REDIRECTS.md) lists how deviantart.com URLs map to these paths,
for use with redirect extensions such as LibRedirect.

### Upstream traffic

DeviantArt blocks egress IPs that make too many requests. SkunkyArt keeps its
upstream traffic low by default:

- API responses are cached in memory, and concurrent requests for the same
  resource share one upstream call.
- Media and avatars are cached on disk.
- Comments load only when requested.
- Pages send `Cache-Control` headers, and crawlers are served a `robots.txt`.
- Each client has a request budget.

All of these are tunable; see [SETUP.md](SETUP.md).

## Public instances

See [INSTANCES.md](INSTANCES.md). CI checks every listed instance daily
(`scripts/check-instances.py`) and fails if one is unreachable.

To list your instance, open a pull request that updates
[instances.json](instances.json) and [INSTANCES.md](INSTANCES.md), or open an
issue. Running `skunkyart --add-instance` updates both files for you.

## Installation

### Docker

Multi-arch images (`linux/amd64`, `linux/arm64`) are published to GHCR for
every release:

```sh
docker pull ghcr.io/krz-labs/skunky-art:latest
```

Each release is tagged `X.Y.Z`, `X.Y`, `X` and `latest`. Pin an exact version
if you want predictable upgrades.

- [compose.example.yaml](compose.example.yaml) is a basic Compose setup.
- [compose.vpn_example.yml](compose.vpn_example.yml) adds an optional VPN
  sidecar for egress traffic.

### Release binaries

Tarballs for Linux, macOS and FreeBSD are attached to each release on the
[GitHub mirror](https://github.com/krz-labs/skunky-art/releases).

### Build from source

Requires Go. The `embed` tag bundles templates and static files into the
binary; leave it out if you plan to edit the templates.

```sh
make build    # embed tag, version taken from git describe
```

Or with `go build` directly:

```sh
go build -tags embed -ldflags "-w -s -X main.version=<version>"
```

Without `-X main.version`, the binary reports its version as `dev`. With
gccgo, pass the linker flags through `-gccgoflags` instead.

### Running

Place the binary in a directory with a `config.json` (and the `static/`
directory, if you built without the `embed` tag), then run:

```sh
./skunkyart -c config.json
```

The config file is optional. Without it, SkunkyArt listens on
`127.0.0.1:3003` with caching enabled.

Example systemd and OpenRC service files are in [services/](services/). Run
SkunkyArt behind a reverse proxy that terminates TLS; [SETUP.md](SETUP.md)
includes an nginx example.

## Configuration

Start from [config.example.json](config.example.json) and pass a different
file with `-c` or `--config`. Every option is documented in
[SETUP.md](SETUP.md).

## Development

```sh
make test    # go test ./... -race
make lint    # golangci-lint
make dist    # release tarballs under dist/
```

CI runs the tests and linter on every push. Planned work is tracked in
[ROADMAP.md](ROADMAP.md).

## Contributing

The repository, issue tracker and pull requests are on
[github.com/krz-labs/skunky-art](https://github.com/krz-labs/skunky-art).

## License

X11 License. See [LICENSE](LICENSE).
