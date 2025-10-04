# Geometry System Documentation

## Overview

The Geometry System provides an interactive canvas for exploring 2D and isometric 3D geometric shapes. Users can manipulate shapes, view live measurements, and learn geometric concepts through hands-on interaction.

## Supported Shapes

### 2D Shapes
- **Triangle**: Interactive vertices with perimeter and area calculations
- **Rectangle**: Corner manipulation with area and perimeter display
- **Circle**: Center and radius adjustment with circumference and area

### 3D Shapes (Isometric)
- **Cube**: Isometric projection with volume and surface area calculations

## Core Data Structures

### Geometry State
```javascript
const geomState = {
  shape: 'triangle',           // Current shape type
  zoom: 1,                     // Zoom level (0.4-5.0)
  offset: { x: 0, y: 0 },      // Pan offset
  dragging: null,              // Current drag operation
  vertices: [                  // Triangle vertices
    {x: -80, y: 60},
    {x: 0, y: -60},
    {x: 80, y: 60}
  ],
  rect: {                      // Rectangle bounds
    x: -100, y: -60, w: 200, h: 120
  },
  circle: {                    // Circle properties
    x: 0, y: 0, r: 80
  },
  cube: {                      // Cube properties
    center: {x: 0, y: 0},
    size: 120,
    rot: 0.3                   // Rotation angle for isometric projection
  }
};
```

## Canvas Rendering

### Coordinate System
- **World Coordinates**: Mathematical coordinates for shape definitions
- **Screen Coordinates**: Pixel coordinates for canvas rendering
- **Conversion Functions**:
  - `worldToScreen(point)`: Convert world → screen coordinates
  - `screenToWorld(point)`: Convert screen → world coordinates

### Rendering Pipeline
```javascript
function drawGeometry() {
  // Clear canvas
  gctx.clearRect(0, 0, width, height);

  // Draw coordinate axes
  drawAxes();

  // Draw current shape based on type
  switch (geomState.shape) {
    case 'triangle': drawTriangle(); break;
    case 'rectangle': drawRectangle(); break;
    case 'circle': drawCircle(); break;
    case 'cube': drawCube(); break;
  }

  // Update measurements display
  updateGeomProps();
}
```

## Shape Rendering Functions

### Triangle Rendering
```javascript
function drawTriangle() {
  const pts = geomState.vertices.map(worldToScreen);

  // Draw triangle fill and stroke
  gctx.fillStyle = 'rgba(125,211,252,0.25)';
  gctx.strokeStyle = 'rgba(125,211,252,0.9)';
  gctx.lineWidth = 2;
  gctx.fill(); gctx.stroke();

  // Draw draggable handles at vertices
  pts.forEach(drawHandle);
}
```

### Rectangle Rendering
```javascript
function drawRectangle() {
  const r = geomState.rect;
  const p = worldToScreen({x: r.x, y: r.y});
  const size = { w: r.w * geomState.zoom, h: r.h * geomState.zoom };

  gctx.fillStyle = 'rgba(96,165,250,0.25)';
  gctx.strokeStyle = 'rgba(96,165,250,0.9)';
  gctx.fillRect(p.x, p.y, size.w, size.h);
  gctx.strokeRect(p.x, p.y, size.w, size.h);

  // Draw handles at corners
  const corners = [
    {x: r.x, y: r.y},
    {x: r.x + r.w, y: r.y},
    {x: r.x + r.w, y: r.y + r.h},
    {x: r.x, y: r.y + r.h}
  ].map(worldToScreen).forEach(drawHandle);
}
```

### Circle Rendering
```javascript
function drawCircle() {
  const c = worldToScreen(geomState.circle);
  const r = geomState.circle.r * geomState.zoom;

  gctx.fillStyle = 'rgba(52,211,153,0.25)';
  gctx.strokeStyle = 'rgba(52,211,153,0.9)';
  gctx.beginPath();
  gctx.arc(c.x, c.y, r, 0, Math.PI * 2);
  gctx.fill(); gctx.stroke();

  // Draw handles at center and edge
  drawHandle({x: c.x + r, y: c.y}); // Radius handle
  drawHandle(c); // Center handle
}
```

### Isometric Cube Rendering
```javascript
function drawCube() {
  const { center, size, rot } = geomState.cube;

  // Isometric projection transformation
  const iso = (x, y, z) => {
    const a = Math.cos(rot), b = Math.sin(rot);
    const ix = (x - z) * a;
    const iy = y + (x + z) * b * 0.5;
    return worldToScreen({x: center.x + ix, y: center.y + iy});
  };

  const s = size / 2;
  const verts = [
    [-s, -s, -s], [s, -s, -s], [s, -s, s], [-s, -s, s],
    [-s, s, -s], [s, s, -s], [s, s, s], [-s, s, s]
  ].map(v => iso(v[0], v[1], v[2]));

  // Draw cube edges
  const edges = [
    [0,1],[1,2],[2,3],[3,0], // Bottom face
    [4,5],[5,6],[6,7],[7,4], // Top face
    [0,4],[1,5],[2,6],[3,7]  // Vertical edges
  ];

  gctx.strokeStyle = 'rgba(245,158,11,0.9)';
  gctx.lineWidth = 2;
  edges.forEach(([i,j]) => {
    gctx.beginPath();
    gctx.moveTo(verts[i].x, verts[i].y);
    gctx.lineTo(verts[j].x, verts[j].y);
    gctx.stroke();
  });

  // Draw center handle
  drawHandle(worldToScreen(center));
}
```

## Measurement Calculations

### `updateGeomProps() → void`
Calculates and displays live measurements for the current shape.

**Triangle Calculations:**
```javascript
const [A, B, C] = geomState.vertices;
const side = (P, Q) => Math.hypot(P.x - Q.x, P.y - Q.y);
const a = side(B, C), b = side(A, C), c = side(A, B);
const s = (a + b + c) / 2;
const area = Math.sqrt(Math.max(0, s * (s - a) * (s - b) * (s - c)));
// Display: `Perimeter = ${perimeter} units, Area = ${area} sq units`
```

**Rectangle Calculations:**
```javascript
const r = geomState.rect;
const area = Math.abs(r.w * r.h);
const perimeter = 2 * (Math.abs(r.w) + Math.abs(r.h));
// Display: `Perimeter = ${perimeter}, Area = ${area}`
```

**Circle Calculations:**
```javascript
const c = geomState.circle;
const area = Math.PI * c.r * c.r;
const circumference = 2 * Math.PI * c.r;
// Display: `Circumference = ${circumference}, Area = ${area}`
```

**Cube Calculations:**
```javascript
const s = geomState.cube.size;
const volume = Math.pow(s, 3);
const surfaceArea = 6 * s * s;
// Display: `Edge = ${s}, Surface Area = ${surfaceArea}, Volume = ${volume}`
```

## Interaction System

### Drag Operations
The system supports multiple types of drag interactions:

#### Vertex Dragging (Triangle)
```javascript
function geomPointerDown(e) {
  const hit = pickHandleAt(e.clientX, e.clientY);
  if (hit.type === 'vertex') {
    geomState.dragging = { type: 'vertex', index: hit.index, start: {x, y} };
  }
}
```

#### Corner Dragging (Rectangle)
```javascript
function geomPointerMove(e) {
  if (geomState.dragging.type === 'rectCorner') {
    const idx = geomState.dragging.index;
    const corners = [/* rectangle corners */];
    const moved = screenToWorld({x: worldToScreen(corners[idx]).x + dx, y: /* ... */});

    // Adjust rectangle bounds based on dragged corner
    const opposite = corners[(idx + 2) % 4];
    geomState.rect.x = Math.min(moved.x, opposite.x);
    geomState.rect.y = Math.min(moved.y, opposite.y);
    geomState.rect.w = Math.abs(moved.x - opposite.x);
    geomState.rect.h = Math.abs(moved.y - opposite.y);
  }
}
```

#### Center and Radius Dragging (Circle)
```javascript
if (geomState.dragging.type === 'circleCenter') {
  geomState.circle.x = newCenter.x;
  geomState.circle.y = newCenter.y;
} else if (geomState.dragging.type === 'circleEdge') {
  geomState.circle.r = Math.max(5, newRadius);
}
```

#### Cube Center Dragging
```javascript
if (geomState.dragging.type === 'cubeCenter') {
  geomState.cube.center = screenToWorld({x: center.x + dx, y: center.y + dy});
}
```

### Hit Detection
```javascript
function pickHandleAt(x, y) {
  const pt = { x, y };
  const hit = (p) => Math.hypot(p.x - pt.x, p.y - pt.y) <= 8;

  // Check different shape types
  if (geomState.shape === 'triangle') {
    const pts = geomState.vertices.map(worldToScreen);
    for (let i = 0; i < pts.length; i++) {
      if (hit(pts[i])) return { type: 'vertex', index: i };
    }
  }
  // Similar checks for other shapes...

  return { type: 'shape' }; // Pan the entire shape
}
```

## Zoom and Pan Controls

### Zoom Controls
```javascript
geomCanvas.addEventListener('wheel', (e) => {
  e.preventDefault();
  const delta = Math.sign(e.deltaY);
  const factor = delta > 0 ? 0.9 : 1.1;
  geomState.zoom = clamp(geomState.zoom * factor, 0.4, 5);
  drawGeometry();
}, { passive: false });
```

### Pan Controls
```javascript
function geomPointerMove(e) {
  if (geomState.dragging.type === 'shape') {
    // Pan the entire shape
    geomState.offset.x += dx;
    geomState.offset.y += dy;
  }
}
```

### Reset Function
```javascript
resetGeometryBtn.addEventListener('click', () => {
  geomState.zoom = 1;
  geomState.offset = { x: 0, y: 0 };
  // Reset all shape data to defaults
  drawGeometry();
});
```

## Shape Selection

### Shape Dropdown
```javascript
shapeSelect.addEventListener('change', (e) => {
  geomState.shape = e.target.value;
  drawGeometry();
});
```

### Available Shapes
- `'triangle'`: Interactive triangle with draggable vertices
- `'rectangle'`: Rectangle with draggable corners
- `'circle'`: Circle with draggable center and radius
- `'cube'`: Isometric cube with draggable center

## Touch Support

The geometry canvas includes full touch support for mobile devices:

```javascript
// Touch event handlers
geomCanvas.addEventListener('touchstart', handleTouchStart, { passive: false });
geomCanvas.addEventListener('touchmove', handleTouchMove, { passive: false });
geomCanvas.addEventListener('touchend', handleTouchEnd, { passive: false });
```

## Responsive Canvas

The canvas automatically resizes with its container:

```javascript
function resizeGeom() {
  const rect = geomWrap.getBoundingClientRect();
  geomCanvas.width = rect.width * devicePixelRatio;
  geomCanvas.height = rect.height * devicePixelRatio;
  gctx.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
  drawGeometry();
}

new ResizeObserver(resizeGeom).observe(geomWrap);
```

## Integration Examples

### Custom Shape Addition
```javascript
// Add a new shape type
function drawPentagon() {
  // Custom pentagon rendering logic
  const center = worldToScreen(geomState.pentagon.center);
  const radius = geomState.pentagon.radius * geomState.zoom;

  gctx.beginPath();
  for (let i = 0; i < 5; i++) {
    const angle = (i * 2 * Math.PI) / 5 - Math.PI / 2;
    const x = center.x + radius * Math.cos(angle);
    const y = center.y + radius * Math.sin(angle);
    if (i === 0) gctx.moveTo(x, y);
    else gctx.lineTo(x, y);
  }
  gctx.closePath();
  gctx.fill(); gctx.stroke();

  // Draw vertex handles
  for (let i = 0; i < 5; i++) {
    const angle = (i * 2 * Math.PI) / 5 - Math.PI / 2;
    const handle = {
      x: center.x + radius * Math.cos(angle),
      y: center.y + radius * Math.sin(angle)
    };
    drawHandle(handle);
  }
}
```

### Measurement Export
```javascript
// Export current measurements
function exportMeasurements() {
  const measurements = {};
  if (geomState.shape === 'triangle') {
    const [A, B, C] = geomState.vertices;
    measurements.sides = [distance(A,B), distance(B,C), distance(C,A)];
    measurements.area = calculateTriangleArea(A, B, C);
  }
  // ... other shapes

  return measurements;
}
```

### Animation Integration
```javascript
// Add animation to geometry
function animateGeometry() {
  if (geomState.shape === 'cube') {
    geomState.cube.rot += 0.02;
    drawGeometry();
    requestAnimationFrame(animateGeometry);
  }
}
```

## Performance Considerations

- **Efficient Rendering**: Only re-renders when state changes
- **Handle Caching**: Reuses handle objects when possible
- **Coordinate Conversion**: Optimized world ↔ screen transformations
- **Responsive Updates**: Automatic canvas resizing and redrawing

This geometry system provides a comprehensive interactive environment for exploring geometric concepts with real-time feedback and measurements.