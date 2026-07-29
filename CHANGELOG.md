# Changelog

All notable changes to LoginCustom are documented here.

## 1.0.0-beta.1-mc1.16-1.20

### Added

- Dedicated Downgrade beta distribution for Minecraft `1.16.x–1.20.x`.
- Separate Bukkit and Paper entry points with a modern `api-version: 1.16`
  descriptor.
- Modern pre-authentication restrictions for block placement, block breaking,
  item pickup, off-hand swaps, armor stands and precise entity interaction.

### Validated

- Paper `1.16.5` with Java 16.
- Paper `1.18.2` with Java 17.
- Paper `1.20.6` with Java 21.
- Bukkit adapter through the Bukkit API on Paper `1.20.6`.
- Velocity and BungeeCord startup with MariaDB.
- Reproducible builds and real SQLite/MariaDB integration.

## 1.0.0-beta.1-mc26

### Added

- Dedicated Minecraft `26.x` distributions for Paper, Bukkit, Velocity and BungeeCord.
- Native Java 25 adapter and proxy bytecode.
- Paper `26.2` API adapter with Adventure-native messages and disconnects.

### Changed

- Removed SQLite JDBC from the Paper and Bukkit JAR payloads.
- Paper uses its bundled SQLite driver.
- Bukkit resolves SQLite through its platform library loader.
- Isolated current event handling from the Legacy and `1.16–1.20` adapters.
- Replaced manual fat-JAR assembly with Shadow in all four distributions.
- Relocated MariaDB, HikariCP and Password4j into LoginCustom's private
  namespace on Paper, Bukkit, Velocity and BungeeCord.
- Merged and rewrote JDBC service descriptors for the relocated MariaDB driver.
- Updated the Velocity baseline to API and runtime `4.1.0`.

### Validated

- Paper and Bukkit adapters on Paper `26.2` with Java 25 and SQLite.
- Paper `26.2`, Velocity `4.1` and BungeeCord with Java 25 and MariaDB.
- Reproducible builds and zero bundled SQLite classes.

## 1.0.0-beta.1-mc1.21

### Added

- Dedicated Minecraft `1.21.x` distributions for Paper, Bukkit, Velocity and BungeeCord.
- Native Java 21 adapter and proxy bytecode.
- Paper-specific Adventure chat handling and Bukkit-compatible fallback handling.
- Paper API baseline updated to `1.21.11`.
- Velocity API baseline updated to the Java 21-compatible `3.6.0` line.
- MariaDB, HikariCP and Password4j relocated in every distribution.

### Validated

- Paper and Bukkit adapters on Paper `1.21.11` with Java 21 and SQLite.
- Paper `1.21.11`, Velocity `3.6` and BungeeCord with Java 21 and MariaDB.
- Real SQLite and MariaDB storage tests.
- Reproducible Java 21 JARs with zero bundled SQLite classes.

## Legacy 1.0.0

### Stable release

- Promoted the Minecraft `1.8.8–1.15.2` line after clean startup validation on
  Paper `1.8.8`, `1.12.2`, `1.15.2` and Spigot `1.8.8`.
- Revalidated Velocity and BungeeCord startup against a real MariaDB instance.
- Added explicit block-break and block-place restrictions before login.
- Rejected overlapping sensitive authentication operations per connection.
- Prevented stale asynchronous results from authenticating a reconnected player.
- Replaced unbounded proxy work queues with bounded, fail-closed queues.
- Made proxy session updates conditional on the exact active connection.
- Verified reproducible JARs and scanned runtime logs for sensitive values.

## 0.1.0-alpha.3

### Added

- Dedicated Bukkit and Paper adapters for Minecraft `1.16.x–1.20.x`.
- Modern restrictions for item pickup, off-hand swaps, armor stands and precise
  entity interaction.
- Modern plugin descriptors with `api-version: 1.16`.

### Validated

- Paper `1.16.5`, `1.18.2` and `1.20.6`.
- Bukkit adapter on a Paper `1.20.6` runtime.
- Velocity and BungeeCord with MariaDB.

## 0.1.0-alpha.2

### Added

- Native selective Premium verification on Velocity and BungeeCord.
- Secure two-step `/premium <current-password>` activation.
- `/unpremium <player>` for registered online or offline accounts.
- Premium auto-login with an explicit success message.
- Authentication reminders at 10 and 5 seconds before the default 15-second timeout.
- Versioned account migration from Alpha 1.
- Credential versions and sensitive-change timestamps.
- Periodic fail-closed validation of proxy sessions.

### Changed

- `/unregister <player>` now works with online or offline registered accounts.
- Password, Premium and account changes revoke active network sessions.
- Proxy-backed players no longer see a temporary `/login` prompt before their
  signed Premium session is verified.
- Bukkit and Paper remain self-contained with the complete SQLite JDBC runtime.

### Security

- Premium ownership is proven through Minecraft's official connection
  handshake; names and offline UUIDs are never accepted as ownership proof.
- Official UUID conflicts are rejected.
- Invalidated or missing proxy sessions disconnect the player.
- Passwords, complete hashes, secrets and reusable tokens are excluded from logs.

## 0.1.0-alpha.1

- Initial Legacy authentication release.
- Bukkit, Paper, Velocity and BungeeCord distributions.
- Argon2id password hashing.
- SQLite standalone storage and MariaDB network storage.
- Pre-authentication gameplay restrictions.
- HMAC-signed LCN/1 proxy-to-backend communication.
