# Fantasy Fútbol Business Sports

## Resumen

Este documento recoge una primera base funcional para el MVP de fantasy fútbol de Business Sports a partir del briefing recibido y de la revisión de la competición pública en Torneos para Empresas.

Objetivo del MVP:

- permitir a cada usuario crear su equipo fantasy
- fichar jugadores reales de la competición
- puntuar por jornada según estrellas
- actualizar mercado de forma automática
- mantener una vía manual de corrección desde administración

## Fuente real analizada

Competición base revisada:

- `https://www.torneosparaempresas.com/mod/soccer/situation.aspx?itm=15473`

Secciones detectadas en la navegación de la competición:

- Situación
- Equipos
- Jugadores
- Técnicos
- Instalaciones
- Clasificación
- Calendario
- Comité competición
- Ranking Tarjetas
- Ranking Equipos
- Ranking Porteros
- Ranking Goleadores
- Ranking FairPlay
- Ranking MVP
- Ranking MVT
- Noticias, comentarios, fotos y multimedia

## Qué datos sí parecen obtenibles

Según la página pública revisada, sí se observan al menos estos datos:

- temporada y nombre del campeonato
- lista de competiciones y divisiones
- clasificación por grupo o fase
- equipos participantes
- resultados de jornadas recientes desde la propia pantalla de situación
- rankings de goleadores, porteros, MVP y MVT
- fichas parciales de jugador visibles en widgets de ranking:
  - nombre
  - dorsal
  - equipo
  - demarcación
  - edad
  - lateralidad cuando existe
- puntuaciones por estrellas mostradas visualmente en fichas o rankings

## Qué datos requieren verificación o vía manual

Con lo visto en la web pública no queda garantizado que siempre podamos extraer de forma estructurada:

- alineaciones completas por partido
- minutos jugados
- asistencias
- expulsiones y tarjetas por jugador en todos los encuentros
- crack del partido y dandy del partido en formato consistente
- detalle completo de eventos por jugador dentro de cada partido

Por eso el MVP debe incluir:

- importación automática con trazabilidad
- estados de incertidumbre por dato
- corrección manual desde admin
- reprocesado idempotente por partido y jornada

## Propuesta de modelo funcional

### Áreas principales

1. Autenticación y usuarios
2. Competición real
3. Plantillas fantasy
4. Mercado
5. Puntuación por estrellas
6. Administración e importaciones
7. Automatización programada

### Roles

- usuario
- administrador

## Esquema base de datos propuesto

### Núcleo de negocio

- `users`
- `accounts`
- `sessions`
- `password_reset_tokens`
- `fantasy_teams`
- `fantasy_team_players`
- `fantasy_lineups`
- `fantasy_lineup_players`

### Competición real

- `seasons`
- `competitions`
- `divisions`
- `real_teams`
- `real_players`
- `real_team_season_players`
- `rounds`
- `matches`
- `match_team_stats`
- `player_match_stats`
- `player_match_ratings`

### Mercado y reglas

- `market_settings`
- `scoring_rules`
- `market_windows`
- `player_market_values`
- `player_market_value_history`
- `transfers`
- `wallet_ledger`

### Importación y control

- `import_runs`
- `import_run_items`
- `source_snapshots`
- `manual_adjustments`
- `admin_audit_logs`
- `recalculation_jobs`

## Pantallas mínimas

- inicio
- registro
- login
- recuperación de contraseña
- dashboard de usuario
- mi plantilla
- mercado
- clasificación general fantasy
- clasificación por jornada
- detalle de jugador
- detalle de jornada
- historial de movimientos
- panel admin
- pantalla de importaciones y logs

## Motor de estrellas MVP

Reglas configurables desde admin:

- juega partido: `+1`
- victoria de su equipo: `+1`
- gol: `+1`
- portería a cero para defensa o portero: `+1`
- crack del partido: `+3`
- dandy del partido: `+1`
- expulsión: `-1`

Restricciones iniciales:

- máximo por partido: `5`
- mínimo por partido: `0`

Se debe guardar por separado:

- estrellas reales detectadas
- puntos fantasy aplicados
- origen del dato
- nivel de confianza
- si hubo ajuste manual

## Fórmula inicial de mercado

### Variables por jugador

- valor actual
- valor anterior
- variación absoluta
- variación porcentual
- media de estrellas últimos 3 partidos jugados
- racha positiva o negativa
- jornadas consecutivas sin jugar

### Lógica propuesta

- 4 o 5 estrellas: subida alta
- 3 estrellas: subida moderada
- 2 estrellas: estabilidad o ligera subida
- 1 estrella: ligera bajada
- 0 estrellas o expulsión: bajada clara
- media alta en últimos 3 partidos: bonus
- media baja en últimos 3 partidos: penalización
- 2 o 3 buenos partidos seguidos: bonus de tendencia
- varias jornadas flojas: penalización adicional
- 2 jornadas sin jugar: bajada configurable
- 3 o más jornadas sin jugar: penalización progresiva

Todo debe parametrizarse desde admin.

## Stack recomendado

- Next.js
- PostgreSQL
- Prisma
- NextAuth o Auth.js
- cron compatible con despliegue
- despliegue en Vercel o plataforma equivalente

## Roadmap de implementación

### Fase 1

- cerrar modelo de datos
- crear estructura base del proyecto
- preparar autenticación
- preparar seeds y configuración

### Fase 2

- construir panel usuario
- construir mercado
- construir reglas de plantilla
- construir panel admin

### Fase 3

- construir scraper/importador
- guardar snapshots y logs
- calcular estrellas y mercado
- programar scheduler `Europe/Madrid` de lunes a viernes a las `01:00`

### Fase 4

- pruebas mínimas
- validación con competición real
- documentación de límites
- preparación de despliegue

## Riesgos detectados

- la navegación pública mezcla contenido útil con estructura HTML irregular
- algunos enlaces parecen depender de estado interno o navegación del sitio
- no todos los eventos de partido están confirmados como datos estructurados y homogéneos
- habrá partidos o jugadores que necesiten corrección manual

## Conclusión

Sí hay base suficiente para construir un MVP fantasy serio apoyado en la competición pública, pero el importador debe diseñarse con tolerancia a fallos, snapshots de origen y un panel admin fuerte para revisión manual.
