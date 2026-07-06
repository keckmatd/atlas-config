# atlas-config

Remote config manifest for [Atlas of the Depths](https://play.google.com/store/apps/details?id=com.atlasofthedepths.app) (issue atlas-of-the-depths#381).

The app fetches `game-config.json` from this repo's raw URL at launch:

```
https://raw.githubusercontent.com/keckmatd/atlas-config/main/game-config.json
```

The fetch is non-blocking with a 3s timeout. Precedence: **validated fetch > last cached copy (AsyncStorage) > bundled defaults**. The app is fully functional offline forever — this manifest can only nudge values within validated bounds.

## Schema

| Field | Type | Default | Notes |
|---|---|---|---|
| `version` | number | `0` | Manifest revision. Informational only — bump on every edit. |
| `zoneUpliftScale` | number | `1.0` | Multiplier on the zone continuity uplift (monster combat stats in later zones, #343). Clamped to `[0.25, 4]`. |
| `survivorGoldScale` | number | `1.0` | Multiplier on grateful survivor gold, applied on top of the chest curve (tunes survivors *relative to* chests, #341). Clamped to `[0.25, 4]`. |
| `chestGoldScale` | number | `1.0` | Multiplier on chest gold rolls (survivor gold shares this curve, #341). Clamped to `[0.25, 4]`. |
| `abyssEnemyScale` | number | `1.0` | Flat multiplier on Abyss boss stat scaling (not the per-floor exponent). Clamped to `[0.25, 4]`. |
| `lategameXpScale` | number | `1.0` | Multiplier on the global monster XP floor curve (#344). Clamped to `[0.25, 4]`. |
| `playerXpScale` | number | `1.0` | Multiplier on all combat/boss XP granted to the player, applied at the XP emitters: monster kill XP, boss XP, Abyss boss XP. Event and bonus-level XP unaffected (#391). Clamped to `[0.25, 4]`. |
| `monsterStatScale` | number | `1.0` | Multiplier on regular/elite monster HP/ATK/DEF/SPD. Explicit-stat bosses excluded — use `bossStatScale` (#391). Clamped to `[0.25, 4]`. |
| `bossStatScale` | number | `1.0` | Multiplier on boss stats: elite bosses, archbosses, and Abyss bosses (stacks with `abyssEnemyScale` in the Abyss) (#391). Clamped to `[0.25, 4]`. |
| `adsEnabled` | boolean | `true` | Kill switch — `false` disables interstitial ads app-wide. |
| `iapEnabled` | boolean | `true` | Kill switch — `false` hides the remove-ads purchase UI and blocks new purchases. Restores keep working; owners never lose their entitlement. |
| `minSupportedVersion` | string \| null | `null` | Semver. App versions below this see a non-dismissable update notice on the title screen. |
| `minVersionMessage` | string | `""` | Player-facing text for the update notice (a default message is used if empty). |
| `configUrl` | string (https) | this repo's raw URL | The manifest's own URL. Change it to migrate hosting without an app update — clients cache it and fetch from the new location next launch. |

## How validation works (client side)

- Each field is type-checked independently; a wrongly-typed field silently keeps its bundled default.
- Scale factors are clamped to `[0.25, 4]` — a typo like `100` becomes `4`, never a 100x distortion.
- Unknown fields are ignored (forward compatible).
- The whole payload is rejected only if it isn't valid JSON or isn't a JSON object; clients then fall back to their cached copy, then bundled defaults.
- `configUrl` must be `https://` or it is ignored.

## Editing safely

1. Edit `game-config.json` on a branch or directly on `main` (raw URL serves `main`).
2. Bump `version`.
3. Validate JSON before committing: `python3 -m json.tool game-config.json`.
4. Keep scale changes small (e.g. `0.9`–`1.15`); they apply to live players on next app launch.
5. Rollback = revert the commit. Clients pick it up on next launch; offline clients keep their last cached copy until then.
