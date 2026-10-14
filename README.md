# IT-Kamianets 3D Engine: Network Schema

`3d-network-schema` defines the engine-independent format for multiplayer sessions in the **IT-Kamianets 3D Engine ecosystem**: who is in a session, what happened in it, and how it ended.

It is the shared contract between the Unity game, a dedicated server in any language, web spectators, stats storage, bots, and AI tools.

### Logic

`3d-scene-schema` says where things are. This schema says what happens between players.

It covers the messages that **matter after the fact**: players joining and leaving, the session moving from lobby to match to result, and the game's own events (a round ending, a player defeated). A server written in Go or Node, a browser spectator, or Firebase stats can all read them without Unity.

It deliberately does **not** cover the high-frequency stream: positions and inputs sent every tick. That traffic must be tiny binary data and belongs to the networking backend used by `3d-unity-network`.

Like every engine package, it holds no game rules. Game events use names namespaced by the game (`temnorid-arena.round-ended`) and carry game-defined data; the `session.*` namespace is reserved for this schema.

**one session → Unity / server / web / stats / AI**

### Schema v1

Files live under [`schema/v1/`](schema/v1/), written as JSON Schema (2020-12), the same as `3d-scene-schema`.

* [`session.schema.json`](schema/v1/session.schema.json): root document. `sessionId`, game `mode`, the `sceneId` of a 3d-scene-schema scene, lifecycle `state` (`lobby`, `starting`, `running`, `ended`), teams, and players (`player`, `spectator`, `bot`).
* [`message.schema.json`](schema/v1/message.schema.json): one message. An envelope (`messageId`, `sessionId`, `seq`, optional `tick`, `sentAt`, `from`, `type`) and a payload typed by `type`.
* [`result.schema.json`](schema/v1/result.schema.json): how a session ended. Reason, winners, and team or player scores with game-defined stats.
* [`replay.schema.json`](schema/v1/replay.schema.json): a finished session as one document. The session as it started, then every server message in order.
* [`common.schema.json`](schema/v1/common.schema.json): shared primitives (`Id`, `Timestamp`, `GameName`, `Data`, `Metadata`).

**Message types:**

| Type | From | Payload |
|---|---|---|
| `session.join` | client | display name, player or spectator |
| `session.joined` | server | the player as added to the session |
| `session.left` | server | playerId and reason (`quit`, `disconnected`, `kicked`) |
| `session.snapshot` | server | the whole session, for a player who joins or reconnects |
| `session.state-changed` | server | new state, and the scene when one is chosen |
| `session.ended` | server | the result |
| `<game>.<name>` | either | game-defined data |

**The server is the authority.** Every server message has a `seq` that starts at 1 and has no gaps, so a client or a replay can tell a message is missing. A client message has no `seq`: it is only a request until the server sends it on as its own message. Only the server may send `session.*` messages other than `session.join`.

**Versioning:** every document carries a `schemaVersion` (semver). Breaking changes get a new `schema/v{n}/` directory rather than changing v1 in place.

Example instances: [`examples/arena-replay.example.json`](examples/arena-replay.example.json) (a short Temnorid Arena match: rounds, a disconnect and reconnect, the result) and [`examples/join-request.example.json`](examples/join-request.example.json) (a client asking to join).

### Status

Schema v1 draft. The examples validate against it. No engine models yet.

### Roadmap

* Session, message, result and replay documents
* Chat and voice-channel messages
* Matchmaking tickets
* Reconnect tokens
* Binary encoding of the same messages (MessagePack)
* TypeScript models
* C# models in `3d-unity-network`
* Stable Network Schema v1
