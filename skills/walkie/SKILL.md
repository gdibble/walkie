---
name: walkie
description: P2P communication between AI agents using walkie-sh CLI. Use when the user asks to set up agent-to-agent communication, create a walkie channel, send/receive messages between agents, or enable real-time coordination between multiple AI agents. Triggers on "walkie", "agent communication", "talk to another agent", "set up a channel", "inter-agent messaging", "collaborate with", "coordinate with".
allowed-tools: Bash(walkie:*)
---

# Walkie — Agent-to-Agent Communication

CRITICAL: You MUST run `export WALKIE_ID=<your-name>` as your very first bash command before ANY walkie command. Pick a name based on your role (e.g., alice, bob, coder, reviewer). If you skip this, all messages will silently fail with "delivered to 0 recipients" and the other agent will never receive anything.

## How to use walkie

Step 1. Set your identity — this is MANDATORY, do not skip:
```bash
export WALKIE_ID=alice
```

Step 2. Create or join a channel:
```bash
walkie create <channel> -s <secret>   # if you're first
walkie join <channel> -s <secret>     # if the other agent created it
```

Step 3. Send and read messages:
```bash
walkie send <channel> "your message"
walkie read <channel>                      # non-blocking
walkie read <channel> --wait               # blocks until a message arrives (30s)
walkie read <channel> --wait --timeout 60  # custom timeout
```

Step 4. Clean up when done:
```bash
walkie leave <channel>
```

## Example

```bash
# Alice's terminal
export WALKIE_ID=alice
walkie create room -s secret
walkie send room "hello from alice"

# Bob's terminal
export WALKIE_ID=bob
walkie join room -s secret
walkie read room
# [14:30:05] alice: hello from alice
```

## Behavior to know

- `delivered: 0` means the message is permanently lost — verify `delivered > 0` for critical messages
- `read` drains the buffer — each message returned only once
- Sender never sees their own messages
- Two agents with the same WALKIE_ID share one buffer and will steal each other's messages
- Daemon auto-starts on first command, runs at `~/.walkie/`
- If the daemon crashes, re-join channels (no message persistence)
- Debug logs: `~/.walkie/daemon.log`

## More

- [references/commands.md](references/commands.md) — full command reference
- [references/polling-patterns.md](references/polling-patterns.md) — polling strategies and patterns
- [references/architecture.md](references/architecture.md) — how the daemon works
