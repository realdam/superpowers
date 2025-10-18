---
name: pest-control-gameplay
description: Use when implementing Pest Control features, designing bot AI for Pest Control, or understanding original game mechanics - complete guide to Pest Control minigame mechanics, monsters, strategies, and rewards as they existed in January 2009 RuneScape
---

# Pest Control Minigame Gameplay (January 2009)

## Overview

**Pest Control** is a cooperative multiplayer combat minigame where 5-25 players defend the Void Knight from monsters while destroying four portals. Players earn Void Knight commendation points for winning, which can be exchanged for experience, equipment, and resources. The minigame is **safe** - players keep items on death and respawn on the lander.

**Core objective:** Destroy all 4 portals before the Void Knight dies OR survive 20 minutes.

## Requirements

### Combat Levels
- **Novice Lander (Boat #1):** Level 40+
- **Intermediate Lander (Boat #2):** Level 70+
- **Veteran Lander (Boat #3):** Level 100+

### Participation Requirements
- **Damage requirement:** 50 points of damage to receive rewards
- **Repairing:** Fixing barricades/gates awards 5 points toward the damage requirement
- **No pets, multicannons, or familiars** allowed

### Starting the Game
- **Minimum players:** 5 players
- **Maximum players:** 25 players per game
- **Auto-start:** Game begins immediately when lander fills with 25 players
- **Wait time:** Otherwise, 5-minute wait before game starts

## Location & Access

**Travel to Pest Control:**
1. Go to Port Sarim docks (south of Lady Lumbridge ship)
2. Speak with the Squire
3. Select "Travel" to Void Knights' Outpost

**The Outpost includes:**
- Bank
- General Store (sells Field rations)
- Anvil (Squire repairs Barrows armor)
- Void Knight Archery Store
- Void Knight Magic Store (body, elemental, mind, chaos, death runes)

**Popular worlds:** 53, 69, 115, 144

## Winning Conditions

### Victory Requirements
1. **Destroy all 4 portals** before the Void Knight dies, OR
2. **Survive for 20 minutes** (very difficult)

### Void Knight Healing
- **Each portal destroyed heals the Void Knight:**
  - **Novice boat:** +50 HP per portal
  - **Intermediate/Veteran:** +60 HP per portal

## Game Mechanics

### Portal Shield System

**Update: July 17, 2007** - Portals start with shields that must drop before attack:

**Shield drop timing:**
- **0:15** - First portal shield drops (15 seconds into game)
- **0:30** - Second portal drops (30 seconds after first)
- **0:30** - Third portal drops (30 seconds after second)
- **0:30** - Fourth portal drops (30 seconds after third)
- **Total:** All portals attackable after 1 minute 45 seconds

**Shield drop patterns (6 possible orders):**
1. Blue → Red → Yellow → Purple
2. Blue → Purple → Red → Yellow
3. Purple → Blue → Yellow → Red
4. Purple → Yellow → Blue → Red
5. Yellow → Purple → Red → Blue
6. Yellow → Red → Purple → Blue

**Pattern rules:**
- If Purple drops first, Red drops last
- If Yellow drops first, Blue drops last
- Red portal never drops first

### Portal Properties

| Portal | Location | Weak To | HP (Novice) | HP (Other) |
|--------|----------|---------|-------------|-----------|
| East (Blue) | | Magic | 200 | 250 |
| Southeast (Yellow) | | Slash | 200 | 250 |
| Southwest (Red) | | Crush/Stab | 200 | 250 |
| West (Purple) | | Ranged | 200 | 250 |

**Portal mechanics:**
- Very high defense - strength/attack enhancing prayers/potions recommended
- Spinners heal portals if not killed first
- Shield visible as faint white lines around portal

## Monsters

### Priority System

**Kill Priority:**
1. **Spinners** (healing portals) - HIGHEST
2. **Ravagers** (destroying gates) - HIGH
3. **Splatters** (near gates) - MEDIUM-HIGH
4. **Defilers/Torchers** (ranged attackers) - MEDIUM
5. **Shifters** (teleporting to knight) - MEDIUM
6. **Brawlers** (blocking paths) - LOWEST

### Monster Details

#### Spinner
**Priority: #1 - Kill first!**
- **Appearance:** Spinning top/jellyfish, floats above ground
- **Threat:** Heals portals
- **Behavior:** If portal destroyed while healing, explodes and poisons nearby players (5 damage poison)
- **Combat levels:** 37, 55, 74, 87, 92
- **Distinct sound** when healing portal

#### Ravager
**Priority: #2**
- **Appearance:** Mole-like with oversized claws, red eyes
- **Threat:** Tears down gates (primary gate destroyer)
- **Behavior:** Continues destroying target before engaging in combat
- **Combat levels:** 36, 52, 71, 89, 106
- **Weakness:** Relatively weak in direct combat

#### Splatter
**Priority: Variable (high near gates)**
- **Appearance:** Giant ball with single eye, liquid inside
- **Threat:** Detonates near fort/gates for massive AoE damage
- **Behavior:** Explodes on death OR when reaching target
- **Chain reactions:** Can trigger other splatters if HP low enough
- **Combat levels:** 22, 33, 44, 55, 66
- **Note:** No prayer protection against detonation
- **Strategic use:** Can detonate near monster groups as AoE

#### Defiler
**Priority: #3-4 (kill if attacking knight)**
- **Appearance:** Snake lower half, humanoid top, cat face
- **Threat:** Ranged attacks over long distances, targets knight
- **Attack:** Throws barbs dealing high ranged damage
- **Behavior:** Can shoot over walls UNLESS directly in front of gate
- **Combat levels:** 33, 50, 66, 80, 100

#### Torcher
**Priority: #3-4 (kill if attacking knight)**
- **Appearance:** Winged worm
- **Threat:** Long-distance magic attacks, targets knight
- **Behavior:** Can attack over walls UNLESS directly in front of gate
- **Combat levels:** 33, 49, 67, 83, 100

#### Shifter
**Priority: #3 (dangerous for defenders)**
- **Appearance:** Spider bottom half, praying mantis scythes
- **Threat:** Teleports through walls directly to Void Knight
- **Behavior:** Excellent melee combat, can teleport past defenses
- **Combat levels:** 36, 57, 76, 90, 104
- **Note:** Most dangerous for knight defenders

#### Brawler
**Priority: #6 (lowest - ignore unless blocking)**
- **Appearance:** Rhino/elephant-like with spikes, pointed snout
- **Threat:** Defends portals, blocks paths
- **Behavior:** Does NOT attack fort normally
- **Combat levels:** 51, 76, 102, 129
- **Unique:** ONLY creature players cannot run through (blocks like walls)
- **Strategic use:** Lure to knight's steps as mobile barricade

## Defensive Strategies

### Gate Management

**CRITICAL: Keep gates closed!**

**Gate mechanics:**
- Defilers/Torchers directly in front of gate CANNOT shoot over it
- Closed gates block most monsters
- East/West gates: Pests only fire if gate is breached or open
- South gate: Always most important (spawns from SE and SW portals converge here)

**Gate priority:**
1. **South gate** - Most important (2 portals' monsters)
2. **East gate** - Has barricade protection
3. **West gate** - Secondary priority

**Repairing gates:**
- Damaged gates become unmovable until repaired
- Check East gate first for exit (barricade protects longer)
- Repairs award 5 damage points

### Defensive Positions

**Priority positions:**

**1. On Void Knight platform (best for low levels)**
- Easy respawn access
- Constant shifter attacks
- Kill shifters immediately

**2. Outside South gate**
- Close gates after players pass
- Kill defilers/torchers shooting over walls
- **Ignore** pests stuck outside gate (can't shoot)
- Kill splatters and ravagers immediately (gate protection)
- Both SE and SW portal pests attack here

**3. East or West gate (if broken)**
- Stand on South square of gate
- Blocks defilers/spinners from entering
- Forms orderly queue of enemies
- Hit each to draw aggro from knight
- Only front enemy can attack you

### Defender Tactics

**Distraction technique:**
- Attack pests before they "see" the Void Knight
- Shifts aggro to you instead of knight
- Protecting knight is objective, not yourself

**Armor recommendation:**
- **Dragonhide armor** - Blocks magic, melee, and ranged decently
- All three combat styles present in Pest Control

**Turn off auto-retaliate:**
- Prevents running around randomly when hit
- Maintains better positioning control

## Offensive Strategies

### Portal Attack Positioning

**Optimal position:**
- Stand **one step to either side** of center between portal and knight
- Allows access to spinners
- Prevents being blocked by monster swarm
- Standing behind portal effective but requires coordinated front attacker

**Bad positions:**
- Direct center (hides spinners from other players, blocks your own access)
- Behind portal alone (can't reach spinners if front blocked)

### Combat Tactics

**Spinner priority:**
- Use Right-Click to target spinners in monster crowds
- Spinners make distinctive sound when healing
- Don't need to kill - just make them attack you instead of healing
- Spinners explode and poison if portal destroyed first

**Brawler handling:**
- Attack brawler, let it attack you, then walk backwards
- Brawler follows, clearing logjam at portal
- Multiple brawlers: repeat for each

**Rangers/Mages:**
- Can draw spinners away from portals
- Spinners follow if path is clear
- Use towers (6 high-level rangers/mages ideal)

### Special Tactics

**Prayer bonuses:** Equip gear matching combat style
**Special attacks:** Useful options include:
- Dragon Battleaxe (with restore potion)
- Dragon Dagger
- Dragon Halberd
- Dragon 2H Sword (hits multiple pests)
- Composite Magic Bow
- Magic Shortbow

**Ancient Magicks:**
- Multi-target spells very effective
- Expensive runes (half XP during PC)
- Good for unreachable spinners

**Chinchompa ranging:**
- Hits multiple targets in multiple squares
- Very effective for portal piles
- Expensive, reduced XP

**Void Seal:**
- Detonates for AoE damage (7-12 damage)
- Hits all nearby monsters
- Does NOT work on portals
- Has charges (8 charges, replacements cost 10 points)

## Rewards

### Commendation Points

**Points per win:**
- **Novice boat:** 2 points
- **Intermediate boat:** 3 points
- **Veteran boat:** 4 points

**Bonus coins:** 10 × combat level if all portals destroyed

**Point cap:** Maximum 250 points
- Warning if boarding at 250 points
- Warning if win would waste points (e.g., 248 points + 4 point win)

**Bonus XP:** Trading 100 points at once gives 10% extra XP per point (10 points = 1% bonus)

### Experience Rewards

**Eligible skills:** Attack, Strength, Defence, Hitpoints, Ranged, Magic, Prayer (all level 25+)

**Experience formula:** `CEILING((Level+25)*(Level-24)/606,1)*N`
- N = 35 (Attack, Strength, Defence, Hitpoints)
- N = 32 (Ranged, Magic)  
- N = 18 (Prayer)

**Experience table (per point):**

| Level Range | Attack/Str/Def/HP | Ranged/Magic | Prayer |
|-------------|-------------------|--------------|--------|
| 25-34 | 35 XP | 32 XP | 18 XP |
| 35-42 | 70 XP | 64 XP | 36 XP |
| 43-48 | 105 XP | 96 XP | 54 XP |
| 49-54 | 140 XP | 128 XP | 72 XP |
| 55-59 | 175 XP | 160 XP | 90 XP |
| 60-64 | 210 XP | 192 XP | 108 XP |
| 65-69 | 245 XP | 224 XP | 126 XP |
| 70-73 | 280 XP | 256 XP | 144 XP |
| 74-77 | 315 XP | 288 XP | 162 XP |
| 78-81 | 350 XP | 320 XP | 180 XP |
| 82-84 | 385 XP | 352 XP | 198 XP |
| 85-88 | 420 XP | 384 XP | 216 XP |
| 89-91 | 455 XP | 416 XP | 234 XP |
| 92-94 | 490 XP | 448 XP | 252 XP |
| 95-97 | 525 XP | 480 XP | 270 XP |
| 98-99 | 560 XP | 512 XP | 288 XP |

**Note:** Experience gained at half rate during game. Reward XP usually exceeds in-game XP.

**Hitpoints consideration:** Using PC for combat skills reduces HP XP ratio. Use ~1 point on HP per 3 points on combat skills to maintain normal ratio.

### Void Knight Equipment

**Weaponry:**
- Void Knight Mace: **250 points**

**Armor:**
- Void Knight Top: **250 points**
- Void Knight Robe: **250 points**
- Void Knight Gloves: **150 points**
- Void Melee Helm: **200 points**
- Void Magic Helm: **200 points**
- Void Ranger Helm: **200 points**

**Other:**
- Void Knight Seal (8 charges): **10 points**

### Resource Packs

**Herb Pack: 30 points**
- Example contents: 2 Harralander, 3 Ranarr, 1 Toadflax, 3 Irit, 4 Avantoe, 2 Kwuarm
- Grimy herbs (random assortment)

**Mineral Pack: 15 points**
- Common: 25 Coal, 18 Iron ore

**Seed Pack: 15 points**
- Common: 3 Sweetcorn seeds, 6 Tomato seeds, 2 Limpwurt seeds

### Charms (2 points each)
- Ravager Charm
- Shifter Charm
- Spinner Charm  
- Torcher Charm
- Purchase amounts: 1, 14, or 28 (costs 2, 28, or 56 points)

## Strategy Tips

### Communication

**Direction confusion:**
- Landing boat faces South
- Tell players **portal colors** instead of directions
- Less confusion than "East" vs "West"

**Help new players:**
- Give advice when asked
- Low combat players can handle spinners to help

### General Tips

**Experience vs Winning:**
- Post-2007 update: half XP during game
- Win rewards usually exceed "portal hanging" XP
- Play for the win, not for in-game kills

**Clan coordination:**
- Pest Control clans cluster by combat level
- Use clan chat for world hopping coordination
- Clans hop when too many low levels join

**Tower usage:**
- Rarely used but effective
- 6 high-level rangers/mages in towers
- Pick off unreachable spinners/brawlers
- 3 high-level melee with knight
- Rest attack portals together

## Quick Reference Card

### Monster Threats
| Monster | Threat | Priority | Action |
|---------|--------|----------|--------|
| Spinner | Heals portal | **1 - Kill First!** | Right-click to target |
| Ravager | Destroys gates | **2 - High** | Kill immediately |
| Splatter | AoE explosion | **High near gates** | Kill or avoid |
| Defiler | Ranged, attacks knight | **Medium** | Kill if attacking knight |
| Torcher | Magic, attacks knight | **Medium** | Kill if attacking knight |
| Shifter | Teleports to knight | **Medium** | Kill if defending |
| Brawler | Blocks paths | **Lowest** | Ignore unless blocking |

### Gate Rules
- ✅ **Always keep closed**
- ✅ South gate most critical
- ✅ Repair damaged gates (+5 damage points)
- ❌ Monsters in front of gate can't shoot over it
- ❌ East/West pests only fire if gate open/breached

### Win Conditions
- Destroy all 4 portals (heals knight per portal)
- OR survive 20 minutes

### Point System
- Need 50 damage OR repair gates (5 points each)
- Novice: 2 points/win
- Intermediate: 3 points/win
- Veteran: 4 points/win
- Max 250 points

## Historical Notes

**Update History:**
- **April 18, 2006:** Pest Control released
- **May 16, 2006:** Added level 40 requirement, 50 damage requirement
- **February 19, 2007:** Blocked pet cats
- **July 17, 2007:** Major update
  - Split into 3 boats (40+, 70+, 100+)
  - Portal shield system added (min 2-3 minutes)
  - Reduced XP per point to 35% of original
  - Players can run through all monsters except Brawlers
  - Caused "World 99 riot" - players protested XP nerfs

**Controversies:**
- "PC Product" - Insult for players who trained primarily via Pest Control
- Referenced in TokTz-Ket-Dill quest ("Void Knight Products")
- Debates about XP formula changes

## Related Mechanics

**Void Seal usage:**
- Only works in Pest Control region
- Rub to unleash power
- AoE damage to nearby monsters
- Multiple charge levels

**Music:**
- Track: "Pest Control"
- Bug: Temporarily changed to "Chef Surprize" after second update

