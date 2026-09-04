# CLAUDE.md

## Repository

`agent-network-digitalocean` is desired state only: no source code. It
configures the
[`agent-network`](https://github.com/getcolors/agent-network) Package Skill to
run a minimal NetBird Agent Network demo on one DigitalOcean Droplet, serving
`agent-network-digitalocean.bigconfig.online` and, one generated label beneath
it, the keyless agent-network endpoint.

This is the second-provider deployment of the workspace Compute Provider
Standard (`../workspace/standards/compute-provider.md`): the same package that
runs `agent-network-vultr`, on DigitalOcean by `provider-compute:
digitalocean` and the `digitalocean-*` keys alone. The hostname is
deliberately distinct from `agent-network-vultr`'s so a rebuild of either can
never collide with the other, and so the two wildcard certificates never
overlap.

`colors.yml` is the only file to edit. Everything else is either generated
(`.colors/`), secret (`.envrc.private`), or a copy of the installed skills.

## The launchers are copies, not symlinks

Root `green`, `red` and `blue` are copies of
`.agents/skills/package-agent-network-<colour>/<colour>`. `npx skills update
-p` rewrites the payloads and leaves the root files alone; after any update
copy each launcher over its root file. Never hand-edit a SHA. Never run two
colours concurrently against the same state.

## Provider switching is a rebuild

Every real `create` and `delete` reads the recorded `params.provider` from
state before validating provider credentials and refuses when it differs from
`provider-compute`. To move this profile to another provider: `delete` on the
recorded provider, then edit `provider-compute` and `create`.

## What converges here

The gateway stack (Traefik, the combined `netbird-server`, the dashboard in
agent-network-only mode, the NetBird reverse proxy in private mode), a
headlessly bootstrapped control plane, and the **isolated agent**: a container
on an internal Docker network with no internet route, running the NetBird
client and headless Claude Code. Acceptance proves the isolation both ways,
the keyless path, both denial classes, attribution, the configured limits,
and that the endpoint refuses callers outside the overlay. The firewall
admits 22 from the SSH sources, 80 and 443 from the HTTP sources, and STUN
(UDP 3478) from the STUN sources; nothing else.

## The Anthropic key is deliberately fake

`COLORS_PAR_ANTHROPIC_API_KEY` in this deployment's `.envrc.private` is the
same fake value `agent-network-vultr` uses. The acceptance suite runs in
fake-key mode: the keyless probes expect Anthropic's own 401 relayed through
the proxy. To upgrade to real completions, put a real key in `.envrc.private`
and re-run `./green create`.

## Do not

- Read or print `.envrc.private`.
- Edit, read as source, or commit `.colors/`.
- Export `COLORS_PAR_PROFILE`; the package refuses to run when it is set.
- Weaken `compute-prevent-destroy` in committed desired state.
- Run a real `create` or `delete` without explicit authorization.
- Regenerate anything under `/etc/agent-network/secrets` on the host.

## Disposability

There are no backups, on purpose. Recovery is a guarded `delete` followed by
`create`, which regenerates the endpoint hostname and every peer identity.
Keygen mode: `~/.ssh/agent-network-digitalocean` is generated and owned by
the package, and the DigitalOcean account key named
`agent-network-digitalocean` belongs to this deployment's state.

## Documentation

`index.html` carries the two analytics tags every repository page carries:
GA4 measurement ID `G-4VKP1WY4QJ`, whose explicit `page_title` must equal the
decoded HTML `<title>`, and the self-hosted Rybbit snippet
`<script src="https://rybbit.getcolors.ai/api/script.js" data-site-id="9fb9c41a6d49" defer></script>`.
Never add one tag without the other. `verification.md` records the live
verification this deployment passed; it carries no address, key id or token.

## Git

Work on the current branch. Do not commit or push unless explicitly authorized.
