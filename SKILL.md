---
name: p5-brush
description: "Generate generative art sketches using p5.js + p5.brush. Describe what you want to draw and get a standalone HTML file with natural, hand-drawn-style artwork."
argument-hint: "a stormy ocean in watercolor with heavy bleed"
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
---

# p5.brush — AI Generative Art Skill

You generate standalone HTML files that produce generative art using **p5.js 2.x** and **p5.brush v2**.

**Library by [@acamposuribe](https://github.com/acamposuribe/p5.brush)**. Design custom brushes visually with the [Brush Maker](https://acamposuribe.github.io/p5.brush/tools/brush-maker.html).

**Companion files**: See `REFERENCE.md` (palettes, recipes, templates) and `UTILITIES.md` (reusable code patterns) in this skill directory for additional depth.

## Workflow

1. User describes what they want (e.g. "a field of wildflowers in charcoal", "abstract watercolor circles")
2. **Plan the artwork** using the Artist's Mindset and Checklist (sections below) before writing any code
3. You write a complete HTML file with an embedded p5.js sketch using the p5.brush API
4. Save to a sensible location (e.g. `./output/<descriptive-name>.html` in the current project, or wherever the user specifies)
5. Open in browser: `open <path>`

## Output Template

Every generated file follows this structure:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>p5.brush — [title]</title>
  <script src="https://cdn.jsdelivr.net/npm/p5@2/lib/p5.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/p5.brush@2.0.0-beta"></script>
  <style>
    html, body { margin: 0; padding: 0; overflow: hidden; background: #1a1a1a; display: flex; justify-content: center; align-items: center; height: 100vh; }
    canvas { display: block; }
  </style>
</head>
<body>
<script>
function setup() {
  createCanvas(800, 800, WEBGL);
  brush.load();
  // Shift WEBGL origin to top-left:
  translate(-width / 2, -height / 2);

  // Set background
  background(245, 240, 235); // warm paper

  // Set a random seed for reproducibility
  randomSeed(42);

  // --- Your generative art code here ---
}
</script>
</body>
</html>
```

## CRITICAL RULES

1. **Always use `WEBGL` mode**: `createCanvas(w, h, WEBGL)` — p5.brush v2 requires it
2. **Always call `brush.load()`** in setup, after `createCanvas(w,h,WEBGL)`. Use `brush.load(buffer)` to target a `p5.Graphics` or `p5.Framebuffer`, and `brush.load()` to switch back
3. **Shift origin**: `translate(-width/2, -height/2)` right after background to work in top-left coordinates
4. **Angles follow `angleMode()`** — radians by default (matching p5). `brush.addField()` angles default to degrees unless you pass `{ angleMode: "radians" }`
5. **`brush.arc()` is stroke-only** — it does NOT fill. `brush.circle()` and `brush.rect()` support stroke + fill + hatch
6. **Use `randomSeed()`** for reproducible outputs — p5.brush syncs automatically (both `randomSeed()` and `noiseSeed()`)
7. **Static sketches only need `setup()`** — no `draw()` loop needed. Use `draw()` only for animations with `brush.refreshField(frameCount/10)`
8. **All drawing in one frame** — put everything in `setup()` for static art
9. **Text labels: use HTML overlay, NOT p5 `text()`** — WEBGL mode requires TTF/OTF fonts loaded via `loadFont()` which is fragile with CDN fonts. For labeled artwork, use absolutely positioned HTML `<div>` elements over the canvas instead
10. **Image brushes require `await`** — `brush.add()` returns a Promise for `type: "image"`. You must `await` it and declare `async function setup()`
11. **Fill budget** — watercolor fills are expensive. Keep filled shape count reasonable (under ~50 for complex fills with high bleed). Use multi-pass washes only at focal areas; periphery can use single fills or stroke-only rendering

---

## ARTISTIC FOUNDATIONS

The sections below are **more important than the API reference**. They teach you to think like a skilled artist. A technically correct sketch with no artistic judgment looks algorithmic; a sketch guided by these principles looks hand-made.

### The Artist's Mindset

Before writing any drawing code, answer these five questions:

1. **Mood** — What feeling should this piece evoke? (Calm, dramatic, joyful, melancholic, energetic?) Mood drives color temperature, value range, edge quality, and brush choice.
2. **Focal point** — Where should the viewer's eye land first? Pick ONE area (canvas coordinates). This area gets maximum contrast, sharpest edges, most detail, and most saturated color.
3. **Eye path** — After the focal point, where does the eye travel? Plan 2-3 secondary areas connected by leading lines, value contrasts, or color echoes.
4. **What to leave out** — What should be merely *suggested*, not fully rendered? Restraint is the hardest skill. A watercolorist's power comes from what they leave unpainted. Plan at least one area of visible paper or loosely sketched suggestion.
5. **Medium** — What combination of brush techniques? (Pure watercolor wash, ink + wash, charcoal study, mixed media?) This determines your brush vocabulary for the piece.

**Translate these decisions into a scene config before any shapes:**

```js
// Plan the artwork before drawing
const FOCAL = { x: 720, y: 480, r: 200 }; // focal point + influence radius
const PALETTE = {
  paper: [252, 248, 240],
  warm: ["#D4785C", "#E8A87C", "#F0C994"],
  cool: ["#5B7D9A", "#8AAFC4", "#BCD4E6"],
  accent: "#C23B22",
  shadow: "#4A3B5C", // chromatic shadow — NOT gray
  mother: "#E8A87C"  // tint all colors slightly toward this
};
```

**The 80/20 rule**: 80% suggestion, 20% full rendering produces more artistic results than 100% rendering. An incomplete shape with confident marks reads better than a fully filled, fully outlined, fully textured one.

### Composition & Visual Hierarchy

**Never place the focal element dead center.** Use these frameworks:

- **Rule of thirds**: Divide canvas into a 3x3 grid. Place the focal point at an intersection. For 1200x800: intersections at (400, 267), (800, 267), (400, 533), (800, 533).
- **S-curve / serpentine**: The eye follows an S-path through the composition — a river, a road, a sequence of elements. The S itself is a compositional device, not just a shape.
- **Steelyard**: Heavy visual mass on one side balanced by a small, intense accent on the other.
- **L-composition**: Strong vertical + strong horizontal create a frame; the focal area sits in the open space.
- **Radiating**: Elements converge toward or emanate from the focal point.

**The 70/25/5 rule for visual weight:**
- 70% of the canvas = supporting area (loose rendering, low detail, soft edges)
- 25% = secondary interest (moderate detail, medium contrast)
- 5% = focal point (maximum contrast, sharpest edges, most detail, strongest color)

**Depth through overlap**: Objects overlapping other objects creates more convincing depth than just lighter colors in the background. Let a tree trunk overlap a distant hill. Let foreground grass strokes cross over a path edge.

**Code pattern — focal-aware rendering:**
```js
// Returns 0 at focal point, 1 at maximum distance
function focalFade(x, y) {
  return constrain(dist(x, y, FOCAL.x, FOCAL.y) / FOCAL.r, 0, 1);
}

// Modulate rendering parameters based on distance from focal point
function renderAt(x, y) {
  let t = focalFade(x, y);
  return {
    opacity: lerp(200, 60, t),
    bleed: lerp(0.06, 0.5, t),
    texture: [lerp(0.15, 0.5, t), lerp(0.1, 0.4, t)],
    weight: lerp(1.2, 0.3, t),
    detail: t < 0.3 ? "full" : t < 0.6 ? "moderate" : "minimal"
  };
}
```

### Value Structure

**Plan values BEFORE color.** Every compelling artwork has a clear pattern of darks, mid-tones, and lights. Without this, the piece looks flat regardless of how many colors you use.

**The 3-value map:**
- **Darks** (opacity 180-255, deep hues): Occupy ~20% of the canvas. Anchor the composition — strong darks near the focal point create drama and contrast.
- **Mid-tones** (opacity 80-150): Occupy ~60% of the canvas. The connective tissue.
- **Lights** (low opacity 20-60, or bare paper): Occupy ~20%. Create luminosity and breathing room. The most common mistake is filling lights with color — leave paper showing instead.

**Maximum contrast at the focal point**: Place your darkest dark directly adjacent to your lightest light at the focal area. This is the single most powerful technique for directing the viewer's eye.

**Build value through layered washes, not single fills:**
```js
// BAD: Single high-opacity fill (flat, lifeless)
brush.fill("#5C4D7D", 180);
brush.polygon(mountainShape);

// GOOD: Three transparent layers building richness
brush.fill("#7B6BA0", 50);   // first wash — light
brush.fillBleed(0.35, "out");
brush.fillTexture(0.4, 0.3);
brush.polygon(mountainShape);

brush.fill("#5C4D7D", 40);   // second wash — darker, slightly inset
brush.fillBleed(0.2, "out");
brush.polygon(innerShape);

brush.fill("#3A2D5C", 30);   // darkest accents — smallest area
brush.fillBleed(0.1, "out");
brush.polygon(shadowAccent);
```

**Multi-pass washes count against the fill budget.** Use them at the focal point (2-3 passes) and for the 1-2 most important background elements. Peripheral areas get single-pass fills or stroke-only treatment.

### Color Theory for Generative Art

**Color temperature creates depth:**
- Warm light (sunset, golden hour) → cool shadows (purple, blue-gray)
- Cool light (overcast, moonlight) → warm shadows (burnt sienna, warm gray)
- Every scene needs BOTH warm and cool — even if one dominates

**Chromatic shadows — never use pure gray:**
```js
// BAD: Gray shadow
brush.fill("#888888", 80);

// GOOD: Chromatic shadow — complement of the light
// Warm scene → cool purple-gray shadow
brush.fill("#5A4B6E", 70);

// Cool scene → warm brown-gray shadow
brush.fill("#7A6B5C", 70);
```

**The "mother color" principle**: Mix a small amount of one hue into EVERY color in the palette. This creates harmony — colors feel like they belong together even when they're different hues. In code, pre-shift your palette:

```js
// Helper: tint a hex color toward a mother hue
function harmonize(hex, motherHex, amount) {
  let c = color(hex);
  let m = color(motherHex);
  return lerpColor(c, m, amount);
}

// Apply to palette — 8-15% tint is usually enough
let sky = harmonize("#6EA6D6", PALETTE.mother, 0.1);
let grass = harmonize("#7CB342", PALETTE.mother, 0.12);
let shadow = harmonize("#4A3B5C", PALETTE.mother, 0.08);
```

**Color echoes**: Repeat your accent color in unexpected places at low opacity to tie the composition together. If the focal area uses a warm red, let a hint of that red appear in distant shadows, in the sky near the horizon, in a foreground reflection.

**Limit your palette**: 3-5 main hues + 1 accent maximum. A skilled artist works within constraints. Use value and temperature shifts to create variety within the palette, not more hues.

### Edge Quality & Lost/Found Edges

**Edge variety is what separates skilled art from algorithm output.** Never make all edges the same quality.

**Three edge types:**

| Type | Visual Effect | p5.brush Translation | Where to Use |
|------|--------------|---------------------|-------------|
| **Hard / found** | Crisp, defined boundary | `fillBleed(0.03-0.08)` + optional stroke | Focal point, important structural lines |
| **Soft** | Gradual transition | `fillBleed(0.25-0.4)` + no stroke | Secondary areas, natural forms |
| **Lost** | Boundary dissolves into adjacent area | `fillBleed(0.5+)` + matching adjacent color at boundary | Background, atmospheric elements, where two similar values meet |

**Hard edge at focal (house against sky):**
```js
brush.fill("#D4A060", 190);
brush.fillBleed(0.05, "out");
brush.set("HB", "#725741", 0.9);  // add stroke for definition
brush.rect(houseX, houseY, w, h);
```

**Lost edge at periphery (distant mountain dissolving into sky):**
```js
// Use a color close to the adjacent sky color + high bleed
brush.noStroke();
brush.fill("#A8B4C0", 45);        // shifted toward sky color
brush.fillBleed(0.55, "out");     // high bleed = soft dissolving edge
brush.fillTexture(0.5, 0.4);
brush.polygon(distantMountain);
```

**Rule of thumb**: Hard edges pull forward, soft edges push back. Use this to reinforce depth — focal point has the hardest edges, background has the softest.

### Selective Rendering & Negative Space

**Not everything needs to be filled.** Visible paper showing through creates luminosity that no color can match. Plan at least 15-20% of your canvas as "breathing room" — areas of bare paper, minimal marks, or very faint washes.

**Rendering hierarchy:**
- **Focal area**: Full treatment — multi-pass washes, strokes, hatching, detail
- **Secondary areas**: Single wash + selective stroke OR hatch only (not both)
- **Peripheral areas**: Stroke-only suggestions, or faint single wash with no stroke
- **Background atmosphere**: Bare paper with scattered faint marks, or a single extremely transparent wash

**Leave shapes open**: An incomplete tree with two confident charcoal strokes and a loose canopy blob reads more artistically than a fully outlined, fully filled, fully textured one. Let the viewer's imagination complete it.

```js
// Fully rendered tree (stiff, algorithmic) — avoid
brush.fill("#2E7D32", 180);
brush.set("HB", "#1B5E20", 1);
brush.circle(x, y - 60, 40);  // filled circle canopy
brush.rect(x - 8, y - 20, 16, 40); // rectangle trunk

// Suggested tree (artistic, alive) — prefer
brush.noFill();
brush.set("charcoal", "#3E2723", 2);
brush.line(x, y, x - 2, y - 55);       // single trunk stroke
brush.fill("#2E7D32", 60);
brush.fillBleed(0.35, "out");
brush.noStroke();
brush.circle(x + 5, y - 50, 35, true); // irregular canopy, transparent
// No outline on canopy — let it dissolve
```

**Negative space is compositional**: The empty areas of paper are not "unfilled" — they're an active design element. A sky with NO wash (just warm paper) can be more evocative than one with four gradient layers.

---

## BRUSH TECHNIQUE GUIDE

### When to Use Each Brush

| Brush | Artistic Purpose | Best For |
|-------|-----------------|----------|
| `2B` | Rich value building, expressive gesture, soft edges | Shadows, underpainting, foreground texture, focal area darks |
| `HB` | Balanced structural lines, moderate texture | Architecture, defined edges at focal point, medium-weight outlines |
| `2H` | Delicate detail, distant textures, subtle marks | Background elements, atmospheric effects, fine cross-hatching, gentle accents |
| `cpencil` | Soft color layering, tonal work | Grass texture, foliage, gentle color transitions, organic flow lines |
| `charcoal` | Bold texture, dramatic darks, gestural energy | Strong value contrasts, expressive marks, foreground drama, tree trunks, rocks |
| `pen` | Clean definition, ink-wash style | Calligraphic outlines at focal point, fine detail, ink-drawing style pieces |
| `rotring` | Precise technical detail, clean hatching | Architectural detail, precise cross-hatching, technical illustration style |
| `marker` | Flat bold color, graphic shapes, solid dots | Color blocking, strong accents, fruit dots, bold foreground marks |
| `spray` | Atmospheric effects, spatter, granular texture | Rain, mist, speckled surfaces, breaking up flat areas, sand/gravel texture |

### Watercolor Techniques in p5.brush

**Wet-on-wet** — Overlapping fills while both have high bleed. The colors don't literally mix, but overlapping transparent shapes with similar bleed create the illusion of bleeding:
```js
brush.noStroke();
brush.fill("#D4785C", 50);
brush.fillBleed(0.45, "out");
brush.circle(300, 300, 80);

brush.fill("#E8A87C", 45);
brush.fillBleed(0.4, "out");
brush.circle(330, 310, 70);  // overlapping — colors interact
```

**Cauliflower / bloom effects** — Overlapping circles of varied sizes and slightly varied color create the organic blooming edges real watercolor makes:
```js
// Sky bloom — NOT horizontal rectangles
for (let i = 0; i < 8; i++) {
  let cx = random(width * 0.2, width * 0.8);
  let cy = random(50, 250);
  let r = random(80, 160);
  brush.fill(harmonize("#B5D8F7", PALETTE.mother, 0.1), random(30, 55));
  brush.fillBleed(random(0.3, 0.55), "out");
  brush.fillTexture(random(0.3, 0.5), random(0.2, 0.4));
  brush.circle(cx, cy, r);
}
```

**Glazing** — Multiple transparent layers of different hues over the same area to build color complexity:
```js
// Glaze a warm tint over a cool wash (creates depth in shadows)
brush.fill("#5B7D9A", 45);  // cool base
brush.fillBleed(0.25, "out");
brush.polygon(shape);

brush.fill("#D4785C", 20);  // warm glaze over
brush.fillBleed(0.3, "out");
brush.polygon(shape);        // same shape, different color = color mixing
```

### Dry Media Techniques

**Gestural strokes** — Confident, sweeping marks with pressure variation:
```js
brush.set("charcoal", "#3E2723", 2.5);
brush.spline([
  [100, 400],
  [250, 350, 1.5],   // high pressure at peak
  [400, 420, 0.4]    // trailing off
], 0.6);
```

**Cross-hatching for value** — Build tone through layered hatch directions:
```js
// Light value: single direction, wide spacing
brush.hatch(18, 45, { rand: 0.15 });
brush.hatchStyle("rotring", "#5A5A6E", 0.35);
brush.polygon(lightArea);

// Dark value: two directions, tight spacing
brush.hatch(8, 45, { rand: 0.1 });
brush.hatchStyle("rotring", "#3A3A4E", 0.5);
brush.polygon(darkArea);
brush.hatch(8, -30, { rand: 0.1 });
brush.polygon(darkArea);
```

**Scumbling** — Short random strokes over a wash for broken texture:
```js
brush.set("2B", "#5D4E3A", 0.6);
brush.wiggle(3);
for (let i = 0; i < 40; i++) {
  let x = randomGaussian(cx, 60);
  let y = randomGaussian(cy, 40);
  brush.line(x, y, x + random(-8, 8), y + random(-6, 6));
}
```

### Mixed Media Combinations

**Wash + selective pencil** (most natural-looking combination):
```js
// 1. Lay watercolor wash first
brush.noStroke();
brush.fill("#7CB342", 65);
brush.fillBleed(0.3, "out");
brush.polygon(hillShape);

// 2. Add pencil marks ON TOP — but only at focal areas
brush.noFill();
brush.set("cpencil", "#4A6B2E", 0.5);
for (let i = 0; i < 50; i++) {
  // Only add grass strokes near focal area
  let x = random(focalMinX, focalMaxX);
  let y = random(focalMinY, focalMaxY);
  brush.flowLine(x, y, random(15, 30), random(70, 110));
}
```

**Ink + wash** — Outline does NOT trace the fill exactly. Offsets and gaps create energy:
```js
// Wash first — slightly larger area
brush.noStroke();
brush.fill("#81D4FA", 65);
brush.fillBleed(0.2, "out");
brush.polygon(riverShape);

// Ink outline — offset, not tracing the fill
brush.noFill();
brush.set("pen", "#1A5276", 0.7);
// Only draw PARTS of the outline — leaving gaps
brush.spline(riverLeftBank.slice(2, 8), 0.6); // partial left bank only
```

---

## ANTI-PATTERNS

**These are the most common mistakes. Each makes the output look algorithmic.**

### 1. Regular vertex spacing
```js
// BAD: Vertices every ~160px
brush.vertex(0, 400); brush.vertex(160, 355); brush.vertex(320, 385);
brush.vertex(480, 340); brush.vertex(640, 375); brush.vertex(800, 335);

// GOOD: Noise-perturbed, irregular spacing
for (let i = 0; i <= 8; i++) {
  let x = (width / 8) * i + random(-40, 40);
  let y = 360 + noise(i * 0.4) * 80 - 40;
  brush.vertex(x, y);
}
```

### 2. Perfect geometric buildings
```js
// BAD: Perfect rectangle + triangle
brush.rect(660, 520, 200, 140);
brush.beginShape(0); brush.vertex(640, 520); brush.vertex(760, 435); brush.vertex(880, 520);

// GOOD: Slightly irregular, hand-drawn feel
brush.beginShape(0.12);
brush.vertex(658, 522); brush.vertex(662, 380); // walls not perfectly vertical
brush.vertex(860, 378); brush.vertex(862, 524);
brush.endShape(CLOSE);
```

### 3. Uniform fillBleed everywhere
```js
// BAD: Same bleed on everything
brush.fillBleed(0.2, "out"); // used on sky, mountains, house, river...

// GOOD: Vary by role
brush.fillBleed(0.05, "out"); // focal point — crisp
brush.fillBleed(0.25, "out"); // mid-ground — moderate
brush.fillBleed(0.5, "out");  // distant bg — atmospheric
```

### 4. Sky as horizontal rectangle bands
```js
// BAD: 4 stacked rects
brush.rect(0, 0, 1200, 280);   // blue band
brush.rect(0, 180, 1200, 200); // lavender band
brush.rect(0, 300, 1200, 150); // peach band

// GOOD: Overlapping organic blooms
for (let i = 0; i < 12; i++) {
  let y = random(0, 400);
  let t = y / 400;
  let col = lerpColor(color("#6EA6D6"), color("#F0B088"), t);
  brush.fill(col, random(25, 50));
  brush.fillBleed(random(0.3, 0.55), "out");
  brush.circle(random(width), y, random(100, 250));
}
```

### 5. Elements at uniform intervals
```js
// BAD: Trees every 80px
drawTree(100, 535); drawTree(180, 545); drawTree(260, 540);

// GOOD: Clustered with irregular gaps
// Main cluster with natural grouping
drawTree(90, 538);  drawTree(135, 550); drawTree(160, 535);
// Gap — breathing room
// Isolated counterpoint tree
drawTree(320, 542);
```

### 6. Everything fully rendered
```js
// BAD: Every element has fill + stroke + full detail

// GOOD: Vary rendering level
// Focal tree: full treatment
drawTreeFull(focalX, focalY);
// Secondary trees: wash + minimal stroke
drawTreeLoose(secondaryX, secondaryY);
// Distant trees: stroke-only suggestion
drawTreeSuggested(distantX, distantY);
```

### 7. No underpainting
```js
// BAD: Single-pass opaque fill
brush.fill("#5C4D7D", 180);
brush.polygon(shape);

// GOOD: Layered transparent passes (see Value Structure section)
```

### 8. Fence posts / elements at exact intervals
```js
// BAD
for (let x = 100; x < 500; x += 50) { brush.line(x, y1, x, y2); }

// GOOD: Base interval + organic jitter + occasional missing post
let x = 100;
while (x < 500) {
  if (random() > 0.1) { // occasionally skip a post
    brush.line(x + random(-3, 3), y1 + random(-2, 2), x + random(-2, 2), y2);
  }
  x += 50 + random(-12, 8);
}
```

### 9. Pure random scatter (no clustering)
```js
// BAD: Uniform random — looks like noise
for (let i = 0; i < 100; i++) {
  drawFlower(random(width), random(600, 800));
}

// GOOD: Gaussian clusters with natural grouping
let clusters = [[200, 720], [550, 750], [900, 710]]; // cluster centers
for (let [cx, cy] of clusters) {
  let n = floor(random(15, 35)); // vary count per cluster
  for (let i = 0; i < n; i++) {
    drawFlower(randomGaussian(cx, 45), randomGaussian(cy, 20));
  }
}
// A few scattered strays between clusters
for (let i = 0; i < 8; i++) {
  drawFlower(random(width), random(680, 800));
}
```

### 10. No color temperature variation
```js
// BAD: All greens are the same temperature
brush.fill("#7CB342", 80); // green hill
brush.fill("#6B9E4A", 70); // another green hill
brush.fill("#4E8C2A", 60); // yet another green hill

// GOOD: Cool green in shadow, warm green in light
brush.fill("#5B8A4E", 80); // cool blue-green — shadowed slope
brush.fill("#A4B844", 70); // warm yellow-green — sunlit slope
brush.fill("#7B9E52", 60); // neutral — transition area
```

---

## ARTIST'S CHECKLIST

**Run through this before outputting code. Every item should be satisfied:**

- [ ] **Focal point**: Identified with approximate canvas coordinates. One area, not everything.
- [ ] **Value structure**: Planned where darks (~20%), mids (~60%), and lights (~20%) go. Darkest meets lightest at focal point.
- [ ] **Color temperature**: Chosen warm/cool strategy. Shadows use the complement of the light, not gray.
- [ ] **Edge variety**: Planned which edges are hard (focal), soft (secondary), and lost (background).
- [ ] **Selective rendering**: Identified areas for full rendering, loose suggestion, and bare paper.
- [ ] **Palette harmony**: 3-5 hues + 1 accent, unified with a mother color tint.
- [ ] **Negative space**: At least 15% of canvas is breathing room (bare paper or faintest wash).
- [ ] **Irregular spacing**: All element positions use jitter/noise/clustering, not uniform intervals.
- [ ] **Brush variety**: Using 3+ brushes with distinct purposes (not just different colors on the same brush).
- [ ] **Variable bleed**: `fillBleed` ranges from tight (≤0.08) at focal to loose (≥0.4) at periphery.
- [ ] **Layered washes**: Focal area uses 2-3 transparent passes to build richness.
- [ ] **Composition framework**: The piece follows a recognizable layout (rule of thirds, S-curve, steelyard, etc.), not just back-to-front stacking.
- [ ] **Fill budget**: Total filled shapes ≤ 50. Multi-pass washes reserved for focal area.

---

## API REFERENCE

### Setup & Config

```js
brush.load()              // Initialize. Call in setup() after createCanvas(w,h,WEBGL)
brush.load(buffer)        // Target a p5.Graphics or p5.Framebuffer
brush.load()              // Switch back to main canvas
brush.scaleBrushes(scale) // Scale all default brush params (weight, scatter, spacing)
brush.instance(p)         // For p5 instance mode — call before setup/draw
```

### Vector Fields

Fields guide brush strokes along organic paths. Activate before drawing.

```js
brush.field("waves")       // Activate: "hand", "curved", "zigzag", "waves", "seabed", "spiral", "columns"
brush.noField()            // Deactivate
brush.wiggle(5)            // Shorthand: activate "hand" field with intensity (1-10)
brush.refreshField(t)      // Update field for animation (use frameCount/10 in draw())
brush.listFields()         // Returns array of available field names
```

Custom field:
```js
brush.addField("myField", function(t, field) {
  for (let col = 0; col < field.length; col++)
    for (let row = 0; row < field[0].length; row++)
      field[col][row] = 45 + noise(col * 0.1, row * 0.1) * 180;
  return field;
}, { angleMode: "degrees" }); // default is degrees
brush.field("myField");
```

### Brush Management

```js
brush.box()               // Returns array of all brush names
brush.pick("2B")           // Select brush without changing color/weight
```

**Built-in brushes:**

| Name | Type | Character |
|------|------|-----------|
| `2B` | pencil | Soft, dark, textured |
| `HB` | pencil | Medium, balanced |
| `2H` | pencil | Hard, light, precise |
| `cpencil` | pencil | Colored pencil, softer opacity |
| `pen` | pen | Ink pen, clean lines |
| `rotring` | pen | Technical pen, very precise |
| `charcoal` | pencil | Heavy texture, wide scatter |
| `spray` | spray | Scattered dots |
| `marker` | marker | Flat, solid strokes |
| `marker2` | marker | Marker variant |
| `hatch_brush` | hatch | Clean lines for hatching |

**Custom brush types:**

```js
// Standard custom brush
brush.add("myBrush", {
  type: "default",     // "default", "spray", "marker", "custom", "image"
  weight: 0.5,         // thickness (canvas units)
  scatter: 0.8,        // sideways wobble (canvas units)
  sharpness: 0.5,      // 0-1, edge softness (default type only)
  grain: 10,           // texture density (default type only)
  opacity: 150,        // 0-255
  spacing: 0.1,        // gap between stamps (1 = no overlap)
  pressure: [1, 0.5],  // [start,end] or [start,mid,end] or (t)=>value
  rotate: "natural",   // "none", "natural", "random"
});

// Custom tip brush — draw your own tip shape
brush.add("diamond", {
  type: "custom",
  weight: 5,
  opacity: 23,
  spacing: 0.6,
  pressure: [0.5, 1.5, 0.5],
  tip: (_m) => { _m.rotate(45); _m.rect(-1.5, -1.5, 3, 3); },
  rotate: "natural",
});

// Image brush — MUST use await + async setup()
await brush.add("watercolor", {
  type: "image",
  weight: 10,
  scatter: 2,
  opacity: 30,
  spacing: 1.5,
  pressure: [1, 0.5],
  image: { src: "./brush_tips/brush.jpg" },
  rotate: "random",
});
```

### Stroke

```js
brush.set("2B", "#333", 1)     // Set brush, color, weight — activates stroke
brush.stroke("#ff0000")         // Set color, activate stroke
brush.strokeWeight(2)           // Weight multiplier
brush.noStroke()                // Disable stroke
```

### Fill (Watercolor)

```js
brush.fill("#4a90d9", 150)           // Color + opacity (0-255), activate fill
brush.fill(244, 15, 24, 75)          // RGB + opacity
brush.fillBleed(0.5, "out")          // Edge bleed: strength 0-1, direction "out"/"in"
brush.fillTexture(0.5, 0.3)          // Texture: strength 0-1, border intensity 0-1
brush.noFill()                       // Disable fill
```

### Hatch

```js
brush.hatch(8, 45, { rand: 0.2, continuous: false, gradient: false })
// dist: line spacing, angle: follows angleMode(), options: { rand, continuous, gradient }
brush.hatchStyle("rotring", "#555", 0.5)  // Brush for hatch lines
brush.noHatch()
brush.hatchArray(polygons)                // Hatch across multiple Polygon objects at once
```

### Drawing Primitives

**Lines & curves** (stroke only):
```js
brush.line(x1, y1, x2, y2)
brush.flowLine(x, y, length, direction)    // Follows active vector field
brush.spline([[x1,y1], [x2,y2,pressure], ...], curvature)  // 0-1 curvature
brush.arc(x, y, radius, startAngle, endAngle)  // STROKE ONLY, follows angleMode()
```

**Shapes** (support stroke + fill + hatch):
```js
brush.rect(x, y, w, h)                     // Default: top-left corner mode
brush.rect(x, y, w, h, "center")           // Center mode
brush.circle(x, y, radius)                 // Supports stroke + fill + hatch
brush.circle(x, y, radius, true)           // With hand-drawn irregularity
brush.polygon([[x1,y1], [x2,y2], ...])     // Not affected by vector fields
```

**Custom shapes** (support stroke + fill + hatch, affected by fields):
```js
brush.beginShape(0.5)        // curvature 0-1
brush.vertex(x, y)           // or brush.vertex(x, y, pressure)
brush.vertex(x2, y2)
brush.endShape(true)          // true = close shape
```

**Manual stroke paths:**
```js
brush.beginStroke("curve", x, y)   // or "segments"
brush.move(angle, length, pressure)
brush.endStroke(angle, pressure)
```

### Clipping

```js
brush.clip([x1, y1, x2, y2])   // Clip strokes/hatches to rectangle (not fills)
brush.noClip()
```

### Classes

**Polygon:**
```js
let poly = new brush.Polygon([[x1,y1], [x2,y2], ...]);
poly.draw()                    // Draw outline with current stroke
poly.fill()                    // Fill with current fill settings
poly.hatch()                   // Hatch with current hatch settings
```

**Plot (reusable stroke paths):**
```js
let plot = new brush.Plot("curve");
plot.addSegment(angle, length, pressure);
plot.endPlot(finalAngle, finalPressure);
plot.draw(x, y);               // Draw at position
plot.fill(x, y);               // Fill at position
plot.hatch(x, y);              // Hatch at position
```

**Position (flow field walker):**
```js
let pos = new brush.Position(x, y);
pos.moveTo(direction, length, stepLength);  // Move along flow field
pos.angle();                   // Get field angle at current position
pos.reset();                   // Reset plotted distance
```

## RECIPES

### Hand-drawn look
```js
brush.wiggle(3);
brush.set("2B", "#333", 1);
brush.line(100, 100, 400, 300);
```

### Watercolor blobs
```js
brush.noStroke();
brush.fill("#4a90d9", 100);
brush.fillBleed(0.6, "out");
brush.fillTexture(0.5, 0.3);
brush.rect(200, 200, 300, 300);
```

### Cross-hatched shapes
```js
brush.noFill();
brush.hatch(5, 45, { rand: 0.1 });
brush.hatchStyle("rotring", "#444", 0.5);
brush.set("pen", "#222", 0.8);
brush.rect(100, 100, 300, 300);
// Layer second direction:
brush.hatch(5, -45, { rand: 0.1 });
brush.rect(100, 100, 300, 300);
```

### Flow field lines
```js
brush.field("curved");
brush.set("cpencil", "#8B4513", 0.8);
for (let i = 0; i < 200; i++) {
  brush.flowLine(random(width), random(height), random(100, 400), random(TWO_PI));
}
```

### Filled circle with watercolor
```js
brush.noStroke();
brush.fill("#e85d75", 120);
brush.fillBleed(0.4);
brush.circle(400, 400, 100);
```

### Mixed media grid (from author's examples)
```js
let palette = ["#7b4800", "#002185", "#003c32", "#fcd300", "#ff2702", "#6b9404"];
let cols = 12, rows = 6, border = 300;
let cw = (width - border) / cols, rh = (height - border) / rows;

brush.field("curved");
for (let i = 0; i < rows; i++) {
  for (let j = 0; j < cols; j++) {
    if (random() < 0.1) {
      brush.fill(random(palette), random(80, 140));
      brush.fillBleed(random(0.05, 0.4));
      brush.fillTexture(0.55, 0.5);
    } else {
      brush.set(random(["2H", "HB", "charcoal"]), random(palette));
      brush.hatchStyle(random(["marker", "marker2"]), random(palette));
      brush.hatch(random(10, 60), random(0, 180));
    }
    brush.rect(border/2 + cw * j, border/2 + rh * i, cw, rh);
    brush.noStroke(); brush.noFill(); brush.noHatch();
  }
}
```

## Design Tips

- **Paper colors**: warm `(245, 240, 235)`, cream `(252, 248, 240)`, cool white `(248, 248, 252)`, dark `(30, 30, 35)`
- **Layering**: Draw background elements first (light opacity), then foreground (higher opacity). Multiple passes build depth.
- **Organic variation**: Use `random()` and `noise()` for position, size, color shifts. Avoid exact repetition.
- **Color palettes**: Pick 3-5 colors. Vary saturation/brightness with `random()` for natural feel.
- **Composition**: Use rule of thirds, S-curve, or steelyard layouts. Leave breathing room.
- **Texture stacking**: Combine hatching + watercolor fill + pencil outlines for rich mixed-media effects. But NOT on every element — reserve for focal area.
- **Scale**: `brush.scaleBrushes(2)` for larger canvases (1200+px) so strokes stay visible.
- **p5 transforms work**: `push()`, `pop()`, `translate()`, `rotate()`, `scale()` — brush state is saved/restored automatically alongside p5's state.
- **Atmospheric perspective**: Distant objects → lighter, cooler, softer edges, less detail. Near objects → darker, warmer, harder edges, more detail.
- **Break symmetry**: If you place something on the left, balance it with something different on the right — not a mirror image.
- **Color echoes**: Repeat your accent color at low opacity in 2-3 unexpected places to unify the piece.
- **Value > color**: A piece with strong value contrast and weak color will always read better than a piece with beautiful color and flat values.
