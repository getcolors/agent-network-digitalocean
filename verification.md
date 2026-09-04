# Live verification

The record of the live verification this deployment passed, per the workspace
Compute Provider Standard §7-8. Commands, outcomes and timings only: no
address, key id or token appears here.

## What was verified

| | |
|---|---|
| Date | 2026-09-04 (UTC) |
| Provider | DigitalOcean, region `ams3` |
| Plan | `s-2vcpu-4gb`, image `ubuntu-24-04-x64` |
| Package | `getcolors/agent-network` at `d228322` (pin commit `99f635c`); first attempt at `f233129` |
| Mode | keygen (no `digitalocean-ssh-keys`), compute name = profile, fake Anthropic key |
| State | R2, `agent-network-digitalocean/agent-network-infrastructure.tfstate` |

## Sequence and outcomes

| # | Command | Outcome |
|---|---|---|
| 1 | `./green create` (at `f233129`) | infrastructure 50 s, ssh-config 2 s, dns 5 s, then **exit 2 in the converge** at the lego install task: `sha256sum: WARNING: 1 computed checksum did NOT match` with one `OK` and one `FAILED`. Upstream's checksums file had gained a `.sbom.json` artifact whose name contains the tarball's, and the play's substring grep fed both lines to the check. A package bug, provider-independent, fixed as `d228322`; the deployment re-pinned. |
| 2 | `./green create` (at `d228322`) | **exit 0.** Stages: infrastructure 4 s (idempotent), ssh-config 2 s, dns 5 s, ansible 461 s (gateway stack, headless bootstrap, both certificates by DNS-01, agent image build), acceptance 18 s (isolation both ways, the keyless path, both denial classes, attribution, limits, endpoint refusing outside callers). |
| 3 | `ssh agent-network-digitalocean agent-network-status` | five containers up, two certificates issued, the endpoint minted beneath the wildcard. |
| 4 | `./red create` | **exit 0**, idempotent: ansible 260 s, acceptance 17 s. |
| 5 | `./blue create` | **exit 0**, idempotent: ansible 262 s, acceptance 17 s. |
| 6 | `COLORS_PAR_PROVIDER_COMPUTE=vultr ./green create` | **exit 2** before any credential or provider call: `state holds a digitalocean machine; set provider-compute back to digitalocean and delete first`. |
| 7 | `COLORS_PAR_PROVIDER_COMPUTE=vultr ./green delete` | **exit 2**, the same refusal, ahead of the prevent-destroy guard. |

Attempt 1 is a converge failure found by the port, not caused by it: the
same play would have failed on Vultr that day. Rows 4 and 5 prove the three
colours manage one state interchangeably; 6 and 7 prove Compute Provider
Standard §4 on a live state.

## Not verified here

- The Vultr side of the package, which `agent-network-vultr` covers.
- Real completions: the Anthropic key is deliberately fake, so the keyless
  probes prove the relayed 401, not a billed response.

## After verification

Deleted the same day under a fresh, explicit authorization, with the one-run
`COLORS_PAR_COMPUTE_PREVENT_DESTROY=false` override: cleanup play, the two DNS
records, `~/.ssh/config` block, droplet and firewall, account key, local
keypair, in that order, exit 0. Verified read-only afterwards that nothing
named after the profile survives at the provider. The repository, `colors.yml`
and the R2 state remain, so the deployment is re-creatable with `./green create`.
