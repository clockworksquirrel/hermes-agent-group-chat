# Hermes Agent Group Chat

Route `@mentions` in a chat to different [Hermes](https://github.com/NousResearch/hermes) agent profiles, and have each one reply **as itself**, in the same room.

You run one main agent. You type `@hank build me a prototype` and `@tommy review the risks`, and Hank and Tommy each answer in their own voice, from their own profile, without your main agent impersonating them. It works on any platform your Hermes gateway speaks (iMessage, Telegram, Discord, Slack, and the rest), because the transport is the Hermes CLI itself.

## How it works

The plugin registers a `pre_gateway_dispatch` hook. On every inbound message it:

1. Looks for `@name` mentions whose name matches a known profile or alias.
2. For each match, invokes that profile out of process via the Hermes CLI (`-p <profile> -z <prompt>`).
3. Posts each reply back into the same room, prefixed with the agent's name.
4. Optionally suppresses the host agent's own reply so the mentioned agents answer cleanly.

Agents are discovered dynamically from your `profiles/` directory and your config, so nothing user-specific is hardcoded.

## The one prerequisite that actually matters

Each agent answers as itself **only if that profile has its own identity.** In a default setup, every profile may authenticate as the same user, so the gateway can serve them all the same persona, and you get copies of your main agent wearing different nametags.

To get genuinely distinct agents, each participating profile needs its own identity at whatever layer your setup uses to choose a persona (its own account/token, or its own onboarded persona record). If your agents come back sounding identical, this is almost always why. The plugin routes correctly; identity is resolved upstream of it.

## Install

Drop the plugin into your Hermes plugins directory:

    cp -R agent_group_chat ~/.hermes/plugins/agent_group_chat

Enable it in `~/.hermes/config.yaml`:

    plugins:
      enabled:
        - '*'
        - agent_group_chat

Then reload the gateway so the plugin loads (run the gateway-restart command from a normal shell, outside any running agent).

## Configure

Add an `agent_group_chat` block to `~/.hermes/config.yaml`. See `config.example.yaml` for a full annotated example. Minimal version:

    agent_group_chat:
      enabled: true
      shared_chats_only: false   # true = only group rooms, not 1:1 DMs
      reply_prefix: '**{agent}**: '
      max_concurrent: 4
      agents:
        hank:
          aliases: hank,builder,coder
          name: Hank
        tommy:
          aliases: tommy,reviewer,review
          name: Tommy

### Config reference

| Key | Default | Purpose |
|-----|---------|---------|
| `enabled` | `true` | Master switch. |
| `shared_chats_only` | `true` | If true, only fire in group/channel rooms. Set `false` to allow 1:1 DMs. |
| `reply_prefix` | `**{agent}**: ` | Prefix on each reply. `{agent}` and `{profile}` are substituted. |
| `max_concurrent` | `4` | Max agents run in parallel per message. |
| `timeout_seconds` | `1800` | Per-agent run timeout. |
| `max_reply_chars` | `12000` | Replies longer than this are truncated. |
| `also_run_current` | `false` | If false, the host agent stays quiet when only other agents are mentioned. |
| `include_default` | `false` | Allow `@default` to address the base profile. |
| `explicit_agents_only` | `false` | Only route to agents listed under `agents:`. |
| `agents` | (auto) | Per-profile names and aliases. Also auto-discovered from `profiles/`. |
| `hermes_command` | `hermes` | Override the CLI binary. |
| `extra_args` | (none) | Extra args for each profile invocation. |

## Usage

In any chat where the plugin is active:

    @hank scaffold a FastAPI service with a /health route
    @tommy what would you watch out for in that design?

Each mentioned agent replies in the same room, in its own voice. Mention several at once and they run in parallel.

## License

MIT. See LICENSE.
