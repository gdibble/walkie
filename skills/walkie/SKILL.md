---
name: walkie
description: P2P communication between AI agents using walkie-sh CLI. Use when the user asks to set up agent-to-agent communication, create a walkie channel, send/receive messages between agents, or enable real-time coordination between multiple AI agents. Triggers on "walkie", "agent communication", "talk to another agent", "set up a channel", "inter-agent messaging".
allowed-tools: Bash(walkie:*), Bash(npx walkie-sh:*)
---

# Walkie — Agent-to-Agent P2P Communication

## Core Workflow

Every agent communication follows this pattern:

1. **Create/Join**: Both agents connect to the same channel with a shared secret
2. **Send**: Push messages to the channel
3. **Read**: Pull messages (non-blocking or blocking)
4. **Cleanup**: Leave the channel and stop the daemon when done

```bash
# Agent A
walkie create ops-room -s mysecret
walkie send ops-room "task complete, results at /tmp/output.json"

# Agent B
walkie join ops-room -s mysecret
walkie read ops-room
# [14:30:05] a1b2c3d4: task complete, results at /tmp/output.json
```

## Essential Commands

```bash
# Channel management
walkie create <channel> -s <secret>   # Create a channel and listen for peers
walkie join <channel> -s <secret>     # Join an existing channel
walkie leave <channel>                # Leave a channel
walkie stop                           # Stop the background daemon

# Messaging
walkie send <channel> "message"       # Send a message to all peers
walkie read <channel>                 # Read pending messages (non-blocking)
walkie read <channel> --wait          # Block until a message arrives (default 30s)
walkie read <channel> -w -t 60        # Block with custom timeout (seconds)

# Status
walkie status                         # Show active channels, peers, buffered messages
```

## Common Patterns

### Two-Agent Collaboration

```bash
# Agent A (coordinator)
walkie create task-room -s sharedsecret
walkie send task-room "analyze /data/users.csv and report top 10"
walkie read task-room --wait --timeout 120   # Wait for result

# Agent B (worker)
walkie join task-room -s sharedsecret
walkie read task-room --wait                 # Get the task
# ... do the work ...
walkie send task-room "done: top 10 users saved to /tmp/report.txt"
```

### Polling Between Task Steps

Check for messages between work steps to receive course corrections, new data, or stop signals mid-operation.

```bash
walkie join task-channel -s secret
# Step 1: do work
walkie read task-channel              # Non-blocking check
# Step 2: do more work
walkie read task-channel              # Non-blocking check
# Step 3: report completion
walkie send task-channel "all steps complete"
```

### Hub-and-Spoke (One Coordinator, Many Workers)

```bash
# Coordinator creates a shared channel
walkie create dispatch -s teamsecret

# Workers all join the same channel
walkie join dispatch -s teamsecret    # Worker 1
walkie join dispatch -s teamsecret    # Worker 2

# Coordinator broadcasts tasks
walkie send dispatch "worker-1: process batch A"
walkie send dispatch "worker-2: process batch B"

# Workers poll for their assignments
walkie read dispatch
```

### Blocking Wait for Response

```bash
walkie send task-room "what is the status?"
walkie read task-room --wait --timeout 60
# Blocks until a reply arrives or 60 seconds elapse
```

## Key Details

- **Daemon auto-starts** on first command, runs in background at `~/.walkie/`
- **Messages buffer locally** — `walkie read` drains the buffer, each message returned once
- **Channel = hash(name + secret)** — both sides must use the same name and secret
- **Encrypted** — all P2P connections use the Noise protocol via Hyperswarm
- **Peer discovery** takes 1–15 seconds via DHT
- **No server** — fully peer-to-peer, works across machines and networks
- **`walkie read` output format**: `[HH:MM:SS] <sender-id>: <message>`

## Cleanup

Always clean up when done to avoid leaked daemon processes:

```bash
walkie leave <channel>   # Leave specific channel
walkie stop              # Stop the daemon entirely
```

## Deep-Dive Documentation

| Reference | When to Use |
|-----------|-------------|
| [references/commands.md](references/commands.md) | Full command reference with all options and output formats |
| [references/architecture.md](references/architecture.md) | How the daemon, IPC, and P2P layers work |
| [references/polling-patterns.md](references/polling-patterns.md) | Agent polling strategies, multi-agent coordination patterns |

## Ready-to-Use Templates

| Template | Description |
|----------|-------------|
| [templates/two-agent-collab.sh](templates/two-agent-collab.sh) | Coordinator sends task, worker executes and reports back |
| [templates/delegated-task.sh](templates/delegated-task.sh) | Delegate work to another agent and wait for result |
| [templates/monitoring.sh](templates/monitoring.sh) | Monitor agent activity from a separate terminal |
