# Default Card Size and Layout Templates

Use this reference only when the source game or design brief does not specify card dimensions, orientation, or layout. Explicit game requirements override every fallback below.

## Default production size

- Physical card: 63 mm wide × 88 mm high (standard poker size).
- Finished raster: 1500 × 2100 px, portrait 5:7.
- Safe area: keep critical text, numbers, and symbols at least 75 px from every edge.
- Corners: use one consistent radius across a deck; 60-90 px is a practical range at this canvas size.
- Build fronts and backs on the same canvas. Do not mix square and portrait cards within a deck unless the game requires it.

## Choose one template per deck

| Template | Use for | Default structure |
|---|---|---|
| `illustrated-action` | Actions, events, equipment, text-heavy effects | Compact top title band; framed art window using 55-65% of height; high-contrast bottom rules box; optional small type icon in a corner |
| `character-role` | Roles, characters, identities | Large character portrait; clear role title; one short ability/footer zone; accent color identifies role family |
| `full-art-stat` | Collectible characters, creatures, locations, stat-led cards | Full-bleed art; restrained title/stat overlays; small rules strip at the bottom; maintain legibility over varied art |
| `minimal-identity` | Simple roles, voting cards, category cards | Flat-color field; one central portrait or symbol; minimal title/supporting text; generous whitespace |

Do not select a different template for every card. Record the deck choice in `cards_plan.json`, then reuse its frame geometry, type hierarchy, stroke weights, padding, icon placement, and text zones across the whole deck.

## Complete-card card0 CLI rule

1. Use the selected template as prompt direction for card0 CLI; do not implement the template with drawing code.
2. Generate the entire ready-to-play card in one image, including illustration, frame, title, full description, values, icons, and labels.
3. Ask card0 CLI to render all supplied text and numbers exactly as written.
4. Keep the approved sample image as the visual reference for every remaining card in that deck.
5. Verify the result at full size and thumbnail size. Text must remain readable and the deck type must be recognizable face-down and face-up.

PIL, HTML, SVG, Canvas, and similar code may compress or convert files, but must not draw or overlay visible card content.

## Card backs

A shared deck back is ornamental, visually balanced under 180° rotation, and recognizable at thumbnail size. Use a central emblem or repeated motif, a restrained deck accent, and the same border language as the fronts. Avoid rules text, component counts, footers, multiple character portraits, and large labels that make the back resemble a front.

Different decks receive distinct backs by default, but those backs should still look like one game family. Cards with mechanically meaningful reverse content keep their per-card backs. Accessories use front = back by default. Explicit game rules always take priority.

## Reference-image boundary

Use reference cards to understand hierarchy, spacing, density, and the balance between art and text. Do not copy protected characters, logos, names, exact frames, or distinctive commercial trade dress.
