# LoginCustom

Secure, modern authentication for offline-mode and mixed-mode Minecraft servers.

[![GitHub release](https://img.shields.io/github/v/release/Imstaxdev/LoginCustom?include_prereleases&label=latest)](https://github.com/Imstaxdev/LoginCustom/releases)
[![Modrinth](https://img.shields.io/modrinth/dt/W5LtRXFX?logo=modrinth&label=Modrinth)](https://modrinth.com/plugin/logincustom)
[![License](https://img.shields.io/badge/license-MIT-2ea44f.svg)](LICENSE)
[![Minecraft](https://img.shields.io/badge/Minecraft-1.8.8--26.x-62b47a)](https://modrinth.com/plugin/logincustom/versions)

LoginCustom protects player accounts against username impersonation. Players register once and authenticate whenever they reconnect; until authentication succeeds, movement and sensitive gameplay actions remain locked.

It works on standalone servers and proxy networks, supports verified Premium accounts, and provides dedicated builds for Bukkit, Spigot, Paper, Purpur, Folia, Velocity and BungeeCord.

## Highlights

- Secure registration and login with Argon2id password hashing
- Verified Premium login through Minecraft's official session handshake
- Optional TOTP two-factor authentication and one-use recovery codes
- SQLite for standalone servers and MariaDB for synchronized networks
- Dedicated Paper, Bukkit, Folia, Velocity and BungeeCord artifacts
- English and Spanish messages with configurable restrictions and timeouts
- Configurable login spawn with `/setlogin`
- Offline account administration with `/unregister` and `/unpremium`
- Asynchronous, bounded authentication and database work
- Fail-closed proxy sessions with signed, replay-protected messages

## Downloads

Download LoginCustom from [Modrinth](https://modrinth.com/plugin/logincustom/versions) or [GitHub Releases](https://github.com/Imstaxdev/LoginCustom/releases).

Always choose the file that matches both your Minecraft version and the software that starts the server:

| Server software | Required file |
| --- | --- |
| Paper or Purpur | `LoginCustom-Paper.jar` |
| Folia | `LoginCustom-Folia.jar` |
| CraftBukkit or Spigot | `LoginCustom-Bukkit.jar` |
| Velocity | `LoginCustom-Velocity.jar` |
| BungeeCord | `LoginCustom-Bungee.jar` |

> A proxy JAR does not replace the backend plugin. Velocity and BungeeCord networks require the matching proxy JAR plus the correct Paper, Folia or Bukkit JAR on every backend.

## Version lines

| Release line | Minecraft | Java | Status |
| --- | --- | --- | --- |
| Legacy `1.0.0` | `1.8.8–1.15.2` | Java 8 | Stable legacy release |
| Downgrade Beta | `1.16.x–1.20.x` | Depends on Minecraft version | Active beta |
| Modern Beta 1.21 | `1.21.x` | Java 21 | Active beta |
| Modern Beta 26 | `26.x` | Java 25 | Active beta |

The 1.21.x and 26.x builds are separate so each line can use its native Java and platform capabilities. Never mix JARs from different release lines.

### Modern platform support

- Paper and compatible Paper forks use `LoginCustom-Paper.jar`.
- CraftBukkit and Spigot use `LoginCustom-Bukkit.jar`.
- Folia uses its dedicated `LoginCustom-Folia.jar`; do not substitute the Paper JAR.
- Velocity and BungeeCord use their matching proxy artifact and require LoginCustom on every backend.
- The Downgrade line does not currently declare Folia support.

Modrinth only enables combinations for which a matching artifact is published. If a Minecraft version or loader appears unavailable, select a supported combination rather than forcing a different JAR.

## Standalone installation

1. Stop the server.
2. Download the artifact matching your Minecraft version and server software.
3. Put exactly one backend JAR in `plugins/`.
4. Start the server once to generate `plugins/LoginCustom/config.yml`.
5. Review the generated security, storage and message settings, then restart.

SQLite is the default storage for standalone servers:

```yaml
spawn-login: true

storage:
  type: sqlite
  sqlite:
    file: accounts.db
```

When `spawn-login` is enabled, connecting players are moved to the point saved with `/setlogin`. If no custom point exists, LoginCustom uses the primary world spawn. Disabling the option preserves the saved location.

## Proxy network installation

```text
Velocity or BungeeCord
          |
          +-- Paper / Purpur / Folia backend
          +-- Bukkit / Spigot backend
```

1. Install the matching LoginCustom proxy JAR.
2. Install the matching backend JAR on every Minecraft server.
3. Configure the same MariaDB database on the proxy and all backends.
4. Configure the same strong network secret at both layers.
5. Enable network mode on every backend.
6. Isolate backend ports with private networking or a firewall.
7. Restart the entire network.

MariaDB is mandatory for proxy deployments. Never expose an offline-mode backend directly to the internet: signed plugin messages protect authentication state, but they do not replace network isolation.

## Commands

| Command | Purpose |
| --- | --- |
| `/register <password> <confirmation>` | Create a LoginCustom account |
| `/login <password>` | Authenticate the current connection |
| `/changepassword <current> <new> <confirmation>` | Change the account password |
| `/premium <current-password>` | Verify and link an official Minecraft account |
| `/2fa <code>` | Complete a two-factor-protected login |
| `/2fa setup <current-password>` | Begin TOTP enrollment |
| `/2fa confirm <code>` | Confirm enrollment and generate recovery codes |
| `/2fa disable <current-password> <code>` | Disable TOTP after verifying both factors |
| `/setlogin` | Save the pre-authentication login location |
| `/unregister <player>` | Delete a registered account, online or offline |
| `/unpremium <player>` | Remove Premium status, online or offline |
| `/logincustom reload` | Reload safe configuration and messages |
| `/logincustom status` | Show runtime and storage status |
| `/logincustom doctor` | Run storage, security and scheduler diagnostics |

Aliases: `/reg`, `/log`, `/cpw` and `/lc`.

## Permissions

Player authentication permissions are available to everyone. Administrative permissions are operator-only by default.

| Permission | Default |
| --- | --- |
| `logincustom.command.login` | Everyone |
| `logincustom.command.register` | Everyone |
| `logincustom.command.changepassword` | Everyone |
| `logincustom.command.premium` | Everyone |
| `logincustom.admin.unregister` | Operators |
| `logincustom.admin.unpremium` | Operators |
| `logincustom.admin.setlogin` | Operators |
| `logincustom.admin.reload` | Operators |
| `logincustom.admin.status` | Operators |
| `logincustom.admin.*` | Operators |

## Security design

- Passwords are hashed with Argon2id and random salts; plaintext passwords are never stored.
- Authentication and database operations run outside the Minecraft tick thread with bounded concurrency.
- Database queries are parameterized and schema changes are versioned.
- Password changes, Premium changes and account removal revoke active sessions.
- Premium status requires proof from Minecraft's official session service; a username or public UUID lookup is never accepted as ownership proof.
- Proxy communication uses HMAC-SHA256 signatures, timestamps, nonces and replay protection.
- Invalid, expired, duplicated or incorrectly signed network messages are rejected.
- Players remain blocked if MariaDB or trusted proxy communication becomes unavailable.
- External password peppers support versioned rotation and database identity validation.

Use a dedicated MariaDB account with a strong password, restrict its network access and maintain regular backups.

## Support and issue reports

Before reporting a problem, run `/logincustom doctor` and keep the relevant startup log. Reports should include the LoginCustom version, Minecraft version, Java version, platform name, standalone or proxy mode, and the diagnostic result. Never publish passwords, database credentials, network secrets, TOTP secrets or recovery codes.

Report reproducible issues through [GitHub Issues](https://github.com/Imstaxdev/LoginCustom/issues).

## License

LoginCustom is distributed under the [MIT License](LICENSE). Bundled third-party components retain their own licenses.
