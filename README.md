# agent-network-digitalocean

Desired state for a minimal
[NetBird Agent Network](https://docs.netbird.io/agent-network) demo on
DigitalOcean: keyless, identity-gated LLM access, demonstrated by an agent
that cannot reach anything else.

- **https://agent-network-digitalocean.bigconfig.online** — dashboard
  (agent-network view), REST API, management and signal gRPC, relay
  WebSocket, embedded IdP
- **https://\<label\>.agent-network-digitalocean.bigconfig.online** — the
  generated agent-network endpoint: tunnel-only, keyless
- **`agent-network-agent`** — the isolated agent container: NetBird client +
  headless Claude Code on an internal Docker network with no internet route

Built by the
[`agent-network`](https://github.com/getcolors/agent-network) Package Skill in
any of its three colours (`package-agent-network-green`, `-red`, `-blue`,
installed under `.agents/skills/`): OpenTofu manages the droplet in the
region's default VPC, its cloud firewall and two unproxied Cloudflare `A`
records (the name and its wildcard); Ansible converges the gateway stack,
bootstraps the control plane headlessly, builds and starts the agent, and
proves the demo's claims.

This is the agent-network package's second-provider deployment under the
workspace Compute Provider Standard: the same package that runs
`agent-network-vultr`, selected onto DigitalOcean by `provider-compute` alone.

## Use

```sh
direnv allow               # once, after cloning
./green build              # render .colors/agent-network-digitalocean/
./green create --dry-run   # walk the workflow, no side effects
./green create             # converge
```

`./red` and `./blue` run the same verbs against the same state; never run two
colours concurrently.

## Fake-key mode

This deployment runs with a **deliberately fake Anthropic key**: acceptance
expects Anthropic's own 401 relayed through the proxy, proving isolation,
tunnel, policy and server-side key injection with nothing billable. Put a
real key in `.envrc.private` and re-run `./green create` to upgrade the demo
to real completions.

## Watching the demo

```sh
ssh agent-network-digitalocean agent-network-status    # containers, endpoint, usage
ssh agent-network-digitalocean agent-network-smoke     # re-run every acceptance gate
docker exec agent-network-agent claude -p 'hi'         # on the host: the governed path
```

Dashboard sign-in: `claude@ululi.it` with the host-generated password at
`/etc/agent-network/secrets/admin_password`.

## Deleting

```sh
COLORS_PAR_COMPUTE_PREVENT_DESTROY=false ./green delete
```

No backups exist, by design; a later `create` rebuilds everything and mints a
fresh endpoint hostname. Changing `provider-compute` on this profile is
refused while a machine is in state: a provider switch is a delete followed
by a create, never an apply.
