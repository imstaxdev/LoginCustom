# LoginCustom

Secure, lightweight authentication for Minecraft servers and proxy networks.

[![Release](https://img.shields.io/github/v/release/Imstaxdev/LoginCustom?include_prereleases&label=release)](https://github.com/Imstaxdev/LoginCustom/releases)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Minecraft](https://img.shields.io/badge/Minecraft-1.8.8--1.15.2-62b47a)](https://github.com/Imstaxdev/LoginCustom/releases)

LoginCustom protects offline-mode servers against username impersonation. New players register an account once, then authenticate on every new connection. Until authentication succeeds, gameplay actions remain restricted.

## Downloads

Choose one backend JAR. Proxy networks also require the matching proxy JAR.

| File | Use it on |
| --- | --- |
| `LoginCustom-Paper-Legacy.jar` | Paper, Purpur and compatible Paper forks |
| `LoginCustom-Bukkit-Legacy.jar` | CraftBukkit, Spigot and compatible Bukkit forks |
| `LoginCustom-Velocity.jar` | Velocity proxy |
| `LoginCustom-Bungee.jar` | BungeeCord and Waterfall-compatible proxies |

Download the latest files from [GitHub Releases](https://github.com/Imstaxdev/LoginCustom/releases).

## Features

- Secure registration and login flow
- Argon2id password hashing with random salts
- SQLite storage for standalone servers
- MariaDB storage for proxy networks
- English and Spanish messages
- Configurable password policy, timeouts and attempt limits
- Asynchronous password hashing with bounded workers
- Protection against movement, chat, commands, inventory access, interaction, damage, vehicles and teleportation before login
- Signed proxy-to-backend communication using HMAC-SHA256
- Timestamp, nonce and replay protection for network messages
- No persistent auto-login: authentication ends when the player disconnects

## Compatibility

### Legacy backends

- Minecraft `1.8.8–1.15.2`
- Java 8 bytecode
- Paper, CraftBukkit, Spigot and compatible forks; Purpur where a build exists for the selected version

The primary compatibility targets are `1.8.8`, `1.12.2` and `1.15.2`. Intermediate Legacy versions use the same compatible API surface.

### Proxies

- Velocity: Java 21
- BungeeCord/Waterfall: Java 8-compatible plugin bytecode
- MariaDB is required for all proxy networks

Velocity modern forwarding requires a Paper `1.13+` backend. Minecraft `1.8–1.12` networks must use legacy-compatible forwarding and keep every backend isolated from direct connections.

## Standalone installation

1. Stop the Minecraft server.
2. Place exactly one backend JAR in `plugins/`:
   - Paper or Purpur: `LoginCustom-Paper-Legacy.jar`
   - Bukkit or Spigot: `LoginCustom-Bukkit-Legacy.jar`
3. Start the server once to generate the configuration.
4. Edit `plugins/LoginCustom/config.yml`.
5. Restart the server completely.

Standalone installations use SQLite by default:

```yaml
storage:
  type: sqlite
  sqlite:
    file: accounts.db
```

On legacy Spigot and Paper servers, disable complete command logging so passwords entered in authentication commands are not written by the server:

```yaml
commands:
  log: false
```

This option belongs in `spigot.yml`.

## Proxy network installation

A network requires LoginCustom on both layers:

```text
Velocity or BungeeCord
          |
          +--> Paper/Purpur backend
          +--> Bukkit/Spigot backend
```

1. Install `LoginCustom-Velocity.jar` or `LoginCustom-Bungee.jar` on the proxy.
2. Install the appropriate backend JAR on every Minecraft server.
3. Configure the same MariaDB database on the proxy and all backends.
4. Use the same `network.proxy-id` and shared secret on both layers.
5. Enable network mode on every backend.
6. Bind backend ports to a private interface or protect them with a firewall.
7. Restart the complete network.

Example backend network configuration:

```yaml
storage:
  type: mariadb
  mariadb:
    jdbc-url: jdbc:mariadb://127.0.0.1:3306/logincustom
    username: logincustom
    password: replace-with-a-strong-password

network:
  enabled: true
  backend-name: survival
  proxy-id: proxy-1
  shared-secret: replace-with-at-least-32-random-bytes
```

Never expose an offline-mode backend directly to the internet. The signed LoginCustom channel protects authentication state, but it does not replace network-level isolation.

## Commands

| Command | Description |
| --- | --- |
| `/register <password> <confirmation>` | Register a new account |
| `/login <password>` | Authenticate the current session |
| `/changepassword <current> <new> <confirmation>` | Change the account password |
| `/unregister <player>` | Remove an account registration |
| `/logincustom reload` | Reload safe configuration and messages |
| `/logincustom status` | Display plugin and storage status |

Aliases: `/reg`, `/log`, `/cpw` and `/lc`.

## Permissions

| Permission | Default |
| --- | --- |
| `logincustom.command.login` | Everyone |
| `logincustom.command.register` | Everyone |
| `logincustom.command.changepassword` | Everyone |
| `logincustom.admin.unregister` | Operators |
| `logincustom.admin.reload` | Operators |
| `logincustom.admin.status` | Operators |
| `logincustom.admin.*` | Operators |

## Security

- Passwords are never stored in plaintext.
- Authentication work runs outside the Minecraft main thread.
- Database operations use parameterized queries.
- Network sessions fail closed if MariaDB or signed communication becomes unavailable.
- Full IP addresses are not used as account identifiers.
- Invalid, expired, duplicated or incorrectly signed network messages are rejected.

Use a dedicated MariaDB user, strong unique credentials and regular database backups.

## License

LoginCustom is distributed under the [MIT License](LICENSE). Third-party libraries bundled with the release retain their respective licenses.
