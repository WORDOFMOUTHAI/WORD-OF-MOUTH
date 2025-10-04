# Component Documentation

This document provides detailed information about the UI components, their structure, interactions, and styling.

## Table of Contents
1. [Layout Structure](#layout-structure)
2. [Header Component](#header-component)
3. [Progress Tab](#progress-tab)
4. [Flashcards Tab](#flashcards-tab)
5. [Geometry Tab](#geometry-tab)
6. [Graphing Tab](#graphing-tab)
7. [Teach-Back Tab](#teach-back-tab)
8. [Settings Tab](#settings-tab)
9. [Styling System](#styling-system)
10. [Responsive Design](#responsive-design)

---

## Layout Structure

### App Container

```html
<div class="app" id="app">
  <header role="banner">...</header>
  <main>...</main>
  <div class="footer" role="contentinfo">...</div>
  <canvas class="celebration" id="confettiCanvas"></canvas>
</div>
```

**Structure:**
- **App Container:** Flexbox column layout spanning full viewport
- **Header:** Sticky positioned with backdrop blur
- **Main:** Scrollable content area with max-width constraint
- **Footer:** Fixed bottom bar with keyboard shortcuts
- **Celebration Canvas:** Full-screen overlay for confetti animations

---

## Header Component

### Structure

```html
<header role="banner">
  <div class="header-inner">
    <div class="brand">...</div>
    <div class="progress-wrap">...</div>
    <div class="actions">...</div>
  </div>
</header>
```

### Brand Section

**Components:**
- **Logo:** Gradient-filled div with rotation and shadow
- **Title:** App name with tagline

**Styling:**
- Flexbox with 12px gap
- Logo: 42×42px with cyan-blue gradient
- Title: 20px font with muted tagline

### Progress Wrap

**Components:**
- **Progress Bar:** Animated fill bar showing completion percentage
- **Progress Stats:** Completed lessons and milestone message

**Features:**
- Real-time updates on lesson completion
- Smooth 600ms transition on fill width
- Dynamic milestone messages at 0%, 25%, 50%, 75%, 100%

**Progress Fill Animation:**
```css
.progress-fill {
  width: 0%;
  background: linear-gradient(90deg, var(--good), var(--accent));
  transition: width 600ms ease;
}
```

### Actions Section

**Components:**
- **Navigation Tabs:** 6 buttons for main sections
- **Tab Indicators:** `aria-current="page"` for active tab

**Styling:**
- Grid layout with auto-flow columns
- Active tab: Highlighted background and border
- Hover: Subtle translateY animation

---

## Progress Tab

### Components

```html
<section class="card" id="tab-progress">
  <div>[Title and Legend]</div>
  <div class="grid grid-2">
    <div>[Stages Container]</div>
    <div>[Lessons Grid]</div>
  </div>
  <div class="card">[Current Lesson Detail]</div>
</section>
```

### Stages Container

**Purpose:** Visualizes the 4 learning stages with completion percentages

**Structure:**
- Each stage is a card with:
  - Color indicator dot
  - Stage title
  - Completion percentage chip
  - Progress bar with stage-specific color

**Rendering:**
```javascript
function renderStages() {
  stages.forEach(st => {
    const done = state.progress.completed
      .slice(st.range[0]-1, st.range[1])
      .filter(Boolean).length;
    const pct = Math.round((done/total) * 100);
    // Create card with progress bar
  });
}
```

### Lessons Grid

**Purpose:** Displays all 30 lessons as interactive cards

**Layout:**
- Grid with `repeat(auto-fill, minmax(180px, 1fr))`
- Responsive: 1 column mobile, 2-5 columns desktop

**Card States:**
- **Completed:** Green background, "Done" pill
- **Current:** Amber background, "Now" pill
- **Future:** Gray background, "Next" pill

**Interactions:**
- Click: Selects lesson and shows details
- Hover: TranslateY(-2px) lift effect

### Current Lesson Detail

**Purpose:** Shows objectives and activities for selected lesson

**Components:**
- Lesson title and stage indicator
- Objective description
- Quick activity prompt
- Action buttons:
  - **Mark Complete:** Updates progress and triggers celebration
  - **Practice Flashcards:** Switches to flashcards tab
  - **Graph Example:** Switches to graphing with pre-filled expression

---

## Flashcards Tab

### Structure

```html
<section class="card" id="tab-flash">
  <div>[Title and Controls]</div>
  <div class="flashcard-wrap">
    <div class="flashcard" id="flashcard">
      <div class="face front">[Front Content]</div>
      <div class="face back">[Back Content]</div>
    </div>
  </div>
  <div class="flash-controls">[Rating Buttons]</div>
</section>
```

### Flashcard Component

**Design:** 3D flip card with CSS transforms

**Structure:**
- **Wrapper:** Provides perspective (1200px)
- **Card:** Preserves 3D transform style
- **Faces:** Front and back with backface-visibility hidden

**Flip Animation:**
```css
.flashcard {
  transform-style: preserve-3d;
  transition: transform 500ms ease;
}
.flashcard.flipped {
  transform: rotateY(180deg);
}
.face.back {
  transform: rotateY(180deg);
}
```

**Interactions:**
- Click card: Toggle flip
- Click "Flip" button: Toggle flip
- Press Enter: Toggle flip
- Press 1-4: Rate card and advance

### Rating Buttons

**Buttons:**
1. **Again (1):** Red background - Complete memory failure
2. **Hard (2):** Amber background - Difficulty recalling
3. **Good (3):** Green background - Correct with effort
4. **Easy (4):** Blue background - Perfect recall

**Visual Feedback:**
- Color-coded backgrounds matching function
- Active state: Scale(0.98) press effect
- Focus: Dashed outline for keyboard navigation

### Due Queue Display

**Components:**
- **Due Count:** Number of cards due for review
- **Card Progress:** Current card index / total due

**Updates:**
- After rating a card
- After shuffling deck
- On tab switch
- After adding custom card

---

## Geometry Tab

### Structure

```html
<section class="card" id="tab-geometry">
  <div>[Title and Shape Selector]</div>
  <div class="canvas-wrap" id="geomWrap">
    <div class="canvas-toolbar">[Instructions]</div>
    <canvas id="geomCanvas"></canvas>
  </div>
  <div class="grid grid-2">
    <div class="card">[Measurements]</div>
    <div class="card">[Concept Notes]</div>
  </div>
</section>
```

### Shape Selector

**Options:**
- Triangle (2D)
- Rectangle (2D)
- Circle (2D)
- Cube (Isometric)

**Behavior:**
- On change: Updates `geomState.shape`
- Triggers `drawGeometry()`
- Maintains pan and zoom state

### Canvas Component

**Size:** Full width × 360px height

**Coordinate System:**
- Origin at canvas center
- Y-axis inverted (down is positive in screen space)
- World coordinates transformed via `zoom` and `offset`

**Rendering Layers:**
1. Background clear
2. Axes (light gray lines)
3. Shape (fill + stroke)
4. Interactive handles (white circles with shadow)

### Interactive Handles

**Types:**
- **Vertex Handles:** For triangle vertices
- **Corner Handles:** For rectangle corners
- **Center Handle:** For circle and cube
- **Edge Handle:** For circle radius adjustment

**Appearance:**
- 6px radius white circles
- Semi-transparent black stroke
- Always drawn last (top layer)

**Hit Detection:**
- 8px radius for touch-friendly interaction
- Returns `{type, index}` object on hit
- Falls back to shape pan if no handle hit

### Measurements Display

**Real-time Updates:**
- **Triangle:** Perimeter, Area (Heron's formula)
- **Rectangle:** Perimeter, Area
- **Circle:** Circumference, Area
- **Cube:** Edge length, Surface Area, Volume

**Format:** Displays to 2 decimal places with unit labels

### Concept Notes

**Content:** Context-specific formulas and facts

**Examples:**
- Triangle: "Heron's formula: A = √(s(s-a)(s-b)(s-c))"
- Circle: "Circumference = 2πr, Area = πr²"
- Cube: "Isometric is a pseudo-3D projection"

---

## Graphing Tab

### Structure

```html
<section class="card" id="tab-graph">
  <div>[Title and Expression Input]</div>
  <div class="canvas-wrap graph-wrap" id="graphWrap">
    <canvas id="graphCanvas"></canvas>
    <div class="overlay-steps" id="solveOverlay"></div>
  </div>
  <div>[Controls Legend]</div>
</section>
```

### Expression Input

**Component:**
- Text input with placeholder: "Enter f(x), e.g., sin(x) + 0.5x"
- Min-width: 280px for readability
- Glassmorphic styling matching theme

**Supported Syntax:**
- Variables: `x`
- Operators: `+`, `-`, `*`, `/`, `^`
- Functions: `sin`, `cos`, `tan`, `sqrt`, `ln`, `log`, `exp`, `abs`, `asin`, `acos`, `atan`
- Constants: `pi`, `e`
- Implicit multiplication: `2x`, `sin(x)cos(x)`

**Examples:**
```
x^2 - 4x + 3
sin(x) + cos(2x)
sqrt(x) * ln(x)
2x^3 - 5x^2 + 3x - 1
```

### Graph Canvas

**Size:** Full width × 420px height

**Coordinate System:**
- Origin movable via `graphState.origin`
- Zoom: 20-400 pixels per unit
- Screen center is reference point

**Grid System:**
- **Minor Grid:** 1/5 of major step, very light
- **Major Grid:** Dynamic spacing (60px target), light
- **Axes:** Medium opacity with arrowheads implied

**Adaptive Grid:**
```javascript
function niceGridStep(scalePxPerUnit) {
  const targetPx = 60;
  const raw = targetPx / scalePxPerUnit;
  // Rounds to 1, 2, 5, or 10 × power of 10
}
```

### Function Rendering

**Algorithm:**
1. Sample function at pixel-aligned x values
2. Transform to screen coordinates
3. Skip non-finite values
4. Break curve at discontinuities

**Performance:**
- Adaptive sampling based on zoom level
- Early termination for invalid values
- Only renders visible domain ±2 units

### Intercept Highlighting

**X-Intercepts (Roots):**
- Color: Red (#ef4444)
- Detection: Sign changes in domain scan
- Refinement: Bisection method (20 iterations)
- Deduplication: 1e-3 unit threshold
- Display: 4px radius circles

**Y-Intercept:**
- Color: Green (#34d399)
- Calculation: f(0)
- Display: 4px radius circle

### Solution Steps Overlay

**Position:** Top-right, absolute positioned

**Styling:**
- Dark semi-transparent background
- Max width: 360px or 90vw
- Scrollable if content overflows
- Border with subtle glow

**Content:**
- **Linear Detection:** Shows slope-intercept form and solving steps
- **Quadratic Detection:** Shows discriminant and quadratic formula
- **General Case:** Newton's method iterations with x, f(x), f'(x) values

**Example Output:**
```
Detected quadratic: f(x) ≈ 1.0000x² + -4.0000x + 3.0000
Discriminant Δ = b² − 4ac = 4.000000
Roots: x = [−b ± √Δ]/(2a)
```

### Interactions

**Pan:**
- Mouse: Click and drag
- Touch: Single-finger drag
- Keyboard: Arrow keys (40px per press)

**Zoom:**
- Mouse: Wheel (towards cursor position)
- Touch: Two-finger pinch
- Keyboard: `+` and `-` keys

**Reset:**
- Button: "Reset View"
- Keyboard: `0` key
- Effect: Zoom to 60px/unit, origin to (0,0)

---

## Teach-Back Tab

### Structure

```html
<section class="card" id="tab-teach">
  <div>[Title and Concept Selector]</div>
  <div class="teach-grid">
    <div class="card">[Generated Prompt]</div>
    <div class="card">[Response Textarea]</div>
    <div class="card">[Progressive Hints]</div>
    <div class="card">[Self-Assessment]</div>
  </div>
</section>
```

### Concept Selector

**Concepts:**
1. Order of Operations (PEMDAS)
2. Solving Linear Equations
3. Right Triangle Trigonometry
4. Quadratics & Roots
5. Area and Perimeter

**Behavior:**
- Dropdown select element
- On change: Regenerates prompt
- Resets hint progression

### Prompt Display

**Purpose:** Presents a teaching challenge for the selected concept

**Prompt Variations:**
- Explain to a 12-year-old with example
- Teach colleague returning to math
- Write step-by-step in 6 sentences
- Describe common mistake and solution

**Generation:**
```javascript
function randomPromptForConcept(concept) {
  const prompts = [ /* 4 variations */ ];
  return prompts[Math.floor(Math.random() * prompts.length)];
}
```

### Response Textarea

**Component:**
- Multi-line textarea
- Placeholder: "Explain as if to a curious friend. Use examples."
- Min-height: 120px, resizable
- Auto-saves on "Save Reflection" button

**Persistence:**
```javascript
{
  concept: string,
  prompt: string,
  text: string,
  timestamp: number,
  selfScore?: 1 | 2 | 3
}
```

### Progressive Hints System

**Structure:**
- Ordered list of hints
- First hint visible by default
- "Reveal next hint" button shows one more
- "Show full example answer" reveals complete solution

**Hint Progression Example (Linear Equations):**
1. "Balance both sides by undoing operations in reverse order."
2. "Isolate x: move constants to one side, x-terms to the other."
3. "Divide by the coefficient of x to solve."
4. Full example: "For 7x − 9 = 3x + 15, subtract 3x both sides → 4x − 9 = 15..."

### Self-Assessment Buttons

**Options:**
1. **Nailed it (3):** Green background - Complete understanding
2. **Almost (2):** Amber background - Good but needs refinement
3. **Need practice (1):** Red background - Requires more work

**Feedback Messages:**
- Score 3: "Excellent clarity."
- Score 2: "Good! Try one more example."
- Score 1: "Great effort. Revisit hints and try again."

**Persistence:**
- Score attached to most recent reflection
- Used for tracking growth over time

---

## Settings Tab

### Structure

```html
<section class="card" id="tab-settings">
  <div class="grid grid-2">
    <div class="card">[Accessibility]</div>
    <div class="card">[Data Management]</div>
  </div>
  <div class="card">[About]</div>
</section>
```

### Accessibility Section

**High Contrast Toggle:**
- Button: "Toggle High Contrast"
- Effect: Adds `.high-contrast` class to `<html>`
- Persists in `state.settings.highContrast`

**High Contrast Changes:**
- Pure black backgrounds
- Pure white text
- Cyan/yellow accents for maximum contrast
- Thicker borders and outlines
- Removes gradients and shadows

### Data Management

**Export:**
- Button: "Export Progress"
- Action: Downloads `math_journey_export.json`
- Format: Pretty-printed JSON with 2-space indent
- Content: Complete state object

**Import:**
- Button: "Import"
- Action: Opens file picker for JSON files
- Validation: Attempts parse, falls back to initial state on error
- Effect: Merges imported data with defaults, refreshes all UI

**Reset All:**
- Button: "Reset All" (red styling)
- Confirmation: "Reset all progress and settings? This cannot be undone."
- Action: Replaces state with `createInitialState()`
- Effect: Clears all progress, flashcard stats, reflections, settings

### About Section

**Content:**
- Application description
- Feature list
- Learning philosophy summary
- Credits and acknowledgments

---

## Styling System

### CSS Custom Properties

**Theme Variables:**
```css
:root {
  --bg-gradient-start: #0e1f2f;
  --bg-gradient-end: #1b3a57;
  --surface: #0f2538;
  --surface-2: #0b1b2a;
  --text: #e8f1f8;
  --muted: #a7c1d6;
  --accent: #7dd3fc;      /* cyan */
  --accent-2: #60a5fa;    /* blue */
  --good: #34d399;        /* green */
  --warn: #f59e0b;        /* amber */
  --future: #6b7280;      /* gray */
  --danger: #ef4444;      /* red */
  --shadow: rgba(0,0,0,0.3);
  --radius: 14px;
  --radius-sm: 10px;
  --radius-lg: 18px;
  --focus: #fde68a;       /* yellow */
}
```

### Card Component

**Base Styling:**
```css
.card {
  background: linear-gradient(180deg, 
    rgba(255,255,255,0.08), 
    rgba(255,255,255,0.04));
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: var(--radius);
  box-shadow: 0 12px 24px var(--shadow);
  padding: 16px;
}
```

**Glassmorphic Effect:**
- Semi-transparent background
- Backdrop blur (on header)
- Subtle border
- Layered shadows

### Button Component

**Base Styling:**
```css
button, .btn {
  background: linear-gradient(145deg, 
    rgba(255,255,255,0.08), 
    rgba(255,255,255,0.02));
  color: var(--text);
  border: 1px solid rgba(255,255,255,0.12);
  padding: 10px 14px;
  border-radius: var(--radius-sm);
  cursor: pointer;
  transition: transform 120ms ease, background 200ms;
}
```

**States:**
- **Hover:** `translateY(-1px)` lift
- **Active:** `scale(0.98)` press
- **Focus:** `outline: 2px dashed var(--focus)` with 2px offset

**Variants:**
- **Active Tab:** Cyan background with increased opacity
- **Danger:** Red-tinted background
- **Good:** Green-tinted background
- **Warn:** Amber-tinted background

### Typography

**Font Stack:**
```css
font-family: system-ui, -apple-system, Segoe UI, Roboto, 
             Ubuntu, Cantarell, Noto Sans, Helvetica, Arial, 
             "Apple Color Emoji", "Segoe UI Emoji";
```

**Hierarchy:**
- **H1:** 20px, letter-spacing 0.4px
- **H2:** Default size, margin-top 0
- **H3:** Default size, margin-top 0 or 8px
- **Body:** Default system size
- **Small:** 12px (stats, labels)
- **Tiny:** 11px (pills, chips)

---

## Responsive Design

### Breakpoints

**Mobile First Approach:**

```css
/* Base: Mobile (320px+) */
.grid { display: grid; gap: 16px; }

/* Tablet/Desktop (900px+) */
@media (min-width: 900px) {
  .grid-2 { grid-template-columns: 1fr 1fr; }
  .grid-3 { grid-template-columns: 1fr 1fr 1fr; }
}
```

### Responsive Patterns

**Header:**
- Mobile: Stacks vertically if needed
- Desktop: 3-column grid (brand, progress, actions)
- Progress bar: Min-width 280px, grows with available space

**Navigation:**
- Mobile: May wrap to multiple rows
- Desktop: Single row with auto-flow columns

**Lesson Grid:**
- Mobile: 1-2 columns
- Tablet: 2-3 columns
- Desktop: 3-5 columns
- Uses: `repeat(auto-fill, minmax(180px, 1fr))`

**Canvas Components:**
- Mobile: Full width, reduced height
- Desktop: Max-width 1200px, centered
- Maintains aspect ratio constraints

### Touch Support

**Geometry Canvas:**
```javascript
geomCanvas.addEventListener('touchstart', (e) => {
  const t = e.touches[0];
  // Translate touch to mouse event
  e.preventDefault();
}, { passive: false });
```

**Graph Canvas:**
- **Single Touch:** Pan
- **Two-Finger Pinch:** Zoom with distance tracking

**Passive: False:**
- Allows `preventDefault()` to block default scrolling
- Enables custom touch interactions

---

## Animation System

### Transition Timing

**Component Transitions:**
```css
transition: transform 120ms ease,  /* Hover/active */
            background 200ms,      /* Color changes */
            width 600ms ease;      /* Progress bar */
```

### Transform Animations

**Hover Effects:**
- Cards: `translateY(-2px)`
- Buttons: `translateY(-1px)`
- Active: `scale(0.98)`

**3D Transforms:**
- Flashcard: `rotateY(180deg)` with `preserve-3d`
- Logo: `rotate(6deg)` for playful tilt

### Canvas Animations

**Confetti:**
- RequestAnimationFrame loop
- Particle physics: gravity, rotation, fade
- Cleans up when particles exit viewport

**Graph Rendering:**
- Continuous arc drawing with `lineTo()`
- Breaks at discontinuities
- No explicit animation (renders on demand)

---

## Accessibility Features

### Semantic HTML

**Structure:**
```html
<header role="banner">
<main role="main">
<footer role="contentinfo">
<nav aria-label="Primary">
<section role="region" aria-labelledby="...">
```

### ARIA Attributes

**Progress Bar:**
```html
<div class="progress-bar" aria-label="Overall progress">
  <div class="progress-fill" id="progressFill"></div>
</div>
```

**Tab Navigation:**
```html
<button class="tab-btn" 
        data-tab="progress" 
        aria-current="page">
  Progress
</button>
```

**Flashcard:**
```html
<div class="flashcard" 
     tabindex="0" 
     aria-label="Flashcard. Press Enter to flip." 
     role="button">
```

**Canvas:**
```html
<canvas id="geomCanvas" 
        aria-label="Geometry canvas" 
        role="img">
</canvas>
```

### Keyboard Navigation

**Focus Management:**
- All interactive elements have `tabindex` (explicit or implicit)
- Visible focus indicator: `outline: 2px dashed var(--focus)`
- Outline offset: 2px for clarity
- Skip to main content via Tab

**Keyboard Handlers:**
- Global: Help shortcut (Shift+?)
- Context-sensitive: Tab-specific shortcuts
- No keyboard traps

### Screen Reader Support

**Live Regions:**
```html
<div class="progress-wrap" aria-live="polite">
  <div class="progress-stats">
    <span id="progressText">0/30 lessons (0%)</span>
    <span id="milestoneText">Let's begin</span>
  </div>
</div>
```

**Descriptive Labels:**
- All buttons have clear text or `aria-label`
- Form inputs have associated `<label>` elements
- Canvas elements have descriptive `aria-label`

### Color Contrast

**Standard Mode:**
- Background: Dark blue gradient
- Text: Light blue-gray (#e8f1f8)
- Ratio: >7:1 (AAA standard)

**High Contrast Mode:**
- Background: Pure black (#000000)
- Text: Pure white (#ffffff)
- Accents: Cyan (#00ffff), Yellow (#ffff00)
- Ratio: 21:1 (maximum)

---

## Component Communication

### Event Flow

```
User Interaction
     ↓
Event Listener (DOM)
     ↓
Update State
     ↓
saveState() → localStorage
     ↓
Notify Subscribers
     ↓
Update UI Components
```

### State Observers

**Subscribe Pattern:**
```javascript
subscribe((state) => {
  // React to state changes
  console.log('State updated:', state);
});
```

**Notification:**
```javascript
state.progress.completed[5] = true;
notify(); // All subscribers called
```

### Cross-Tab Communication

**Not Implemented** but could use:
```javascript
window.addEventListener('storage', (e) => {
  if (e.key === APP_KEY) {
    state = loadState();
    updateAllUI();
  }
});
```

---

## Performance Optimizations

### Canvas Rendering

**Device Pixel Ratio:**
```javascript
canvas.width = rect.width * devicePixelRatio;
canvas.height = rect.height * devicePixelRatio;
ctx.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
```

**Resize Observer:**
```javascript
new ResizeObserver(() => {
  resizeCanvas();
  redraw();
}).observe(wrapElement);
```

**Benefits:**
- Crisp rendering on retina displays
- Automatic responsive sizing
- No polling or window resize thrashing

### State Management

**Batched Updates:**
- Single `saveState()` call per user action
- `updateProgressUI()` called once after save
- Avoids repeated DOM manipulation

**Lazy Evaluation:**
- Intercepts calculated only when expression changes
- Measurements computed only when shape updates
- Stats objects created on first access

### Memory Management

**Confetti Cleanup:**
```javascript
confettiParticles = confettiParticles.filter(
  p => p.life > 0 && p.y < canvasHeight + 20
);
```

**Canvas Clearing:**
```javascript
ctx.clearRect(0, 0, width, height);
// Only clears what's needed
```

---

## Browser Compatibility Notes

### Required APIs

✅ **Supported:**
- Canvas 2D Context
- localStorage
- ES6+ (arrow functions, const/let, template literals, destructuring)
- CSS Grid & Flexbox
- CSS Custom Properties
- ResizeObserver
- Touch Events

❌ **Not Used:**
- WebGL
- Service Workers
- IndexedDB
- Web Components
- Shadow DOM

### Polyfill Opportunities

**ResizeObserver:**
```javascript
if (!('ResizeObserver' in window)) {
  // Fallback to window.resize with debouncing
  window.addEventListener('resize', debounce(resizeCanvas, 100));
}
```

---

## Testing Considerations

### Unit Testing Targets

**Pure Functions:**
- `clamp()`, `lerp()`, `formatPercent()`, `uid()`
- `tokenize()`, `toRPN()`, `evalRPN()`
- `worldToScreen()`, `screenToWorld()`

**State Management:**
- `createInitialState()`
- `getCardStats()`
- `getCurrentLessonIndex()`

### Integration Testing Targets

**User Flows:**
1. Complete a lesson → Verify progress updates
2. Rate flashcard → Verify stats update and reschedule
3. Draw geometry → Verify measurements update
4. Plot expression → Verify graph renders and intercepts found
5. Save reflection → Verify persistence

### Visual Regression Testing

**Key Screens:**
- Progress tab with various completion states
- Flashcard flip animation
- Geometry canvas with each shape
- Graph canvas with various functions
- High contrast mode comparison

---

## Customization Guide

### Adding a New Tab

1. **HTML:** Add section with `id="tab-{name}"`
2. **Button:** Add tab button with `data-tab="{name}"`
3. **Switch Logic:** Update `switchTab()` to handle new tab
4. **Active Check:** Update `getActiveTabId()`
5. **Initialization:** Add setup in `init()` if needed

### Creating a Custom Component

```javascript
// Define state
const customState = { /* ... */ };

// Create render function
function renderCustom() {
  const element = document.getElementById('custom');
  element.innerHTML = `<div>${customState.value}</div>`;
}

// Add event listeners
element.addEventListener('click', () => {
  customState.value++;
  renderCustom();
});

// Initialize
renderCustom();
```

### Extending Theme

```css
/* Add custom color */
:root {
  --custom: #ff00ff;
}

/* Use in component */
.custom-button {
  background: var(--custom);
  color: var(--text);
}
```

---

This component documentation provides a comprehensive guide to understanding, modifying, and extending the UI components of the 30-Day Math Journey application.
