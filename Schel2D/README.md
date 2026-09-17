# Schelling 2D simulation

A browser-based visualization of a two-type Schelling segregation model. Agents occupy a square grid and exchange positions according to local preferences, allowing you to watch clusters develop from a random initial arrangement.

## Run the app

Open `schel2D-js.html` in a modern browser. No build step or backend is required. The page loads Chroma.js from a CDN for color generation, so an internet connection is needed for that dependency, particularly when resetting or changing parameters.

`example.html` is a separate 1D simulation used as the visual reference for the tune panel and toolbar highlights.

## Controls

| Control | Action |
| --- | --- |
| **start / pause / resume** | Start the simulation, pause it, or continue the current run. Displays **finished** when the stopping condition is reached. |
| **reset** | Generate a new random grid and update its colors using the current parameters. Clear the recorded animation. |
| **rules** | Open the in-app explanation. Click outside the popup to close it. |
| **tune** | Open the parameter panel. Click outside the panel to close it. |
| **svg** | Download the current grid as a vector image. |
| **gif** | Download the recorded stages as a looping animation. Available after starting a run. |

The `g` key also starts, pauses, or resumes the simulation; `q` reloads the page.

### Parameters

| Parameter | Default | Range | Meaning |
| --- | --- | --- | --- |
| **Size** | 150 | 50–250 | Grid side length, `N`. The grid contains `N × N` agents; the default is 22,500 agents. |
| **Radius** | 4 | 1–10 | Neighborhood radius, `w`, in both grid directions. |
| **Tolerance** | 0.45 | 0–1, step 0.01 | Minimum fraction of the neighborhood that must have the agent's own type for it to be happy. |

Changing any slider immediately resets the simulation and clears its recording. Press **start** to run the new configuration.

## Transition process

### Initial state and neighborhoods

Every cell contains one agent, represented internally as `+1` or `-1`. Each initial type is chosen independently with probability 0.5, so the two population sizes are approximately equal. The display colors can change on reset; the code calls the types red and blue regardless of their displayed colors.

The grid wraps across both edges, forming a torus. Each agent's neighborhood is a square of side `2w + 1`, including the agent's own cell. Its size is therefore `K = (2w + 1)²`: radius 4 gives 81 cells.

The simulation stores a neighborhood bias `B`, the sum of the `+1` and `-1` values in that neighborhood. This determines the fraction of each type:

- A `+1` agent is unhappy when `B < K × (2τ − 1)`.
- A `-1` agent is unhappy when `B > K × (1 − 2τ)`.

Here `τ` is tolerance. Equality counts as happy. For example, with radius 4 and tolerance 0.45, an agent needs at least 37 same-type cells among the 81 neighborhood cells, including itself.

### One transition

1. Randomly select one agent from each type's unhappy list.
2. At tolerance at most 0.5, accept the selected pair. At higher tolerance, apply the additional `swapTest` condition described below and retry rejected pairs.
3. Exchange the two agents' positions. Each successful transition preserves both population counts, and every cell remains occupied.
4. Update neighborhood biases around the changed cells and maintain the unhappy lists. Reassess the two swapped positions as well.
5. Increment the successful-swap counter and continue.

For tolerance above 0.5, `swapTest` rejects a pair if the bias at the selected `+1` position exceeds the bias at the selected `-1` position by more than 2. It also rejects equality at a difference of 2 when the absolute coordinate differences in both directions are at most `w`. This proximity check currently uses raw coordinate differences rather than wrapped distances.

After `N²` unsuccessful random retries, the code attempts an exhaustive search over the unhappy pairs. If that search finds no permitted pair, it sets the halt flag.

### Animation and stopping

Each animation callback performs up to 40 transition attempts, redraws the grid, and records a frame if the successful-swap count changed. The recording includes the initial grid; it captures displayed stages rather than every individual swap.

The simulation stops when either type's unhappy list is empty or the exhaustive search sets the halt flag. Stopping does not necessarily mean every agent is happy. Although a `swapmax` variable exists, the animation loop does not enforce a maximum swap count.

GIF exports replay the recorded stages at 20 frames per second, hold the last stage for one second, and loop continuously. SVG and GIF exports contain the grid without the toolbar.

## Implementation notes

HTML, CSS, simulation logic, canvas rendering, and export code live in `schel2D-js.html`. Useful entry points are `resetSimulation`, `buildInitBiasArr`, `doSwapFollowConv`, `updateBiasLists`, and `animate`.

The in-app rules text describes some behavior differently from the implementation: the actual grid is fully occupied, transitions swap opposite types, and neighborhoods include the center cell and use the configured radius. The description above follows the code.

The current transition implementation also has limitations that matter when interpreting results:

- The exhaustive-search result omits `exhaustswap` and the blue-list index expected by its caller, so a permitted pair found through that fallback is not applied.
- The incremental unhappy-list update contains inconsistent coordinate and membership checks that can leave the lists out of sync with the grid.

These behaviors should be corrected and validated before treating the app as a quantitative reference implementation of the model.
