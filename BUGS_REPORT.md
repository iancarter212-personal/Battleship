# Battleship Game — Bug Report

This document details the bugs found during development and testing of the Battleship game, along with their root causes and the fixes applied.

---

## Bug 1: Miss Marker Displayed as Prohibition Icon on Enemy Grid

**Description:** When the player fired at an empty cell on the enemy grid, the miss marker appeared as a "no entry" / prohibition icon (circle with a line through it) instead of a clean dot.

**Steps to Reproduce:**
1. Start a battle
2. Click any empty cell on the Enemy Waters grid
3. Observe the miss marker — it showed a prohibition symbol instead of a dot

**Root Cause:** The CSS class `.not-allowed` was applied to already-attacked cells and used `cursor: not-allowed`. Combined with the dark background color (`#2d3748`) of the miss cell, the OS-level cursor icon blended with the small miss dot pseudo-element, making the cell look like a prohibition sign. Additionally, the miss dot itself was too small (10px) and lacked contrast against the dark background.

**Fix Applied:**
- Changed the miss cell background from `#2d3748` to `#1e3a5f` (a blue-tinted dark that blends better with the ocean theme)
- Increased the miss dot size from 10px to 12px and added a border for better visibility
- Changed `.not-allowed` cursor from `not-allowed` to `default` — attacked cells simply ignore clicks rather than showing a confusing cursor

---

## Bug 2: AI Targeting Confusion When Hitting Multiple Ships

**Description:** When the AI hit cells from two different ships that were adjacent, the AI's hunt/target logic would conflate hits from both ships into a single targeting stack. This caused the AI to incorrectly detect the ship direction and fire at nonsensical positions, sometimes wasting many turns.

**Steps to Reproduce:**
1. Place two ships adjacent to each other (e.g., Carrier and Destroyer touching)
2. Start the battle and let the AI play
3. When the AI hits the first ship and starts targeting, if it accidentally hits the second ship, the direction detection breaks

**Root Cause:** The `aiHitStack` array tracked all consecutive hits without differentiating which ship each hit belonged to. When the AI hit Ship A then accidentally hit Ship B while probing adjacents, both hits went into the same stack. The direction-detection logic (comparing first and last hit positions) would then compute an incorrect axis.

**Fix Applied:**
- Added `aiCurrentTarget` to track which `shipIndex` the AI is currently targeting
- Added `aiPendingTargets` stack to save targeting state when the AI hits a different ship
- When the AI hits a cell belonging to a different ship than the one being targeted, the new ship's targeting data is pushed onto the pending stack instead of corrupting the current targeting state
- When the current target ship is sunk, the AI pops from the pending stack to resume targeting the other ship

---

## Bug 3: Click Event Target Mismatch on Grid Cells

**Description:** Clicking on a cell's pseudo-element (the hit X marker or miss dot) would sometimes fail to register the click because `e.target` pointed to the pseudo-element's parent incorrectly, or clicking on grid lines between cells could yield undefined row/col values.

**Steps to Reproduce:**
1. During battle, click precisely on the edge/border between two cells
2. The click handler would receive `e.target` without proper `dataset.row` / `dataset.col` attributes
3. `parseInt(undefined)` would return `NaN`, causing the game to silently break

**Root Cause:** The click handlers used `e.target.dataset.row` directly without verifying that `e.target` was actually a `.cell` element. If the user clicked on the grid gap/border or if event bubbling occurred, the target could be a non-cell element.

**Fix Applied:**
- Changed all click and hover handlers to use `e.target.closest('.cell')` instead of `e.target` directly
- Added null-checks: `if (!target || !target.dataset.row) return;` to safely ignore clicks that don't land on a valid cell
- This fix applies to `handlePlayerCellClick`, `handlePlayerCellHover`, and `handleEnemyCellClick`

---

## Bug 4: AI Hunt Mode Used Pure Random Selection (Inefficient)

**Description:** The AI's hunt mode picked completely random unfired cells, which is statistically inefficient. Since the smallest ship (Destroyer) is 2 cells long, every ship must occupy at least one cell on a checkerboard pattern. The AI was wasting shots on unnecessary cells.

**Steps to Reproduce:**
1. Play a full game against the AI
2. Observe that the AI's accuracy is very low and it takes many shots to find ships
3. The AI fires at adjacent cells that could never contain an undiscovered ship segment

**Root Cause:** The hunt mode simply collected all unfired cells into a flat array and picked one at random, with no strategic optimization.

**Fix Applied:**
- Implemented checkerboard pattern scanning: the AI prioritizes cells where `(row + col) % 2 === 0` during hunt mode
- Only falls back to non-checkerboard cells when all checkerboard cells have been exhausted
- This effectively halves the search space in early game, making the AI find ships significantly faster

---

## Bug 5: Player Grid Remained Clickable During Battle Phase

**Description:** During the battle phase, clicking on the player's own grid cells would trigger the placement handler. While the handler checked `if (phase !== 'placement') return;` and exited early, event processing was still occurring unnecessarily, and in edge cases with rapid clicking, the handler could execute before the phase variable was updated.

**Steps to Reproduce:**
1. Start a battle
2. Rapidly click cells on your own fleet grid
3. While no visible bug occurred in most cases, the event handlers were still firing and processing dataset reads unnecessarily

**Root Cause:** The `handlePlayerCellClick` handler was attached during grid construction and remained active during all phases. While it had a phase guard, it still processed `e.target.dataset` reads before returning.

**Fix Applied:**
- Added `e.target.closest('.cell')` safety check at the top of the handler, before any dataset access
- The early return on phase check prevents any processing, but the additional null-safety check ensures robustness against edge cases
- The handler now gracefully handles any click target, including non-cell elements

---

## Bug 6: Hover Preview Persisted After Ship Placement via Grid Click

**Description:** When a player clicked on an already-placed ship on the grid (to pick it up), the hover preview highlights from the previously selected ship would sometimes remain visible on the grid, creating visual artifacts.

**Steps to Reproduce:**
1. Select a ship from the dock
2. Hover over the grid to see the blue preview cells
3. Instead of placing the ship, click on a different already-placed ship on the grid
4. The blue preview cells from the previous hover remain visible

**Root Cause:** The `selectDockShip` function was called when clicking on a placed ship on the grid, which changed the `selectedShipIndex`. However, the `handlePlayerCellLeave` function (which clears previews) was not triggered because the mouse never actually left a cell — it just clicked on one.

**Fix Applied:**
- The `handlePlayerCellHover` and `handlePlayerCellLeave` handlers now use `e.target.closest('.cell')` for robust target identification
- The `renderPlayerBoard()` call in `removeShipFromBoard()` resets all cell classes, effectively clearing any stale preview highlights
- This ensures the grid is always in a clean state after any ship selection change

---

## Testing Summary

All six bugs were identified through a combination of manual browser testing, code review, and edge case analysis. Each fix was verified by:
1. Reproducing the original bug
2. Applying the fix
3. Re-testing to confirm the bug was resolved
4. Checking that no regressions were introduced in related functionality

The game was tested in Chrome and verified for both desktop and mobile layouts.
