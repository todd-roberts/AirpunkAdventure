# Airpunk Adventure Remixable World

Welcome to the guide for the Airpunk Adventure Remixable World! This document provides an overview of the project, its features, and instructions on how to remix and customize your own version of the world.

## Overview

Airpunk Adventure is a sky-themed world featuring two opposing airships. Players can travel from one airship to the other by using a balloon that’s deployed with the **B** button (**F** on desktop). Players also receive a physics-based, auto-reloading crossbow in their right hand upon joining the world. They can shoot other players with arrows to deplete their health—or simply pop their balloon to send them plummeting to their doom. Players who run out of health or collide with the respawn trigger near the cloudline are respawned on their team’s ship.

The world ships with a demo **Capture the Flag** mode. More on that in [Capture the Flag](#capture-the-flag).

## Hierarchy

The hierarchy is a small, well-organized set of objects. The root-level objects are:

- **Environment**  
  An environment gizmo with children that give the world its sky theme. It contains moving clouds, music, and wind sound effects.

- **PlayerRespawnTrigger**  
  A massive rectangular trigger beneath the ships that respawns players to their respective ship if they fall out of the sky.

- **PlayerGearAssetPool**  
  An asset pool gizmo that contains the player-gear objects assigned to players when they join the game. More on this in [PlayerGear](#playergear).

- **CaptureTheFlag**  
  An object that contains the Capture the Flag game’s main manager and UI. See [Capture the Flag](#capture-the-flag).

- **Airships**  
  Holds both example airships. This object is not merely organizational—it contains a script that manages air-traffic control between the ships. See [Airship Movement](#airship-movement).

- **Cannonballs**  
  This is just a pool of example cannonballs for the cannons. This needs to be outside of Airships because the balls are physics-enabled, so they can't be descendents of animated entities.

## Codebase

Similar to the hierarchy, the code “framework” is intended to leave a lightweight footprint in your scripts directory. It includes the following files:

- **`AirpunkAdventure_ClientComponents.ts`**  
  All client-side components used in the world: crossbow, balloon, health UI, etc.

- **`AirpunkAdventure_ServerComponents.ts`**  
  All server-side components used in the world: airship movement, texture swapping, Capture the Flag, etc.

- **`AirpunkAdventure_Config.ts`**  
  Configuration variables used across the world. See [Configuration](#configuration).

- **`AirpunkAdventure_Events.ts`**  
  All events used by the world.

The scripts are prefixed intentionally so they sort together in IDEs and don’t obscure your custom scripts. Another nice side effect is that you can easily copy/paste the entire codebase into something like ChatGPT.

> If you prefer to refactor these components into separate files, remember to designate any client-side scripts with **Local** execution mode.

## Configuration

Configuration is done in two ways:

1. **Global feel** is set in **`AirpunkAdventure_Config.ts`** (movement speeds, gravity, health, etc.).  
2. **Singleton entities** (one instance in the world) expose **props** you can edit directly in the editor—for example, the `CaptureTheFlagManager` and `CaptureTheFlagUI`. See [Capture the Flag](#capture-the-flag) and [Texture Swapping](#texture-swapping).

## PlayerGear

The player devices described in the [Overview](#overview) are provided by objects with the `PlayerGear` **client-side** script attached. These objects live inside the **`PlayerGearAssetPool`**. When a player joins, everything nested under `PlayerGear` is automatically assigned to that player. Items like the crossbow are force-held once delivery to the client is confirmed.

### Ideas for Improvement / Extension
- Add a PPV for player preferences (e.g., handedness). Deliver that via event to `PlayerGear` to decide which hand to force-hold the crossbow in.
- Add an explicit **recovery** mechanism if the asset pool falters—let players claim an owner-less `PlayerGear` or spawn a new one.
- Consider a recurring **heartbeat** event from `PlayerGear` to the asset pool—if a heartbeat isn’t received, assume missing gear and reassign.

## Capture the Flag

Capture the Flag was an obvious choice for a demo mode. It gives players a reason to visit each other’s ships and interact, and it resets automatically for continuous play.

You can find gameplay options—such as **scores required for a win**—in the **`CaptureTheFlag`** object’s children (e.g., `CaptureTheFlagManager` and `CaptureTheFlagUI`). See [Configuration](#configuration) for global tuning.

### Ideas for Other Games

- **Loot Run:** Players raid a doubloon-filled chest on each ship and bring loot back home.  
- **Airship Battles:** Make the cannonballs functional and try to sink the other team’s ship.  
- **Exploration:** Add floating islands or sky temples with treasures to discover.

## Airships

### Airship Movement

Airships move **on rails**: players steer ships left/right across discrete “lanes” using the helm wheel. Under the hood:

- **Traffic controller:** `AirshipTrafficController` computes a per-ship **origin X** and the **left/right hop sizes** from the spacing between ships.  
  - `maxHops` caps how far from origin a ship can slide.  
  - `minSeparation` guarantees a safety buffer between ships at all times.
- **Wheel input:** The `Wheel` component handles steering. When a player triggers the left/right arrow, the ship slides toward the next allowed hop, plays a short yaw animation while moving, and then returns upright.
- **Movement feel:** Speed and visual yaw are tuned in `AirpunkAdventure_Config.ts` (`airship.movementSpeed`, `airship.rotationAmount`). Blinking arrow hints help telegraph direction changes.

**Why rails?** Three reasons:

1. **No overlap / no crashes**  
   Rails + `minSeparation` make layout constraints explicit and safe.
2. **Easier, more immersive navigation**  
   Free 3D flight is cognitively heavy. Rails let players focus on timing and tactics instead of piloting a helicopter.
3. **Prebaked navmeshes (the subtle win)**  
   Predictable rest positions mean you can prebake navmeshes and do believable **on-deck NPC** movement when ships aren’t sliding. That’s almost impossible with free-flight platforms.

#### What to tweak first

- `AirshipTrafficController.maxHops` — widen or reduce lateral range.  
- `AirshipTrafficController.minSeparation` — increase to make collisions visually impossible, decrease for tenser near-misses.  
- `Config.airship.movementSpeed` — slow for stately cruisers, faster for arcadey feel.  
- `Config.airship.rotationAmount` — increase for more dramatic banking while moving.

### Texture Swapping

Team textures are **the** primary theming lever, and they propagate automatically:

- **Set once** on `CaptureTheFlagManager`: `team1Texture` and `team2Texture`.  
- Those textures then apply to:
  - **Airship meshes** (each team’s ship)
  - **Team flags**
  - **Each player’s balloon**

  - **Each player's top hat**
  - **Ship props/furniture** (e.g., chairs)

#### How to re-skin fast

1. In the repo root, locate the base texture template: `./FantasyShip_BR.png`.  
2. Duplicate it to create variants (for example):  
   - `./FantasyShip_BR_Team1.png` (e.g., Purple)  
   - `./FantasyShip_BR_Team2.png` (e.g., Green)
3. Edit the duplicates in your image editor of choice (preserve alpha and dimensions).  
4. In-world, set:
   - `CaptureTheFlagManager.team1Texture` → `FantasyShip_BR_Team1.png`  
   - `CaptureTheFlagManager.team2Texture` → `FantasyShip_BR_Team2.png`

That single change re-skins ships, flags, player balloons, and any themed props you add.

## Questions?
Feel free to open an issue against this repo or reach out to `scarcesoft` within Horizon Worlds.