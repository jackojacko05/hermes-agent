# Keep one Discord gateway owner

Hermes can run a local Desktop/backend and a remote VPS gateway at the same
time. The Discord bot connection is different: every active gateway that uses
the same bot token receives the same Discord event. If both gateways process
that event, the channel can receive duplicate replies and each host can use a
different model or workspace.

Use one gateway owner for each Discord bot token. For a stable deployment,
keep Discord on the VPS and use the Mac for Desktop or local CLI development.

## VPS configuration

Keep the Discord token in the VPS-only `.env` file. Do not commit the token.
The VPS `config.yaml` may make ownership explicit:

```yaml
platforms:
  discord:
    enabled: true
```

## Mac configuration

Disable the Discord adapter in the Mac profile. This setting is behavioral
configuration and belongs in `config.yaml`, not `.env`:

```yaml
platforms:
  discord:
    enabled: false
```

The Mac can still run the Desktop backend, CLI, local models, and local file
operations. It simply must not start a Discord adapter with the shared bot
token.

If a Mac gateway was already running, stop it and disable its launch agent
before changing the configuration:

```bash
hermes gateway stop
launchctl disable "gui/$(id -u)/ai.hermes.gateway"
```

After changing `config.yaml`, restart the local gateway if it is needed for
non-Discord development. It should report Discord as disabled and must not log
`Connected as Hermes Personal AI Bot`.

## Verification

Check that the Mac has no Discord gateway process and that the VPS is the only
connected owner:

```bash
pgrep -af 'hermes_cli.main gateway run' || true
hermes gateway status
```

On the VPS, check the gateway status endpoint or service logs and confirm the
Discord adapter is `connected`. Do not test by sending duplicate production
messages; compare the process and adapter status instead.

## Temporary Mac Discord testing

When Discord must be tested from the Mac, stop or disable the VPS Discord
adapter first, then enable the Mac adapter. Never run both hosts with the same
token. A separate Discord application/token and test channel are safer when
simultaneous testing is required.

## Codex fallback model policy

When the VPS primary provider (Grok) is unavailable, keep the first Codex
fallback on the lightweight `gpt-5.6-luna` model:

```yaml
fallback_providers:
  - provider: openai-codex
    model: gpt-5.6-luna
```

This default does not restrict explicit model selection. The Codex catalog
continues to expose `gpt-5.6-sol` and `gpt-5.6-terra` (plus their `-pro`
variants) when a model is selected directly.
