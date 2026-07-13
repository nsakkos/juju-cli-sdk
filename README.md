# juju-cli Workshop SDK

A **thin** [Workshop](https://ubuntu.com/workshop/docs/) SDK that ships the
**Juju CLI client** as a prebuilt binary, tracking the stable **3.x** series.

> **Not a Juju controller.** This SDK installs only the `juju` *client*. It does
> not bootstrap or run a controller, and holds no credentials or cloud config.
> It talks to a **remote** controller you register or bootstrap at runtime.
> (The name is `juju-cli`, not `juju`, precisely to avoid the impression that a
> controller is being stood up inside the workshop.)

## What it ships

- The `juju` CLI (from the official `github.com/juju/juju` release tarball),
  installed at `$SDK/bin/juju` and added to the system `PATH`.

Only the client binary is primed — the helper binaries bundled in the upstream
tarball are dropped to keep the SDK thin.

## Plugs

| Plug | Interface | Auto-connect | Purpose |
|------|-----------|--------------|---------|
| `ssh-agent` | `ssh-agent` | No (opt-in) | Reach machines with `juju ssh`/`juju scp`/`juju debug-hooks` and do manual provisioning. Connect with `workshop connect <workshop>/juju-cli:ssh-agent`. |

The SSH access is opt-in — nothing crosses the host boundary unless you wire it.

## Slots

This SDK declares **no slots** — it consumes capabilities, it does not provide
them; it is just the client binary on `PATH`.

## Controller data persistence (optional, discouraged)

Registering or bootstrapping a controller writes endpoints, accounts, and
**credentials/macaroons** under `~/.local/share/juju`. By default that lives
only inside the ephemeral workshop and does **not** survive a rebuilding refresh
— which is the safe default. The SDK deliberately does **not** declare a mount
plug for it: a `mount` plug *always* auto-connects (there is no per-plug
opt-out), so shipping one would persist Juju credentials onto a host directory
by default. If you genuinely need that data to persist, graft a `mount` at that
path in your own workshop definition (extending the SDK's interfaces without the
publisher's involvement — see
[Plugs, slots, connections](https://ubuntu.com/workshop/docs/explanation/workshops/concepts/#exp-workshop-definition-connections)),
and understand you are then writing live controller credentials to the host.

## Build and try locally

```bash
sdkcraft try                     # packs juju-cli_<arch>_<base>.sdk into the try area
# add `- name: try-juju-cli` under sdks: in a scratch workshop, then:
workshop launch --verbose --wait-on-error
workshop exec -- juju version
```

Iterate with `sdkcraft clean && sdkcraft try` then `workshop refresh`.

## Test

```bash
sdkcraft test        # runs the spread tests under tests/ against a clean container
```

## Versioning

The `VERSION` file is the single source of truth for the pinned upstream Juju
release; `sdkcraft.yaml` adopts it at build time (`adopt-info` +
`craftctl set version`). Renovate (see `renovate.json`) opens PRs to bump
`VERSION` within the stable **3.x** line (`allowedVersions: /^3\./`) as new Juju
releases ship — no separate literal to keep in sync. To track a different major
(e.g. 4.x), publish from a `4` branch with a matching `renovate.json` rule.

## License and copyright

Copyright 2026 Canonical Ltd.

This program is free software: you can redistribute it and/or modify it under
the terms of the
[GNU Lesser General Public License version 2.1 (LGPLv2.1)](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html)
as published by the Free Software Foundation.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the GNU Lesser General Public License for more details.

The packaged binary is Juju, licensed under the GNU Affero General Public
License v3.0 (AGPL-3.0). This SDK only downloads and repackages the official
release tarball; the `license:` field reflects the bundled binary's terms.
