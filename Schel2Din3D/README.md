# Schelling 2D in 3D

Open `schel2Din3D-js.html` in a modern browser with WebGL enabled. Keep the bundled `chroma.min.js` beside the HTML file. No installation, server, or network connection is required. `example.html` is the reference for the control styling.

The app simulates two populations on a square grid and displays their evolution as a 3D point scene. The initial grid lies at depth zero; the current grid moves along the depth axis as swaps accumulate. Colored points between them mark the new occupants of swapped cells, showing the transition history. On page load and reset, the two display colors are randomized using the same Chroma.js logic as `example.html`: random colors brightened by 2 and saturated by 3, with a minimum contrast ratio of 4.5. Red and blue below refer to population identities, regardless of their display colors.

## Controls

- **start / pause / resume** starts or toggles the simulation without moving the buttons. The button continues to toggle pause/resume after completion; the finished grid stays unchanged. Reset or change parameters to run a new simulation.
- **reset** stops playback, restores the exact original population for the current run, and clears swap history and GIF recording. It chooses a new random color pair and preserves parameters and camera settings.
- **tune** opens sliders styled after the example and pauses playback. Size (`numbhum`) is the number of cells along each side, so size 200 means 40,000 agents. Radius (`w`) controls the square neighborhood. Tolerance (`tau`) is the minimum fraction of the same population required for happiness. Releasing a slider applies its value automatically, creates a fresh random population, and clears history. Click outside the panel or press Escape to close it. The run remains paused.
- **svg** saves the current 3D canvas view as vector points, including both grids and swap history, using the current camera and colors. Controls are excluded.
- **gif** becomes available immediately after start. It saves a looping animation from the initial view through the current displayed stage. Recording continues across pause/resume and is cleared by reset or changing tune settings.

Drag the canvas to rotate. Arrow keys translate the view; `p` / `P` decrease/increase zoom. `j` / `J` change grid extent, `k` / `K` change depth spacing, and `r` randomizes the two colors. Resize the window to resize the canvas.

## Transition rules

1. Each cell is independently initialized red or blue with equal probability. There are no empty cells.
2. A cell's neighborhood includes itself and all cells within `w` rows and `w` columns. Boundaries are clipped at the grid edges; the default model does not wrap around.
3. An agent is unhappy when its same-color fraction in that neighborhood is strictly below `tau`. Equality is happy. Internally, red is +1 and blue is −1; the bias array stores their neighborhood sum.
4. A transition selects an unhappy red agent and an unhappy blue agent uniformly from their respective lists and exchanges their positions. Population totals remain constant.
5. For `tau > 0.5`, the existing constrained-swap rule rejects pairs within one another's neighborhoods and pairs whose red-site bias is at least the blue-site bias plus two. Rejected selections count toward a retry limit. Once that limit is reached, the process stops without executing the rejected swap.
6. After an accepted swap, local bias values and unhappy lists are updated, and both changed cells are added to the 3D history. Up to ten swaps are performed per animation frame.
7. The run ends when either unhappy list is empty, after `500 × numbhum` swaps, or after that many rejected selections. Ending does not necessarily mean everyone is happy or complete segregation has occurred.

## Export details

SVG projects the same point geometry and camera transformation used by WebGL and orders the points by depth. Browser rasterization can differ slightly from WebGL at point edges.

GIF records each displayed simulation stage (up to ten swaps per stage), including the initial and final stages, rather than every individual swap. Frames use a fixed aspect ratio captured at start, a maximum width of 640 pixels, and a 256-color RGB332 palette. Resized views are fitted into that frame. Playback uses 80 ms per stage and holds the final stage for one second; pause durations are omitted. An export before any swaps contains one frame. Exporting keeps the simulation state intact.

The GIF encoder is embedded in the HTML and stores compressed frames in memory. Long runs or large grids can consume substantial memory because both swap history and animation frames are retained. The parameter panel limits size to 10–400 and radius to 1–30 (and less than size).
