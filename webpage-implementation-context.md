# Property Results Dashboard - Webpage Implementation Context

This document is a fast-reference guide for future AI/code contributors working on `index.html`.

## Purpose

The webpage is a **single-file frontend dashboard** for property listings.
It consumes JSON data, renders map + cards, and supports both grid (tile) and list views.

Primary goals:
- Show key listing details and model score context quickly
- Support status lifecycle (`active`/`expired`)
- Surface price movement with fallback logic
- Keep interactions lightweight (no framework/build step)

## Project Files

- `index.html`: entire UI (HTML, CSS, JS)
- `webpage-schema-instructions.md`: schema/UI rules for status + price changes

No bundler, no component framework, no backend code in this repo.

## Runtime Data Source

In `index.html`, `DATA_URL` points to a remote JSON source.

Expected shape:
- Top-level object map; values are property records
- Code uses `Object.values(data || {})` to produce `properties`

## High-Level UI Structure

- Brand/header block
- Toolbar with filters/sort/view mode
- Main content grid:
  - Left: results cards (`#results`)
  - Right: map (`#map`)

### Result Views

Two render modes controlled by `viewMode`:
- `grid` (tile cards)
- `list` (compact row cards)

`toggleView(mode)` switches mode and calls `render()`.

## State Model (JS)

Key globals:
- `properties`: full dataset in memory
- `filtered`: currently filtered/sorted subset
- `selectedId`: selected listing (for map/card sync)
- `viewMode`: `grid` or `list`
- `topPickSet`: fixed top-3 records (computed once on load)
- `map`, `markers`, `mapMessageControl`: Leaflet map state

## Data Mapping Rules

### Status / Expiry

Status handling via:
- `getStatusDisplay(property)`

Behavior:
- `status === "expired"` -> expired badge + muted card style
- `expired_at` shown as a formatted date if present

### Price Change

Primary helper:
- `getPriceChangeDisplay(property)`

Fallback chain:
1. Use `property.price_change` when direction exists
2. Else derive from `price_history` (last two entries)
3. Else show no change indicator

Formatting helpers:
- `formatPrice(value)`
- `formatPriceChangeLabel(direction, value)`
- `renderPriceChangeHTML(change)`

Current display behavior:
- Up/down only show badge
- Flat/null do not render badge
- `(was £X)` is shown only when direction is up/down

## Filters and Sorting

Toolbar controls:
- Sort: score/price/first seen asc/desc
- Source filter (`locationFilter`)
- Status filter (`statusFilter`): all/active/expired
- Price movement filter (`priceMovement`): down/up/flat/none

`applyFilters()` performs:
1. Filter pipeline
2. Sort
3. Reset `selectedId`
4. Re-render + map refresh + status text

## Top Pick Logic

Top picks are intentionally stable and not filter-dependent.

- `computeTopPickSet(properties, 3)` runs once after load
- `isTopPick(property)` checks set membership
- Result: badges do not recalculate when filter set changes

## Card Composition

Both card modes include:
- Title/location
- Price (current + optional previous)
- Property metric chips
- Seen/completion metadata
- Expandable detail controls:
  - Key Features
  - Score Breakdown

### Grid/Tiles

- `View Listing` remains a full-width button near the bottom
- Detail controls are stacked vertically in the right side of metadata row

### List

Current intended pattern:
- Left metadata block remains unchanged (`First Seen`, `Last Seen`, completion)
- Right controls are aligned as a compact row:
  - Key Features | Score Breakdown | View Listing

## Tooltip/Details Behavior

Shared open/close logic:
- `toggleScoreBreakdown(event, button)`
- `closeScoreBreakdowns(exceptWrap)`

Any control using `.score-wrap` + `.score-tooltip` participates in this behavior.
Only one tooltip should be open at a time.

## Key Features Rendering

Helpers:
- `normalizeKeyFeature(feature)`
- `renderKeyFeatures(features)`

Long text handling:
- Uses `.feature-text` with wrapping (`overflow-wrap: anywhere`)
- Avoids truncating long key feature descriptions

## Styling System

- CSS variables in `:root` define palette, spacing, radius, shadows
- Earthy/light visual theme (greens/beiges)
- Shared utility classes: `.truncate`, chip/badge/button patterns
- Responsive breakpoints: 1280, 1120, 900, 640, 400

## Map Integration

Uses Leaflet from CDN.

Key behavior:
- Markers for filtered properties with valid lat/lng
- Selected card highlights corresponding marker
- Marker click scrolls to card and selects it
- If no coordinates: shows inline map message

## Demo/Test Mode

A temporary demo fixture block exists in `index.html`:
- Enabled with `?demo=1`
- Injects synthetic records for expired/up-down price changes
- Intended to help UI testing when live data lacks these states

Look for block comments:
- `DEMO MODE BLOCK START`
- `DEMO MODE BLOCK END`

## Safe Editing Guidelines for Future AI

1. Preserve `render()` structure unless strictly necessary.
2. Keep filtering/sorting in `applyFilters()` only.
3. Reuse existing helper functions before adding new ones.
4. Avoid changing top-pick behavior unless explicitly requested.
5. Keep list view compact (height-sensitive).
6. Ensure both `grid` and `list` templates are updated together when adding UI fields.
7. After edits, validate no syntax errors and check both views.

## Common Regression Risks

- Misaligned list controls due to accidental CSS inheritance from generic `.detail-controls`
- Tooltip clipping/overlap from width/position changes
- Reintroducing flat/no-change badge when not desired
- Breaking fallback logic for older records with history-only price fields
- Top Pick becoming filter-dependent again

## Quick Manual Test Checklist

- Load normal mode and `?demo=1`
- Switch grid/list views
- Test all filters and sort options
- Verify expired badge/muted style/date
- Verify up/down badge + previous price behavior
- Verify long key feature text wraps in tooltip
- Verify list controls align as one row: Key Features | Score Breakdown | View Listing
- Verify map marker/card selection sync
