---
name: 2009scape-bot-testing
description: Use when writing automated tests for bot scripts in 2009scape, verifying bot behavior, or testing game systems that interact with bots - covers MockPlayer, TestUtils, and JUnit 5 testing patterns
---

# Testing Server-Sided Bots in 2009scape

## Overview

2009scape uses **JUnit 5** for automated testing with custom `MockPlayer` and `TestUtils` infrastructure to simulate bots and game state without running a full server. Tests verify bot behavior, game mechanics, and system interactions.

## Test Infrastructure

### TestUtils - Test Environment Setup

**Location:** `Server/src/test/kotlin/TestUtils.kt`

Provides utilities for setting up test environments:

**Key functions:**
```kotlin
// Initialize game world, repositories, definitions
preTestSetup()

// Create mock player (human or bot)
getMockPlayer(name: String, isBot: Boolean = false): MockPlayer

// Advance game ticks for time-based tests
advanceTicks(ticks: Int, waitForPulses: Boolean = true)
```

**Usage pattern:**
```kotlin
class MyBotTest {
    init {
        TestUtils.preTestSetup()  // Run once before tests
    }
    
    @Test
    fun testBotBehavior() {
        val bot = TestUtils.getMockPlayer("testbot", isBot = true)
        // Test logic
    }
}
```

### MockPlayer - Simulated Player

**Location:** `Server/src/test/kotlin/TestUtils.kt`

MockPlayer extends Player with test-friendly features:

**Key capabilities:**
```kotlin
// Create mock player
val player = TestUtils.getMockPlayer("testuser")

// Configure properties
player.location = Location.create(3222, 3218, 0)
player.skills.setLevel(Skills.ATTACK, 60)

// Simulate relogging (save/load cycle)
player.relog(ticksToWait = 10)

// Clean up after test
player.close()
```

**AutoCloseable support:**
```kotlin
TestUtils.getMockPlayer("test").use { player ->
    // Test code
} // Automatically closed
```

### MockSession

Simulated network session for testing:
- No actual network I/O
- Allows packet testing
- Enables login/logout simulation

## Test Configuration

### test.conf

**Location:** `Server/src/test/resources/test.conf`

Test-specific configuration:
```toml
[server]
secret_key = "2009scape_development"
noauth_default_admin = true  # All test players are admins

[database]
database_name = "global"
database_username = "2009scapetest"
database_password = "2009scapetest"

[world]
enable_bots = true
max_adv_bots = 100
enable_botting = false  # Disable player botting in tests
start_gui = false
```

## Writing Bot Tests

### Basic Test Structure

```kotlin
import org.junit.jupiter.api.Test
import org.junit.jupiter.api.Assertions.*

class MyBotTest {
    var testBot: MockPlayer
    
    init {
        TestUtils.preTestSetup()
        testBot = TestUtils.getMockPlayer("bot", isBot = true)
    }
    
    @Test
    fun `bot should perform action`() {
        // Arrange: Set up test conditions
        testBot.location = Location.create(3222, 3218, 0)
        
        // Act: Execute bot behavior
        // Bot script logic here
        
        // Assert: Verify results
        assertTrue(testBot.inventory.contains(expectedItem))
    }
}
```

### Pest Control Bot Tests

**Location:** `Server/src/test/kotlin/content/minigame/pestcontrol/bots/BrawlerHandlingTest.kt`

Example test structure for complex bot behavior:

```kotlin
class BrawlerHandlingTest {
    @Test
    fun `bot should prioritize spinner over portal`() {
        // Test that spinners healing portals are top priority
        // Requires mocking framework
        assertTrue(true) // Placeholder
    }
    
    @Test
    fun `bot should attack blocking brawler when portal is accessible`() {
        // Test that brawlers directly blocking paths are handled
        assertTrue(true)
    }
    
    @Test
    fun `bot should ignore non-threatening brawlers when attacking portal`() {
        // Test that distant brawlers are deprioritized
        assertTrue(true)
    }
}
```

**Key test scenarios:**
- Priority targeting (spinners vs brawlers vs portals)
- Pathfinding decisions (attack vs path around blockers)
- Rate limiting (detection intervals)
- Threat assessment (threatening vs non-threatening NPCs)

## Test Patterns

### Mocking Game State

```kotlin
@Test
fun testBotInZone() {
    val bot = TestUtils.getMockPlayer("bot", isBot = true)
    val zone = ZoneBorders(3200, 3200, 3250, 3250)
    
    bot.location = zone.randomLoc
    assertTrue(zone.insideBorder(bot))
}
```

### Testing Combat Behavior

```kotlin
@Test
fun testBotAttacks() {
    val bot = TestUtils.getMockPlayer("bot", isBot = true)
    bot.skills.setLevel(Skills.ATTACK, 99)
    
    val npc = NPC.create(1, bot.location.transform(1, 0, 0))
    npc.init()
    
    // Simulate bot attacking
    bot.attack(npc)
    
    TestUtils.advanceTicks(5, true)
    
    assertTrue(bot.inCombat())
}
```

### Testing Inventory Operations

```kotlin
@Test
fun testBotLooting() {
    val bot = TestUtils.getMockPlayer("bot", isBot = true)
    val item = Item(Items.BONES_526)
    
    // Simulate item drop
    GroundItemManager.create(item, bot.location, bot)
    
    // Bot should pick it up
    // ... bot script logic ...
    
    assertTrue(bot.inventory.contains(Items.BONES_526))
}
```

### Testing State Machines

```kotlin
@Test
fun testStateTransitions() {
    val bot = TestUtils.getMockPlayer("bot", isBot = true)
    val script = MyBotScript()
    script.bot = bot
    
    assertEquals(State.IDLE, script.state)
    
    script.tick()
    assertEquals(State.MOVING, script.state)
    
    TestUtils.advanceTicks(10, true)
    script.tick()
    assertEquals(State.COMBAT, script.state)
}
```

### Testing with Time Progression

```kotlin
@Test
fun testBotOverTime() {
    val bot = TestUtils.getMockPlayer("bot", isBot = true)
    val script = MyBotScript()
    
    // Run bot for 100 game ticks
    repeat(100) {
        if (!bot.pulseManager.hasPulseRunning()) {
            script.tick()
        }
        TestUtils.advanceTicks(1, true)
    }
    
    // Verify bot completed task
    assertTrue(script.taskCompleted)
}
```

### Testing Relog Persistence

```kotlin
@Test
fun testBotPersistence() {
    val bot = TestUtils.getMockPlayer("bot", isBot = true)
    bot.inventory.add(Item(Items.COINS_995, 1000))
    bot.setAttribute("/save:test_value", 42)
    
    // Simulate relog (save and restore)
    bot.relog(ticksToWait = 5)
    
    assertEquals(1000, bot.inventory.getAmount(Items.COINS_995))
    assertEquals(42, bot.getAttribute("test_value", 0))
}
```

## Test Organization

### Test Class Lifecycle

```kotlin
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class MyBotTests {
    companion object {
        init {
            TestUtils.preTestSetup()  // Once for all tests
        }
    }
    
    @Test
    fun test1() { /* ... */ }
    
    @Test
    fun test2() { /* ... */ }
    
    @AfterAll
    fun cleanup() {
        // Clean up resources if needed
    }
}
```

### Parameterized Tests

```kotlin
@ParameterizedTest
@ValueSource(strings = ["Cow", "Chicken", "Goblin"])
fun testBotAttacksDifferentNPCs(npcName: String) {
    val bot = TestUtils.getMockPlayer("bot", isBot = true)
    // Test against different NPC types
}
```

## Assertions and Verification

### Common Assertions

```kotlin
// Location checks
assertEquals(expectedLocation, bot.location)
assertTrue(bot.location.withinDistance(target, radius))
assertTrue(zone.insideBorder(bot))

// Inventory checks
assertTrue(bot.inventory.contains(itemId))
assertEquals(expectedAmount, bot.inventory.getAmount(itemId))
assertTrue(bot.inventory.freeSlots() > 0)

// Combat checks
assertTrue(bot.inCombat())
assertEquals(expectedTarget, bot.properties.combatPulse.victim)

// Skill checks
assertEquals(expectedLevel, bot.skills.getLevel(Skills.ATTACK))

// Attribute checks
assertEquals(expectedValue, bot.getAttribute("key", defaultValue))

// State checks
assertFalse(bot.pulseManager.hasPulseRunning())
assertTrue(bot.locks.isMovementLocked)
```

## Advanced Testing

### Testing Bot Scripts with Real Scripts

```kotlin
@Test
fun testGenericSlayerBot() {
    val bot = TestUtils.getMockPlayer("slayerbot", isBot = true)
    val script = GenericSlayerBot()
    script.bot = bot
    script.init(false)
    
    // Verify initial setup
    assertEquals(99, bot.skills.getStaticLevel(Skills.SLAYER))
    assertTrue(bot.inventory.contains(Items.LOBSTER_379))
    
    // Run bot through states
    assertEquals(State.GETTING_TASK, script.state)
    script.tick()
    // ... verify state transitions
}
```

### Testing NPC Interactions

```kotlin
@Test
fun testBotVsNPC() {
    val bot = TestUtils.getMockPlayer("bot", isBot = true)
    bot.skills.setLevel(Skills.ATTACK, 99)
    bot.skills.setLevel(Skills.STRENGTH, 99)
    
    val npc = NPC.create(NPCIds.COW_81, bot.location.transform(3, 0, 0))
    npc.init()
    
    bot.attack(npc)
    
    // Run combat for several ticks
    repeat(20) {
        TestUtils.advanceTicks(1, true)
        if (npc.skills.lifepoints <= 0) break
    }
    
    assertFalse(npc.isActive)  // NPC should be dead
}
```

### Testing with Multiple Bots

```kotlin
@Test
fun testMultipleBots() {
    val bot1 = TestUtils.getMockPlayer("bot1", isBot = true)
    val bot2 = TestUtils.getMockPlayer("bot2", isBot = true)
    
    bot1.location = Location.create(3222, 3218, 0)
    bot2.location = Location.create(3223, 3218, 0)
    
    assertTrue(bot1.location.withinDistance(bot2.location, 2))
    
    bot1.close()
    bot2.close()
}
```

## Testing Pest Control Bots

### Test Structure for Minigame Bots

```kotlin
class PestControlBotTest {
    lateinit var bot: MockPlayer
    lateinit var session: PestControlSession
    
    @BeforeEach
    fun setup() {
        TestUtils.preTestSetup()
        bot = TestUtils.getMockPlayer("pcbot", isBot = true)
        
        // Mock PC session
        // session = createMockPCSession()
        // bot.addExtension(PestControlSession::class.java, session)
    }
    
    @Test
    fun `bot prioritizes spinners healing portals`() {
        // Arrange: Create portal and spinner
        // val portal = createMockPortal()
        // val spinner = createMockSpinner(nearPortal = true)
        
        // Act: Run bot combat state
        // val combatState = CombatState(bot)
        // combatState.goToPortals()
        
        // Assert: Bot targets spinner first
        // assertEquals(spinner, bot.properties.combatPulse.victim)
    }
    
    @AfterEach
    fun cleanup() {
        bot.close()
    }
}
```

## Common Testing Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `preTestSetup()` | Call in `init` block or `@BeforeAll` |
| Not advancing ticks | Use `TestUtils.advanceTicks()` for time-based tests |
| Not closing MockPlayers | Use `.use {}` or `.close()` in cleanup |
| Testing with real database | Use test.conf with test database |
| Assuming instant actions | Many actions require ticks to complete |
| Not checking pulse state | Verify `!bot.pulseManager.hasPulseRunning()` |

## Quick Reference

| Task | Method |
|------|--------|
| Setup tests | `TestUtils.preTestSetup()` |
| Create bot | `TestUtils.getMockPlayer("name", isBot = true)` |
| Advance time | `TestUtils.advanceTicks(count, waitForPulses)` |
| Set location | `bot.location = Location.create(x, y, z)` |
| Set skill | `bot.skills.setLevel(Skills.ATTACK, level)` |
| Add item | `bot.inventory.add(Item(itemId, amount))` |
| Check combat | `bot.inCombat()` |
| Check pulse | `bot.pulseManager.hasPulseRunning()` |
| Relog test | `bot.relog(ticksToWait)` |
| Cleanup | `bot.close()` or use `.use {}` |

## Example: Complete Bot Test

```kotlin
import org.junit.jupiter.api.*
import org.junit.jupiter.api.Assertions.*

class SimpleCombatBotTest {
    lateinit var bot: MockPlayer
    lateinit var script: SimpleCombatBot
    
    companion object {
        @BeforeAll
        @JvmStatic
        fun setupClass() {
            TestUtils.preTestSetup()
        }
    }
    
    @BeforeEach
    fun setup() {
        bot = TestUtils.getMockPlayer("testbot", isBot = true)
        script = SimpleCombatBot()
        script.bot = bot
        script.init(false)
    }
    
    @Test
    fun `bot initializes with correct stats`() {
        assertEquals(40, bot.skills.getStaticLevel(Skills.ATTACK))
        assertEquals(40, bot.skills.getStaticLevel(Skills.STRENGTH))
        assertTrue(bot.inventory.contains(Items.LOBSTER_379))
    }
    
    @Test
    fun `bot eats food when damaged`() {
        val initialFood = bot.inventory.getAmount(Items.LOBSTER_379)
        bot.skills.lifepoints = 10  // Damage bot
        
        script.tick()
        
        // Should have eaten
        assertTrue(bot.skills.lifepoints > 10)
        assertEquals(initialFood - 1, bot.inventory.getAmount(Items.LOBSTER_379))
    }
    
    @Test
    fun `bot attacks target in range`() {
        bot.location = Location.create(3222, 3218, 0)
        
        val cow = NPC.create(NPCIds.COW_81, 
            bot.location.transform(5, 0, 0))
        cow.init()
        
        script.tick()
        TestUtils.advanceTicks(2, true)
        
        assertTrue(bot.inCombat())
        assertEquals(cow, bot.properties.combatPulse.victim)
    }
    
    @AfterEach
    fun cleanup() {
        bot.close()
    }
}
```

## Related Files

- `TestUtils.kt` - Test utilities and MockPlayer
- `APITests.kt` - Example API tests
- `BrawlerHandlingTest.kt` - Pest Control bot tests
- `ExchangeTests.kt` - Grand Exchange testing
- `test.conf` - Test configuration

