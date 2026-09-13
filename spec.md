# Eurobox Stacker — v1 Product Spec

## Goal

Build a small web app for planning dimensionally accurate stacks of Euro-standard storage boxes, initially for under-stairs storage.

The app should let the user build a physically valid stack from real Eurobox sizes, see an isometric representation of the result, and understand the total dimensions and approximate cost.

The initial catalogue should be based on products from Storage Box Shop’s Euro solid storage box range.

## Primary Use Case

The user starts with a large base footprint, typically 600 × 400 mm, and builds upward using smaller or same-sized Euroboxes.

Multiple boxes may occupy the same vertical level where the footprint allows it.

Example:

- 1 × 600 × 400 box as the base
- 2 × 300 × 200 boxes side by side above it
- further boxes above those where structurally valid

The purpose is to explore compact arrangements that fit into constrained spaces such as under stairs.

## Box Catalogue

Use real catalogue items rather than generic box classes.

Each box SKU should contain:

- external width
- external depth
- external height
- approximate capacity
- current or guide price
- footprint family
- source URL
- optional product name / SKU

Initial footprint families:

- 600 × 400 mm
- 400 × 300 mm
- 300 × 200 mm

Initial heights should be taken from the Storage Box Shop Euro solid storage range.

Where two products differ in real external height, treat them as separate SKUs rather than combining them into an approximate size.

## Dimensional Accuracy

The visualisation must be generated from real dimensions.

Do not use fixed graphical box proportions.

A 600 × 400 × 120 box must appear proportionally different from a 400 × 300 × 320 box.

All placement, fit, height calculations and collision rules must use the actual external dimensions stored in the catalogue.

## Coordinate Model

Use a Cartesian 3D model internally:

- X = width
- Y = depth
- Z = height

All boxes should occupy explicit rectangular volumes.

The isometric view should be a projection of this model.

## Stack Structure

The stack is not a simple single-column list.

Each horizontal region may contain multiple boxes.

A larger box can therefore provide a surface that is subdivided into smaller boxes above it.

Examples that should be representable include:

- one 600 × 400 box
- two 300 × 200 boxes placed side by side on part or all of a larger surface
- multiple smaller boxes occupying distinct positions on a common supporting surface

Boxes should snap to valid positions.

Free-floating arbitrary placement is not required.

## Placement Rules

A box may only be added where its complete footprint is physically supported.

Allowed cases include:

1. A box stacked directly on another box of the same footprint.

2. A smaller box placed fully within the supported top surface of a larger box.

3. Multiple smaller boxes occupying different regions of a larger supporting surface.

A box must not overhang unsupported space.

A larger box must not automatically be considered supported merely because several smaller boxes below happen to tile the same overall footprint.

For example:

four 300 × 200 boxes arranged into a 600 × 400 rectangle should not automatically allow a 600 × 400 box to sit directly above them.

The upper box would bridge several independent box rims and internal seams.

## Lid / Support Rule

Use real place-on Eurobox lids as structural components.

A lid creates a continuous supported surface matching its footprint.

If a proposed box arrangement would require bridging multiple independent boxes below, the app should require an appropriate lid before allowing the placement.

Example:

- four 300 × 200 boxes form a 600 × 400 overall rectangle
- placing a 600 × 400 box directly above should be rejected
- inserting a suitable 600 × 400 support/lid surface should make the arrangement valid

Lids should:

- have real external dimensions
- have real thickness
- have a price
- contribute to total stack height
- contribute to total cost
- be visibly rendered in the isometric view

Use place-on lids rather than hinged lids for v1.

The user generally cannot access a lower box without removing boxes above it anyway, so hinged-access mechanics are outside scope.

## Lid UX

Do not silently insert lids.

When the user attempts a placement that requires one, show a clear prompt such as:

"600 × 400 support lid required"

The user can then add the recommended lid.

The app may suggest the correct catalogue lid and show its price before insertion.

## User Interaction

The primary interaction should happen directly through the stack.

The user must be able to:

- select a box from the catalogue
- add it to a valid position
- remove a box
- replace an existing box with another valid size
- add a lid when required
- remove a lid where doing so does not invalidate boxes above

The interface should prevent structurally invalid arrangements.

Possible placements should snap to valid Eurobox positions.

## Orientation

Boxes may be rotated 90 degrees where the footprint geometry allows it.

For example, a 400 × 300 footprint may be used as either:

- 400 wide × 300 deep
- 300 wide × 400 deep

The visualisation and support model must respect orientation.

## Isometric View

Use a fixed isometric view for v1.

True free-camera 3D is not required.

The visualisation should clearly show:

- individual boxes
- different heights
- different footprint sizes
- box boundaries where several boxes share a level
- lids/support surfaces
- relative position of all objects

The rendering should remain legible for tall stacks.

Pan/zoom may be added if easy, but camera orbit is not necessary.

## Measurements

Always show:

- total maximum width
- total maximum depth
- total stack height
- total box count
- total lid count
- estimated total cost

Dimensions should be shown in millimetres.

Optionally show metres where useful for larger totals.

## Price Model

Use catalogue or guide prices stored with each product.

For v1, each SKU should have one working price used in the total.

The UI may additionally display price guidance such as:

- good price
- normal range
- expensive threshold

but the stack total should use one clearly defined current/guide price per SKU.

## Catalogue Editing

Keep product data separate from the rendering logic.

Use a simple structured dataset such as JSON.

Example concept:

```js
{
  id: "euro-600-400-175",
  width: 600,
  depth: 400,
  height: 175,
  price: 13.19,
  type: "box",
  source: "..."
}
```

Lids should use the same catalogue approach:

```js
{
  id: "lid-600-400",
  width: 600,
  depth: 400,
  height: 15,
  price: 6.84,
  type: "lid",
  source: "..."
}
```

Do not hard-code product dimensions into UI components.

## Under-Stairs Mode

Under-stairs geometry is not required for the first implementation milestone, but the architecture should allow it later.

A future mode should support a sloped ceiling / stair underside defined by parameters such as:

- usable floor depth
- height at one end
- height at the other end

The stack could then be checked for collision against the available envelope.

For v1, focus on building and measuring the stack itself.

## Suggested UI

Desktop-first single page.

Left or bottom panel:

- catalogue
- footprint filter
- box height
- price
- add-box controls

Main area:

- isometric stack view
- selectable boxes
- valid placement targets

Summary panel:

- width
- depth
- height
- box count
- lids
- total price

When a selected box has multiple valid positions, show those positions visually on the stack.

When no valid position exists, explain why.

## Invalid Placement Feedback

Avoid generic errors.

Examples:

- "This box would overhang the supporting box."
- "A 600 × 400 lid is required before this box can be placed here."
- "This footprint cannot fit in the selected position."
- "Removing this lid would leave the box above unsupported."

## Persistence

For personal-use v1, store the current arrangement in localStorage.

Include:

- New stack
- Save automatically
- Reset stack

No account or backend is required.

## Technical Preference

A self-contained browser application is sufficient.

Preferred implementation:

- HTML
- CSS
- JavaScript or TypeScript
- SVG or Canvas for rendering

A lightweight framework is acceptable, but do not introduce a backend or heavyweight 3D engine unless it provides a clear benefit.

SVG is attractive because the isometric geometry is simple, selectable elements are useful, and dimensions can be rendered crisply.

## Important Implementation Principle

The source of truth must be the geometric model, not the drawing.

The application should:

1. calculate valid positions
2. validate structural support
3. calculate dimensions and price
4. derive the isometric drawing from that state

Do not infer geometry from DOM positions or manually positioned graphics.

## Initial Acceptance Criteria

The implementation is successful when the user can:

1. Start with a 600 × 400 Eurobox.

2. Add another 600 × 400 box directly above it.

3. Replace the upper box with one or more smaller valid Euroboxes.

4. Place multiple boxes on the same supporting level.

5. Rotate compatible boxes where necessary.

6. Be prevented from creating overhangs.

7. Be prevented from placing a large box across several smaller unsupported boxes.

8. Be told when an appropriate lid is required.

9. Add that lid and then complete the previously invalid placement.

10. Remove or replace boxes without corrupting the stack.

11. See the stack update immediately in an accurate isometric view.

12. See correct overall dimensions.

13. See a running estimated total price.

14. Refresh the browser and retain the current stack.

## Out of Scope 

- user accounts
- cloud persistence
- warehouse inventory
- labels / box contents
- colours as meaningful metadata
- weight/load calculations
- animated physics
- accessibility of boxes while stacked
- arbitrary free-form 3D movement
- pallet planning
- multiple separate stacks
- automatic under-stairs optimisation

## Likely v2

Once the stack-builder itself works correctly:

- under-stairs envelope
- visual slope / stair underside
- drag stack along the available floor
- automatically flag collisions
- suggest better combinations
- optimise for storage volume
- optimise for cost
- optimise for number of boxes
- compare alternate stack designs


## Deployment / Packaging

The app must be a **single self-contained HTML file** that can be opened directly in a browser.

No infrastructure should be required.

Specifically:

- no backend
- no server
- no build step required for the final artifact
- no database
- no authentication
- no external API dependency at runtime
- no package installation required to use it
- no CDN dependency if avoidable
- all CSS embedded in the HTML
- all JavaScript embedded in the HTML
- product catalogue embedded in the HTML as structured data
- persistence via `localStorage`
- should work from a local `file://` URL

The final deliverable should therefore be something like:

`eurobox-stacker.html`

The user should be able to copy that file to another computer, open it in a modern browser, and use the utility immediately.

### Technical implication

Prefer:

- plain HTML
- plain CSS
- vanilla JavaScript
- SVG for the isometric rendering

Avoid React, Vue, npm, bundlers, WebGL frameworks, or any other dependency unless they are compiled fully into the single HTML artifact and provide a substantial benefit.

The application should remain understandable and maintainable as one file.

### Offline requirement

Once the HTML file has been created, all core functionality must work offline:

- build stacks
- add/remove/change boxes
- validate placement
- add lids
- calculate dimensions
- calculate price
- save/load current state locally

External product links may be present as references, but the app must not require them to function.