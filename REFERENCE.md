# p5.brush — Art Reference Library

Companion to `SKILL.md`. Provides reusable palettes, brush recipes, composition templates, and advanced technique snippets. Read from this file selectively when you need depth — you don't need to internalize the whole file for every generation.

---

## Palette Recipes

Each palette includes: paper background, 3-5 main hues, 1 accent, shadow color, highlight treatment. The "mother" color is used to harmonize (tint all others 8-12% toward it).

### Golden Hour / Warm Evening
```js
const GOLDEN_HOUR = {
  paper: [252, 245, 232],
  main: ["#D4785C", "#E8A87C", "#F0C994", "#8B6F4E", "#5B7D9A"],
  accent: "#C23B22",
  shadow: "#4A3B5C",     // cool purple shadow (complement of warm light)
  highlight: null,        // leave as paper — warm paper IS the highlight
  mother: "#E8A87C",
  mood: "Warm, nostalgic, serene. Dominant warm temperature with cool shadow accents."
};
```

### Misty Morning / Overcast
```js
const MISTY_MORNING = {
  paper: [248, 248, 250],  // cool white paper
  main: ["#9BB5C9", "#B8C9D4", "#7A98A8", "#A8B8A0", "#C4B8A8"],
  accent: "#D4785C",       // warm accent against cool scene
  shadow: "#6B7D8A",       // cool blue-gray
  highlight: null,          // paper is the light
  mother: "#9BB5C9",
  mood: "Quiet, contemplative, high-key value range. Minimal contrast except at focal point."
};
```

### Stormy / Dramatic
```js
const STORMY = {
  paper: [235, 232, 228],
  main: ["#3A4A5C", "#5C6B7A", "#7A8B8A", "#4A5A4A", "#8A7A6A"],
  accent: "#E8C44A",       // isolated bright warm accent
  shadow: "#2A2A3A",       // near-black chromatic dark
  highlight: "#F0E8D4",    // rare — used sparingly for lightning/breaks in cloud
  mother: "#5C6B7A",
  mood: "Moody, low-key. 70% darks, narrow value range with rare bright accents."
};
```

### Spring Pastoral
```js
const SPRING = {
  paper: [252, 250, 242],
  main: ["#7CB87A", "#A4C88C", "#B8D4A0", "#E8D4A8", "#8AAFC4"],
  accent: "#E87CAA",       // pink blossom accent
  shadow: "#5A6B4A",       // warm green-gray
  highlight: null,
  mother: "#A4C88C",
  mood: "Fresh, cheerful, medium-high key. Yellow-greens dominate with pink/blue accents."
};
```

### Autumn Harvest
```js
const AUTUMN = {
  paper: [250, 244, 232],
  main: ["#B87A4A", "#D4A050", "#E8C878", "#8A6B3A", "#6B8A5A"],
  accent: "#C23B22",       // deep red leaf accent
  shadow: "#4A3B2A",       // warm dark brown
  highlight: "#F0E8C8",    // pale gold
  mother: "#D4A050",
  mood: "Warm earth tones, rich and grounded. Burnt sienna, raw umber, gold, olive."
};
```

### Nocturne / Moonlit
```js
const NOCTURNE = {
  paper: [30, 32, 38],     // dark paper
  main: ["#2A3A5C", "#3A5A7A", "#4A7A8A", "#1A2A3A", "#5A6A5A"],
  accent: "#E8C87A",       // warm moonlight/lamplight
  shadow: "#1A1A2A",       // deepest blue-black
  highlight: "#C4D4E8",    // cool silver-blue moonlight
  mother: "#2A3A5C",
  mood: "Deep blues, selective warm accents. 80% dark, 15% mid, 5% light."
};
```

### Mediterranean / Sun-bleached
```js
const MEDITERRANEAN = {
  paper: [252, 248, 240],
  main: ["#D4B88A", "#E8D4B0", "#7AA0B8", "#5A8AA0", "#B8A888"],
  accent: "#C87A4A",       // terracotta
  shadow: "#5A6B7A",       // cool blue-gray
  highlight: null,          // bleached paper = highlight
  mother: "#D4B88A",
  mood: "Faded warmth, bleached by sun. Warm whites, terracotta, faded blue, sandy tones."
};
```

### Ink Wash / Sumi-e
```js
const INK_WASH = {
  paper: [252, 250, 245],
  main: ["#2A2A2A", "#4A4A4A", "#7A7A7A", "#B0B0B0"], // pure value scale
  accent: "#C23B22",       // single red seal/accent (optional)
  shadow: "#1A1A1A",
  highlight: null,
  mother: null,             // no mother color — pure values
  mood: "Monochromatic, meditative. All expression through value, not hue."
};
```

---

## Brush Combination Recipes

### Expressive Watercolor Landscape
Primary medium for natural scenes with atmospheric depth.

```
Layers (bottom to top):
1. Sky:         noStroke + fill (circle blooms) — cpencil or 2H for texture accents
2. Distance:    noStroke + fill (high bleed, low opacity) — NO stroke outlines
3. Mid-ground:  noStroke + fill (medium bleed) + sparse cpencil flow lines
4. Focal area:  fill (low bleed) + HB stroke (selective) + multi-pass wash
5. Foreground:  charcoal gestural marks + cpencil detail + scattered marker accents
6. Finishing:   spray for atmospheric spatter, 2H for finest details at focal
```

### Ink and Wash
Bold outlines with loose watercolor fills. Outlines do NOT trace fills exactly.

```
Layers:
1. Light washes: fill only, high bleed, no stroke — lay down color broadly
2. Selective ink outlines: pen or rotring — only 30-40% of edges, gaps create energy
3. Value darks: 2B or charcoal for shadow areas — hatching or solid marks
4. Accents: marker for bold color dots/marks at focal point
```

### Charcoal Study
Monochromatic value study. No watercolor fills. All expression through mark-making.

```
Tools:
- charcoal: broad value areas, gestural sweeps, darkest darks
- 2B: mid-tone building, crosshatch, blending strokes
- 2H: lightest marks, fine detail, erased-highlight effect (light strokes over dark)
- Eraser effect: use paper-color strokes over dark areas for "lifted" highlights
```

### Botanical Illustration
Precise, delicate rendering with controlled color.

```
Layers:
1. Faint pencil outline: 2H at 0.3 weight — structural guide
2. Light wash layers: fill with low bleed (0.08-0.15) — build color gradually
3. Detail strokes: cpencil for veins, texture, gradation
4. Hatch shading: rotring for form-describing hatching on stems/leaves
5. Fine accents: 2H for stamens, edge highlights, tiny detail
```

### Abstract Expressionist
Bold, energetic, gesture-driven. Process over representation.

```
Tools:
- marker: large bold shapes, flat color blocks
- charcoal: sweeping gestural lines across the whole canvas
- spray: spatter texture, atmosphere
- fill: high bleed watercolor pools — large, overlapping
- flow fields: "curved" or custom fields for organic sweep patterns
```

---

## Composition Coordinate Templates

### Landscape with Focal Tree — 1200x800

```
Rule of thirds grid:
  Verticals: x = 400, 800
  Horizontals: y = 267, 533

Focal point: (800, 533) — right-lower third intersection
Eye path: horizon line at y ≈ 400 → leading line to focal → foreground texture

Major masses:
  Sky:        y = 0-400 (top half)
  Distance:   y = 300-500 (overlapping sky)
  Mid-ground: y = 400-650
  Foreground: y = 600-800

Focal tree at: (800, 533) — tallest element, most contrast, hardest edges
Counter-balance: smaller element at (200, 450) — secondary interest
```

### Seascape with Lighthouse — 1400x900

```
Steelyard composition:
  Heavy mass (sea + cliffs): left 60% of canvas
  Lighthouse (small, bright): right third, at (1050, 350)

Horizon: y ≈ 450 (slightly above center — more sea than sky)
Darkest darks: cliff face at (300, 400-600)
Lightest light: lighthouse + sky break at (1050, 300-400)
Leading line: wave foam curves from lower-left toward lighthouse
```

### Abstract Grid — 1200x1200

```
Cruciform composition:
  Strong vertical band: x = 450-550 (center column of grid)
  Strong horizontal band: y = 450-550 (center row)
  Focal cell: intersection (500, 500)

Grid irregularity: cells vary from 80-140px, not uniform
Fill density: 30% of cells filled, 70% stroke/hatch only or empty
Color: one warm column, one cool row — intersection = accent
```

---

## Advanced Technique Snippets

### Sky with Organic Bloom Effects
Replace horizontal rectangle bands with overlapping circles of varied size and color.

```js
function drawOrganicSky(palette, horizonY) {
  brush.noStroke();
  brush.noHatch();

  // Large background washes — fewest, biggest, faintest
  for (let i = 0; i < 6; i++) {
    let y = random(0, horizonY * 0.7);
    let t = y / horizonY;
    let col = lerpColor(color(palette.main[4]), color(palette.main[0]), t);
    brush.fill(col, random(20, 40));
    brush.fillBleed(random(0.4, 0.6), "out");
    brush.fillTexture(random(0.35, 0.5), random(0.25, 0.4));
    brush.circle(random(width * 0.1, width * 0.9), y, random(150, 300));
  }

  // Smaller accent blooms near horizon
  for (let i = 0; i < 4; i++) {
    let y = random(horizonY * 0.5, horizonY * 0.9);
    brush.fill(palette.main[2], random(25, 45));
    brush.fillBleed(random(0.3, 0.5), "out");
    brush.circle(random(width), y, random(60, 140));
  }
}
```

### Atmospheric Depth Gradient
Modulate color, contrast, and detail based on distance from foreground.

```js
// depth: 0 = far background, 1 = near foreground
function atmosphericParams(depth) {
  return {
    opacity: lerp(35, 180, depth),
    bleed: lerp(0.5, 0.08, depth),
    saturation: lerp(0.3, 1.0, depth),     // desaturate distance
    temperatureShift: lerp(0.25, 0, depth), // shift to cool at distance
    strokeWeight: lerp(0.2, 1.2, depth),
    detail: depth > 0.7 ? "full" : depth > 0.3 ? "moderate" : "minimal"
  };
}

// Desaturate a color toward a cool gray for atmospheric haze
function atmosphericColor(hexColor, depth) {
  let c = color(hexColor);
  let haze = color("#9AA8B4"); // blue-gray atmospheric haze
  return lerpColor(c, haze, (1 - depth) * 0.5);
}
```

### Water Reflections
Reflections are: inverted, shifted slightly, higher bleed, lower opacity, cooler color.

```js
function drawReflection(bankElements, waterSurface, waterBottom) {
  brush.noStroke();
  for (let elem of bankElements) {
    // Mirror position vertically around water surface
    let reflY = waterSurface + (waterSurface - elem.y);
    if (reflY > waterBottom) continue; // clip to water area

    // Reflected color: cooler, less saturated, more transparent
    let reflColor = atmosphericColor(elem.color, 0.3);
    brush.fill(reflColor, elem.opacity * 0.35);
    brush.fillBleed(elem.bleed + 0.2, "out"); // softer in reflection
    brush.fillTexture(0.4, 0.3);

    // Slightly distorted shape
    brush.circle(elem.x + random(-5, 5), reflY, elem.size * 0.9);
  }

  // Horizontal ripple lines over reflection
  brush.noFill();
  brush.set("2H", "#E1F5FE", 0.3);
  for (let y = waterSurface + 10; y < waterBottom; y += random(15, 30)) {
    let x1 = random(0, width * 0.3);
    let x2 = x1 + random(40, 120);
    brush.line(x1, y, x2, y + random(-2, 2));
  }
}
```

### Foliage with Natural Clustering
Poisson-like distribution: elements cluster but don't overlap too tightly.

```js
function clusterPositions(centers, countRange, spreadX, spreadY) {
  let positions = [];
  for (let [cx, cy] of centers) {
    let n = floor(random(countRange[0], countRange[1]));
    for (let i = 0; i < n; i++) {
      let x = randomGaussian(cx, spreadX);
      let y = randomGaussian(cy, spreadY);
      // Reject positions too close to existing ones
      let tooClose = positions.some(p => dist(x, y, p[0], p[1]) < 15);
      if (!tooClose) positions.push([x, y]);
    }
  }
  return positions;
}

// Usage: tree positions in natural groupings
let treeCenters = [[150, 540], [850, 530], [1100, 550]];
let treePositions = clusterPositions(treeCenters, [2, 5], 50, 15);
```

### Ground Plane with Perspective Depth
Grass strokes that get shorter, thinner, and sparser toward the horizon.

```js
function drawGrassField(horizonY, bottomY, focalX, focalY) {
  brush.field("curved");
  brush.noFill();
  brush.noHatch();

  for (let i = 0; i < 300; i++) {
    let y = random(horizonY, bottomY);
    let depth = (y - horizonY) / (bottomY - horizonY); // 0=horizon, 1=bottom

    // Skip more strokes at distance for natural thinning
    if (depth < 0.3 && random() > 0.4) continue;

    let x = random(0, width);
    let w = lerp(0.15, 0.6, depth);
    let len = lerp(8, 35, depth);

    // More detail near focal point
    let fDist = dist(x, y, focalX, focalY);
    if (fDist < 150) { w *= 1.3; len *= 1.2; }

    // Color: lighter/cooler at distance, darker/warmer near
    let col = depth > 0.5
      ? (random() > 0.5 ? "#4A7C2E" : "#3D6B25")
      : "#7BA060";

    brush.set("cpencil", col, w);
    brush.flowLine(x, y, len, random(70, 110));
  }
  brush.noField();
}
```

### Irregular Vertex Generation for Organic Shapes
Replace hardcoded, evenly-spaced vertices with noise-perturbed organic curves.

```js
// Generate organic edge vertices between two endpoints
function organicEdge(x1, y1, x2, y2, segments, amplitude, noiseScale) {
  let verts = [];
  for (let i = 0; i <= segments; i++) {
    let t = i / segments;
    let x = lerp(x1, x2, t) + noise(t * noiseScale, 0) * amplitude - amplitude / 2;
    let y = lerp(y1, y2, t) + noise(0, t * noiseScale) * amplitude - amplitude / 2;
    // Add micro-jitter for hand-drawn feel
    x += random(-amplitude * 0.1, amplitude * 0.1);
    y += random(-amplitude * 0.1, amplitude * 0.1);
    verts.push([x, y]);
  }
  return verts;
}

// Usage: mountain ridge with organic irregularity
let ridge = organicEdge(0, 400, 1200, 380, 10, 60, 3.5);
// Close the shape by adding bottom corners
ridge.push([1200, 600], [0, 600]);
brush.polygon(ridge);
```
