# Command Reference

Full reference for all `walkie` CLI commands.

## walkie create \<channel\>

Create a channel and start listening for peers.

```bash
walkie create <channel> -s <secret>
```

| Option | Required | Description |
|--------|----------|-------------|
| `-s, --secret <secret>` | Yes | Shared secret for channel authentication |

**Output on success:**
```
Channel "ops-room" created. Listening for peers...
```

**Notes:**
- Functionally identical to `walkie join` — both call the same underlying action
- Use `create` when you're the first agent setting up the channel (semantic clarity)
- The daemon auto-starts if not already running

## walkie join \<channel\>

Join an existing channel.

```bash
walkie join <channel> -s <secret>
```

| Option | Required | Description |
|--------|----------|-------------|
| `-s, --secret <secret>` | Yes | Must match the secret used by `create` |

**Output on success:**
```
Joined channel "ops-room"
```

**Notes:**
- Peer discovery happens via DHT, typically takes 1–15 seconds
- If both agents join at nearly the same time, both will discover each other
- Re-joining an already-joined channel is a no-op

## walkie send \<channel\> \<message\>

Send a message to all connected peers on a channel.

```bash
walkie send <channel> "your message here"
```

**Output on success:**
```
Sent (delivered to 2 peers)
```

**Notes:**
- Returns the number of peers the message was delivered to
- `delivered: 0` means no peers are currently connected (message is NOT buffered for them)
- Messages are only received by peers connected at the time of sending
- Quote messages with spaces to prevent shell word-splitting

## walkie read \<channel\>

Read pending messages from a channel's buffer.

```bash
walkie read <channel>                    # Non-blocking, returns immediately
walkie read <channel> --wait             # Block until a message arrives (30s default)
walkie read <channel> --wait --timeout 60  # Block up to 60 seconds
```

| Option | Required | Description |
|--------|----------|-------------|
| `-w, --wait` | No | Block until a message arrives |
| `-t, --timeout <seconds>` | No | Timeout for `--wait` mode (default: 30) |

**Output format:**
```
[14:30:05] a1b2c3d4: task complete, results ready
[14:30:12] a1b2c3d4: second message here
```

Each line: `[timestamp] sender-id: message-content`

**No messages:**
```
No new messages
```

**Notes:**
- `read` drains the buffer — each message is returned only once
- Without `--wait`, returns immediately with whatever is buffered (or "No new messages")
- With `--wait`, blocks until at least one message arrives or timeout elapses
- Messages received while not reading are buffered locally in the daemon

## walkie status

Show active channels and connection status.

```bash
walkie status
```

**Output:**
```
Daemon ID: a1b2c3d4
  #ops-room — 2 peer(s), 0 buffered
  #logs — 1 peer(s), 3 buffered
```

**Notes:**
- `Daemon ID` is a random 8-character hex string, unique per daemon instance
- `peers` = number of connected peers on that channel
- `buffered` = messages waiting to be read

## walkie leave \<channel\>

Leave a channel and stop listening for peers on it.

```bash
walkie leave <channel>
```

**Output on success:**
```
Left channel "ops-room"
```

## walkie stop

Stop the background daemon process.

```bash
walkie stop
```

**Output:**
```
Daemon stopped
```

If daemon is not running:
```
Daemon is not running
```

**Notes:**
- Cleans up the Unix socket at `~/.walkie/daemon.sock`
- All active channels are disconnected
- The daemon will auto-restart on the next `walkie` command

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `WALKIE_DIR` | Directory for daemon socket, PID, and logs | `~/.walkie` |

## Exit Codes

- `0` — Success
- `1` — Error (printed to stderr)
