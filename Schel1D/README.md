# Schelling 1D simulation

An interactive visualization of a one-dimensional Schelling segregation model. Two types of agents occupy a circular sequence of positions. Agents that have too few neighbors of their own type exchange places, showing how local preferences can produce larger groups of the same type.

The app is contained in [schel1D-js.html](schel1D-js.html), with its HTML, styles, simulation logic, and export code in one file. It runs in the browser without a build step or backend.

## Running the app

Open `schel1D-js.html` in a modern browser. An internet connection is needed to load the Chroma.js color library from its CDN. The initial view is a preview; pressing **start** creates a fresh random population and color pair and begins the simulation.

## Controls

The panel is arranged as **[start] [reset] [tune] [svg] [gif] [rules]**. Button widths stay fixed as labels change.

| Button | Action |
| --- | --- |
| **start** | Begin a new simulation. The button becomes **pause**. |
| **pause** | Pause the current run without losing its population or history. The button becomes **resume**. |
| **resume** | Continue the same run. The button becomes **pause** again. |
| **reset** | Stop and restore the current run’s initial population and colors. Clear transitions and GIF history, disable GIF export, and return the main button to **start**. |
| **tune** | Open the parameter sliders. Click outside the panel to close it. Changes apply when a new simulation starts, not when a paused run resumes. |
| **svg** | Download the current visualization as a vector SVG, including the rings and transition history. Available before and after starting. |
| **gif** | Download an animated GIF of the current run from its initial state through the export point. Enabled after **start** is pressed. |
| **rules** | Show the model explanation. Click outside the popup to close it. |

After the simulation finishes, the main button returns to **start**. Starting again replaces the previous run and its GIF history. Use **reset** to return to the initial state at any time. Pressing **start** after resetting runs that same initial population again; random swap choices can produce a different history. If you change the parameters before starting, the app creates a fresh population using those settings.

Keyboard shortcuts: **g** starts or resumes; **q** pauses. Opening the tune or rules panel does not pause the simulation.

## Parameters

| Parameter | Default | Range | Meaning |
| --- | --- | --- | --- |
| Size | 4,000 | 1,000–20,000 | Number of agents and positions on the circle. |
| Radius | 50 | 1–100 | Number of positions considered on each side of an agent. |
| Tolerance | 0.4 | 0–1 | Minimum same-type proportion required for an agent to be happy. |
| Minority | 0.5 | 0–1 | Probability that each initial agent belongs to the first type. The actual proportion varies randomly. Values above 0.5 make that type the majority. |

The two types are named red and blue internally, but each run uses a randomly selected contrasting color pair.

## How transitions work

1. **Initialize the circle.** Each position independently receives one of the two agent types according to the Minority setting. The number of agents of each type remains constant during the run.
2. **Evaluate happiness.** The program counts the types within each local neighborhood, wrapping around the ends of the sequence. It maintains separate lists of unhappy agents of each type.
3. **Select a pair.** One unhappy agent of each type is chosen randomly. For tolerance at or below 0.5, this pair can swap directly. Above 0.5, an additional selection rule rejects pairs within the neighborhood radius of each other, including across the circular boundary, and pairs whose neighborhood biases fail the implemented comparison `bias[first] <= bias[second] + 2`.
4. **Swap positions.** The two agents exchange places. This counts as one transition (one swap), and the program records the new color at both positions along with the transition number.
5. **Update nearby agents.** Neighborhood counts and unhappy lists are updated around the two changed positions. The next pair is selected from those updated lists.
6. **Repeat or stop.** The run stops when either type has no unhappy agents left, or when the circle contains exactly two boundaries between types, meaning each type occupies one contiguous group. Stopping does not necessarily mean every agent is happy: there may simply be no unhappy opposite-type partner available.

The implementation includes the agent itself in its happiness calculation, using a neighborhood of `2 × radius + 1` positions. An agent is happy when its same-type count divided by that neighborhood size is at least the tolerance. The in-app rules currently describe a neighborhood excluding the agent; the calculation described here reflects the actual code.

One swap is attempted per animation callback, and the visualization normally redraws every five swaps. Pausing, exporting, and reaching the stopping condition also redraw the current state. The speed therefore depends on the browser and the amount of history being rendered.

## Reading the visualization

- **Small inner ring:** the initial population, preserved throughout the run.
- **White marks near the inner ring:** agents that were unhappy initially; these marks do not track current happiness.
- **Large outer ring:** the current population. Each angular position represents the same location in the circular sequence.
- **Colored dots between the rings:** the history of swaps. Each swap adds two dots at the affected angular positions, colored by the type occupying each position after the swap.

History progresses outward: early transitions lie near the center and recent transitions lie near the outer ring. A dot's radius is calculated as `40 + 290 × transitionIndex / (currentTransitionCount + 1)`. As the run grows, earlier dots move inward because the entire history is rescaled. These dots are a record of position changes, not paths traced by individual agents.

## Export details

SVG exports use the same drawing geometry as the canvas at its native 1,000 × 800 size. They contain vector paths and can be scaled or edited in a vector graphics application.

GIF exports are 500 × 400 pixels with a 256-color palette. They include the initial frame, subsequent displayed stages, and the current stage at export time. They do not include a separate frame for every individual swap. Playback loops, using an 80 ms delay per frame and a one-second hold on the final frame; time spent paused is not recorded. Palette conversion can slightly change the colors compared with the canvas or SVG.

Both exports download locally as `schelling-<transition-count>.svg` or `schelling-<transition-count>.gif`. Exporting does not reset or pause a running simulation. Compressed GIF frames are retained in browser memory for the current run, so long runs can use more memory.

## Current limitation

For tolerance above 0.5, the pair-selection loop has no retry limit. If no eligible pair can be found, it can block the browser rather than finish the simulation. The declared `swapmax` value is not currently enforced as a stopping condition.
