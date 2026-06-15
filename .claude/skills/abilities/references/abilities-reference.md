# Abilities Reference

Complete documentation for `tabs/abilities.json`.

## Schema

```typescript
interface AbilityDefinition {
  name: string                       // ✅ Display name, must match object key
  description: string                // ✅ What the ability does
  requirements: AbilityRequirement[] // ✅ Prerequisites to unlock
  bonus: number                      // ✅ Power bonus when used in actions
  cooldown: number                   // ✅ Turns between uses
}
```

Ability names are matched against player input case-insensitively (whitespace is also normalized), so "Fireball", "fireball", and "FIREBALL" in player input all detect an ability named `Fireball`. Pick whichever capitalization reads best for your world.

### Legend

- ✅ **Predefine-able**: Can be set in config, preserved via spread
- ⚠️ **Calculated default**: Has fallback logic if not predefined
- ❌ **Always overwritten**: Set by initialization regardless of what exists in config

## AbilityRequirement Schema

A requirement is one of two shapes. Most types name a target via `variable`; `characterLevel` has no target, so it omits `variable` entirely.

```typescript
// resource | attribute | skill | trait
interface ReferencedAbilityRequirement {
  type: 'resource' | 'attribute' | 'skill' | 'trait'  // What to check
  variable: string                // Name of the requirement target
  amount: number                  // Required value (ignored for trait type)
}

// characterLevel — no variable
interface CharacterLevelAbilityRequirement {
  type: 'characterLevel'
  amount: number                  // Required character level
}
```

| Type | Shape | Checks | Example |
|------|-------|--------|---------|
| `resource` | with `variable` | Resource max >= amount | `{ type: "resource", variable: "mana", amount: 50 }` |
| `attribute` | with `variable` | Attribute value >= amount | `{ type: "attribute", variable: "strength", amount: 14 }` |
| `skill` | with `variable` | Skill level >= amount | `{ type: "skill", variable: "melee", amount: 3 }` |
| `trait` | with `variable` | Has trait with that name | `{ type: "trait", variable: "fire affinity", amount: 1 }` |
| `characterLevel` | no `variable` | Character level >= amount | `{ type: "characterLevel", amount: 5 }` |

## Runtime State

When a character unlocks an ability, it becomes an `Ability` with additional runtime fields:

```typescript
interface Ability extends AbilityDefinition {
  lastUsedTick: number            // ❌ Always set to -1 on creation
}
```

Characters track:
- `unlockedAbilities`: Abilities they've unlocked
- `availableCustomAbilities`: AI-generated abilities available to unlock
- `freeAbilityPoints`: Unspent ability points

## Bonus Mechanics

When a predefined ability is used in an action, its own `bonus` value is added to the relevant check directly, as authored. There is no global multiplier applied to it.

`combatSettings.abilityBonus` does **not** scale predefined abilities. It is the default `bonus` assigned to abilities the AI generates for a character during play (newly "learned" abilities). Predefined abilities keep the `bonus` you set on them.

## Cooldown Mechanics

After using an ability:
1. `lastUsedTick` is set to current tick
2. The ability is unavailable for its own `cooldown` turns

`combatSettings.abilityCooldown` does **not** modify predefined abilities' cooldowns. It applies to AI-generated (learned) abilities in two ways: it is the default `cooldown` they're given, and it is the minimum number of turns that must pass before a character can learn another new ability. Predefined abilities use the `cooldown` you author on them.

## Unlocking Abilities

Players can unlock predefined abilities through:

1. **Spending ability points**: Meeting requirements + spending `freeAbilityPoints`
2. **Trait abilities**: Automatically granted by certain traits
3. **Gameplay learning**: `generateLearnedAbility` task creates custom abilities

## Cross-References

| Field | References |
|-------|------------|
| `requirements[].variable` (skill) | `tabs/skills.json` |
| `requirements[].variable` (attribute) | `attributeSettings.attributeNames` in `tabs/settings.json` |
| `requirements[].variable` (resource) | `resourceSettings` keys in `tabs/settings.json` |
| `requirements[].variable` (trait) | `tabs/traits.json` |
