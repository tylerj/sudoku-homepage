# Sudoku Launchpad — Homepage

Product landing page for [Sudoku Launchpad](https://sudokulaunchpad.com), a companion app for sudoku enthusiasts who solve on SudokuPad and follow creators like Cracking The Cryptic.

## What is Sudoku Launchpad?

Sudoku Launchpad syncs your puzzle progress from SudokuPad, lets you replay your solves move-by-move, and watch them synchronized alongside the original YouTube videos. It also aggregates daily puzzles from the NYT and featured puzzles from CTC into one place.

## Tech Stack

Static single-page site — no build step, no dependencies.

- **HTML** with embedded CSS and JavaScript
- **Light/dark mode** via `data-theme` attribute with system preference detection
- **Theme-aware screenshots** — light and dark image variants swap automatically on theme toggle

## Local Development

Serve the directory with any static file server:

```sh
python3 -m http.server 8080
```

Then open `http://localhost:8080`.

## Screenshot Placeholders

The `images/` directory contains SVG placeholders for app screenshots. Each has a light and dark variant:

| Section | Light | Dark |
|---|---|---|
| Homepage hero | `homepage-light.svg` | `homepage-dark.svg` |
| Video Replay | `replay-video-light.svg` | `replay-video-dark.svg` |
| Calendar | `calendar-light.svg` | `calendar-dark.svg` |
| Statistics | `stats-light.svg` | `stats-dark.svg` |

Replace these with actual screenshots (`.png` or `.webp`) and update the `src` attributes in `index.html`.

## Project Structure

```
├── index.html          # Landing page (HTML, CSS, JS all-in-one)
├── 404.html            # Custom error page
├── favicon.svg         # SVG favicon
├── robots.txt          # Robots config
└── images/             # Screenshot placeholders (light + dark)
```
