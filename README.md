# LoginCustom

Secure authentication for Minecraft servers and proxy networks.

[![Release](https://img.shields.io/github/v/release/Imstaxdev/LoginCustom?include_prereleases&label=release)](https://github.com/Imstaxdev/LoginCustom/releases)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Minecraft](https://img.shields.io/badge/Minecraft-1.8.8--26.x-62b47a)](https://github.com/Imstaxdev/LoginCustom/releases)

LoginCustom protects offline-mode servers against username impersonation. Players register once and authenticate on every new connection. Until authentication succeeds, gameplay remains restricted.

## Downloads

Each release is built for a specific Minecraft generation. Do not mix JARs from different release lines.

### Which file should I download?

1. Select your **exact Minecraft version** on Modrinth.
2. Select the software that actually starts your server.
3. Download the file offered for that exact combination.

| Your server software | File to install |
| --- | --- |
| Paper or Purpur | `LoginCustom-Paper.jar` |
| Folia | `LoginCustom-Folia.jar` |
| CraftBukkit or Spigot | `LoginCustom-Bukkit.jar` |
| Velocity proxy | `LoginCustom-Velocity.jar` |
| BungeeCord proxy | `LoginCustom-Bungee.jar` |

The Velocity and BungeeCord files do not replace the backend plugin. A proxy
network needs the proxy JAR **and** the matching Paper, Folia or Bukkit JAR on
every backend server.

### Why are some Modrinth versions shown in red?

Modrinth filters downloads by the exact combination of Minecraft version and
loader. A red or unavailable version means LoginCustom does not currently
publish a file for that exact combination. It does not indicate a problem with
your Modrinth account or the download page.

The current Folia beta is declared for:

- `1.21.4`, `1.21.5`, `1.21.6`, `1.21.8` and `1.21.11`
- `26.2`

Other Folia patch versions are not currently claimed as supported. If your
exact Folia version is unavailable, do not force a different LoginCustom JAR
and do not install the Paper JAR on Folia. Update the server to a listed Folia
version, use Paper/Purpur with the Paper JAR, or wait for a matching build.

| Release line | Minecraft | Java | Backend files |
| --- | --- | --- | --- |
| Legacy `1.0.0` | `1.8.8–1.15.2` | Java 8 | `LoginCustom-Paper-Legacy.jar`, `LoginCustom-Bukkit-Legacy.jar` |
| Downgrade Beta | `1.16.x–1.20.x` | Version-dependent | `LoginCustom-Paper.jar`, `LoginCustom-Bukkit.jar` |
| Modern Beta 1.21 | `1.21.x` | Java 21 | `LoginCustom-Paper.jar`, `LoginCustom-Bukkit.jar`, `LoginCustom-Folia.jar` |
| Modern Beta 26 | `26.x` | Java 25 | `LoginCustom-Paper.jar`, `LoginCustom-Bukkit.jar`, `LoginCustom-Folia.jar` |

Proxy networks additionally require `LoginCustom-Velocity.jar` or `LoginCustom-Bungee.jar` from the same release line. Download files from [GitHub Releases](https://github.com/Imstaxdev/LoginCustom/releases).

## Features

- Argon2id password hashing with random salts
- SQLite for standalone servers and MariaDB for networks
- English and Spanish messages
- Configurable password policy, timeout and attempt limits
- Bounded asynchronous authentication workers
- Gameplay restrictions until authentication succeeds
- HMAC-SHA256 signed proxy-to-backend communication
- Timestamp, nonce and replay protection
- Fail-closed network sessions
- Offline account administration with `/unregister` and `/unpremium`
- Selective Premium verification through Minecraft's official login handshake
- Session revocation after password, Premium or account changes
- Versioned external password pepper keyring with database identity checks
- Optional TOTP two-factor authentication with one-use recovery codes
- Region-aware Folia scheduling through dedicated Folia artifacts

## Platform compatibility

### Beta 1.21

- Minecraft `1.21.x`
- Java 21 bytecode
- Paper/Purpur through the Paper JAR
- CraftBukkit/Spigot through the Bukkit JAR
- Folia through the dedicated Folia JAR
- Directly started on Paper `1.21.11`
- Folia `1.21.11` build 14 validated with registration, login and timeout flows
- Velocity `3.6.x` through the Java 21 proxy JAR

### Beta 26

- Minecraft `26.x`
- Java 25 bytecode
- Paper/Purpur through the Paper JAR
- CraftBukkit/Spigot through the Bukkit JAR
- Folia through the dedicated Folia JAR
- Directly started on Paper `26.2`
- Folia `26.2` build 1 validated for startup, diagnostics and clean shutdown
- Velocity `4.1.x` through the Java 25 proxy JAR

The two beta lines are separate on purpose: the 26.x build can use the current Java runtime without reducing its bytecode target for older servers. The Downgrade Beta does not currently declare Folia support; use the Folia JAR only with its matching 1.21.x or 26.x line.

### Proxies

- Velocity and BungeeCord-compatible builds are provided for each beta line.
- Proxy bytecode follows its release line: Java 21 for Beta 1.21 and Java 25
  for Beta 26.
- MariaDB is mandatory for proxy networks.
- Velocity modern forwarding requires a compatible Paper backend.
- Every backend must be inaccessible from the public network.

## Standalone installation

1. Stop the server.
2. Download the release matching your Minecraft and Java version.
3. Place exactly one backend JAR in `plugins/`.
4. Start the server once to generate `plugins/LoginCustom/config.yml`.
5. Review the configuration and restart the server.

Standalone servers use SQLite by default:

```yaml
spawn-login: true

storage:
  type: sqlite
  sqlite:
    file: accounts.db
```

Paper already supplies its SQLite driver. The compact Bukkit beta downloads `org.xerial:sqlite-jdbc` through Bukkit's library loader on first startup, so that first start requires repository access.

All modern distributions relocate MariaDB, HikariCP and Password4j into
LoginCustom's private namespace. This prevents dependency collisions without
bundling SQLite inside the JAR.

## Proxy network installation

```text
Velocity or BungeeCord
          |
          +--> Paper/Purpur/Folia backend
          +--> Bukkit/Spigot backend
```

1. Install the matching Velocity or Bungee JAR on the proxy.
2. Install the matching backend JAR on every Minecraft server.
3. Configure the same MariaDB database on the proxy and backends.
4. Copy the same network secret to both layers.
5. Enable network mode on every backend.
6. Isolate backend ports using private networking or a firewall.
7. Restart the entire network.

Never expose an offline-mode backend directly to the internet. Signed LoginCustom messages protect authentication state; they do not replace network isolation.

## Commands

| Command | Description |
| --- | --- |
| `/register <password> <confirmation>` | Register a new account |
| `/login <password>` | Authenticate the current session |
| `/changepassword <current> <new> <confirmation>` | Change the password |
| `/premium <current-password>` | Verify and link the official account |
| `/2fa <code>` | Complete a login protected by two-factor authentication |
| `/2fa setup <current-password>` | Start TOTP enrollment |
| `/2fa confirm <code>` | Confirm TOTP enrollment and create recovery codes |
| `/2fa disable <current-password> <code>` | Disable TOTP after both factors are verified |
| `/setlogin` | Save the location used when players connect before authentication |
| `/unregister <player>` | Delete a registered account |
| `/unpremium <player>` | Disable Premium for a registered account |
| `/logincustom reload` | Reload safe configuration and messages |
| `/logincustom status` | Show plugin and storage status |
| `/logincustom doctor` | Run storage, security and platform scheduler diagnostics |

Aliases: `/reg`, `/log`, `/cpw` and `/lc`.

## Permissions

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

## Security model

- Passwords are never stored in plaintext.
- Authentication and database work stay off the Minecraft main thread.
- Queries are parameterized.
- Invalid, expired, duplicated or incorrectly signed network messages are rejected.
- Network sessions stay blocked if MariaDB or signed communication is unavailable.
- Premium status is accepted only after Minecraft's official session service verifies the connection.
- Names, offline UUIDs and public profile lookups are never treated as proof of ownership.

Use a dedicated MariaDB user, strong unique credentials and regular backups.

When `spawn-login: true` (the default), backend players are moved to the
location saved by `/setlogin` as they connect. Until a custom point is saved,
LoginCustom uses the primary world's spawn. The location is stored separately
in `plugins/LoginCustom/login-spawn.yml`; setting the option to `false` disables
the connection teleport without deleting the saved point.

## License

LoginCustom is distributed under the [MIT License](LICENSE). Third-party components retain their own licenses.
