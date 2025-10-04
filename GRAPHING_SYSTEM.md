# Graphing System Documentation

## Overview

The Graphing System provides a powerful mathematical function visualizer with an integrated expression parser, interactive canvas, and educational features including intercept highlighting and step-by-step solution methods.

## Core Features

- **Expression Parser**: Shunting-yard algorithm supporting complex mathematical expressions
- **Interactive Canvas**: Pan, zoom, and touch support
- **Educational Tools**: Intercept detection, symbolic solving, and Newton method visualization
- **Responsive Design**: Automatic scaling and high-DPI support

## Expression Parser

### Supported Operations

#### Arithmetic Operators
- `+` (Addition)
- `-` (Subtraction)
- `*` (Multiplication)
- `/` (Division)
- `^` (Exponentiation)

#### Mathematical Functions
- `sin(x)`, `cos(x)`, `tan(x)` - Trigonometric functions
- `asin(x)`, `acos(x)`, `atan(x)` - Inverse trigonometric functions
- `sqrt(x)` - Square root
- `abs(x)` - Absolute value
- `ln(x)` - Natural logarithm
- `log(x)` - Base-10 logarithm
- `exp(x)` - Exponential function (e^x)

#### Constants
- `pi` - π (3.14159...)
- `e` - Euler's number (2.71828...)

#### Variables
- `x` - Independent variable

### Parser Architecture

#### `tokenize(expr: string) → string[]`
Converts expression string into tokens, handling implicit multiplication.

**Examples:**
```javascript
tokenize("2x + 3");           // ["2", "x", "+", "3"]
tokenize("2(x+1)");           // ["2", "(", "x", "+", "1", ")"]
tokenize("sin(x) + x^2");      // ["sin", "(", "x", ")", "+", "x", "^", "2"]
```

#### `toRPN(tokens: string[]) → string[]`
Converts infix tokens to Reverse Polish Notation using the shunting-yard algorithm.

**Algorithm Features:**
- Proper operator precedence handling
- Left and right associativity support
- Parentheses and function call parsing
- Unary minus detection

#### `compile(expr: string) → string[]|null`
Complete compilation pipeline from expression string to RPN tokens.

**Error Handling:**
- Returns `null` for invalid expressions
- Catches parsing errors and logs warnings

#### `evalRPN(rpn: string[], x: number) → number`
Evaluates RPN expression at a specific x value.

**Stack-based Evaluation:**
```javascript
function evalRPN(rpn, x) {
  const stack = [];
  for (const token of rpn) {
    if (/^[0-9]*\.?[0-9]+$/.test(token)) stack.push(parseFloat(token));
    else if (token === 'x') stack.push(x);
    else if (token in CONSTANTS) stack.push(CONSTANTS[token]);
    else if (token === 'u-') stack.push(-stack.pop());
    else if (token in FUNCTIONS) {
      const fn = FUNCTIONS[token];
      stack.push(fn(stack.pop()));
    }
    // Handle binary operators...
  }
  return stack.pop();
}
```

#### `safeEval(rpn: string[], x: number) → number`
Safe wrapper around `evalRPN` with error handling.

```javascript
function safeEval(rpn, x) {
  try { return evalRPN(rpn, x); } catch { return NaN; }
}
```

## Canvas Rendering

### Graph State Management
```javascript
const graphState = {
  expr: 'sin(x)',              // Current expression string
  compiled: null,              // Compiled RPN tokens
  zoom: 60,                    // Pixels per unit
  origin: { x: 0, y: 0 },      // Pan offset in pixels
  dragging: false,             // Drag state
  dragStart: { x: 0, y: 0 },   // Drag start position
  lastPan: { x: 0, y: 0 },    // Last pan position
  intercepts: { x: [], y: 0 }, // Detected intercepts
  stepsHtml: ''                // Solution steps HTML
};
```

### Rendering Pipeline
```javascript
function drawGraph() {
  // Clear canvas
  g2.clearRect(0, 0, width, height);

  // Setup coordinate transformation
  g2.save();
  g2.translate(width/2 + graphState.origin.x, height/2 + graphState.origin.y);

  // Draw grid and axes
  drawGrid();
  drawAxes();
  drawAxisLabels();

  // Plot function if compiled
  if (graphState.compiled) {
    plotFunction();
    highlightIntercepts();
  }

  g2.restore();
}
```

### Grid System
```javascript
function drawGrid(ctx, width, height, scale, step) {
  // Calculate grid line positions
  const startX = Math.floor((left/scale) / step) * step;

  // Draw vertical grid lines
  for (let x = startX; x < (right/scale); x += step) {
    ctx.beginPath();
    ctx.moveTo(x * scale, -height);
    ctx.lineTo(x * scale, height);
    ctx.stroke();
  }

  // Draw horizontal grid lines (similar logic)
}
```

### Function Plotting
```javascript
function plotFunction() {
  g2.strokeStyle = '#7dd3fc';
  g2.lineWidth = 2;
  g2.beginPath();

  const sampleStep = 1 / graphState.zoom;
  let first = true;

  for (let x = minX; x <= maxX; x += sampleStep) {
    const y = safeEval(graphState.compiled, x);
    if (!isFinite(y)) { first = true; continue; }

    const px = x * graphState.zoom;
    const py = -y * graphState.zoom; // Flip Y axis

    if (first) { g2.moveTo(px, py); first = false; }
    else g2.lineTo(px, py);
  }

  g2.stroke();
}
```

## Interaction System

### Mouse/Touch Controls

#### Panning
```javascript
graphCanvas.addEventListener('mousedown', (e) => {
  graphState.dragging = true;
  graphState.dragStart = { x: e.clientX, y: e.clientY };
  graphState.lastPan = { ...graphState.origin };
});

window.addEventListener('mousemove', (e) => {
  if (!graphState.dragging) return;

  const dx = e.clientX - graphState.dragStart.x;
  const dy = e.clientY - graphState.dragStart.y;

  graphState.origin.x = graphState.lastPan.x + dx;
  graphState.origin.y = graphState.lastPan.y + dy;

  drawGraph();
});
```

#### Zooming
```javascript
graphCanvas.addEventListener('wheel', (e) => {
  e.preventDefault();
  const delta = Math.sign(e.deltaY);
  const factor = delta > 0 ? 0.9 : 1.1;

  // Zoom towards mouse position
  const mouse = {
    x: e.offsetX - graphCanvas.width/devicePixelRatio/2,
    y: e.offsetY - graphCanvas.height/devicePixelRatio/2
  };

  const before = {
    x: (mouse.x - graphState.origin.x) / graphState.zoom,
    y: (mouse.y - graphState.origin.y) / graphState.zoom
  };

  graphState.zoom = clamp(graphState.zoom * factor, 20, 400);

  const after = {
    x: (mouse.x - graphState.origin.x) / graphState.zoom,
    y: (mouse.y - graphState.origin.y) / graphState.zoom
  };

  graphState.origin.x += (after.x - before.x) * graphState.zoom;
  graphState.origin.y += (after.y - before.y) * graphState.zoom;

  graphState.intercepts = findIntercepts(graphState.compiled);
  drawGraph();
});
```

#### Touch Support
```javascript
// Single finger pan
graphCanvas.addEventListener('touchstart', (e) => {
  if (e.touches.length === 1) {
    const t = e.touches[0];
    graphState.dragging = true;
    graphState.dragStart = { x: t.clientX, y: t.clientY };
    graphState.lastPan = { ...graphState.origin };
  }
  e.preventDefault();
});

// Pinch to zoom
graphCanvas.addEventListener('touchmove', (e) => {
  if (e.touches.length === 2) {
    const [a, b] = e.touches;
    const dist = Math.hypot(a.clientX - b.clientX, a.clientY - b.clientY);

    if (!graphState._pinchStart) {
      graphState._pinchStart = dist;
      graphState._zoomStart = graphState.zoom;
    } else {
      const factor = dist / graphState._pinchStart;
      graphState.zoom = clamp(graphState._zoomStart * factor, 20, 400);
      graphState.intercepts = findIntercepts(graphState.compiled);
      drawGraph();
    }
  }
  e.preventDefault();
});
```

## Mathematical Analysis

### Intercept Detection
```javascript
function findIntercepts(rpn) {
  const xs = [];
  const minX = -width/2/zoom - origin.x/zoom - 10;
  const maxX = width/2/zoom - origin.x/zoom + 10;
  const step = (maxX - minX) / 200;

  let prevX = minX;
  let prevY = safeEval(rpn, prevX);

  for (let x = minX + step; x <= maxX; x += step) {
    const y = safeEval(rpn, x);
    if (isFinite(prevY) && isFinite(y) && prevY * y <= 0) {
      const root = refineRootBisection(rpn, prevX, x, 20);
      if (isFinite(root)) xs.push(root);
    }
    prevX = x; prevY = y;
  }

  const y0 = safeEval(rpn, 0);
  return { x: dedupeSorted(xs), y: y0 };
}
```

### Root Refinement
```javascript
function refineRootBisection(rpn, a, b, iterations = 20) {
  let fa = safeEval(rpn, a), fb = safeEval(rpn, b);

  if (!isFinite(fa) || !isFinite(fb) || fa * fb > 0) return NaN;

  for (let i = 0; i < iterations; i++) {
    const m = (a + b) / 2;
    const fm = safeEval(rpn, m);

    if (Math.abs(fm) < 1e-6) return m;
    if (fa * fm <= 0) { b = m; fb = fm; }
    else { a = m; fa = fm; }
  }

  return (a + b) / 2;
}
```

### Symbolic Solving
```javascript
function trySymbolicSteps(expr, rpn) {
  const f = (x) => safeEval(rpn, x);
  const f0 = f(0), f1 = f(1), f_1 = f(-1), f2 = f(2);

  // Linear detection
  if ([f0, f1, f_1].every(Number.isFinite)) {
    const d1 = f1 - f0;
    const d2 = f2 - f1;
    if (Math.abs(d2 - d1) < 1e-6) {
      const a = d1; // slope
      const b = f0; // intercept
      return [
        `Detected linear: f(x) ≈ ${a.toFixed(4)}x + ${b.toFixed(4)}`,
        `Solve f(x)=0: ${a.toFixed(4)}x + ${b.toFixed(4)} = 0`,
        `x = −b/a = ${(-b/a).toFixed(6)}`
      ];
    }
  }

  // Similar logic for quadratic detection...
}
```

## Keyboard Controls

### Graph Navigation
```javascript
document.addEventListener('keydown', (e) => {
  if (getActiveTabId() === 'tab-graph') {
    if (e.key === '+') { /* zoom in */ }
    if (e.key === '-') { /* zoom out */ }
    if (e.key === '0') { /* reset view */ }
    if (['ArrowLeft','ArrowRight','ArrowUp','ArrowDown'].includes(e.key)) {
      /* pan view */
    }
  }
});
```

### Expression Input
```javascript
exprInput.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') {
    plotCurrentExpression();
  }
});
```

## Responsive Design

### High-DPI Support
```javascript
function resizeGraph() {
  const rect = graphWrap.getBoundingClientRect();
  graphCanvas.width = rect.width * devicePixelRatio;
  graphCanvas.height = rect.height * devicePixelRatio;
  g2.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
  drawGraph();
}

new ResizeObserver(resizeGraph).observe(graphWrap);
```

### Grid Scaling
```javascript
function niceGridStep(scalePxPerUnit) {
  const targetPx = 60; // Target pixel spacing
  const raw = targetPx / scalePxPerUnit;
  const pow = Math.pow(10, Math.floor(Math.log10(raw)));
  const norm = raw / pow;

  let step = 1;
  if (norm < 1.5) step = 1;
  else if (norm < 3.5) step = 2;
  else if (norm < 7.5) step = 5;
  else step = 10;

  return step * pow;
}
```

## Integration Examples

### Custom Function Addition
```javascript
// Add a custom function to the parser
const originalFunctions = { ...FUNCTIONS };
FUNCTIONS.cbrt = Math.cbrt; // Cube root function

// Update tokenizer to recognize 'cbrt'
const originalTokenize = tokenize;
tokenize = function(expr) {
  // Call original and add cbrt recognition
  return originalTokenize(expr);
};
```

### Advanced Analysis
```javascript
// Add derivative plotting
function plotDerivative() {
  if (!graphState.compiled) return;

  const h = 1e-4;
  const derivative = (x) => {
    return (safeEval(graphState.compiled, x + h) - safeEval(graphState.compiled, x - h)) / (2 * h);
  };

  // Plot derivative alongside original function
  g2.strokeStyle = '#ef4444';
  g2.lineWidth = 2;
  g2.beginPath();

  for (let x = minX; x <= maxX; x += sampleStep) {
    const y = derivative(x);
    if (!isFinite(y)) continue;

    const px = x * graphState.zoom;
    const py = -y * graphState.zoom;
    g2.lineTo(px, py);
  }

  g2.stroke();
}
```

### Export Capabilities
```javascript
// Export graph as image
function exportGraph() {
  const link = document.createElement('a');
  link.download = `graph_${graphState.expr}.png`;
  link.href = graphCanvas.toDataURL();
  link.click();
}
```

## Error Handling

### Expression Errors
```javascript
function plotCurrentExpression() {
  const expr = exprInput.value.trim();
  if (!expr) return;

  const compiled = compile(expr);
  graphState.compiled = compiled;

  if (compiled) {
    graphState.intercepts = findIntercepts(compiled);
    const symbolic = trySymbolicSteps(expr, compiled);
    graphState.stepsHtml = symbolic ?
      `<div><ol>${symbolic.map(s => `<li>${s}</li>`).join('')}</ol></div>` :
      '<div>Parse error. Please check expression.</div>';
  } else {
    graphState.stepsHtml = '<div>Parse error. Please check expression.</div>';
  }

  solveOverlay.innerHTML = graphState.stepsHtml;
  drawGraph();
}
```

### Mathematical Edge Cases
- **Division by zero**: Returns NaN, handled gracefully in plotting
- **Undefined functions**: Returns NaN (e.g., `sqrt(-1)`, `ln(0)`)
- **Overflow**: Large numbers return `Infinity` or `-Infinity`

## Performance Optimizations

- **Efficient Sampling**: Adaptive step size based on zoom level
- **Coordinate Caching**: Reuses coordinate transformations
- **Selective Redrawing**: Only redraws when state changes
- **Memory Management**: Automatic cleanup of event listeners

This graphing system provides a comprehensive mathematical visualization platform with robust parsing, interactive controls, and educational features.