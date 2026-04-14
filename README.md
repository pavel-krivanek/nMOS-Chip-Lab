# nMOS Chip Lab

nMOS Chip Lab is a browser-based layout editor and circuit simulator for small depletion-load nMOS circuits.

It is meant as a compact laboratory for understanding how a chip-like layout turns into an electrical network, how parasitics influence behavior, and how simple logic structures such as an inverter or NAND gate behave over time.

This is not a full IC CAD flow and not a full SPICE implementation. It sits in the middle: more realistic than a pure logic-gate toy, much simpler than a production semiconductor simulator.

## On-line version

[Try online](https://pavel-krivanek.github.io/nMOS-Chip-Lab/)

For the latest version, use this repository.

## What the project does

From a cell-based layout, the simulator extracts:

- distributed resistance in diffusion, poly, and metal1
- contact resistance between layers
- parasitic capacitance per cell
- junction capacitance at p/n boundaries
- automatically extracted nMOS transistors where poly crosses p or depletion-load diffusion with n-diffusion source/drain regions on opposite sides

It then solves the resulting network using:

- **DC operating-point solve**
- **transient time-step simulation** with capacitors and time-varying inputs

The UI lets you:

- draw a design on an infinite zoomable grid
- define explicit chip bounds if desired
- place VCC, GND, inputs, outputs, contacts, and labels
- define input waveforms and expected output waveforms
- run simulation and inspect waveforms
- save and load `.chipdesign` files
- drag and drop a saved design onto the page to load it
- switch between System, Light, and Dark themes

## Files

- `index.html` — single-page UI application
- `_ui_extract.js` — UI logic and editor behavior
- `sim-core.js` — extraction and simulation engine
- `tests.js` — Node-based regression tests
- `README.md` — this document

## Running it

### Open the app

Open `index.html` in a browser.

No build step is required.

### Run tests

```bash
node tests.js
```

Current test status for this bundle:

- 13 passed
- 0 failed

## What is more-or-less realistically simulated

The simulator is intentionally simplified, but several parts are still grounded in real physical effects.

### 1. Geometry-derived connectivity

Connectivity is not hardcoded at the logic-gate level. It is derived from the drawn layout:

- adjacent cells on the same conductive layer connect through sheet resistance
- contacts connect diffusion or poly to metal1 through finite resistance
- labels and ports attach to actual extracted nodes
- p/n boundaries become junctions rather than shorts

That means wire length, routing choice, and where a contact is placed all matter.

### 2. Distributed RC behavior

The simulator includes:

- resistance per square for p diffusion, n diffusion, poly, and metal1
- capacitance per cell for conductive regions
- contact capacitance
- junction capacitance at p/n boundaries

So long narrow wires, high-resistance poly, and diffusion-heavy nodes really do slow edges down.

### 3. Automatic transistor extraction from layout patterns

An nMOS transistor is extracted when:

- a poly cell crosses a p-region or depletion-load p-region cell
- n diffusion exists on opposite sides of the gate cell
- source/drain-adjacent contacts are not violating the extraction rule used by this prototype

This gives the editor a genuine layout-to-device step rather than a hand-entered schematic.

### 4. Distinction between enhancement and depletion devices

The simulator distinguishes:

- **enhancement-mode nMOS** in regular p regions
- **depletion-load nMOS** in depletion-marked p regions

This is important for classic depletion-load nMOS logic, where a depletion device acts as a pull-up load.

### 5. Nonlinear transistor behavior

The transistor model is simplified, but it is not just an ideal switch.
It includes:

- threshold voltage
- an effective strength term (`beta` per extracted width)
- a linear region and saturation region split
- channel-length modulation via `lambda`
- off-state leakage through a large `ROFF`

So the pull-down is voltage-dependent and not purely digital.

### 6. Time-domain response

Transient simulation includes capacitive memory and time-stepped solving, so you can see:

- finite rise and fall times
- output lag behind inputs
- the effect of heavier loading
- different responses caused by parameter changes

## What is simplified

This is the most important section to read if you want to understand the simulator honestly.

### 1. It is not SPICE

The simulator does **not** implement full modified nodal analysis with detailed MOSFET equations, body effect, subthreshold modeling, temperature dependence, or a full Jacobian-based Newton-Raphson flow.

Instead, each extracted transistor is treated as a voltage-dependent effective conductance between two nodes, derived from a compact simplified nMOS model.

That makes the simulator useful and responsive, but still approximate.

### 2. Only a very small process stack is modeled

The extracted layers are limited to:

- p diffusion / p body region
- depletion-load p region (`pDepl`)
- n diffusion
- polysilicon
- metal1
- three contact types between metal1 and diffusion/poly

There is no support for:

- multiple metal layers
- vias between higher metals
- explicit wells
- implant rules beyond the simple depletion marker
- design-rule checking beyond a few extraction constraints

### 3. Device extraction is pattern-based and narrow in scope

Transistor extraction is intentionally simple and cell-local.

It assumes a very specific topology:

- one gate cell location
- n diffusion on opposite sides of the gate
- poly crossing the p or depletion region

It does not try to infer arbitrary device geometries or full width merging like a modern layout extractor would.

### 4. Capacitances are lumped approximations

Capacitances are estimated using fixed per-cell and per-junction constants. They are not geometry-calibrated from a real fabrication process.

This means the simulator gives a useful **qualitative** sense of loading and delay, but not fabrication-grade timing numbers.

### 5. Numerical methods are practical, not industrial

The current transient engine uses:

- fixed timesteps
- dense Gaussian elimination
- a damped nonlinear iteration

That is fine for small educational layouts, but it is not the most advanced or scalable numerical strategy.

### 6. The editor is layout-first, not rule-driven

The tool lets you draw structures that a real process might reject or that would need far more process context to validate.

The simulator can warn about some net conflicts, but it is not a full DRC/LVS environment.

### 7. Some built-in logic examples are macro shortcuts in the core

The UI focuses on **Inverter** and **NAND** as editable prebuilt examples.
The simulation core still contains internal helper layouts/macros for NOR, AND, and OR mainly for regression testing.
Those are not meant to imply full physically extracted implementations in the current UI workflow.

## When to trust the simulator

You can trust it for:

- learning how a layout turns into a resistive-capacitive transistor network
- comparing one layout against another inside the same simplified model
- seeing why long poly or diffusion routes slow circuits down
- understanding depletion-load nMOS style logic qualitatively
- checking whether a small layout behaves like an inverter or NAND

You should **not** trust it for:

- tape-out decisions
- real process signoff
- exact delay or power prediction
- accurate analog behavior
- modern CMOS design analysis

## Small tutorial

## 1. Start with a built-in example

1. Open `index.html`.
2. In the **File** menu, choose either **Inverter** or **NAND** from the prebuilt circuit selector.
3. Click **Load**.
4. The editor will fit the view to the loaded design automatically.

That is the fastest way to see a working layout before editing anything.

## 2. Understand the main tabs

The main workspace has four tabs:

- **Die** — the layout editor
- **Waveforms** — simulation results
- **Waveform definitions** — input and expected output waveforms
- **Parameters** — electrical model parameters

A good first workflow is:

1. load an example
2. inspect the die
3. edit waveform definitions
4. run simulation
5. inspect the waveform tab

## 3. Navigate the die editor

In the **Die** tab:

- mouse wheel zooms
- middle mouse button pans
- left mouse button edits using the active tool

The grid is infinite. You can still define explicit chip bounds if you want a finite work area.

Useful tools in the left sidebar include:

- **Select**
- **Erase**
- diffusion, poly, and metal layers
- contact tools
- one tool per currently defined input and output port

The **View** box controls what layers and overlays are visible.

## 4. Read the layout conventions

The built-in examples show the intended style:

- **metal1** is used for rails and interconnect
- **poly** is used for gates and some routing
- **n diffusion** forms source/drain regions
- **p** or **pDepl** under the gate identifies the channel type
- **VCC** and **GND** are supply ports
- named input and output ports are attached to real extracted nodes

If a transistor is extracted, the UI shows an oriented butterfly marker at the gate location.

## 5. Define input and expected output waveforms

Open **Waveform definitions**.

There you can:

- set **Interval ps** — the duration of one waveform interval
- set **Intervals** — how many intervals exist
- add or remove inputs and expected outputs
- rename signals
- toggle each interval high or low
- draw a waveform directly with the mouse on the small waveform canvas

For simulation, defined input names should match actual input ports in the layout.
Expected outputs are for comparison and documentation in the UI.

## 6. Run simulation

Use the toolbar:

- **Run** to extract and simulate
- **Replay** to animate the transient on the die
- **Stop** to stop replay

After running:

- the **Waveforms** tab shows the main waveform results
- the right-side waveform panel also shows waveforms
- the status and summary panels show extracted information and any conflicts

## 7. Edit parameters carefully

Open the **Parameters** tab.

Each parameter includes a short description. Some useful experiments:

- increase **R□ poly** to make poly routing slower
- increase **C n** or **C m1** to make transitions slower
- change **Vth enh** to see when pull-down devices switch more weakly or strongly
- change **Depl scale** to make depletion loads stronger or weaker

These parameters are best treated as model knobs, not calibrated process values.

## 8. Save and load your work

Use the **File** menu or the toolbar to:

- save the current design to a `.chipdesign` file
- load a previously saved design

You can also drag a `.chipdesign` file from your desktop and drop it onto the page.

The app warns about unsaved changes when you try to replace the current design.

## 9. A practical first exercise

A good first exercise is:

1. load the **Inverter** example
2. run it once and inspect the waveforms
3. make the poly input run longer
4. run again and see the slower response
5. increase metal or diffusion area on the output node
6. run again and compare the new delay

That quickly shows the main idea of the project: **layout decisions change circuit behavior**.

## Implementation notes

A few details from the current codebase are useful to know:

- transient simulation now extracts the static network once per run and updates only time-varying input drives at each timestep
- passive stamping is cached between nonlinear iterations for better performance
- the core is written so it can run both in the browser and in Node-based tests

## Summary

nMOS Chip Lab is best understood as an educational and exploratory tool for small **layout-derived depletion-load nMOS logic**.

It is realistic enough to make routing, resistance, capacitance, contacts, and device placement matter.
It is simplified enough to stay understandable, hackable, and fast enough to run inside a single-page browser app.
