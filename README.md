# Battleship: Devin vs the Competiton

Live link: https://battleship-game-fvnrvjkm.devinapps.com/

Tech Stack: Built with vanilla HTML, CSS, and JavaScript — no frameworks or dependencies. Spec sheet is attached separately here.

Description:
A fully functional, browser-based Battleship game where you compete against a smart AI opponent. Built as a single HTML file with no external dependencies — just open it in any modern browser and play. The game features ship placement with drag-and-rotate controls, a hunt/target AI strategy with checkerboard scanning, hit/miss/sunk animations, mobile-responsive layout, and full game statistics tracking.



Minor UI updates made after the first iteration: 
Title → "BATTLESHIP: Cognition vs the Competition"
Subtitle → "Sink the coding agent before they can beat Devin."
Grid labels → "DEVIN FLEET" / "ENEMY CODING AGENT FLEET"
Stats → "DEVIN'S STATS" / "COMPETITION'S STATS"
Button → "Start Game"
All status/toast messages updated (AI → "the competition", player → "Devin")
Win screen → "Devin Won!", Lose screen → "Game Over, Devin Didn't Win This Time"
AI delay increased to 1.5s
Font set to IBM Plex Sans (with system-ui fallback since no CDN allowed)

## How the AI Works

The AI uses a **hunt/target strategy** with checkerboard optimization:

- **Hunt Mode**: The AI fires at random cells on a checkerboard pattern (like a chess board), which guarantees it can find every ship since the smallest ship is 2 cells long. This is more efficient than pure random firing.
- **Target Mode**: When the AI scores a hit, it switches to target mode and tries all four adjacent cells (up, down, left, right). Once it finds the direction of the ship, it continues along that axis until the ship is sunk.
- **Multi-ship tracking**: If the AI accidentally hits a different ship while targeting another, it stacks the new target and returns to it after sinking the current ship.
- Once a ship is fully sunk, the AI returns to hunt mode to find the next ship.

## Run Locally

1. Clone or download this repository
2. Open `index.html` in any modern web browser (Chrome, Firefox, Safari, or Edge)
3. No server, build step, or dependencies required — the game runs entirely in the browser

## Tech Stack

- Single HTML file (HTML5, CSS3, Vanilla JavaScript)
- No frameworks, no external dependencies, no build tools
- Works 100% offline once loaded
- Responsive design for desktop (1024px+) and mobile (375px+)
