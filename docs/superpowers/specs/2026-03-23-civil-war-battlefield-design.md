# Civil War Battlefield — Game Design Spec

## Overview

A real-time tactical battle game built in Godot 4 with GDScript. Two armies (Confederate vs Union) clash on a battlefield with infantry, cavalry, and artillery. Faithful to the combat sequences of North & South (1989). Single-player vs AI. Desktop + Web (HTML5) export.

## Architecture

**Scene-per-unit approach.** Each unit type is its own Godot scene with encapsulated behavior. The battlefield is a parent scene that spawns and manages unit groups. AI is a separate node that reads game state and issues commands.

### Scene Tree

```
Main (Node2D)
├── Battlefield (Node2D)
│   ├── Terrain (TileMapLayer)
│   │   ├── Ground tiles (grass, dirt)
│   │   ├── River tiles (impassable)
│   │   ├── Bridge tiles (passable)
│   │   └── Hill tiles (defense bonus)
│   ├── UnionArmy (Node2D)
│   │   ├── Infantry (CharacterBody2D)
│   │   ├── Cavalry (CharacterBody2D)
│   │   └── Artillery (CharacterBody2D)
│   ├── ConfederateArmy (Node2D)
│   │   ├── Infantry (CharacterBody2D)
│   │   ├── Cavalry (CharacterBody2D)
│   │   └── Artillery (CharacterBody2D)
│   └── Projectiles (Node2D)
│       └── Cannonball (Area2D) — spawned dynamically
├── HUD (CanvasLayer)
│   ├── UnitSelector (shows which unit is selected)
│   ├── HealthBars (per-unit health display)
│   └── GameStatus (victory/defeat text)
├── AIController (Node)
└── GameManager (Node)
```

## Battlefield

- **Orientation:** Landscape. Confederate (red) deploys on the left, Union (blue) on the right.
- **View:** Top-down 2D, fixed camera — entire battlefield visible at once, no scrolling.
- **Resolution:** 1280x720 base, scaled for different displays.
- **Terrain:** Tile-based using TileMapLayer.
  - **Open ground** — default, no modifiers.
  - **Hills** — defense bonus (multiply incoming damage by 0.75) for units positioned on them. Applies to both melee and ranged damage.
  - **River** — impassable except at bridge crossing points.
  - **Bridge** — narrow crossing over the river, creates a chokepoint. Width fits one unit at a time.

## Unit Types

Each unit is a CharacterBody2D scene with its own script, sprite, and collision shape. Units have collision with both friendly and enemy units (no stacking/overlapping).

### Collision & Detection

- **Unit collision shape:** 32x32px bounding box for all unit types.
- **Friendly collision:** Enabled — units block each other. This makes the bridge chokepoint meaningful.
- **Melee detection radius:** 80px for Infantry and Cavalry. When an enemy enters this radius, the unit auto-moves toward the enemy to engage.
- **Ranged detection radius:** 400px for Artillery.
- **Movement model:** Player input is 4-directional, but movement is continuous (not tile-snapped). AI uses `NavigationAgent2D` for pathfinding around rivers and obstacles.

### Infantry
- **Speed:** 120 px/s (medium)
- **Health:** 100
- **Attack:** 10 damage/s (melee, on contact)
- **Detection radius:** 80px
- **Behavior:** Solid all-rounder, holds ground. Deals bonus damage to artillery (+50%).

### Cavalry
- **Speed:** 200 px/s (fast)
- **Health:** 80
- **Attack:** 20 damage on charge (burst on first contact), then 8 damage/s sustained
- **Detection radius:** 80px
- **Behavior:** Fast flanker. Charge bonus on first contact (resets after 3 seconds without contact). Deals bonus damage to infantry (+50%). Takes bonus damage from artillery (+50%).

### Artillery
- **Speed:** 40 px/s (very slow, cannot move while firing)
- **Health:** 60
- **Attack:** 25 damage per cannonball, fires every 2 seconds
- **Melee attack:** 3 damage/s (minimal self-defense when enemies close in)
- **Detection radius:** 400px (ranged), 80px (melee fallback)
- **Behavior:** Stationary when firing. Deals bonus damage to cavalry (+50%).

### Cannonball Projectiles

- **Speed:** 300 px/s
- **Friendly fire:** No — cannonballs only damage enemy units.
- **AOE:** None — single target hit.
- **Targeting:** Fires at the target's current position (no leading).
- **Despawn:** On hitting an enemy, or after 3 seconds, or on leaving the screen.

### Combat Resolution

- Damage is applied continuously while units overlap (melee) or per projectile hit (artillery).
- Rock-paper-scissors balance: Cavalry > Infantry > Artillery > Cavalry.
- Bonus damage (+50%) applied based on the matchup.
- A unit is eliminated when health reaches 0.
- An army loses when all its units are eliminated.
- **Stalemate rule:** If no damage is dealt by either side for 15 seconds, the battle ends as a draw. If only artillery vs artillery remains, the side with more total HP wins; if equal, it is a draw.

## Controls

Keyboard-only, faithful to the original:

- **Arrow keys** or **WASD** — move the currently selected unit in 4 directions (continuous movement, not tile-snapped).
- **Tab** — cycle through living units (skip eliminated ones).
- **1/2/3** — directly select Infantry/Cavalry/Artillery. No-op if that unit is eliminated.
- **Escape** — pause/unpause the game (freezes physics, shows "Paused" overlay).
- **Selected unit indicator** — visible glow/highlight on the active unit.
- **Auto-combat** — units engage automatically when an enemy enters their detection radius.
- **Default selection:** Infantry is selected at battle start.
- **No mouse required.**

## AI Opponent

The AI controls the Confederate army with a simple state-based approach:

### AI States
1. **Advance** — move units toward the bridge / center of the battlefield.
2. **Engage** — when enemy units are within detection range, move to attack with favorable matchups (send cavalry toward infantry, infantry toward artillery).
3. **Retreat** — if a unit is below 30% health and not engaged, pull back toward starting position. Re-engages if: it is the last surviving unit, or an enemy enters its detection range while retreating.
4. **Artillery position** — AI keeps artillery behind other units and targets the nearest enemy. AI avoids sending units through prolonged open-ground artillery fire; prefers to commit multiple units simultaneously or use terrain cover.

### AI Decision Loop
- Runs every 0.5 seconds (not every frame — gives it a human-like reaction time).
- Evaluates each unit independently.
- Prioritizes favorable matchups when choosing targets.
- Basic threat assessment: if outnumbered, consolidate units together.

## Game Flow

### States
1. **Setup** — battlefield loads, units spawn at starting positions. Brief "Battle Begins!" text.
2. **Battle** — real-time combat. Player controls Union army, AI controls Confederate.
3. **Victory/Defeat/Draw** — triggered when one army is eliminated or stalemate rule activates. Display result and option to restart.

### Spawn Formation
Units spawn in a column on their respective side, vertically centered on the map:
- **Cavalry** — front (closest to center)
- **Infantry** — middle
- **Artillery** — rear (closest to edge)
- Spacing: 80px between units. 100px from the map edge (artillery position).

### Transitions
- Setup → Battle: automatic after 2-second countdown.
- Battle → Victory/Defeat/Draw: when one army has no surviving units, or stalemate rule triggers.
- Victory/Defeat/Draw → Setup: player presses Enter/Space to restart.

## HUD

Minimal, non-intrusive:

- **Top-left:** Player's unit health bars (3 small bars, colored by unit type). Eliminated units shown greyed out.
- **Top-right:** Enemy unit health bars.
- **Bottom-center:** Currently selected unit name + icon.
- **Center (overlay):** Game state messages ("Battle Begins!", "Victory!", "Defeat!", "Draw!", "Paused").

## Visual Style

- **Placeholder art initially** — colored rectangles/simple shapes for units, solid tiles for terrain. Replace with pixel art or sprites later.
- **Color coding:** Union = blue tones, Confederate = red/grey tones.
- **Unit differentiation:** Different shapes per unit type (rectangle=infantry, triangle=cavalry, circle=artillery).
- **Animations (later):** Movement animations, attack effects, cannon smoke, unit death.

## Audio (Deferred)

Not in initial scope. Can add later:
- Battle ambient sounds
- Cannon fire effects
- Cavalry charge sounds
- Victory/defeat fanfares

## Technical Notes

- **Godot version:** 4.x (latest stable)
- **Language:** GDScript
- **Export targets:** Desktop (Windows, Linux, macOS) + Web (HTML5)
- **No external dependencies** — pure Godot.
- **Project name:** `civil-war-battlefield`

## Out of Scope (v1)

- Strategic map layer
- Multiple battle maps
- Mini-games (fort capture, train robbery)
- Online multiplayer
- Unit upgrades or progression
- Sound and music (deferred)
