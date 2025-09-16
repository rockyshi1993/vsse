# Changelog

All notable changes to this project will be documented in this file.

The format is based on Keep a Changelog, and this project adheres to Semantic Versioning.

## [Unreleased]
### Added
- Global broadcast listeners: `onBroadcast(cb)` to receive messages without `requestId` (e.g., system announcements). Returns `unsubscribe`.
- destroy(): clean up global event listeners and timers to prevent memory leaks.
- Event parsing compatibility: if JSON has no `event` field, fall back to SSE native `ev.type`.
- Docs: Expanded README with heartbeat, backoff, sseWithCredentials, per-call overrides, lifecycle, FAQ and troubleshooting; added global broadcast docs and examples.

### Changed
- dispatch(): now forwards the full message object to callbacks (all top-level fields are preserved). Backward compatible.
- Lazy connection and idle-close now consider both per-request listeners and global broadcast listeners.
- Default `sseWithCredentials` is now `false`. Enable explicitly when the server allows credentials.

### Fixed
- JSDoc: `SSEClientOptions` now documents `sseWithCredentials`.

