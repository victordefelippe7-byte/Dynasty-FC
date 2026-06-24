# Dynasty FC - Initial Game Design Document

## 1. Core Concept and Genre
* **Genre**: Arcade Sports Simulation / Sports Management Hybrid
* **Core Concept**: A dynasty-building soccer game where the player controls a single field star and manages a franchise across three divisions. The game balances top-down arcade gameplay with front-office management.
* **Player Objectives**: 
  * Win matches to earn Money, Development Points (Dev Pts), Reputation, and Legacy.
  * Compete in a 10-team league to earn promotion from League Two to League One, and eventually the Championship.
  * Manage a two-player roster (one Field Player, one Goalkeeper) by handling transfers, contracts, and attribute training throughout their careers.

*(References: `dynasty-fc/index.html` - UI Header subtitle and Init state).*

## 2. Implemented Gameplay Systems and Mechanics
* **Match Engine**: 2D top-down physics-based match simulation running on an HTML Canvas. The player actively controls their field player against an AI field player, while goalkeepers on both sides are AI-controlled.
* **Season Phases**: The game progresses through fixed states: `Regular Season`, `Playoffs`, `Transfer Window`, `Contracts`, and `Offseason`.
* **League Simulation**: Three tiers exist (`0`: Championship, `1`: League One, `2`: League Two). Matches are simulated between AI teams to calculate standings. The top 2 teams earn automatic promotion, 3rd/4th place play a promotion playoff, and the bottom 3 are relegated.
* **Player Aging and Retirement**: During the Offseason, players age by 1 year. Attributes improve until age 24 and degrade after age 32. Players retire at 36 (or at 33 if their overall rating is too low) and can enter the Hall of Fame if they meet statistical milestones.
* **Transfers**: During the Transfer Window, a randomized market of 6 players is available to sign. Players can also be sold for Money, requiring a replacement before the next match.
* **Training / Development**: Dev Points earned from matches can be spent to incrementally increase specific attributes (+1) or improve Role level.
* **Contracts**: Players are signed for multiple years. The user must spend money to renew them (+1 or +3 years), otherwise they leave in free agency and are replaced by minimum-wage free agents.
* **Training / Local Multiplayer Mode**: A standalone, 2-player local exhibition mode exists below the main game panel, allowing practice without affecting franchise stats.

*(References: `dynasty-fc/index.html` - `doAIPromoRelegation()`, `agePlayers()`, `simLeagueWeek()`, `updateBall()`)*

## 3. Player Controls, Input, and Abilities

### Controls Table
| Mode | Action | Keybinding | Context |
| :--- | :--- | :--- | :--- |
| **Main Match** | Move | `W/A/S/D` or `Arrow Keys` | Active during match |
| **Main Match** | Shoot | `Spacebar` | Offense (when possessing ball) |
| **Main Match** | Slide Tackle | `Spacebar` | Defense (when ball is loose/owned by AI) |
| **Main Match** | Pause | `Escape` | Toggles pause overlay |
| **Training** | Move (Left Player) | `W/A/S/D` | Exhibition local mode |
| **Training** | Shoot (Left Player)| `F` | Exhibition local mode |
| **Training** | Move (Right Player)| `Arrow Keys` | Exhibition local mode |
| **Training** | Shoot (Right Player)| `L` | Exhibition local mode |

### Player Attributes (Scaled 10-99)
| Attribute | Abbreviation | Effect |
| :--- | :--- | :--- |
| **Speed** | SPD | Determines movement speed on the pitch. |
| **Shot** | SHT | Determines shot power and accuracy. |
| **Dribble** | DRB | Determines ball retention when being tackled. |
| **Defense** | DEF | Determines slide tackle success rate. |
| **Keeper** | SAV | Goalkeeper-specific stat for determining save success probability. |

*(References: `dynasty-fc/index.html` - Input event listeners, `shootOrTackle()`, `moveUser()`, `updateTraining()`)*

## 4. Game Entities and Objects

### Game Objects
* **The Ball**: Physics object with position, velocity, and spin. Responds to collisions with pitch boundaries, goals, and keeper saves. Can be "owned" (attached to a player) or "released".
* **Field Players (User & AI)**: Visualized as articulated 2D figures with walk cycles and slide animations. 
* **Goalkeepers (User & AI)**: Visualized similarly to field players but with gloves. They have unique diving animations and movement constraints (staying near the goal).
* **Teams**: Contain an ID, name, tier, win/loss/draw records, goals for/against, money, reputation, and exactly two players (Field and Keeper).

### Player Roles (Behavior Modifiers)
| Role | Name | Description | Base Position |
| :--- | :--- | :--- | :--- |
| `SK` | Sweeper Keeper | Aggressive positioning, moves further from the goal line. | Goalkeeper |
| `BPK` | Ball-Playing Keeper| Holds the ball after a save and distributes it accurately to their field player. | Goalkeeper |
| `F9` | False 9 | Drops deep to defend and link play. Shoots with slightly less power. | Field Player |
| `PC` | Poacher | Aggressive attacking positioning. High shot power and likelihood to shoot. | Field Player |
| `STD` | Standard | Baseline behavior without special logic modifiers. | Both |

*(References: `dynasty-fc/index.html` - `makePlayer()`, `drawDetailedPlayer()`, `drawGoalkeeper()`)*

## 5. Progression, Scoring, and Game States
* **Match Scoring**: Standard soccer rules. Ball entering the goal increments the score. Matches run for a simulated 90 minutes.
* **Match Result**: Win (3 points), Draw (1 point), Loss (0 points). In playoff matches, draws are resolved by a hidden team-power tiebreaker.
* **Franchise Progression**: 
  * Wins grant Money, Dev Pts, Reputation, and Legacy.
  * Losses break win streaks and provide minimal compensation.
  * Playoff victories grant higher rewards and result in tier promotion.
* **Game States**: The UI dynamically updates based on the active `phase` string (`Regular Season`, `Playoffs`, `Transfer Window`, `Contracts`, `Offseason`) and `match.active` boolean.
  * Matches can be paused via the `pauseOverlay` UI element.
  * Halftime stats are displayed via a modal when the timer hits 45.

*(References: `dynasty-fc/index.html` - `finishMatch()`, `showHalftime()`, `determinePlayoffs()`)*

## 6. Levels, Scenes, and World Structure
* **Single Page Application**: The entire game is rendered within a single DOM structure.
* **The Pitch**: A 2D top-down `<canvas>` element (960x600 resolution). Features a grass pattern, pitch lines, penalty areas, goals, and a crowd silhouette at the top.
* **Attack Zones**: Visual colored tints on the left and right sides of the pitch indicating team zones.
* **Fans**: Animated crowd entities rendered behind the goals, colored based on the competing teams' primary and secondary colors.

*(References: `dynasty-fc/index.html` - `drawPitch()`, `drawFans()`)*

## 7. UI Systems, Menus, and HUD
* **Layout**: A responsive CSS Grid layout (`.shell`) divided into three columns on desktop:
  * **Left Aside**: Franchise stats (Money, Dev Pts), Roster cards, and Action Buttons (Play Match, Develop, Transfers).
  * **Center Main**: The active Match Canvas, match banner/scoreboard, Next Match info, and Training canvas.
  * **Right Aside**: League Table, Dynasty Log (event feed), and Records (top scorers/keepers).
* **HUD**: Floating overlays on the canvas during matches (Home/Away Score, Timer).
* **Modals & Toasts**: Popups used for match stats, the transfer market, training upgrades, and temporary system notifications (`toast()`).
* **Customization**: A modal allows changing the club's name, 3-letter badge, and kit colors (Primary, Secondary, Keeper).

*(References: `dynasty-fc/index.html` - CSS styles `.shell`, `openModal()`, `toast()`, `openCustomize()`)*

## 8. Technical Architecture
* **Vanilla Stack**: The game runs entirely on client-side JavaScript, HTML5, and CSS variables within a single `index.html` file.
* **Global State**: All game data is stored in a single mutable `S` object (aliased to `window.DS`).
* **Game Loop**: A custom `requestAnimationFrame` loop (`loop(ts)`) manages physics (`updateMatch`), AI movement (`moveAi`), and rendering (`drawMatch`).
* **Canvas Rendering**: Immediate mode 2D rendering using `ctx`. Characters are drawn programmatically using arcs, paths, and gradients without external image sprites.
* *(Note: Despite the presence of Vite, React, and Tailwind dependencies in `package.json`, they are completely unused in the current `index.html` implementation.)*

*(References: `dynasty-fc/index.html` - `const S = {}`, `loop()`, `dynasty-fc/package.json`)*

## 9. Incomplete, Placeholder, or Experimental Features
* **Role Implementations**: `F9` and `PC` have distinct logic in `moveAi` and `shootOrTackle`, but `STD` has no unique behavior modifier. Role Pips (proficiency) simply grant raw stat boosts.
* **Playoff Tiebreakers**: Currently handled by a random coin-flip weighted by Team Power, rather than extra time or penalties.
* **Match Engine Stuns**: The `slideTimer` and `fallTimer` act as simple stuns, locking player movement.
* **Training Mode Disconnect**: The 2-player local exhibition mode (`trainingPitch`) maintains completely separate state (`TG`) and a separate loop (`trainingLoop`), disconnected from the franchise mode.
* **Package Architecture**: `package.json` outlines a React/Vite/Tailwind setup that is currently ignored by the Vanilla JS implementation.

*(References: `dynasty-fc/index.html` - `handlePlayoffResult()`, `TG` state, `dynasty-fc/package.json`)*

## 10. Open Questions and Unknowns
* **Technology Stack**: Is the intention to eventually migrate this Vanilla JS canvas game into the React/Vite architecture defined in `package.json` and the workspace `lib` folders?
* **Monetization / Long-term loop**: The player earns Money and Dev Pts constantly. Is there an intended economy sink for late-game besides increasingly expensive transfers?
* **Matches per Season**: Teams play every other team in their tier exactly once (9 matches). Are there plans for multi-round robins or cup tournaments?
* **Goalkeeper Controls**: The user currently has no direct control over their goalkeeper. Is this a permanent design choice or a placeholder for future AI/Control improvements?

## 11. Recommended Next Documentation Pass
* **State Management Audit**: Document the entire schema of the global `S` object to assist with potential migration to React/Redux or Zod validation (given `@workspace/api-zod` is present in `lib`).
* **AI Behavior Deep Dive**: Map out a state machine diagram for the AI field player and Goalkeepers to better understand how `walkPhase`, `diveTimer`, and ball `owner` interact.
* **Stat Formula Reference**: Extract and formalize all mathematical formulas used for shooting power, save probability, and tackle success into a centralized design spreadsheet.
