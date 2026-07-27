# LoginCustom

Sistema de autenticación para servidores de Minecraft Java Edition, desarrollado en Java y diseñado para funcionar de forma segura, liviana y mantenible en varias generaciones de Bukkit, Spigot, Paper, Purpur, Folia, Velocity y BungeeCord.

> Estado actual: `0.1.0-alpha.1` Legacy implementada para evaluación.  
> Versión estable objetivo: `1.0.0`.

La primera alpha genera cuatro distribuciones desde un único núcleo:

- `LoginCustom-Bukkit-Legacy.jar`: CraftBukkit/Spigot `1.8.8`, `1.12.2` y `1.15.2`.
- `LoginCustom-Paper-Legacy.jar`: Paper y forks compatibles en esas mismas versiones.
- `LoginCustom-Velocity.jar`: adaptador de proxy; requiere el plugin en cada backend.
- `LoginCustom-Bungee.jar`: adaptador de proxy; requiere el plugin en cada backend.

Las versiones indicadas forman la matriz de pruebas Legacy. Otras versiones entre `1.8.x` y `1.15.x`
son compatibles por diseño, pero no se declararán verificadas hasta ejecutar su prueba real.

Construcción local:

```text
gradlew.bat clean test assembleRelease --no-parallel
```

Los artefactos y sus SHA-256 se generan en `release/0.1.0-alpha.1/`. La instalación, las
limitaciones y la evidencia ejecutada están documentadas en `docs/INSTALL_LEGACY.md`,
`docs/ALPHA_LIMITATIONS.md` y `docs/TEST_REPORT.md`.

## Objetivo

LoginCustom protegerá servidores que admitan jugadores sin autenticación oficial, evitando que otra persona pueda entrar usando un nombre registrado. El jugador deberá registrarse la primera vez y autenticarse en las conexiones siguientes.

El proyecto tendrá un núcleo común y adaptadores separados por plataforma. No se intentará forzar todas las APIs y versiones de Java dentro de un único JAR, porque eso aumenta el peso, rompe compatibilidad y dificulta el mantenimiento.

## Ediciones compatibles

| Edición | Minecraft | Plataformas objetivo | Java del servidor |
| --- | --- | --- | --- |
| Normal | `1.21.x` a `26.x` | Bukkit/Spigot, Paper/Purpur, Folia | Java 21 hasta `1.21.11`; Java 25 desde `26.1` |
| Upgrade | `1.16.x` a `1.20.x` | Bukkit/Spigot, Paper/Purpur y forks compatibles | Según la versión del servidor; artefactos compilados con el mínimo necesario |
| Legacy | `1.8.x` a `1.15.x` | Bukkit/Spigot y forks compatibles | Java 8 |

Los adaptadores de Velocity y BungeeCord se versionarán de forma independiente, ya que las versiones de los proxies no siguen la numeración de Minecraft. Velocity `3.5.x+` requiere Java 21.

La edición Normal tendrá implementaciones separadas para `1.21.x` y `26.x`, aunque ambas pertenezcan a la misma línea de producto. Esto permite usar correctamente las APIs actuales de Paper sin abandonar servidores `1.21.x`.

## Plataformas y artefactos

La distribución final tendrá JARs pequeños y específicos:

- `LoginCustom-Bukkit`: Bukkit, Spigot y forks compatibles.
- `LoginCustom-Paper`: Paper y Purpur, con optimizaciones propias de sus APIs.
- `LoginCustom-Folia`: planificadores regionales y de entidad; sin acceder al mundo desde hilos incorrectos.
- `LoginCustom-Velocity`: protección centralizada para redes Velocity.
- `LoginCustom-Bungee`: protección centralizada para BungeeCord.

El núcleo de autenticación, persistencia, validaciones y mensajes será compartido. Cada adaptador contendrá únicamente la integración necesaria para su plataforma.

## Flujo del jugador

### Primera conexión

```text
/register pass1234 pass1234
```

También estará disponible el alias:

```text
/reg pass1234 pass1234
```

Si ambas contraseñas coinciden y cumplen la política configurada, la cuenta queda registrada y la sesión se autentica.

### Conexiones siguientes

```text
/login pass1234
```

Alias:

```text
/log pass1234
```

El jugador tendrá un máximo configurable de intentos; el valor predeterminado será `3`. Al agotarlos será expulsado y se aplicará un tiempo de espera para frenar ataques de fuerza bruta.

### Mientras no está autenticado

El jugador no podrá:

- moverse más allá de una distancia mínima configurable;
- enviar chat ni ejecutar comandos no autorizados;
- recoger, soltar, mover o usar objetos;
- interactuar con bloques, entidades o inventarios;
- causar o recibir daño;
- teletransportarse, cambiar de mundo o aprovechar vehículos;
- ocultar la pantalla de autenticación entrando a otro servidor de la red.

Solo se permitirán los comandos de autenticación y los comandos adicionales incluidos explícitamente en una lista segura.

## Comandos

| Comando | Alias | Uso | Acceso |
| --- | --- | --- | --- |
| `/login <contraseña>` | `/log` | Iniciar sesión | Jugador no autenticado |
| `/register <contraseña> <confirmación>` | `/reg` | Registrar la cuenta por primera vez | Jugador no registrado |
| `/changepassword <actual> <nueva> <confirmación>` | `/cpw` | Cambiar la propia contraseña | Jugador autenticado |
| `/premium <contraseña>` | — | Solicitar la vinculación de una cuenta oficial verificada | Jugador autenticado |
| `/unpremium <jugador>` | — | Retirar el modo premium de una cuenta | Operador o permiso administrativo |
| `/unregister <jugador>` | — | Eliminar el registro para que el jugador pueda registrarse nuevamente | Operador o permiso administrativo |
| `/logincustom reload` | `/lc reload` | Recargar mensajes y opciones seguras | Operador o permiso administrativo |

`/unregister` nunca mostrará ni recuperará una contraseña. El administrador elimina el registro y el jugador crea una contraseña nueva en su próxima conexión.

## Permisos

```text
logincustom.command.login
logincustom.command.register
logincustom.command.changepassword
logincustom.command.premium
logincustom.admin.unpremium
logincustom.admin.unregister
logincustom.admin.reload
logincustom.admin.status
logincustom.admin.*
```

Los permisos administrativos estarán denegados por defecto y solo se concederán a operadores o grupos configurados expresamente.

## Cuentas premium

`/premium` no será un simple interruptor basado en el nombre del jugador. Eso permitiría robar cuentas en una red configurada en modo offline.

El modo premium solo quedará activo cuando:

1. el jugador ya esté autenticado con su contraseña;
2. confirme la operación con la contraseña actual;
3. el proxy o servidor demuestre una sesión oficial válida;
4. el UUID oficial verificado coincida con la vinculación almacenada;
5. la siguiente conexión vuelva a superar la verificación oficial.

Si la identidad oficial no puede verificarse de forma confiable, LoginCustom rechazará la activación y mantendrá el login normal. La seguridad fallará de forma cerrada: un error de red o de verificación nunca concederá bypass.

`/unpremium <jugador>` solo podrá ejecutarlo un operador o alguien con `logincustom.admin.unpremium`. La acción quedará registrada en la auditoría sin incluir datos sensibles.

## Identidad y UUID

- Cuenta premium: se conservará el UUID oficial recibido mediante una conexión autenticada.
- Cuenta no premium: se generará un identificador interno aleatorio y persistente, independiente del nombre visible.
- Un cambio de mayúsculas o minúsculas no creará otra cuenta.
- Los cambios de nombre de cuentas premium se resolverán por UUID oficial, no por el nombre anterior.
- Nunca se confiará en un UUID enviado por un backend o proxy mal configurado sin validar el canal de forwarding.

## Seguridad de contraseñas

Las contraseñas nunca se guardarán, registrarán ni transmitirán en texto plano fuera del instante necesario para validarlas.

La base de datos almacenará únicamente:

- hash de contraseña resistente a ataques offline;
- salt criptográfico único;
- parámetros y versión del algoritmo;
- identificadores y metadatos mínimos de seguridad.

Diseño previsto:

- Argon2id como algoritmo principal, con parámetros calibrados y migración transparente cuando cambien;
- pepper opcional almacenado fuera de la base de datos, mediante variable de entorno o secreto del host;
- comparación en tiempo constante;
- borrado temprano de datos sensibles de memoria cuando la API de Java lo permita;
- longitud mínima y máxima configurables para evitar claves débiles y abuso de recursos;
- prohibición total de contraseñas en logs, mensajes de error, métricas, backups de configuración o comandos de consola;
- respuestas genéricas para no revelar si una cuenta existe;
- invalidación de todas las sesiones después de cambiar contraseña, desregistrar o quitar el modo premium.

Un hash puede verse en la base de datos, pero no revela directamente la contraseña original. LoginCustom no ofrecerá funciones para “ver” o descifrar contraseñas.

## Protección contra ataques

- Tres intentos de login por sesión de forma predeterminada.
- Límites por cuenta, IP y ventana de tiempo.
- Retardo progresivo y expulsión configurable.
- Bloqueos temporales, nunca permanentes por defecto, para reducir ataques de denegación de servicio contra otros usuarios.
- Límite de conexiones simultáneas en proceso de autenticación.
- Consultas parametrizadas para impedir inyección SQL.
- Operaciones de base de datos fuera del hilo principal, del hilo de red y de los hilos regionales de Folia.
- Timeouts estrictos y circuit breaker para bases de datos remotas.
- Protección contra replay de sesiones y tokens aleatorios de un solo uso.
- Forwarding moderno y secreto compartido obligatorio cuando se use Velocity.
- Backends aislados de Internet y firewall configurado para aceptar únicamente al proxy.
- Auditoría de cambios administrativos, sin IP completa ni secretos salvo que el administrador habilite expresamente una política compatible con su normativa.

## Persistencia

### Servidor único

SQLite será el almacenamiento predeterminado, con WAL, migraciones versionadas, consultas preparadas e índices mínimos.

### Red de servidores

MariaDB/MySQL o PostgreSQL serán las opciones recomendadas. Una red no deberá compartir un archivo SQLite entre varias máquinas.

El acceso se realizará mediante un pool pequeño y configurable. Ninguna consulta bloqueará el tick del servidor, una región de Folia ni el event loop del proxy.

Esquema conceptual:

```text
accounts
  internal_id
  normalized_name
  official_uuid
  password_hash
  hash_algorithm
  hash_parameters
  premium_enabled
  created_at
  password_changed_at

sessions
  session_id_hash
  account_id
  server_scope
  expires_at

login_attempts
  account_id
  address_fingerprint
  result
  created_at

audit_events
  actor_id
  action
  target_id
  created_at
```

Las sesiones y tokens también se guardarán como hashes; nunca como valores reutilizables en texto plano.

## Configuración y mensajes

Todos los mensajes serán personalizables desde archivos YAML y tendrán valores predeterminados en español e inglés.

Ejemplos:

```yaml
messages:
  login-required: "<yellow>Usá <white>/login <contraseña></white> para ingresar."
  register-required: "<yellow>Usá <white>/register <contraseña> <contraseña></white> para registrarte."
  login-success: "<green>Sesión iniciada correctamente."
  invalid-credentials: "<red>Credenciales incorrectas. Intentos restantes: <attempts>."
  too-many-attempts: "<red>Demasiados intentos. Volvé a intentarlo más tarde."
```

En versiones modernas se usará Adventure/MiniMessage donde la plataforma lo admita. La edición Legacy convertirá los mensajes al formato compatible sin cargar Adventure completo si no es necesario.

Las variables de mensajes se limitarán a una lista conocida para evitar inyecciones de formato. La recarga validará toda la configuración antes de reemplazar la versión activa.

## Arquitectura prevista

```text
LoginCustom/
├─ login-api/
├─ login-core/
├─ login-storage-sqlite/
├─ login-storage-sql/
├─ login-platform-bukkit/
├─ login-platform-paper/
├─ login-platform-folia/
├─ login-platform-velocity/
├─ login-platform-bungee/
├─ login-distribution-normal/
├─ login-distribution-upgrade/
├─ login-distribution-legacy/
└─ login-test-suite/
```

Principios:

- Java y Gradle con Kotlin DSL.
- Dependencias reducidas, fijadas y revisadas.
- API interna pequeña y sin referencias a Bukkit, Paper o proxies dentro del núcleo.
- Inyección explícita de reloj, generador aleatorio, almacenamiento y scheduler para facilitar pruebas.
- Migraciones de base de datos compatibles hacia adelante y copias de seguridad antes de cambios destructivos.
- Métricas desactivadas por defecto y sin información sensible.
- Sin llamadas de red ocultas ni actualizaciones automáticas del JAR.

## Rendimiento

“Ultra liviano” se tratará como un requisito medible, no como una afirmación publicitaria. Antes de publicar `1.0.0` se definirán y ejecutarán pruebas para:

- tiempo de habilitación del plugin;
- memoria retenida en reposo;
- latencia p50, p95 y p99 de registro y login;
- impacto en MSPT/TPS bajo conexiones simultáneas;
- comportamiento del pool de base de datos;
- carga de Argon2id con parámetros seguros;
- ausencia de operaciones bloqueantes en Bukkit, Folia, Velocity y BungeeCord.

Los parámetros de hash tendrán un límite de concurrencia para que una oleada de logins no agote CPU o memoria.

## Pruebas mínimas antes de publicar

- Registro, login correcto y tres intentos fallidos.
- Reinicio completo y persistencia de cuentas.
- Cambio de contraseña e invalidación de sesiones anteriores.
- `unregister` y `unpremium` con y sin permisos.
- Cuenta premium verificada, UUID correcto y rechazo ante identidad no verificable.
- Nombres con distintas mayúsculas, caracteres permitidos y longitudes límite.
- Caída, lentitud y reconexión de la base de datos.
- SQLite local y base SQL remota.
- Ataques de fuerza bruta y conexiones concurrentes.
- Red con proxy y backends inaccesibles directamente desde Internet.
- Scheduler correcto en Folia.
- Matriz real de servidores para Normal, Upgrade y Legacy.
- Revisión de logs para comprobar que nunca aparezcan contraseñas, tokens o secretos.

## Criterios de aceptación de `1.0.0`

- Todos los comandos y permisos documentados funcionan.
- Ninguna contraseña se almacena o registra en texto plano.
- Las tres ediciones pasan su matriz de compatibilidad declarada.
- Paper/Purpur, Folia, Velocity y BungeeCord usan adaptadores propios.
- El bypass premium solo se concede a una identidad oficial verificada.
- No existen consultas ni tareas de hash bloqueando hilos sensibles.
- Configuración y mensajes pueden personalizarse y validarse.
- Las migraciones y recuperaciones ante fallos están probadas.
- Los resultados de rendimiento se publican con entorno, carga y metodología reproducibles.

## Referencias técnicas

- [Paper: requisitos y versiones de Java](https://docs.papermc.io/paper/getting-started/)
- [Paper: configuración actual de proyectos](https://docs.papermc.io/paper/dev/project-setup/)
- [Paper/Folia: soporte correcto de schedulers](https://docs.papermc.io/paper/dev/folia-support/)
- [Velocity: preguntas frecuentes y Java requerido](https://docs.papermc.io/velocity/faq/)
- [Velocity: creación de plugins](https://docs.papermc.io/velocity/dev/creating-your-first-plugin/)
- [SpigotMC: desarrollo para BungeeCord](https://www.spigotmc.org/wiki/bungeecord-plugin-development/)

## Nota de alcance

Este README define el producto completo y sus condiciones de seguridad. La alpha Legacy implementa
únicamente el alcance documentado en `docs/ALPHA_LIMITATIONS.md`; no implica que las ediciones
Upgrade/Normal, premium ni todas las combinaciones de `1.0.0` estén terminadas. La compatibilidad
solo se declarará públicamente después de ejecutar la matriz real de pruebas.
