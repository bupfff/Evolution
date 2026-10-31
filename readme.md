# EVOLUTION

**An evolving army. An adaptive enemy. A battlefield shaped by
intelligence.**

EVOLUTION is a top-down 2D pixel-art real-time strategy and simulation
game built in Unity. The player recruits, trains, equips, and directs a
persistent army of autonomous soldiers. Each soldier has individual
behavioral tendencies, experience, memories, and a history. Soldiers can
learn during their lifetime, share useful tactical discoveries with
their army, and pass selected inherited traits to future generations.

The central promise is not simply that units have AI. It is that the
player's army becomes observably more capable over time---and that its
tactics emerge from the interaction of individual decisions, shared
knowledge, training, and evolution.

> **Design principle:** The player sets goals and creates conditions for
> learning. Soldiers interpret situations and make their own combat
> decisions.

------------------------------------------------------------------------

## Project status

-   **Status:** Concept / pre-production
-   **Engine:** Unity 2D
-   **Language:** C#
-   **Genre:** Real-time strategy / autonomous-agent simulation /
    evolutionary strategy
-   **Visual direction:** Top-down pixel art
-   **First playable target:** EVOLUTION Arena, 20 friendly soldiers
    versus 20 enemy soldiers
-   **Primary technical priority:** A deterministic, testable simulation
    separated from rendering

## 1. Vision and pillars

### 1.1 The player fantasy

The player begins with a small, inexperienced force. Through battles,
training, recruitment, experimentation, and selective breeding, that
force develops into a distinct military organization. Veterans become
recognizable. Soldiers develop different tendencies. Successful tactics
spread. Enemy factions adapt. The player gradually becomes the caretaker
and strategic commander of an evolving military ecosystem.

### 1.2 Core pillars

1.  **Autonomous individuals:** Soldiers perceive local conditions and
    select actions without constant micromanagement.
2.  **Meaningful individual development:** A soldier's behavior can
    change through its own experience during its lifetime.
3.  **Collective intelligence:** The army records tactical evidence and
    makes useful discoveries available to other soldiers.
4.  **Evolution and legacy:** Inheritance and mutation shape future
    generations while preserving behavioral diversity.
5.  **Adaptive opposition:** Enemy factions can develop counters to
    repeated player strategies in later milestones.
6.  **Observable simulation:** The player can inspect decisions,
    histories, lineages, battle events, and performance trends.
7.  **Strategic command, not unit babysitting:** The player issues
    objectives and constraints; soldiers retain local autonomy.

### 1.3 What success looks like

After many battles, a player should be able to identify veterans,
recognize the tendencies of different lineages, notice tactics spreading
through the army, and occasionally see soldiers coordinate in ways the
player did not explicitly script.

The game must not fake this with random animations or descriptive text.
Any claim that the army is improving must be supported by repeatable
simulation data.

------------------------------------------------------------------------

## 2. Core gameplay loop

1.  Review the army, soldier histories, equipment, morale, and tactical
    knowledge.
2.  Recruit new soldiers, including eligible captured enemies.
3.  Assign equipment, training, squads, and optional lineage policies.
4.  Choose a battlefield, enemy faction, and strategic objective.
5.  Issue high-level orders such as attack, defend, hold, escort, flank,
    capture, or retreat.
6.  Watch autonomous soldiers interpret the orders and fight.
7.  Pause, speed up, inspect units, and review important battle events.
8.  Review casualties, captures, objectives, tactical outcomes, and
    individual performance.
9.  Update personal experience and army knowledge from recorded
    evidence.
10. Run evolution / inheritance for eligible new recruits or the next
    generation.
11. Prepare for an opponent that may be stronger or may have adapted to
    familiar tactics.

The first prototype only needs a subset of this loop. Campaign
progression, complex equipment, enemy evolution, and advanced training
are later features.

------------------------------------------------------------------------

## 3. Soldier model

Every soldier is a persistent data entity, not merely a scene object.
The simulation data must remain valid when a battle is unloaded and must
be serializable.

### 3.1 Core soldier data

Each soldier should eventually contain:

-   Stable unique ID
-   Name or callsign
-   Faction and current squad
-   Health and combat statistics
-   Equipment references
-   Personality / behavioral parameters
-   Current action and current strategic order
-   Experience and battle count
-   Battles survived, kills, assists, damage dealt / received
-   Morale and surrender state
-   Personal memory summary
-   Learned behavioral parameters
-   Neural-network parameters, when that milestone is enabled
-   Parent IDs, lineage ID, and generation
-   Full or summarized battle history
-   Alive / dead / captured / retired status

Separate persistent data (`SoldierData`) from runtime simulation state
(`SoldierRuntime`) and visual presentation (`SoldierView`). Do not make
the data model depend on a `MonoBehaviour`.

### 3.2 Personality is a tendency, not a class

Initial traits should be continuous values, preferably normalized from 0
to 1:

-   Aggression
-   Caution / risk aversion
-   Ally protection
-   Formation discipline
-   Flanking preference
-   Opportunism
-   Retreat willingness
-   Preferred engagement distance

Avoid rigid classes such as "the aggressive unit can only attack." A
defensive soldier should still be able to attack when the situation
warrants it. Traits bias decisions; they do not hard-code them.

A soldier's current behavioral parameters may change through learning.
Its inherited baseline should remain separately recorded so the player
can distinguish inherited tendencies from personal development.

### 3.3 Individual memories

Do not store every simulation tick as a permanent memory. Record
meaningful events, for example:

-   Survived while heavily outnumbered
-   Successfully retreated from a dangerous engagement
-   Helped eliminate a high-threat target
-   Protected an ally or commander
-   Was repeatedly punished by a particular enemy tactic
-   Participated in a successful flank
-   Failed an objective because of poor positioning

Use a bounded memory system: recent events, notable events, and
aggregated statistics. Memory entries should include context, outcome,
confidence, and timestamp / battle ID where practical.

------------------------------------------------------------------------

## 4. Intelligence architecture

Use a layered decision system. Keep the first implementation
understandable and measurable.

### 4.1 Layer A --- Utility AI (first decision system)

Utility AI evaluates possible actions using the soldier's current
situation, personality, order, and available information.

Initial actions:

-   Attack
-   Retreat
-   Defend
-   Flank
-   Follow / regroup
-   Capture
-   Hold position

Each action receives a score. Scores should be based on explicit factors
and normalized where practical. Add a small hysteresis / commitment rule
so units do not switch actions every tick due to tiny score changes.

Examples:

-   Attack becomes more attractive when the target is vulnerable and the
    engagement is favorable.
-   Retreat becomes more attractive when health is low, the soldier is
    surrounded, or nearby enemy strength is overwhelming.
-   Defend becomes more attractive when the unit has a defend order or
    an ally / objective needs protection.
-   Flank becomes more attractive when the enemy is distracted and a
    viable route exists.
-   Capture becomes more attractive when the target is surrendering and
    it is safe to approach.

A simple, reliable Utility AI is the baseline benchmark. It must work
even when neural networks are disabled.

### 4.2 Layer B --- Individual learning

A soldier should be able to adapt within its own lifetime. Start with
interpretable behavioral parameters rather than immediately training a
large neural network.

Examples of learnable parameters:

-   Risk tolerance
-   Preferred distance to enemies
-   Retreat threshold
-   Flank preference
-   Ally-support priority
-   Formation adherence
-   Target preference

Learning must be evidence-based. A single lucky event should not
radically change a soldier. Use small updates, confidence, and minimum
sample counts. Record before / after values and the evidence that caused
a change.

Do not let an individual learn a harmful behavior simply because it
earned a short-term reward. Consider survival, objective progress, team
outcomes, and context.

### 4.3 Layer C --- Neural network (later milestone)

A small feed-forward neural network may eventually influence action
preferences. Keep it plain C# initially; no external ML framework is
required for the prototype.

Proposed initial shape:

-   12 normalized inputs
-   16 hidden neurons
-   6 outputs corresponding to the initial action set

Potential inputs include health, nearest enemy distance, nearby ally /
enemy counts, local strength estimate, surrounded state, target
vulnerability, cover / terrain advantage, current order, and army
tactical recommendation.

Important: A neural network does not learn merely because it exists. The
project must implement and test an actual weight-update or evolutionary
mechanism. Initially, Utility AI remains the safety / sanity layer.
Neural outputs should bias or modify utility scores within bounded
limits, not force nonsensical actions.

Do not implement neural training and genetic evolution simultaneously in
the first prototype. Add the network only after the baseline simulation
and telemetry are reliable.

### 4.4 Layer D --- Collective army intelligence

The army maintains shared tactical knowledge. This is not a single
omniscient brain that directly puppets every unit.

Initial knowledge categories:

-   Flanking effectiveness
-   Defensive formation effectiveness
-   Retreat success
-   Formation cohesion
-   Enemy faction behavior
-   Terrain advantages
-   Coordination effectiveness
-   Target-priority effectiveness

Store evidence, not just a score. For each tactic, track attempts,
successful outcomes, failed outcomes, contexts, and confidence. A tactic
should not become "good" simply because it was used often.

Example: A soldier participates in a successful flank against distracted
archers. The event is recorded. The army increases confidence in that
tactic for similar contexts. Other soldiers can consider the
recommendation when their own local situation is appropriate.

Collective knowledge should be contextual and confidence-weighted. A
tactic that works against archers in open terrain may not work against
shielded knights in a narrow passage.

### 4.5 Decision flow

A typical soldier decision should follow this sequence:

1.  Read current local perception.
2.  Read the active high-level order.
3.  Read relevant army tactical recommendations.
4.  Evaluate possible actions using Utility AI.
5.  If enabled, use the neural network to adjust action preferences
    within safe bounds.
6.  Apply action commitment / hysteresis.
7.  Select an action and execute it through the simulation.
8.  Record meaningful outcomes for later learning.

The player order is an important objective signal, but should not erase
local autonomy. A retreating soldier may temporarily disengage even
while the army has an attack order if survival logic makes that
appropriate.

------------------------------------------------------------------------

## 5. Rewards, evidence, and fitness

### 5.1 Reward design

Avoid a reward system that simply counts kills. Reward useful outcomes
across several dimensions:

-   Damage dealt when tactically appropriate
-   Eliminating high-value threats
-   Surviving / avoiding unnecessary losses
-   Protecting allies
-   Assisting teammates
-   Successfully retreating from danger
-   Completing objectives
-   Capturing surrendering enemies
-   Maintaining formation / supporting a successful maneuver

Penalties may include unnecessary deaths, abandoning critical
objectives, friendly-fire incidents, or charging into overwhelming force
without a strategic reason.

Reward values are tuning parameters, not facts. Log reward components
separately so the developer can see why a soldier received a score.

### 5.2 Team contribution

A soldier that enables a team victory should not be ranked below a
soldier that farms kills. Track individual contribution and team
outcomes separately. Avoid assigning all team success to every unit
equally; use participation / proximity / role context where feasible.

### 5.3 Evolution fitness

A starting fitness model may be:

`Fitness = 0.30 * WinContribution + 0.20 * Survival + 0.25 * Teamwork + 0.15 * ObjectiveContribution + 0.10 * Adaptability`

All components must be normalized to 0--1 and defined precisely before
implementation. The weights are initial tuning values, not final
balance.

Use multiple test scenarios. Preserve behavioral diversity instead of
allowing one strategy to dominate every lineage. Consider novelty /
diversity bonuses or separate sub-populations for different behavioral
niches.

Never evolve from one battle alone. Evaluate candidates over multiple
seeds, maps, and opponent styles to reduce luck and overfitting.

------------------------------------------------------------------------

## 6. Evolution and lineages

Evolution affects future generations. It is distinct from an individual
soldier's lifetime learning.

### 6.1 Inheritance

When creating a new recruit or successor, inheritance may include:

-   Baseline personality parameters
-   Selected neural network weights (once implemented)
-   Lineage identity and generation
-   A limited inherited tactical tendency or preference

Do not copy the parent's complete memories into a new soldier as if the
recruit personally experienced them. If the design includes inherited
knowledge, represent it explicitly as cultural / army knowledge or a
small inherited predisposition.

### 6.2 Mutation and selection

-   Evaluate candidates over repeatable benchmark battles.
-   Preserve multiple successful strategies.
-   Select parents from a diverse pool, not only the single highest
    scorer.
-   Apply small, bounded mutations.
-   Keep some randomized recruits to maintain diversity.
-   Record parentage, mutation summary, and generation.
-   Allow the player to inspect family trees and lineage statistics.

### 6.3 Legacy and loss

Veterans should matter because of their history, not only because their
stats are higher. Their death can be meaningful, but the game should not
make every loss catastrophic. Provide a clear record of notable service
and descendants.

Potential veteran retirement / mentorship mechanics are later features,
not required for the first prototype.

------------------------------------------------------------------------

## 7. Enemy factions and adaptation

Enemy behavior should begin as authored faction profiles. Full enemy
evolution is a later system.

Example faction archetypes:

-   Goblins: scatter when surrounded; favor numbers and opportunistic
    attacks.
-   Knights: resist frontal attacks; maintain disciplined formations.
-   Archers: prefer distance; vulnerable to coordinated pressure.
-   Beasts: become more aggressive when wounded.
-   Heavy infantry: durable but less effective in difficult terrain.

These are starting design examples, not permanent rules. Faction
behavior must be implemented as tunable data rather than scattered
hard-coded checks.

### Later enemy adaptation

In a later milestone, enemy factions may observe repeated player tactics
and develop counters. Adaptation should be gradual and legible:

1.  Track repeated tactics and their outcomes.
2.  Identify patterns with sufficient evidence.
3.  Adjust faction doctrine or unit preferences.
4.  Give the player clues through intelligence reports.
5.  Let the player experiment with counter-tactics.

Do not make enemies omniscient or instantly counter the player. Enemy
adaptation should be fair, contextual, and measurable.

------------------------------------------------------------------------

## 8. Training and experimentation

A training system gives the player a way to influence development
without directly controlling every decision.

Later features may include:

-   Training grounds for basic combat behaviors
-   Scenario drills against knights, archers, beasts, or formations
-   Experimental arena for repeated simulation
-   Squad-level practice
-   Training policies that bias what situations recruits experience
-   Benchmark scenarios to compare lineages

Training should not simply grant invisible stat bonuses. It should
expose soldiers to controlled situations and produce recorded learning
evidence. The initial prototype does not need a training UI; automated
benchmark battles are sufficient.

------------------------------------------------------------------------

## 9. Battle simulation

### 9.1 First prototype configuration

  Setting                                                  Initial target
  --------------------- -------------------------------------------------
  Friendly army                                               20 soldiers
  Enemy army                                                  20 soldiers
  Battlefield                                           One bounded arena
  Combat                       Basic melee / simple ranged only if needed
  AI                                                           Utility AI
  Individual learning                            Simple parameter updates
  Collective learning                     Basic contextual tactic records
  Evolution                                     Small between-battle test
  Persistence                                 Soldier and army data saved
  Rendering               2D top-down pixel art or temporary placeholders

The first prototype should prove that autonomous units can fight, that
decisions are understandable, and that recorded experience can change
future behavior. Do not require the full feature list to call the
prototype successful.

### 9.2 Simulation timing

-   Simulation decisions: initially 5--10 times per second.
-   Rendering: target 60 FPS, independently of simulation decisions.
-   Use a fixed simulation timestep.
-   Avoid expensive decision calculations every rendered frame.
-   Keep randomness seedable so battles can be reproduced.

### 9.3 Perception

Initial perception data:

-   Health percentage
-   Nearest enemy distance
-   Nearest ally distance
-   Nearby enemy count
-   Nearby ally count
-   Estimated local friendly / enemy strength
-   Surrounded / threatened state
-   Target vulnerability
-   Cover / terrain advantage
-   Current order
-   Relevant army recommendation

Perception should be local and limited. Do not give every soldier
perfect knowledge of the entire battlefield.

### 9.4 Movement and pathfinding

Begin with simple movement and collision avoidance. Add formations and
robust pathfinding only when the basic arena works. Use spatial
partitioning (uniform grid or spatial hash) for nearby-unit queries once
needed. Avoid checking every soldier against every other soldier on
every tick.

### 9.5 Combat and captures

Start with a small, deterministic combat model:

-   Attack range
-   Attack cooldown
-   Damage
-   Health
-   Death
-   Basic target selection

Captures, morale, surrender conditions, and recruitment are later
milestones. A surrendering unit should enter an explicit state and
should not continue normal combat behavior.

### 9.6 Battle records

Record:

-   Winner and objective result
-   Friendly / enemy casualties
-   Survivors
-   Damage and assists
-   Captures
-   Time to objective
-   Formation / flank events
-   Individual performance
-   Tactical evidence
-   Simulation seed and scenario ID

------------------------------------------------------------------------

## 10. Architecture and code organization

Keep simulation logic independent from Unity rendering. Use plain C#
classes for data, AI, learning, evolution, and simulation wherever
possible. Use `MonoBehaviour` primarily for scene integration and visual
presentation.

Suggested folder structure:

``` text
Assets/
  _Project/
    Art/
      Characters/
      Environment/
      Effects/
      UI/
    Audio/
    Prefabs/
      Units/
      Environment/
      UI/
    Scenes/
      Bootstrap.unity
      EvolutionArena.unity
    ScriptableObjects/
      Factions/
      Equipment/
      Scenarios/
    Scripts/
      Core/
        GameBootstrap.cs
        SimulationClock.cs
        SeededRandom.cs
      Data/
        SoldierData.cs
        SoldierRuntime.cs
        ArmyData.cs
        BattleRecord.cs
        LineageData.cs
      Simulation/
        BattleSimulation.cs
        MovementSystem.cs
        CombatSystem.cs
        PerceptionSystem.cs
        SpatialGrid.cs
        CaptureSystem.cs
      AI/
        SoldierBrain.cs
        UtilityEvaluator.cs
        ActionScores.cs
        DecisionContext.cs
      Learning/
        IndividualLearningSystem.cs
        MemorySystem.cs
        RewardCalculator.cs
        TacticalKnowledge.cs
        ArmyBrain.cs
      Evolution/
        FitnessEvaluator.cs
        GeneticAlgorithm.cs
        MutationSystem.cs
        LineageManager.cs
      Commands/
        ArmyOrder.cs
        OrderController.cs
      Persistence/
        SaveService.cs
        SaveData.cs
      Presentation/
        SoldierView.cs
        BattleCamera.cs
        SelectionController.cs
        BattleHUD.cs
        DebugOverlay.cs
      UI/
        SoldierInspector/
        ArmyDashboard/
        BattleResults/
    Tests/
      EditMode/
      PlayMode/
```

This is a suggested structure, not a requirement to create every file
immediately. Create folders and classes only when a milestone needs
them.

### 10.1 Separation rules

-   `SoldierData` contains persistent data and should not reference
    Unity scene objects.
-   `SoldierRuntime` contains temporary state for the active battle.
-   `SoldierView` reads simulation state and renders it; it does not
    make AI decisions.
-   `BattleSimulation` owns simulation progression and deterministic
    tick order.
-   AI systems return decisions; they should not directly manipulate
    transforms.
-   Learning systems consume recorded events / outcomes; they should not
    secretly modify unrelated combat code.
-   Save / load must serialize stable data, not `GameObject` references.
-   Use ScriptableObjects for authored faction, equipment, and scenario
    definitions---not as mutable storage for an individual soldier's
    save data.

### 10.2 Determinism and reproducibility

Use a seeded random-number generator owned by the simulation. A given
seed, army setup, scenario, and simulation version should produce
reproducible results as far as practical. Avoid relying on Unity's
global random state for core simulation decisions.

Record the seed and version with each battle. This is essential for
debugging and comparing AI changes.

------------------------------------------------------------------------

## 11. UI, inspection, and replay

The interface should make the simulation understandable without
overwhelming the player.

### 11.1 Soldier inspector

Selecting a soldier should eventually show:

-   Name, ID, faction, squad
-   Health and equipment
-   Personality parameters
-   Current order and current action
-   Current action scores and decision explanation
-   Experience and battle count
-   Kills, assists, damage, survival record
-   Recent notable memories
-   Learned parameter changes
-   Lineage, generation, parents, descendants
-   Neural network information when that system exists

For debugging, show a concise explanation such as: "Retreat selected:
low health + outnumbered + retreat order." Do not expose raw neural
weights as the only explanation.

### 11.2 Army intelligence dashboard

Display:

-   Win rate across a defined sample
-   Average casualties
-   Objective success
-   Current tactical knowledge and confidence
-   Formation effectiveness
-   Faction-specific observations
-   Generation and lineage diversity
-   Most notable soldiers
-   Behavioral diversity / distribution

### 11.3 Battle replay

Later, add event-based replay with:

-   Pause / resume
-   Speed controls (1x, 2x, 4x, 10x)
-   Timeline markers for major events
-   Unit selection and inspection
-   Toggle for movement paths, targets, and tactical zones
-   Display of the order active at each event

Do not store every rendered frame. Store simulation seeds and / or
compact event logs, depending on the final replay architecture.

------------------------------------------------------------------------

## 12. Performance and scale

The initial target is 40 units. The long-term ambition is hundreds of
autonomous soldiers.

Potential optimizations, only when profiling justifies them:

-   Spatial hashing / uniform grid for neighbor queries
-   Fixed simulation ticks
-   Different AI update frequencies by distance / relevance
-   Cached perception and tactical recommendations
-   Object pooling for effects and transient objects
-   Batched simulation
-   Unity Job System / Burst for suitable data-oriented workloads

Do not prematurely optimize. First measure CPU time, allocations, and
query counts. The simulation should eventually run headlessly or without
graphical presentation for automated training and benchmark battles.

------------------------------------------------------------------------

## 13. Development roadmap

Each milestone must leave the project in a working state. Do not begin a
later milestone while the current one has unresolved compile errors or
broken acceptance tests.

### Milestone 0 --- Project audit and foundation

-   Inspect the existing Unity project before changing it.
-   Confirm Unity version, render pipeline, input setup, and existing
    assets.
-   Create a concise architecture / decision log.
-   Set up folders and assembly boundaries only where useful.
-   Establish a working bootstrap scene and version-control-friendly
    project structure.

**Acceptance criteria:** Project opens and compiles; a documented scene
starts without errors.

### Milestone 1 --- Arena foundation

-   Create a bounded top-down arena.
-   Spawn 20 friendly and 20 enemy units.
-   Implement health, movement, collision avoidance, target detection,
    basic attacks, damage, and death.
-   Use simple placeholder sprites if final art is not ready.
-   Add camera movement / zoom only if needed for testing.

**Acceptance criteria:** 40 units can spawn, move, engage, take damage,
die, and finish a battle without manual unit control. No null-reference
errors or stuck units in the standard test scenario.

### Milestone 2 --- Autonomous Utility AI

-   Implement local perception.
-   Implement action scoring for attack, retreat, defend, flank, follow,
    and hold.
-   Implement strategic orders.
-   Add action commitment / hysteresis.
-   Add a basic decision debug display.

**Acceptance criteria:** Units make sensible decisions in several
repeatable scenarios. A developer can inspect why a unit chose its
current action.

### Milestone 3 --- Telemetry and reproducible battles

-   Add seeded randomness.
-   Record battle outcomes and unit events.
-   Build a basic results screen / debug report.
-   Create benchmark scenarios and run batches of battles.
-   Record win rate, survival, objective performance, and behavioral
    metrics.

**Acceptance criteria:** The same setup and seed can be rerun for
comparison; results are saved and can be inspected. The project has a
non-learning AI baseline.

### Milestone 4 --- Persistent soldiers and individual development

-   Separate persistent soldier data from battle runtime.
-   Save and load soldier IDs, stats, personality, and history.
-   Add notable-event memories.
-   Implement small, interpretable individual parameter updates.
-   Show learned changes in the inspector.

**Acceptance criteria:** A soldier's identity and learned behavioral
parameters persist across battles. Learning changes are logged and
explainable.

### Milestone 5 --- Collective army intelligence

-   Add contextual tactical knowledge records.
-   Record successes and failures with context.
-   Share recommendations with soldiers.
-   Ensure recommendations influence rather than dictate decisions.
-   Compare results against the baseline.

**Acceptance criteria:** A tactic's confidence changes based on
evidence; soldiers can use recommendations in relevant situations;
benchmark results show whether the system helps.

### Milestone 6 --- Evolution and lineages

-   Add fitness evaluation over multiple scenarios / seeds.
-   Add parent selection, crossover if appropriate, and bounded
    mutation.
-   Preserve behavioral diversity.
-   Add lineage IDs, generations, and parent records.
-   Add an evolution summary.

**Acceptance criteria:** New recruits can inherit selected parameters;
mutations are recorded; multiple viable behavioral styles survive;
results are comparable across generations.

### Milestone 7 --- Neural network experimentation

-   Implement the small feed-forward network in plain C#.
-   Add serialization and deterministic tests.
-   Integrate outputs as bounded action-score modifiers.
-   Compare against Utility AI alone.
-   Add an explicit learning / evolution method and measure whether it
    improves outcomes.

**Acceptance criteria:** Network behavior is reproducible, inspectable,
saveable, and demonstrably useful in benchmark tests. If it does not
outperform the simpler baseline, keep it optional.

### Milestone 8 --- Recruitment, morale, and capture

-   Add morale and surrender states.
-   Implement capture conditions and safe capture behavior.
-   Preserve captured soldiers' original data / traits.
-   Integrate recruits into the player's army and collective knowledge.

**Acceptance criteria:** Eligible enemies can surrender, be captured,
and persist as recruits without losing identity or lineage information.

### Milestone 9 --- Training and experimental arena

-   Add repeatable training scenarios.
-   Allow the player to expose soldiers to selected opponent types /
    situations.
-   Record training evidence separately from battle history.
-   Compare trained and untrained groups.

**Acceptance criteria:** Training changes are measurable and do not
simply grant unexplained stat bonuses.

### Milestone 10 --- Enemy adaptation and campaign systems

-   Add faction doctrine data.
-   Track player tactic patterns.
-   Add gradual, contextual enemy counter-strategies.
-   Add campaign progression and additional battlefields.
-   Expose enemy observations to the player through reports.

**Acceptance criteria:** Enemy adaptation is understandable, gradual,
and can be countered by player experimentation.

### Milestone 11 --- Presentation and polish

-   Replace placeholders with a consistent pixel-art set.
-   Add animation, combat feedback, particles, audio, and UI.
-   Add battle replay and timeline markers.
-   Optimize only after profiling.

**Acceptance criteria:** A complete battle can be watched, understood,
paused, inspected, replayed, and reviewed.

------------------------------------------------------------------------

## 14. Technical risks and design safeguards

  -----------------------------------------------------------------------
  Risk                                Safeguard
  ----------------------------------- -----------------------------------
  Units oscillate between actions     Hysteresis, cooldowns, and action
                                      commitment

  Soldiers learn from lucky outcomes  Confidence, sample thresholds,
                                      contextual evidence

  Kill farming dominates evolution    Teamwork, survival, objective, and
                                      contribution metrics

  One strategy takes over every       Diversity preservation and multiple
  lineage                             benchmark scenarios

  Enemy adaptation feels unfair       Gradual changes, clear evidence,
                                      player counterplay

  Neural network is hard to debug     Keep Utility AI baseline and show
                                      action explanations

  Simulation becomes too slow         Profile, spatial partitioning,
                                      decoupled update rates

  Save data becomes fragile           Versioned data schema and migration
                                      strategy

  Art assets do not match             Establish a style guide and test
                                      asset scale / camera before buying
                                      many packs

  Scope grows too quickly             Milestone acceptance criteria;
                                      defer later systems
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 15. Instructions for Claude Code

Claude Code should work as an incremental engineering partner, not
attempt to build the entire game in one pass.

### 15.1 Required workflow

At the start of every task:

1.  Inspect the existing repository and Unity project. Do not assume the
    project is empty.
2.  Read this README and any existing architecture notes.
3.  Identify the current milestone and its acceptance criteria.
4.  Inspect relevant scripts, scenes, packages, and project settings
    before editing.
5.  Propose a short implementation plan and list files expected to
    change.
6.  Implement only the requested milestone slice.
7.  Compile / run available tests. If Unity Editor automation is
    unavailable, state that honestly and provide exact manual
    verification steps.
8.  Fix errors introduced by the change.
9.  Summarize changed files, behavior, tests run, known limitations, and
    the next smallest task.

### 15.2 Engineering rules

-   Do not rewrite working systems without a concrete reason.
-   Do not add third-party packages unless explicitly approved.
-   Do not invent APIs or assume a Unity package is installed.
-   Keep simulation logic independent of `MonoBehaviour` and rendering.
-   Prefer small, cohesive classes and clear interfaces.
-   Avoid giant manager classes and unnecessary abstraction.
-   Use serialized configuration / ScriptableObjects for authored
    content.
-   Keep save data versioned.
-   Make random behavior seedable.
-   Add EditMode tests for pure C# logic and PlayMode tests for Unity
    integration where practical.
-   Do not silently change the design. Explain meaningful tradeoffs and
    ask before major architectural changes.
-   Do not implement future milestones early just because they appear in
    this README.
-   Never claim that AI is learning or improving without a benchmark or
    test that supports the claim.
-   Keep the game runnable at the end of each task.

### 15.3 Suggested first prompt for Claude Code

``` text
You are helping develop EVOLUTION, a Unity 2D top-down pixel-art autonomous army
simulation. Read README.md completely before making changes.

For this task, do not attempt to implement the entire game.

First inspect the existing repository and Unity project:
- Identify the Unity version and render pipeline.
- Identify existing scenes, scripts, packages, and project conventions.
- Determine whether any systems already exist that should be preserved.
- Report compile errors or project setup problems you can verify.

Then propose a minimal implementation plan for Milestone 1: Arena Foundation.
The target is a 20-versus-20 battle with basic movement, target detection,
health, attacks, damage, and death. Use temporary visuals if necessary.

Keep simulation data and logic separate from MonoBehaviour presentation.
Do not add neural networks, genetic algorithms, recruitment, enemy evolution,
or advanced UI in this milestone.

Before editing, list the files you expect to create or modify. Then implement
the smallest complete vertical slice, test it as far as the available Unity
tooling permits, and report:
1. What changed
2. How to run the scene
3. Tests / compilation checks performed
4. Known limitations
5. The next smallest recommended task

Leave the project in a working, understandable state.
```

### 15.4 Milestone handoff prompt

Use this after completing each milestone:

``` text
Review the current EVOLUTION implementation against the corresponding
milestone in README.md.

Do not add new features yet. Verify the acceptance criteria, inspect for
bugs and architectural drift, run available tests, and identify missing
requirements. Fix only issues that belong to the current milestone.

Report each acceptance criterion as PASS, FAIL, or NOT VERIFIED. Explain
how each result was checked. Do not claim Unity compilation or PlayMode
testing succeeded unless it was actually run.
```

------------------------------------------------------------------------

## 16. First playable definition of done

The first playable prototype is complete when:

-   A 20v20 battle starts and finishes reliably.
-   Units move, detect enemies, attack, take damage, and die.
-   Units choose actions autonomously using Utility AI.
-   The player can issue at least attack, defend, hold, and retreat
    orders.
-   Individual decisions can be inspected.
-   Battle results are recorded.
-   Battles can be rerun with a known seed.
-   A non-learning baseline exists for comparison.
-   Soldier identity can persist between at least two battles.
-   The project opens and runs without known blocking errors.

Neural networks, advanced evolution, captures, enemy adaptation,
training grounds, hundreds of units, and polished final art are **not**
required for this first playable milestone.

------------------------------------------------------------------------

## 17. Long-term possibilities

Once the core loop is proven, consider:

-   Hundreds of autonomous soldiers
-   Squad-level and commander-level intelligence
-   More sophisticated neural architectures
-   Reinforcement-learning experiments
-   Procedural personalities and faction doctrines
-   Advanced formations and terrain-dependent tactics
-   Multiple enemy factions and adaptive commanders
-   Veteran mentorship and retirement
-   Family trees and lineage specialization
-   Player-designed training scenarios
-   Procedural battlefields
-   Campaign maps and persistent wars
-   Advanced replay, analytics, and simulation tools

These are possibilities, not commitments. Add them only when they
improve the central experience.

------------------------------------------------------------------------

## 18. Final design statement

EVOLUTION is about cultivating an army rather than manually controlling
one.

The player creates the conditions, chooses the objectives, and guides
the development of a persistent force. Individual soldiers gain
experience, the army shares discoveries, and future generations inherit
selected strengths. Enemies may eventually adapt in response.

The defining moment is when the player watches their soldiers execute a
coordinated, effective tactic that emerged from the systems they
built---not from a scripted cutscene or a command to every individual
unit.

**An army that learns to win.**
