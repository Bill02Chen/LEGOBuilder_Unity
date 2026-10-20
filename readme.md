# LEGO Builder with hand tracking

[Portfolio](https://bill02chen.github.io/chenyanrui-site/) · [GitHub](https://github.com/Bill02Chen)

A Unity brick-building prototype. The player points at the world, opens a radial
menu, chooses a brick and colour, and places it on a snapped grid. The original goal was Ultraleap hand tracking. This repository contains the Unity
game and reads pointer/axis input; the Ultraleap runtime and SDK are not vendored
here.

**Key result:** a playable prototype with a first-person build mode, a radial
brick/colour menu, grid snapping, collision-aware placement and a transparent
preview of the next brick.

## Problem

Placing bricks in 3D with a mouse is precise but slow, and direct hand tracking
introduces two problems:

1. **Selection** — many brick shapes and colours must be reachable without a
   keyboard.
2. **Placement** — a free-hand position is noisy; bricks must land on a grid and
   must not intersect existing bricks.

The prototype explores both: a radial menu for selection and a snap-and-check
step for placement.

## Approach

```text
hand / pointer
      │
      ▼
Controller ── Tab ──► Build mode
      │                  │
      │                  ├─ Q → Ring menu → brick + colour
      │                  │
      ▼                  ▼
Player / camera     PlaceBrick: raycast → SnapToGrid → OverlapBox check
      │                  │
      ▼                  ▼
move + jump         preview (transparent) → left click → place
```

### Modes and input

`Controller.cs` implements three modes:

- **Play** — first-person movement with `Player.cs`;
- **Build** — `Tab` toggles; a transparent brick follows the pointer;
- **Menu** — `Q` opens the radial menu in build mode.

The input layer reads Unity's pointer and axis APIs. The repository does **not**
vendor the Ultraleap runtime; the original setup used hand tracking as the input
source, and a fresh clone needs that runtime connected to the pointer input.

### Radial menu

`Assets/Scripts/UI/RingMenu.cs` builds a ring from `Ring` data assets:

- each element is a pie slice with an icon;
- the slice under the pointer is highlighted;
- a slice can open a sub-ring (brick shape → colour);
- the final selection calls back with a path such as `1/2`, and
  `Controller.MenuClick` passes the brick and colour indices to `PlaceBrick`.

### Placement

`PlaceBrick.cs` runs on every frame in build mode:

1. raycast from the camera;
2. snap the hit point to the LEGO grid (`LegoLogic.Grid = 0.2 × 0.1 × 0.2`);
3. test an `OverlapBox` at the candidate position against the Brick/Lego layers;
4. if the position is blocked, lift the brick by one grid step and retry (up to
   10 times);
5. show the brick with a transparent material; on left click, enable its collider,
   apply the selected material and place it.

`LegoLogic.cs` holds the grid and layer masks; `Player.cs` handles movement,
jumping and gravity.

## Key result

- A playable build loop: select a brick from the radial menu, preview it, and
  place it on the grid.
- Collision-aware placement prevents a brick from being placed inside an existing
  one; the preview turns into a solid brick only on a valid position.
- The scene includes a first-person player, a build/play toggle and a working
  ring menu with sub-menus.

## Evidence

- [Controller.cs](Assets/Scripts/Controller.cs) — modes, camera and menu entry.
- [PlaceBrick.cs](Assets/Scripts/PlaceBrick.cs) — raycast, snap, collision check,
  preview and placement.
- [LegoLogic.cs](Assets/Scripts/LegoLogic.cs) — grid and layer masks.
- [Player.cs](Assets/Scripts/Player.cs) — movement, jump and gravity.
- [RingMenu.cs](Assets/Scripts/UI/RingMenu.cs) — radial menu and sub-rings.
- [SampleScene.unity](Assets/Scenes/SampleScene.unity) — the playable scene.
- [Assets/Object/Bricks](Assets/Object/Bricks) — brick prefabs and meshes.
- [Assets/UI](Assets/UI) — ring and icon sprites.

No recorded demo or screenshots are committed; the scene and scripts are the
evidence.

## Run

1. Open the project in Unity (the project has no `ProjectVersion.txt`, so use a
   Unity version compatible with the package set).
2. Open `Assets/Scenes/SampleScene.unity`.
3. Press Play.
4. Controls:
   - mouse / pointer: aim and place;
   - `Tab`: toggle play/build;
   - `Q`: open the ring menu in build mode;
   - left click: select a ring item or place a brick;
   - mouse wheel: zoom; mouse move: rotate the camera;
   - `WASD` + space: move/jump in play mode.

To use hands, install the Ultraleap Unity plugin and map hand position/clicks to
the pointer input the scripts already read. The plugin is not included.

## Limitations

- The Ultraleap SDK/runtime is not vendored, so hand tracking does not work from
  a fresh clone without the plugin.
- The ring menu is driven by pointer angle; it is usable with a mouse and was
  designed for a hand-driven cursor.
- No automated tests; the project is a prototype, not a shipped game.

## Original note

The early README said: "This is a lego game project combine with ultraleap motion
to make users place bricks using hand motion control." That is the project's
goal; the implementation detail is in the scripts above.
