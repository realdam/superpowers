---
name: pest-control-implementation
description: Use when developing Pest Control features in 2009scape, implementing similar minigames, debugging PC issues, or creating PC bots - deep dive into technical implementation including dynamic regions, session management, specialized NPCs, and bot AI
---

# Pest Control Technical Implementation in 2009scape

## Overview

Pest Control in 2009scape uses a **dynamic region system** with session management, specialized NPC plugins, bot AI with state machines, and activity coordination. The implementation spans multiple systems working together to create the cooperative minigame experience.

## Architecture Overview

### Core Components

1. **PestControlActivityPlugin** - Activity management and player coordination
2. **PestControlSession** - Individual game instance management
3. **DynamicRegion** - Isolated game instance
4. **PC Monster NPCs** - Specialized NPC behavior (Portal, Squire, Monsters)
5. **Bot AI** - PestControlTestBot with CombatState
6. **Zones** - PCLanderZone and PCIslandZone

## Activity System

### PestControlActivityPlugin

**Location:** `Server/src/main/content/minigame/pestcontrol/PestControlActivityPlugin.java`

Manages the Pest Control activity globally:

**Boat types:**
```java
enum class BoatType(val requirement: Int) {
    NOVICE(40),      // Level 40+
    INTERMEDIATE(70), // Level 70+
    VETERAN(100)     // Level 100+
}
```

**Key responsibilities:**
- Player queue management (`waitingPlayers`)
- Session creation and coordination
- Lander interface management
- Player entry/exit handling
- Registration of all PC plugins

**Registration (startup):**
```java
@Override
public void register() {
    // Load NPC plugins
    ClassScanner.definePlugin(new PCPortalNPC());
    ClassScanner.definePlugin(new PCSquireNPC());
    ClassScanner.definePlugin(new PCTorcherNPC());
    ClassScanner.definePlugin(new PCDefilerNPC());
    // ... all monster types ...
    ClassScanner.definePlugin(new VoidSealPlugin());
    
    // Configure zones
    ZoneBuilder.configure(new PCLanderZone(activities));
    ZoneBuilder.configure(new PCIslandZone());
}
```

**Activity pulse:**
```java
// Runs every tick to check for game start conditions
if (waitingPlayers.size() >= 25 || 
    (waitingPlayers.size() >= 5 && waitForTime())) {
    startNewSession();
}
```

## Session Management

### PestControlSession

**Location:** `Server/src/main/content/minigame/pestcontrol/PestControlSession.java`

Represents a single active game instance:

**Core fields:**
```java
private DynamicRegion region;              // Isolated game area
private PestControlActivityPlugin activity; // Parent activity
private int ticks;                          // Game time counter
private NPC squire;                         // The Void Knight
private final NPC[] portals = new NPC[4];  // The 4 portals
private final List<NPC> aportals;          // Attackable portal list
private boolean active = true;              // Session state
```

**Game tick progression:**
```java
public boolean update() {
    if (squire != null && !squire.isActive()) {
        activity.end(this, false);  // Knight died - loss
        return true;
    }
    
    switch (++ticks) {
        case 50:   // 0:15 - Drop first shield
        case 100:  // 0:30 after first
        case 150:  // 0:30 after second
        case 200:  // 0:30 after third - all portals attackable
            int index = (ticks / 50) - 1;
            removePortalShield(ids[index]);
            return false;
        case 2000: // 20 minutes - time win
            activity.end(this, true);
            return true;
    }
    return false;
}
```

**Shield removal:**
```java
private void removePortalShield(int portalIndex) {
    NPC portal = portals[portalIndex];
    portal.transform(portal.getId() - 4); // Shield off
    aportals.add(portal);                 // Now attackable
    sendMessage("The " + getPortalName(portalIndex) + 
                " portal shield has dropped!");
}
```

**Player management:**
```java
private void addPlayer(Player p, String portalHealth) {
    p.setAttribute("pc_zeal", 0);  // Damage counter
    p.addExtension(PestControlSession.class, this);
    p.getInterfaceManager().openOverlay(new Component(408)); // PC overlay
    
    // Teleport to random spawn
    Location l = region.getBaseLocation();
    p.getProperties().setTeleportLocation(
        l.transform(32 + random.nextInt(4), 
                   49 + random.nextInt(6), 0));
}
```

## Dynamic Region System

### DynamicRegion

**Purpose:** Creates isolated game instances so multiple games run simultaneously

**Key features:**
- Each session gets own region copy
- NPCs/objects independent per region
- Players in different sessions don't see each other
- Allows 3+ concurrent games (one per boat type)

**Region lifecycle:**
```java
// Create
DynamicRegion region = DynamicRegion.create(baseRegionId);

// Add player
region.getPlanes()[0].add(player);

// Destroy when game ends
region.setActive(false);
```

## NPC Implementations

### PCPortalNPC

**Location:** `Server/src/main/content/minigame/pestcontrol/PCPortalNPC.java`

Implements portal behavior:

**Key features:**
```java
@Override
public void handleTickActions() {
    if (!isShielded() && spawnTimer++ % spawnRate == 0) {
        spawnMonster();  // Spawn pest every N ticks
    }
}

@Override
public void finalizeDeath(Entity killer) {
    PestControlSession session = getExtension(PestControlSession.class);
    session.portalDestroyed(this);
    session.healSquire(portalHealAmount);
    session.checkWinCondition();
}
```

**Spawn rate varies by boat difficulty:**
```java
int spawnRate = switch(boatType) {
    case NOVICE -> 15;       // Spawn every 15 ticks
    case INTERMEDIATE -> 12;  // Faster spawns
    case VETERAN -> 10;       // Fastest spawns
};
```

### PCSquireNPC (Void Knight)

**Location:** `Server/src/main/content/minigame/pestcontrol/PCSquireNPC.java`

The defendable NPC:

**Properties:**
```java
// Higher HP for higher difficulties
int squireHP = switch(boatType) {
    case NOVICE -> 2000;
    case INTERMEDIATE -> 2500;
    case VETERAN -> 3000;
};

// Cannot be poisoned
@Override
public boolean isPoisonImmune() { return true; }

// Death triggers loss
@Override
public void finalizeDeath(Entity killer) {
    session.end(false);  // Players lose
}
```

### Monster NPCs

All Pest Control monsters extend specialized NPC classes:

**PCBrawlerNPC:**
```java
// Blocks pathing
@Override
public boolean isBlockingPath() { return true; }

// High defense, defends portals
@Override
public void handleTickActions() {
    if (nearPortal() && !inCombat()) {
        guardPortal();
    }
}
```

**PCSpinnerNPC:**
```java
// Heals portals
@Override
public void handleTickActions() {
    NPC nearestPortal = findNearestDamagedPortal();
    if (nearestPortal != null) {
        healPortal(nearestPortal, HEAL_AMOUNT);
        playHealSound();
    }
}

// Explodes and poisons if portal destroyed first
@Override
public void onPortalDestroyed(NPC portal) {
    if (isHealingPortal(portal)) {
        explodeAndPoison();
    }
}
```

**PCRavagerNPC:**
```java
// Attacks gates and barricades
@Override
public void selectTarget() {
    GameObject nearestGate = findNearestGate();
    if (nearestGate != null) {
        attackObject(nearestGate);
    }
}
```

**PCShifterNPC:**
```java
// Can teleport through walls
@Override
public void handleTickActions() {
    if (shouldTeleport()) {
        Location knightLoc = session.getSquire().getLocation();
        teleport(knightLoc.randomize(3));
        attack(session.getSquire());
    }
}
```

**PCSplatterNPC:**
```java
// Detonates on death or reaching fort
@Override
public void finalizeDeath(Entity killer) {
    detonateAOE(EXPLOSION_RADIUS, EXPLOSION_DAMAGE);
}

@Override
public void handleTickActions() {
    if (reachedTarget()) {
        detonateAOE(EXPLOSION_RADIUS, EXPLOSION_DAMAGE);
        finalizeDeath(null);
    }
}
```

## Bot AI Implementation

### PestControlTestBot

**Location:** `Server/src/main/content/minigame/pestcontrol/bots/` (referenced in code)

Bot script for Pest Control:

**Core structure:**
```kotlin
class PestControlTestBot : Script() {
    var customState = ""
    var movetimer = 0
    var justStartedGame = true
    var openedGate = false
    
    override fun tick() {
        if (movetimer > 0) {
            movetimer--
            return  // Wait before next action
        }
        
        val combatState = CombatState(this)
        combatState.goToPortals()
    }
}
```

### CombatState - Bot AI Logic

**Location:** `Server/src/main/content/minigame/pestcontrol/bots/CombatState.kt`

Implements strategic decision-making:

**Priority targeting system:**
```kotlin
fun goToPortals() {
    val portal = session.getNextPortal()
    
    when {
        // PRIORITY 1: Spinners healing portals
        spinners.isNotEmpty() -> {
            bot.attack(spinners.random())
        }
        
        // PRIORITY 2: Brawlers blocking portal access
        blockingBrawlers.isNotEmpty() -> {
            if (Random.nextInt(100) < 60) {
                bot.attack(blockingBrawlers.random())  // 60% attack
            } else {
                pathAroundBrawler()  // 40% path around
            }
        }
        
        // PRIORITY 3: Portal itself
        portal != null -> {
            bot.attack(portal)
        }
        
        // FALLBACK: Attack nearby threats
        else -> {
            attackNearbyThreats()
        }
    }
}
```

**Brawler handling optimization:**
```kotlin
private var lastBrawlerCheckTick = 0
private val BRAWLER_CHECK_INTERVAL = 5  // Check every 5 ticks

private fun findNearbyBrawlersOptimized(): List<NPC> {
    val currentTick = GameWorld.ticks
    if (currentTick - lastBrawlerCheckTick < BRAWLER_CHECK_INTERVAL) {
        return emptyList()  // Rate limiting
    }
    
    lastBrawlerCheckTick = currentTick
    return findNearbyBrawlers()
}
```

**Threat assessment:**
```kotlin
private fun isNearSquire(location: Location): Boolean {
    val squireLoc = session.squire?.location ?: return false
    return location.withinDistance(squireLoc, 15)
}

private fun isNearOtherBots(location: Location): Boolean {
    return RegionManager.getLocalPlayers(bot).any { player ->
        player !== bot && 
        player is PvMBots &&
        player.location.withinDistance(location, 8)
    }
}

// Only attack threatening brawlers
val threateningBrawlers = nearbyBrawlers.filter { brawler ->
    isNearSquire(brawler.location) || 
    isNearOtherBots(brawler.location)
}
```

**Path blocking detection:**
```kotlin
private fun isPathBlockedByBrawler(targetLoc: Location): Boolean {
    val brawlers = findNearbyBrawlers(8)
    return brawlers.any { brawler ->
        val botToBrawler = bot.location.getDistance(brawler.location)
        val brawlerToTarget = brawler.location.getDistance(targetLoc)
        val botToTarget = bot.location.getDistance(targetLoc)
        
        // Brawler is blocking if on direct path (within 2 tiles)
        (botToBrawler + brawlerToTarget) <= (botToTarget + 2)
    }
}
```

### CombatStateIntermediate

**Location:** `Server/src/main/content/minigame/pestcontrol/bots/CombatStateIntermediate.kt`

Alternative/simplified bot AI variant for intermediate difficulty bots.

## Zones and Areas

### PCLanderZone

**Location:** Zone configuration for lander areas

**Responsibilities:**
- Detects player entry to lander
- Starts activity for player
- Manages waiting queue
- Updates lander interface

### PCIslandZone

**Location:** Zone configuration for game island

**Responsibilities:**
- Detects player entry/exit
- Manages combat restrictions
- Handles death respawning
- Enforces game rules (no pets, etc.)

## Helper Systems

### PestControlHelper

**Location:** `Server/src/main/content/minigame/pestcontrol/PestControlHelper.kt`

Utility functions and constants:

```kotlin
object PestControlHelper {
    val GATE_ENTRIES = listOf(14233, 14235)
    val Portals_AttackableN = listOf(6142, 6143, 6144, 6145)
    val Portals_AttackableI = listOf(6150, 6151, 6152, 6153)
    val Portals_AttackableV = listOf(7551, 7552, 7553, 7554)
    
    fun isInPestControlInstance(p: Player): Boolean {
        return p.getAttribute<Any?>("pc_zeal") != null
    }
    
    fun getMyPestControlSession(p: Player): PestControlSession? {
        return p.getExtension(PestControlSession::class.java)
    }
}
```

### VoidSealPlugin

**Location:** `Server/src/main/content/minigame/pestcontrol/VoidSealPlugin.java`

Handles Void Knight Seal item:

```java
@Override
public boolean handle(Player player, Node node, String option) {
    // Check in PC region
    if (player.getViewport().getRegion().getRegionId() != 10536) {
        player.sendMessage("You can only use the seal in Pest Control.");
        return true;
    }
    
    // AoE damage to nearby pests
    player.graphics(Graphics.create(1177));
    for (NPC npc : RegionManager.getLocalNpcs(player, 2)) {
        if (canTarget(npc)) {
            npc.getImpactHandler().manualHit(player, 
                7 + RandomFunction.randomize(5), 
                HitsplatType.NORMAL, 1);
            npc.graphics(Graphics.create(1176));
        }
    }
    
    // Consume charge
    decreaseCharges(item);
    return true;
}
```

## Damage Tracking System

### Player Damage Counter

Players must deal 50 damage to qualify for rewards:

```java
// When player damages pest or portal
@Override
public void onDamageDealt(Player attacker, int damage) {
    int currentZeal = attacker.getAttribute("pc_zeal", 0);
    attacker.setAttribute("pc_zeal", currentZeal + damage);
}

// When repairing gate/barricade
@Override
public void onRepair(Player player) {
    int currentZeal = player.getAttribute("pc_zeal", 0);
    player.setAttribute("pc_zeal", currentZeal + 5);  // +5 per repair
}

// At game end - check eligibility
public void giveRewards(Player player) {
    int zeal = player.getAttribute("pc_zeal", 0);
    if (zeal < 50) {
        player.sendMessage("You did not do enough damage to earn points.");
        return;
    }
    awardPoints(player);
}
```

## Win Condition Checking

```java
public void checkWinCondition() {
    // Check if all portals destroyed
    boolean allPortalsDestroyed = true;
    for (NPC portal : portals) {
        if (portal.isActive()) {
            allPortalsDestroyed = false;
            break;
        }
    }
    
    if (allPortalsDestroyed) {
        activity.end(this, true);  // Victory!
        return;
    }
    
    // Check if 20 minutes elapsed (handled in update())
    // Check if squire died (handled in squire death handler)
}
```

## Rewards System

### Point Calculation

```java
public void awardPoints(Player player) {
    int points = switch(activity.getType()) {
        case NOVICE -> 2;
        case INTERMEDIATE -> 3;
        case VETERAN -> 4;
    };
    
    int currentPoints = player.getAttribute("pc_points", 0);
    int newTotal = Math.min(currentPoints + points, 250); // Cap at 250
    
    player.setAttribute("/save:pc_points", newTotal);
    
    // Bonus coins if all portals destroyed
    if (allPortalsDestroyed) {
        int bonus = player.getProperties().getCurrentCombatLevel() * 10;
        player.getInventory().add(new Item(Items.COINS_995, bonus));
    }
}
```

### Experience Exchange

```java
public void exchangeForExperience(Player player, int skill, int points) {
    int level = player.getSkills().getStaticLevel(skill);
    if (level < 25) {
        player.sendMessage("You need level 25 to get XP in that skill.");
        return;
    }
    
    // Formula: CEILING((Level+25)*(Level-24)/606,1)*N
    double baseXP = Math.ceil((level + 25) * (level - 24) / 606.0);
    int multiplier = getMultiplier(skill);  // 35, 32, or 18
    double xpPerPoint = baseXP * multiplier;
    
    // Bonus for bulk trades
    double bonus = points >= 100 ? 1.10 : (points >= 10 ? 1.01 : 1.0);
    
    double totalXP = xpPerPoint * points * bonus;
    player.getSkills().addExperience(skill, totalXP);
}
```

## Testing Infrastructure

### BrawlerHandlingTest

**Location:** `Server/src/test/kotlin/content/minigame/pestcontrol/bots/BrawlerHandlingTest.kt`

Test cases for bot AI:

```kotlin
@Test
fun `bot should prioritize spinner over portal`() {
    // Test: Spinner healing portal = top priority
    // Mock: Create portal, spinner, bot
    // Assert: Bot targets spinner first
}

@Test
fun `bot should attack blocking brawler when portal is accessible`() {
    // Test: Brawler directly blocking path
    // Mock: Create portal, blocking brawler, bot
    // Assert: Bot attacks brawler OR paths around (60/40)
}

@Test
fun `bot should use rate limiting for brawler detection`() {
    // Test: Detection only every 5 ticks
    // Mock: Time progression
    // Assert: findNearbyBrawlers called at correct intervals
}
```

## Configuration Integration

### Boat Configuration

```kotlin
enum class BoatInfo(
    val boatBorder: ZoneBorders,
    val outsideBoatBorder: ZoneBorders,
    val ladderId: Int
) {
    NOVICE(
        ZoneBorders(2656, 2639, 2662, 2643),
        ZoneBorders(2660, 2638, 2663, 2644),
        25631
    ),
    INTERMEDIATE(
        ZoneBorders(2643, 2644, 2649, 2648),
        ZoneBorders(2640, 2643, 2651, 2650),
        25632
    ),
    VETERAN(
        ZoneBorders(2634, 2652, 2641, 2657),
        ZoneBorders(2632, 2651, 2643, 2660),
        25633
    )
}
```

## Performance Considerations

### Optimization Strategies

**1. Portal spawn rate limiting:**
```java
if (spawnTimer++ % spawnRate == 0) {
    spawnMonster();  // Not every tick
}
```

**2. Bot AI rate limiting:**
```kotlin
// Check brawlers every 5 ticks, not every tick
if (currentTick - lastCheck >= 5) {
    checkBrawlers()
}
```

**3. Region isolation:**
- Each session in separate DynamicRegion
- Prevents cross-session interference
- Reduces NPC search radius per session

**4. NPC cleanup:**
```java
@Override
public void finalizeDeath(Entity killer) {
    super.finalizeDeath(killer);
    region.removeNPC(this);  // Clean up immediately
}
```

## Common Implementation Patterns

### Session Access Pattern

```kotlin
// From player
val session = player.getExtension(PestControlSession::class.java)

// From NPC
val session = npc.getExtension(PestControlSession::class.java)

// From bot AI
val session = getMyPestControlSession1(bot)
```

### State Synchronization

```java
// Send updates to all players in session
public void broadcastMessage(String message) {
    for (Player p : region.getPlanes()[0].getPlayers()) {
        if (p.isActive()) {
            p.sendMessage(message);
        }
    }
}

// Update interface for all players
public void updatePortalHealth(int portalIndex, int health) {
    sendString("" + health, 13 + portalIndex);
}
```

### Portal Shield Pattern

```java
// Portals start shielded (ID + 4)
portal.transform(baseId + 4);  // Shielded

// Remove shield when time comes
portal.transform(portal.getId() - 4);  // Unshielded
aportals.add(portal);  // Now attackable
```

## Quick Reference

| Component | Purpose | Key Files |
|-----------|---------|-----------|
| PestControlActivityPlugin | Activity management | PestControlActivityPlugin.java |
| PestControlSession | Game instance | PestControlSession.java |
| PCPortalNPC | Portal behavior | PCPortalNPC.java |
| PCSquireNPC | Void Knight | PCSquireNPC.java |
| Monster NPCs | Specialized pest behavior | PC*NPC.java files |
| CombatState | Bot AI decision-making | CombatState.kt |
| PestControlHelper | Utilities | PestControlHelper.kt |
| VoidSealPlugin | Void seal item | VoidSealPlugin.java |
| BrawlerHandlingTest | Bot AI tests | BrawlerHandlingTest.kt |

## Related Systems

- **Activity System** - Base minigame framework
- **DynamicRegion** - Instanced area system
- **NPC Plugin System** - Custom NPC behavior
- **Bot AI Framework** - Script and AIPlayer system
- **Zone System** - Area-based triggers
- **Combat System** - Damage dealing and tracking
- **Reward System** - Point tracking and exchange

