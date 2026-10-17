# Changelog

All notable changes to Refrakt are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project
uses [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added

- Library modules: `cors`, `jobs`, `mailer`, `paginate`, `pubsub`,
  `rate_limit`, `telemetry`, `token`, `upload`.
- CLI commands: `refrakt assets` (Tailwind/esbuild setup) and `refrakt seed`.
- `gen resource --belongs-to` for foreign-key relationships.
- Dev error page with error details, stack trace and request info.

### Fixed

- Newly generated projects failed to compile because they pinned
  `mist < 6`, which pulls in an incompatible `glisten` (#1, #3).
- The library failed to compile against `gleam_otp` 1.x.
- `jobs` now enforces `max_concurrency` and survives crashing jobs.
- `token.verify` accepts data that contains dots.
- `upload` strips NUL bytes from filenames.

### Changed

- Generated projects require `mist >= 6.0`, `wisp >= 2.2.2`,
  `lustre >= 5.7` and `refrakt >= 0.2`.
- Library dependencies updated; `gleam_otp` and `gleam_crypto` are now
  direct dependencies, and the unused `glint` dependency was removed.
- Development toolchain pinned to Gleam 1.19.

## [0.2.0] - 2026-03-09

### Added

- JSON API mode for `gen resource`.
- `gen live` with a WebSocket transport.
- Smarter `refrakt migrate` and a file-watching `refrakt dev`.

## [0.1.0] - 2026-03-09

### Added

- `refrakt new` with optional PostgreSQL or SQLite support.
- Generators: `gen page`, `gen resource`, `gen auth`, `gen migration`,
  and `gen island` for Lustre islands.
- `refrakt migrate`, `refrakt dev`, `refrakt routes`, `refrakt build`.
- Library helpers for validation, flash messages, migrations and testing.
- Documentation, tutorial and examples.

[Unreleased]: https://github.com/raskell-io/refrakt/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/raskell-io/refrakt/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/raskell-io/refrakt/releases/tag/v0.1.0
