# Hysteria — Mistgate fork

This is [HyNetworks/hysteria](https://github.com/HyNetworks/hysteria) (formerly apernet/hysteria) at `core/v2.14.0` plus **a few small patches to the
`core` module**, on the branch `mistgate`. Everything else is upstream, unchanged; for what Hysteria is and how to use
it, read the upstream repository and <https://v2.hysteria.network/>. License: MIT, as upstream.

## What is changed (all in `core/`)

- **Connection events** (`server/config.go`, `server/server.go`): `ConnEventLogger`, an optional extension of
  `EventLogger` with `Accepted(conn)`, `ConnConnect(conn, id, tx)` and `ConnDisconnect(conn, id, err)`, so a server
  application gets the QUIC connection itself and can read its statistics.
- **Lost packets** (`internal/congestion/utils.go`, `server/`, `client/client.go`): `server.LostPackets(conn)` and
  the same count in the client's `ConnectionStats()`. quic-go counts lost packets only in its own congestion
  controller; BBR and Brutal are wrapped in a counter.
- **Why a handshake failed** (`server/config.go`, `server/server.go`): `QUICConfig.ConnContext`, passed to
  `quic.Transport.ConnContext`. Its context ends with the close error as the cause, also for a handshake that never
  completed (a timeout, a TLS alert either way). One callback per connection, nothing per packet.
- **Bytes the request hook puts back** (`server/server.go`): `handleTCPRequest` counts them in `LogTraffic`, so they
  are not missing from the user's counted upload.

Nothing changes for an application that does not use the new hooks. The patches are meant to go upstream as pull
requests. Details and how to move them to a newer version: [core/MISTGATE.md](core/MISTGATE.md).

## See the patches

```
git diff core/v2.14.0 mistgate -- core
```

## Who uses it

[Mistgate](https://github.com/Mistgate/mistgate), a VPN panel, builds its nodes with this `core` (it keeps the same
files in `third_party/hysteria-core`). You can use it without Mistgate through a `replace` of
`github.com/apernet/hysteria/core/v2` in your `go.mod`.
