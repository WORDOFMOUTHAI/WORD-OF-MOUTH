# 30-Day Math Journey - Component Documentation

## Overview

This document provides detailed documentation for all UI components and interactive elements in the 30-Day Math Journey application. Each component includes usage examples, customization options, and integration patterns.

## Table of Contents

1. [Header Components](#header-components)
2. [Progress Components](#progress-components)
3. [Flashcard Components](#flashcard-components)
4. [Geometry Components](#geometry-components)
5. [Graphing Components](#graphing-components)
6. [Teach-Back Components](#teach-back-components)
7. [Settings Components](#settings-components)
8. [Layout Components](#layout-components)

---

## Header Components

### Brand Component

The application header features a distinctive brand section with animated logo and title.

```html
<div class="brand" aria-label="App title and identity">
  <div class="logo" aria-hidden="true"></div>
  <div class="title">
    <h1>30-Day Math Journey</h1>
    <small>Rigor meets confidence. Welcome back to math.</small>
  </div>
</div>
```

**Features:**
- Rotating gradient logo with subtle animation (`transform: rotate(6deg)`)
- Two-line title with primary heading and descriptive tagline
- Responsive typography that scales with viewport

**CSS Customization:**
```css
.logo {
  background: radial-gradient(circle at 30% 30%, var(--accent), var(--accent-2));
  box-shadow: 0 6px 18px var(--shadow) inset, 0 8px 20px rgba(96,165,250,0.35);
}
```

### Progress Header

Real-time progress indicator in the header showing overall completion.

```html
<div class="progress-wrap" aria-live="polite">
  <div class="progress-bar" aria-label="Overall progress">
    <div class="progress-fill" id="progressFill"></div>
  </div>
  <div class="progress-stats">
    <span id="progressText">0/30 lessons (0%)</span>
    <span id="milestoneText">Let's begin</span>
  </div>
</div>
```

**Dynamic Updates:**
- Progress bar width animates with CSS transitions (`transition: width 600ms ease`)
- Text updates show fraction and percentage
- Milestone messages provide encouragement at key completion points

**Milestone Messages:**
- 0%: "Let's begin"
- 25%: "Great start—25% complete!"
- 50%: "Halfway there—keep going!"
- 75%: "75% milestone—momentum is yours!"
- 100%: "Incredible! You finished. 🎉"

### Navigation Tabs

Accessible tab navigation with visual current page indicator.

```html
<nav class="nav" aria-label="Primary">
  <button class="tab-btn" data-tab="progress" aria-current="page">Progress</button>
  <button class="tab-btn" data-tab="flash">Flashcards</button>
  <button class="tab-btn" data-tab="geometry">Geometry</button>
  <button class="tab-btn" data-tab="graph">Graphing</button>
  <button class="tab-btn" data-tab="teach">Teach-Back</button>
  <button class="tab-btn" data-tab="settings">Settings</button>
</nav>
```

**Accessibility Features:**
- `aria-current="page"` for active tab
- Keyboard navigation support
- Focus indicators with custom styling

---

## Progress Components

### Stage Cards

Visual representation of learning stages with completion tracking.

```html
<div class="card">
  <div style="display:flex;justify-content:space-between;align-items:center;">
    <div style="display:flex;align-items:center;gap:8px;">
      <div style="width:14px;height:14px;border-radius:3px;background:#7dd3fc"></div>
      <strong>Stage 1: Foundation Reconstruction</strong>
    </div>
    <span class="chip">40% complete</span>
  </div>
  <div class="progress-bar" style="margin-top:8px">
    <div class="progress-fill" style="background:#7dd3fc;width:40%"></div>
  </div>
</div>
```

**Stage Definitions:**
1. **Stage 1**: Foundation Reconstruction (Days 1-10) - `#7dd3fc`
2. **Stage 2**: Intermediate Integration (Days 11-18) - `#60a5fa`
3. **Stage 3**: Classical Mathematical Thinking (Days 19-25) - `#34d399`
4. **Stage 4**: Sacred Geometry Synthesis (Days 26-30) - `#f59e0b`

### Lesson Grid

Interactive lesson selector with status indicators.

```html
<div class="lesson-grid">
  <div class="lesson-card completed">
    <div style="display:flex;justify-content:space-between;align-items:center;">
      <div>
        <div class="lesson-title">Day 1: PEMDAS Warm-up</div>
        <div style="color:var(--muted);font-size:12px;">Stage 1</div>
      </div>
      <span class="pill green">Done</span>
    </div>
  </div>
</div>
```

**Lesson States:**
- **Completed**: Green background, "Done" pill
- **Current**: Amber background, "Now" pill  
- **Future**: Default background, "Next" pill

**Interactive Features:**
- Click to select any lesson
- Hover animations (`transform: translateY(-2px)`)
- Responsive grid layout (`grid-template-columns: repeat(auto-fill, minmax(180px, 1fr))`)

### Current Lesson Detail

Comprehensive lesson information panel with action buttons.

```html
<div class="card">
  <h3>Current Lesson</h3>
  <div style="display:grid;gap:8px">
    <div style="display:flex;justify-content:space-between;align-items:center;">
      <div>
        <strong>Day 5: Solving Linear Equations II</strong>
        <div style="color:var(--muted);">Stage 1</div>
      </div>
      <div style="display:flex;gap:8px;">
        <button id="markCompleteBtn">Mark Complete</button>
        <button id="jumpToFlashBtn">Practice Flashcards</button>
        <button id="openGraphWithSampleBtn">Graph Example</button>
      </div>
    </div>
    <div><em>Objective:</em> Work multi-step equations</div>
    <div><em>Quick Activity:</em> Solve: (x/3) + 4 = 9</div>
  </div>
</div>
```

**Action Buttons:**
- **Mark Complete**: Updates progress and triggers celebrations
- **Practice Flashcards**: Switches to flashcard tab
- **Graph Example**: Pre-loads relevant expression in graphing tab

---

## Flashcard Components

### Flashcard 3D Interface

Animated flip card with 3D CSS transforms.

```html
<div class="flashcard-wrap">
  <div class="flashcard" id="flashcard" tabindex="0" role="button">
    <div class="face front">
      <div id="flashFront">What is the order of operations?</div>
    </div>
    <div class="face back">
      <div id="flashBack">PEMDAS: Parentheses, Exponents, Multiplication/Division, Addition/Subtraction</div>
    </div>
  </div>
</div>
```

**CSS Animation:**
```css
.flashcard {
  transform-style: preserve-3d;
  transition: transform 500ms ease;
}
.flashcard.flipped {
  transform: rotateY(180deg);
}
.face {
  backface-visibility: hidden;
}
.face.back {
  transform: rotateY(180deg);
}
```

**Interaction Methods:**
- Click anywhere on card to flip
- Enter key to flip
- Touch support for mobile devices

### Flashcard Controls

Spaced repetition rating system with color-coded buttons.

```html
<div class="flash-controls" aria-label="Answer quality">
  <button id="flipBtn">Flip (Enter)</button>
  <button id="againBtn" style="background:rgba(239,68,68,0.2);">Again (1)</button>
  <button id="hardBtn" style="background:rgba(245,158,11,0.2);">Hard (2)</button>
  <button id="goodBtn" style="background:rgba(16,185,129,0.2);">Good (3)</button>
  <button id="easyBtn" style="background:rgba(96,165,250,0.2);">Easy (4)</button>
</div>
```

**Rating Colors:**
- **Again**: Red (`#ef4444`) - Immediate review needed
- **Hard**: Amber (`#f59e0b`) - Reduced interval
- **Good**: Green (`#10b981`) - Normal progression
- **Easy**: Blue (`#60a5fa`) - Extended interval

### Deck Management

Tools for customizing and organizing flashcard decks.

```html
<div style="display:flex;gap:8px;align-items:center;">
  <button id="shuffleDeckBtn" title="Shuffle deck (S)">Shuffle</button>
  <button id="addCardBtn" title="Add your own card">Add Card</button>
  <span class="chip" id="dueCount">Due: 12</span>
  <span class="chip" id="cardProgress">3/12</span>
</div>
```

**Features:**
- **Shuffle**: Randomizes card order with keyboard shortcut
- **Add Card**: Modal prompts for custom cards
- **Due Count**: Real-time count of cards needing review
- **Progress**: Current position in due queue

---

## Geometry Components

### Interactive Canvas

Multi-touch canvas with shape manipulation and live measurements.

```html
<div class="canvas-wrap">
  <div class="canvas-toolbar">
    <span class="chip">Drag vertices/shape • Scroll to zoom</span>
  </div>
  <canvas id="geomCanvas" aria-label="Geometry canvas" role="img"></canvas>
</div>
```

**Supported Shapes:**
- **Triangle**: Draggable vertices with area/perimeter calculation
- **Rectangle**: Corner handles with dimension display
- **Circle**: Center and radius manipulation
- **Cube**: Isometric 3D visualization with rotation

**Interaction Features:**
- Mouse drag for desktop
- Touch gestures for mobile
- Scroll wheel zoom (0.4x to 5x)
- Pan by dragging empty space

### Shape Selector

Dropdown for switching between geometric shapes.

```html
<select id="shapeSelect" title="Choose shape">
  <option value="triangle">Triangle (2D)</option>
  <option value="rectangle">Rectangle (2D)</option>
  <option value="circle">Circle (2D)</option>
  <option value="cube">Cube (Isometric)</option>
</select>
```

### Measurement Display

Real-time calculation panel with formulas and values.

```html
<div class="card">
  <h3>Measurements</h3>
  <div id="geomProps">Perimeter = 24.00 units, Area = 24.00 sq units</div>
</div>
<div class="card">
  <h3>Concept Notes</h3>
  <ul id="geomNotes">
    <li>Area rectangle = width × height</li>
    <li>Perimeter rectangle = 2(w+h)</li>
  </ul>
</div>
```

**Measurement Types:**
- **Triangle**: Perimeter, Area (Heron's formula)
- **Rectangle**: Perimeter, Area
- **Circle**: Circumference, Area
- **Cube**: Surface Area, Volume, Edge length

---

## Graphing Components

### Graph Canvas

Advanced coordinate plane with adaptive grid and function plotting.

```html
<div class="canvas-wrap graph-wrap">
  <canvas id="graphCanvas" aria-label="Graph canvas" role="img"></canvas>
  <div class="overlay-steps" id="solveOverlay" hidden>
    <strong>Detected linear:</strong> f(x) ≈ 2.0000x + 3.0000
    <div>Solve f(x)=0: 2.0000x + 3.0000 = 0</div>
    <div>x = −b/a = -1.500000</div>
  </div>
</div>
```

**Visual Features:**
- Adaptive grid with smart spacing
- Color-coded axes and labels
- Function curve rendering with sampling
- Intercept highlighting (red for x-intercepts, green for y-intercept)

### Expression Input

Mathematical expression parser with real-time validation.

```html
<input id="exprInput" 
       placeholder="Enter f(x), e.g., sin(x) + 0.5x" 
       style="min-width:280px;">
<button id="plotBtn">Plot (Enter)</button>
<button id="resetViewBtn">Reset View</button>
```

**Supported Syntax:**
- **Functions**: sin, cos, tan, asin, acos, atan, sqrt, abs, ln, log, exp
- **Constants**: pi, e
- **Operators**: +, -, *, /, ^ (with proper precedence)
- **Implicit multiplication**: 2x, sin(x)cos(x)

### Solution Overlay

Step-by-step analysis panel with automatic function detection.

**Linear Functions:**
```
Detected linear: f(x) ≈ 2.0000x + 3.0000
Solve f(x)=0: 2.0000x + 3.0000 = 0  
x = −b/a = -1.500000
```

**Quadratic Functions:**
```
Detected quadratic: f(x) ≈ 1.0000x² + -5.0000x + 6.0000
Discriminant Δ = b² − 4ac = 1.000000
Roots: x = [−b ± √Δ]/(2a)
```

**General Functions:**
```
Newton's method steps:
• Iter 1: x = 1.000000, f(x) = 0.841471, f'(x) ≈ 0.540302, next ≈ -0.557407
• Iter 2: x = -0.557407, f(x) = -0.527860, f'(x) ≈ 0.848771, next ≈ 0.064624
```

---

## Teach-Back Components

### Concept Selector

Dropdown for choosing mathematical concepts to explain.

```html
<select id="teachConceptSelect">
  <option value="pemdas">Order of Operations (PEMDAS)</option>
  <option value="linear">Solving Linear Equations</option>
  <option value="trig">Right Triangle Trigonometry</option>
  <option value="quadratic">Quadratics & Roots</option>
  <option value="geometry">Area and Perimeter</option>
</select>
```

### Teaching Prompt

Dynamic prompt generation with varied question styles.

```html
<div class="card">
  <div id="teachPrompt">
    Order of Operations (PEMDAS): Explain this concept as if to a 12-year-old with one real-life example.
  </div>
</div>
```

**Prompt Variations:**
- "Explain this concept as if to a 12-year-old with one real-life example"
- "Teach it to a colleague returning to math after years; focus on confidence"
- "Write a step-by-step method in under 6 sentences"
- "Describe a common mistake and how to avoid it"

### Response Area

Large text input with formatting and save functionality.

```html
<div class="card">
  <label for="teachResponse"><strong>Your explanation</strong></label>
  <textarea id="teachResponse" 
            placeholder="Explain as if to a curious friend. Use examples."
            style="min-height:120px;"></textarea>
  <div style="display:flex;gap:8px;align-items:center;">
    <button id="saveTeachBtn">Save Reflection</button>
    <span id="teachSaveMsg" style="color:var(--muted)"></span>
  </div>
</div>
```

### Progressive Hints

Expandable hint system with controlled revelation.

```html
<div class="card">
  <h3>Progressive Hints</h3>
  <ol id="hintList">
    <li>Which operation should be handled first in 3 + 2 × 4?</li>
  </ol>
  <button id="revealNextHintBtn">Reveal next hint</button>
  <button id="showFullAnswerBtn">Show full example answer</button>
</div>
```

**Hint Strategy:**
- Start with one guiding question
- Reveal additional hints on demand
- Full worked example as final option
- Prevents over-reliance while providing support

### Self-Assessment

Three-point rating system for reflection quality.

```html
<div class="card">
  <h3>Self-Assessment</h3>
  <div style="display:flex;gap:8px;">
    <button class="selfBtn" data-score="3" 
            style="background:rgba(16,185,129,0.2);">Nailed it</button>
    <button class="selfBtn" data-score="2" 
            style="background:rgba(245,158,11,0.2);">Almost</button>
    <button class="selfBtn" data-score="1" 
            style="background:rgba(239,68,68,0.2);">Need practice</button>
    <span id="selfMsg" style="color:var(--muted)"></span>
  </div>
</div>
```

**Assessment Feedback:**
- **Score 3**: "Excellent clarity."
- **Score 2**: "Good! Try one more example."
- **Score 1**: "Great effort. Revisit hints and try again."

---

## Settings Components

### Accessibility Controls

High contrast mode toggle with immediate visual feedback.

```html
<div class="card">
  <h3>Accessibility</h3>
  <div style="display:flex;gap:8px;align-items:center;">
    <button id="contrastToggleBtn">Toggle High Contrast</button>
    <span class="chip">Keyboard tips live in the footer</span>
  </div>
</div>
```

**High Contrast Mode:**
- Black background with white text
- Enhanced color contrast for all UI elements
- Bright accent colors for better visibility
- Persistent setting saved to localStorage

### Data Management

Export, import, and reset functionality for user data.

```html
<div class="card">
  <h3>Data</h3>
  <div style="display:flex;gap:8px;align-items:center;">
    <button id="exportBtn">Export Progress</button>
    <input type="file" id="importInput" accept="application/json" style="display:none">
    <button id="importBtn">Import</button>
    <button id="resetAllBtn" style="background:rgba(239,68,68,0.2);">Reset All</button>
  </div>
</div>
```

**Features:**
- **Export**: Downloads complete state as `math_journey_export.json`
- **Import**: Validates and restores from JSON file
- **Reset**: Confirmation dialog prevents accidental data loss

### About Section

Application information and feature overview.

```html
<div class="card">
  <h3>About</h3>
  <p>This single-file application was designed for adults returning to mathematics. It blends structure and warmth: rigorous visuals and step-by-step reasoning with encouragement and reflection.</p>
  <ul>
    <li>Four stages guiding 30 days of learning</li>
    <li>Spaced repetition to reinforce concepts that need attention</li>
    <li>Canvas-based geometry and graphing to engage visual intuition</li>
    <li>Teach-back prompts and self-assessment to consolidate understanding</li>
  </ul>
</div>
```

---

## Layout Components

### Card System

Consistent card-based layout with elevation and transparency.

```css
.card {
  background: linear-gradient(180deg, rgba(255,255,255,0.08), rgba(255,255,255,0.04));
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: var(--radius);
  box-shadow: 0 12px 24px var(--shadow);
  padding: 16px;
}
```

**Usage Patterns:**
- Main content containers
- Nested cards for sub-sections
- Consistent spacing and visual hierarchy

### Grid Layouts

Responsive grid system with breakpoint adaptation.

```css
.grid {
  display: grid;
  gap: 16px;
}

@media (min-width: 900px) {
  .grid-2 { grid-template-columns: 1fr 1fr; }
  .grid-3 { grid-template-columns: 1fr 1fr 1fr; }
}
```

### Button Styles

Consistent interactive element styling with hover effects.

```css
button, .btn {
  background: linear-gradient(145deg, rgba(255,255,255,0.08), rgba(255,255,255,0.02));
  color: var(--text);
  border: 1px solid rgba(255,255,255,0.12);
  padding: 10px 14px;
  border-radius: var(--radius-sm);
  cursor: pointer;
  transition: transform 120ms ease, background 200ms;
}
button:hover { transform: translateY(-1px); }
button:active { transform: translateY(0px) scale(0.98); }
```

### Status Pills

Color-coded indicators for various states and progress.

```html
<span class="pill green">Completed</span>
<span class="pill yellow">Current</span>
<span class="pill gray">Future</span>
```

**Pill Colors:**
- **Green**: Success states, completion
- **Yellow**: Active/current states  
- **Gray**: Future/inactive states
- **Red**: Error or attention needed

### Footer

Persistent keyboard shortcut reference.

```html
<div class="footer" role="contentinfo">
  <span>Flashcards: <span class="kbd">Enter</span> flip, <span class="kbd">1-4</span> rate</span>
  <span>Graphing: <span class="kbd">Drag</span> pan, <span class="kbd">Wheel / + / -</span> zoom</span>
  <span>Global: <span class="kbd">Tab</span> navigate, <span class="kbd">Shift+?</span> help</span>
</div>
```

**Keyboard Styling:**
```css
.kbd {
  display: inline-block;
  border: 1px solid rgba(255,255,255,0.35);
  border-bottom-width: 3px;
  padding: 2px 6px;
  border-radius: 6px;
  font-size: 12px;
  background: rgba(255,255,255,0.08);
}
```

---

## Responsive Design

### Mobile Adaptations

The application includes comprehensive mobile support:

**Touch Events:**
- Full gesture support for canvas interactions
- Pinch-to-zoom on graphs and geometry
- Touch-friendly target sizes (minimum 44px)

**Layout Adjustments:**
- Flexible header layout with wrapping navigation
- Responsive grid systems that stack on mobile
- Appropriate font scaling and spacing

**Performance Optimizations:**
- Passive event listeners where appropriate
- Efficient canvas rendering with requestAnimationFrame
- Minimal DOM manipulation during interactions

### Accessibility Features

**Screen Reader Support:**
- Semantic HTML with proper heading hierarchy
- ARIA labels and live regions
- Role attributes for interactive elements

**Keyboard Navigation:**
- Full tab order accessibility
- Custom focus indicators
- Logical keyboard shortcuts

**Visual Accessibility:**
- High contrast mode option
- Sufficient color contrast ratios
- Clear visual hierarchy and spacing

This component documentation provides comprehensive coverage of all UI elements, their styling, interactions, and customization options within the 30-Day Math Journey application.