---
name: 2009scape-bot-architecture
description: Use when implementing new bot scripts in 2009scape, debugging bot behavior, or understanding how bots interact with game systems - covers AIPlayer architecture, Script class, GeneralBotCreator, and state machines
---

# Understanding Server-Sided Bots in 2009scape

## Overview

2009scape uses **AIPlayer** (Artificial Intelligent Player) objects to create server-sided bots that simulate real player behavior. These bots run scripts that control their actions through the `Script` abstract class and are managed by the `GeneralBotCreator` system.

## Core Architecture

### AIPlayer - The Bot Entity

**Location:** `Server/src/main/core/game/bots/AIPlayer.java`

AIPlayer extends the Player class, making bots full-fledged entities in the game world:

- **Unique identification:** Each bot has a UID and generated username
- **Session handling:** Uses `ArtificialSession` instead of network sessions
- **Appearance:** Can load randomized equipment/stats from configuration files
- **Repository integration:** Added to `Repository.getPlayers()` like real players

Key characteristics:
```java
public AIPlayer(Location l) {
    super.setLocation(startLocation = l);
    super.artificial = true;  // Marks as bot
    super.getDetails().setSession(ArtificialSession.getSingleton());
    Repository.getPlayers().add(this);
    this.updateRandomValues();  // Randomize appearance/stats
    this.init();
}
```

### Script - The Bot Brain

**Location:** `Server/src/main/core/game/bots/Script.java`

The abstract Script class defines bot behavior:

**Core components:**
- `scriptAPI` - Helper methods for common bot actions (teleport, attack, walk, bank)
- `inventory` - Starting inventory items
- `equipment` - Starting equipped items
- `skills` - Skill levels to set on initialization
- `quests` - Quest completion states
- `tick()` - Abstract method called every game tick

**Lifecycle:**
1. Bot spawned with Script instance
2. `init(boolean isPlayer)` sets skills, quests, equipment, inventory
3. `tick()` called every game tick (doesn't run if bot has active pulse)
4. Script controls bot through `scriptAPI` and direct `bot` reference

### GeneralBotCreator - The Bot Launcher

**Location:** `Server/src/main/core/game/bots/GeneralBotCreator.kt`

Creates bots and submits their scripts to the game pulse system:

**Constructor patterns:**
```kotlin
// Create bot with script at location
GeneralBotCreator(botScript: Script, loc: Location?)

// Create bot with pre-existing AIPlayer
GeneralBotCreator(botScript: Script, bot: AIPlayer?)

// Turn real player into bot (for botting commands)
GeneralBotCreator(botScript: Script, player: Player, isPlayer: Boolean)
```

**BotScriptPulse:**
- Inner class that runs the script's `tick()` method every game tick
- Registered in `AIRepository.PulseRepository` for tracking
- Rate-limited: max pulses per tick tracked via `botPulsesTriggeredThisTick`

## Writing a Bot Script

### Basic Structure

```kotlin
class MyBot : Script() {
    var state = State.IDLE
    
    init {
        // Set starting stats, inventory, equipment
        skills[Skills.ATTACK] = 60
        skills[Skills.STRENGTH] = 65
        inventory.add(Item(Items.LOBSTER_379, 10))
        equipment.add(Item(Items.RUNE_SCIMITAR_1333))
    }
    
    override fun tick() {
        // Only runs when bot doesn't have active pulse
        scriptAPI.eat(Items.LOBSTER_379)
        
        when(state) {
            State.IDLE -> { /* logic */ }
            State.COMBAT -> { /* logic */ }
        }
    }
    
    override fun newInstance(): Script {
        return MyBot()
    }
    
    enum class State { IDLE, COMBAT }
}
```

### ScriptAPI Methods

**Movement:**
- `walkTo(Location)` - Walk to specific location
- `teleport(Location)` - Instant teleport

**Combat:**
- `attackNpcInRadius(bot, npcName, radius)` - Attack nearest NPC by name
- Attack through `bot.attack(target)` directly

**Items:**
- `eat(foodId)` - Eat food if HP is low
- `forceEat(foodId)` - Eat regardless of HP
- `takeNearestGroundItem(itemId)` - Pick up ground items
- `withdraw(itemId, amount)` - Withdraw from bank

**Objects:**
- `getNearestNode(id, true)` - Find nearest object/NPC by ID
- `getNearestNode(name, true)` - Find nearest by name

**Grand Exchange:**
- `sellAllOnGe()` - Sell all tradeable items

### State Machines

Bots typically use state machines for complex behavior:

```kotlin
enum class State {
    GETTING_TASK,    // Initialize new task
    GOING_TO_HUB,    // Travel to area
    KILLING_ENEMY,   // Combat + looting
    GOING_TO_BANK,   // Travel to bank
    BANKING,         // Bank items
    GOING_TO_GE,     // Travel to GE
    SELLING          // Sell items
}
```

Each state handles specific logic and transitions to the next state when complete.

### Pulses for Delays

Use Pulse for actions that need delays:

```kotlin
bot.pulseManager.run(object: Pulse(delayTicks) {
    override fun pulse(): Boolean {
        // Action after delay
        return true  // Stop pulse
    }
})
```

Use MovementPulse for pathfinding:

```kotlin
bot.pulseManager.run(object: MovementPulse(bot, destination, Pathfinder.SMART) {
    override fun pulse(): Boolean {
        // Reached destination
        return true
    }
})
```

## Bot Spawning Systems

### ImmerseWorld - Global Bot Spawner

**Location:** `Server/src/main/core/game/world/ImmerseWorld.kt`

StartupListener that spawns bots on server start:
- `CombatBotAssembler` - Spawns combat bots in various areas
- `SkillingBotAssembler` - Spawns skilling bots
- Configurable via `GameWorld.settings.max_adv_bots` and `enable_bots`

Spawning methods:
```kotlin
fun spawnBots() {
    immerseSeersAndCatherby()
    immerseLumbridgeDraynor()
    immerseVarrock()
    // etc...
}
```

### Bot Assemblers

**CombatBotAssembler** - Builds combat-focused bots
**SkillingBotAssembler** - Builds skilling-focused bots

Both use builders/factories to create diverse bot populations with randomized:
- Combat levels
- Equipment tiers
- Skill levels
- Behavior patterns

## AIRepository

**Location:** `Server/src/main/core/game/bots/AIRepository.kt`

Central registry for bot management:
- `PulseRepository` - Maps bot username to BotScriptPulse
- `groundItems` - Tracks items that belong to specific bots
- Allows lookup and management of active bots

## Command Interface

**Location:** `Server/src/main/core/game/system/command/sets/BottingCommandSet.kt`

Admin/testing commands:
- `::script <identifier>` - Start bot script on player (requires warning acknowledgement)
- `::stopscript` - Stop currently running script
- Scripts must be registered in `PlayerScripts.identifierMap`

**Important:** Running a bot script permanently removes player from highscores.

## Common Patterns

### Looting System

```kotlin
val items = AIRepository.groundItems.get(bot)
if (items != null && items.isNotEmpty()) {
    scriptAPI.takeNearestGroundItem(items[0].id)
}
```

### Zone-Based Behavior

```kotlin
if (taskZone.insideBorder(bot)) {
    // Do task
} else {
    scriptAPI.walkTo(taskZone.randomLoc)
}
```

### Teleport Flags

```kotlin
var teleportFlag = false

if (!teleportFlag) {
    scriptAPI.teleport(destination)
    teleportFlag = true
} else {
    // Do actions at destination
}
```

### Banking Pattern

```kotlin
for (item in bot.inventory.toArray()) {
    item ?: continue
    if (item.shouldKeep()) continue
    bot.bank.add(item)
}
bot.inventory.clear()
// Re-add essential items
```

## Configuration

Bots respect server configuration:
- `enable_bots` - Master switch for bot spawning
- `max_adv_bots` - Maximum number of adventurer bots
- `enabled_botting` - Allow players to use bot scripts

## Performance Considerations

- **Tick budget:** Bot pulses are rate-limited per tick
- **Pulse management:** Don't run unnecessary actions when bot has active pulse
- **Region loading:** Bots load/unload with regions like players
- **Memory:** Each bot is a full Player object with all systems

## Quick Reference

| Task | Method/Pattern |
|------|----------------|
| Create bot | `GeneralBotCreator(script, location)` |
| Walk | `scriptAPI.walkTo(location)` |
| Attack NPC | `scriptAPI.attackNpcInRadius(bot, name, radius)` |
| Eat food | `scriptAPI.eat(foodId)` |
| Pick up item | `scriptAPI.takeNearestGroundItem(itemId)` |
| Find object | `scriptAPI.getNearestNode(id, true)` |
| Delay action | `Pulse(delayTicks)` |
| Check in zone | `zoneBorders.insideBorder(bot)` |
| Set skill level | `skills[Skills.ATTACK] = 99` |

## Example: Simple Combat Bot

```kotlin
class SimpleCombatBot : Script() {
    val FOOD = Items.LOBSTER_379
    val TARGET_NAME = "Cow"
    val COMBAT_ZONE = ZoneBorders(3200, 3200, 3250, 3250)
    
    init {
        skills[Skills.ATTACK] = 40
        skills[Skills.STRENGTH] = 40
        skills[Skills.DEFENCE] = 40
        inventory.add(Item(FOOD, 10))
    }
    
    override fun tick() {
        scriptAPI.eat(FOOD)
        
        // Loot first
        val items = AIRepository.groundItems.get(bot)
        if (items?.isNotEmpty() == true) {
            scriptAPI.takeNearestGroundItem(items[0].id)
            return
        }
        
        // Stay in zone
        if (!COMBAT_ZONE.insideBorder(bot)) {
            scriptAPI.walkTo(COMBAT_ZONE.randomLoc)
            return
        }
        
        // Attack
        scriptAPI.attackNpcInRadius(bot, TARGET_NAME, 20)
    }
    
    override fun newInstance() = SimpleCombatBot()
}
```

## Related Files

- `AIPlayer.java` - Bot entity implementation
- `Script.java` - Bot script base class
- `ScriptAPI.kt` - Helper methods for bot actions
- `GeneralBotCreator.kt` - Bot launching system
- `AIRepository.kt` - Bot registry and tracking
- `ImmerseWorld.kt` - Global bot spawning
- `GenericSlayerBot.kt` - Complete bot example
- `BottingCommandSet.kt` - Bot commands

