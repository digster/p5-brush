# p5.brush — Reusable Utility Patterns

Companion to `SKILL.md`. These are **code patterns to inline** into generated HTML files when needed. Since each output is a standalone HTML file, these are documented as copy-paste patterns, not an importable module.

Include only the utilities a specific sketch actually uses. Don't inline all of them.

---

## Spacing & Positioning

### irregularSpacing
Generate positions with natural irregularity instead of uniform intervals.

```js
// Returns array of x-positions with organic jitter
// base: starting x, end: ending x, count: approximate number, jitter: max offset
function irregularSpacing(base, end, count, jitter) {
  let positions = [];
  let step = (end - base) / count;
  for (let i = 0; i < count; i++) {
    let x = base + step * i + random(-jitter, jitter);
    // Occasionally skip a position for natural gaps
    if (random() > 0.08) positions.push(x);
  }
  return positions;
}
```

### clusterPositions
Gaussian-distributed positions around cluster centers. Produces natural grouping with rejection-based minimum spacing.

```js
// centers: [[cx, cy], ...], countRange: [min, max] per cluster
// spreadX, spreadY: standard deviation of gaussian distribution
// minDist: minimum distance between any two points
function clusterPositions(centers, countRange, spreadX, spreadY, minDist) {
  minDist = minDist || 12;
  let positions = [];
  for (let [cx, cy] of centers) {
    let n = floor(random(countRange[0], countRange[1]));
    for (let i = 0; i < n; i++) {
      let x = randomGaussian(cx, spreadX);
      let y = randomGaussian(cy, spreadY);
      let tooClose = positions.some(p => dist(x, y, p[0], p[1]) < minDist);
      if (!tooClose) positions.push([x, y]);
    }
  }
  // Add a few isolated strays between clusters
  for (let i = 0; i < floor(positions.length * 0.15); i++) {
    positions.push([random(0, width), random(0, height)]);
  }
  return positions;
}
```

---

## Focal-Aware Rendering

### renderParams
Returns rendering parameters modulated by distance from the focal point. Use this to automatically vary bleed, opacity, weight, and detail level across the canvas.

```js
// FOCAL should be defined at the top of your sketch
// const FOCAL = { x: 720, y: 480, r: 250 };

// Returns 0 at focal center, 1 at edge of influence, >1 beyond
function focalFade(x, y) {
  return constrain(dist(x, y, FOCAL.x, FOCAL.y) / FOCAL.r, 0, 1);
}

// Returns an object of rendering parameters
function renderParams(x, y) {
  let t = focalFade(x, y);
  return {
    opacity: lerp(200, 55, t),
    bleed: lerp(0.06, 0.5, t),
    texture: [lerp(0.15, 0.5, t), lerp(0.1, 0.4, t)],
    weight: lerp(1.2, 0.3, t),
    detail: t < 0.3 ? "full" : t < 0.6 ? "moderate" : "minimal"
  };
}

// Usage example:
// let rp = renderParams(treeX, treeY);
// brush.fill(treeColor, rp.opacity);
// brush.fillBleed(rp.bleed, "out");
// brush.fillTexture(...rp.texture);
// if (rp.detail === "full") { /* add hatching, extra passes */ }
// if (rp.detail === "minimal") { /* stroke only, no fill */ }
```

---

## Color & Palette

### harmonize
Tint a hex color toward a mother hue to unify the palette.

```js
// Tint hexColor toward motherHex by amount (0-1, typically 0.08-0.15)
function harmonize(hexColor, motherHex, amount) {
  let c = color(hexColor);
  let m = color(motherHex);
  return lerpColor(c, m, amount);
}

// Usage: apply to every color before use
// let skyColor = harmonize("#6EA6D6", PALETTE.mother, 0.1);
// let grassColor = harmonize("#7CB342", PALETTE.mother, 0.12);
```

### atmosphericColor
Shift a color toward a cool haze for atmospheric perspective. depth: 0 = far (most haze), 1 = near (no haze).

```js
function atmosphericColor(hexColor, depth) {
  let c = color(hexColor);
  let haze = color("#9AA8B4"); // blue-gray atmospheric haze
  let hazeAmount = (1 - depth) * 0.45;
  return lerpColor(c, haze, hazeAmount);
}

// Usage for layered mountain ranges:
// let farMountain = atmosphericColor("#5C7D4E", 0.15);  // very hazy
// let midMountain = atmosphericColor("#5C7D4E", 0.5);   // moderate haze
// let nearMountain = atmosphericColor("#5C7D4E", 0.9);  // nearly original color
```

### chromaticShadow
Generate a chromatic shadow color from a base color. Uses complementary hue shift rather than just darkening.

```js
// warmLight: true for warm light scenes (shadow shifts cool)
// warmLight: false for cool light scenes (shadow shifts warm)
function chromaticShadow(hexColor, warmLight) {
  let c = color(hexColor);
  let r = red(c), g = green(c), b = blue(c);

  if (warmLight) {
    // Cool shadow: shift toward purple/blue
    return color(r * 0.5, g * 0.4, b * 0.6 + 40);
  } else {
    // Warm shadow: shift toward brown/sienna
    return color(r * 0.6 + 30, g * 0.4, b * 0.3);
  }
}
```

---

## Shape Generation

### organicVertices
Generate noise-perturbed vertices for organic shapes. Replaces hardcoded, evenly-spaced coordinate arrays.

```js
// Generate an organic edge between two points
// segments: vertex count, amplitude: max deviation, nScale: noise smoothness
function organicVertices(x1, y1, x2, y2, segments, amplitude, nScale) {
  nScale = nScale || 3.5;
  let verts = [];
  for (let i = 0; i <= segments; i++) {
    let t = i / segments;
    let x = lerp(x1, x2, t);
    let y = lerp(y1, y2, t);
    // Perpendicular noise offset
    x += (noise(t * nScale, 0) - 0.5) * amplitude;
    y += (noise(0, t * nScale) - 0.5) * amplitude;
    // Micro-jitter for hand-drawn quality
    x += random(-amplitude * 0.08, amplitude * 0.08);
    y += random(-amplitude * 0.08, amplitude * 0.08);
    verts.push([x, y]);
  }
  return verts;
}

// Usage: mountain ridge
// let ridge = organicVertices(0, 380, 1200, 390, 10, 80, 4);
// ridge.push([1200, 600], [0, 600]); // close shape at bottom
// brush.polygon(ridge);
```

### closedOrganicShape
Generate a complete closed organic shape (blob) around a center point.

```js
function closedOrganicShape(cx, cy, radiusX, radiusY, points, wobble) {
  let verts = [];
  for (let i = 0; i < points; i++) {
    let angle = (TWO_PI / points) * i;
    let r = 1 + (noise(i * 0.5, cx * 0.01) - 0.5) * wobble;
    let x = cx + cos(angle) * radiusX * r;
    let y = cy + sin(angle) * radiusY * r;
    verts.push([x, y]);
  }
  return verts;
}

// Usage: organic cloud shape
// let cloudShape = closedOrganicShape(400, 120, 80, 35, 8, 0.4);
// brush.polygon(cloudShape);
```

---

## Layered Rendering

### layeredWash
Apply multiple transparent wash layers to build color richness. Most impactful at the focal area.

```js
// shape: vertex array for brush.polygon()
// layers: array of { color, opacity, bleed, inset }
//   inset: 0 = same shape, >0 = contract vertices toward center
function layeredWash(shape, layers) {
  brush.noStroke();
  brush.noHatch();

  // Calculate centroid for inset shrinking
  let cx = 0, cy = 0;
  for (let [x, y] of shape) { cx += x; cy += y; }
  cx /= shape.length; cy /= shape.length;

  for (let layer of layers) {
    brush.fill(layer.color, layer.opacity);
    brush.fillBleed(layer.bleed || 0.25, "out");
    brush.fillTexture(layer.texture || 0.35, layer.textureBorder || 0.25);

    if (layer.inset && layer.inset > 0) {
      // Shrink shape toward centroid
      let insetShape = shape.map(([x, y]) => [
        lerp(x, cx, layer.inset),
        lerp(y, cy, layer.inset)
      ]);
      brush.polygon(insetShape);
    } else {
      brush.polygon(shape);
    }
  }
}

// Usage: rich mountain with 3 passes
// layeredWash(mountainShape, [
//   { color: "#7B6BA0", opacity: 45, bleed: 0.35 },              // light base
//   { color: "#5C4D7D", opacity: 35, bleed: 0.2, inset: 0.1 },  // darker mid
//   { color: "#3A2D5C", opacity: 25, bleed: 0.1, inset: 0.25 }  // darkest accent
// ]);
```

### tieredDetail
Draw an element at different detail levels depending on distance from focal point.

```js
// Call one of these based on renderParams().detail
function drawElementFull(x, y, size, colors) {
  // Multi-pass wash + stroke + detail marks
  // ... full implementation
}
function drawElementModerate(x, y, size, colors) {
  // Single wash + selective stroke, no hatching
  // ... moderate implementation
}
function drawElementMinimal(x, y, size, colors) {
  // Stroke-only suggestion, no fill
  // ... minimal implementation
}

// Usage:
// let rp = renderParams(x, y);
// if (rp.detail === "full") drawElementFull(x, y, sz, colors);
// else if (rp.detail === "moderate") drawElementModerate(x, y, sz, colors);
// else drawElementMinimal(x, y, sz, colors);
```

---

## Texture & Effects

### paperGrain
Add subtle paper grain texture. Use as a final overlay pass.

```js
function paperGrain(density, grainColor, grainAlpha) {
  density = density || 1500;
  grainColor = grainColor || [97, 78, 59];
  grainAlpha = grainAlpha || 10;
  stroke(...grainColor, grainAlpha);
  for (let i = 0; i < density; i++) {
    point(random(width), random(height));
  }
  noStroke();
}
```

### colorEchoes
Repeat a color at low opacity in scattered positions to unify the composition.

```js
function colorEchoes(echoColor, count, opacity, sizeRange) {
  brush.noStroke();
  brush.noHatch();
  brush.fill(echoColor, opacity);
  brush.fillBleed(random(0.3, 0.5), "out");
  brush.fillTexture(0.4, 0.3);
  for (let i = 0; i < count; i++) {
    let x = random(width * 0.1, width * 0.9);
    let y = random(height * 0.1, height * 0.9);
    let r = random(sizeRange[0], sizeRange[1]);
    brush.circle(x, y, r);
  }
  brush.noFill();
}

// Usage: echo the accent color across the composition
// colorEchoes(PALETTE.accent, 4, 15, [30, 80]);
```
