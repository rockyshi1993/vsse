# Changelog

All notable changes to this project will be documented in this file.

The format is based on Keep a Changelog, and this project adheres to Semantic Versioning.

## [Unreleased]
### Added
- destroy(): clean up global event listeners and timers to prevent memory leaks.
- Event parsing compatibility: if JSON has no `event` field, fall back to SSE native `ev.type`.
- Docs: Expand README with heartbeat, backoff, sseWithCredentials, per-call overrides, lifecycle, FAQ and troubleshooting.

### Changed
- Idle strategy: do not close SSE due to user inactivity when there are active listeners; receiving messages counts as activity.
- Default `sseWithCredentials` is now `false`. Enable explicitly when the server allows credentials.

### Fixed
- JSDoc: `SSEClientOptions` now documents `sseWithCredentials`.

