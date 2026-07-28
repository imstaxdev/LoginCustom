# Changelog

All notable changes to LoginCustom are documented here.

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
