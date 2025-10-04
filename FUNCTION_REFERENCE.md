# 30-Day Math Journey - Function Reference

## Overview

This document provides a comprehensive reference for all JavaScript functions in the 30-Day Math Journey application. Functions are organized by module with detailed parameter specifications, return values, and usage examples.

## Table of Contents

1. [State Management Functions](#state-management-functions)
2. [Progress Tracking Functions](#progress-tracking-functions)
3. [Flashcard System Functions](#flashcard-system-functions)
4. [Geometry Functions](#geometry-functions)
5. [Graphing & Parsing Functions](#graphing--parsing-functions)
6. [Teach-Back Functions](#teach-back-functions)
7. [UI & Navigation Functions](#ui--navigation-functions)
8. [Utility Functions](#utility-functions)
9. [Event Handler Functions](#event-handler-functions)

---

## State Management Functions

### `createInitialState()`

Creates a fresh application state object with default values for all modules.

**Parameters:** None

**Returns:** `Object` - Complete initial state structure

**Return Structure:**
```javascript
{
  progress: { completed: boolean[30] },
  currentLessonIndex: number,
  flashcards: { deck: Flashcard[], stats: {} },
  settings: { highContrast: boolean },
  teachBack: { reflections: [] }
}
```

**Example:**
```javascript
const freshState = createInitialState();
console.log(freshState.progress.completed); // [false, false, ..., false] (30 items)
console.log(freshState.flashcards.deck.length); // 10 (default cards)
```

### `loadState()`

Loads application state from localStorage with migration safety and error handling.

**Parameters:** None

**Returns:** `Object` - Loaded state or initial state if load fails

**Error Handling:**
- JSON parse errors return initial state
- Missing properties are merged with defaults
- Logs warnings for recovery attempts

**Example:**
```javascript
const currentState = loadState();
// Automatically handles missing or corrupted data
```

### `saveState()`

Persists current application state to localStorage and triggers UI updates.

**Parameters:** None

**Returns:** `void`

**Side Effects:**
- Writes to localStorage with key `math_journey_v1`
- Calls `updateProgressUI()` to refresh displays
- No error throwing on storage failure

**Example:**
```javascript
state.progress.completed[0] = true;
saveState(); // Persists change and updates progress bar
```

### `subscribe(listener)`

Registers a callback function to be called when application state changes.

**Parameters:**
- `listener: Function` - Callback that receives state object

**Returns:** `void`

**Example:**
```javascript
subscribe((newState) => {
  console.log('Progress:', newState.progress.completed.filter(Boolean).length);
});
```

### `notify()`

Triggers all registered state change listeners with current state.

**Parameters:** None

**Returns:** `void`

**Example:**
```javascript
state.currentLessonIndex = 5;
notify(); // Calls all subscribed listeners
```

---

## Progress Tracking Functions

### `updateProgressUI()`

Refreshes all progress-related UI elements including header bar, stages, and lessons.

**Parameters:** None

**Returns:** `void`

**Updates:**
- Progress bar width and percentage text
- Milestone messages based on completion
- Stage completion percentages
- Lesson grid with current/completed styling

**Example:**
```javascript
state.progress.completed[4] = true;
updateProgressUI(); // Reflects lesson 5 completion
```

### `getCurrentLessonIndex()`

Determines the index of the current lesson based on completion status.

**Parameters:** None

**Returns:** `number` - Zero-based lesson index (0-29)

**Logic:**
- Returns index of first incomplete lesson
- Returns 29 (last lesson) if all lessons complete

**Example:**
```javascript
const current = getCurrentLessonIndex();
console.log(`Working on lesson ${current + 1}`);
```

### `selectLesson(index)`

Sets a specific lesson as active and updates lesson detail display.

**Parameters:**
- `index: number` - Zero-based lesson index (0-29)

**Returns:** `void`

**Side Effects:**
- Updates `state.currentLessonIndex`
- Calls `saveState()`
- Renders lesson details and action buttons

**Example:**
```javascript
selectLesson(9); // Jump to lesson 10 (Stage 1 final)
```

### `renderCurrentLesson()`

Generates and displays the current lesson detail panel with objectives and activities.

**Parameters:** None

**Returns:** `void`

**Generated Content:**
- Lesson title and stage information
- Learning objectives and activities
- Action buttons (complete, flashcards, graphing)

**Example:**
```javascript
state.currentLessonIndex = 15;
renderCurrentLesson(); // Shows lesson 16 details
```

### `celebrateIfMilestone()`

Checks if current completion count hits milestone and launches confetti if so.

**Parameters:** None

**Returns:** `void`

**Milestones:** `[1, 5, 10, 15, 20, 25, 30]` lessons completed

**Example:**
```javascript
state.progress.completed[4] = true; // Complete lesson 5
celebrateIfMilestone(); // Triggers confetti for 5-lesson milestone
```

### `getStageIdForDay(day)`

Determines which stage (1-4) contains the given lesson day.

**Parameters:**
- `day: number` - Lesson day number (1-30)

**Returns:** `number` - Stage ID (1-4)

**Stage Ranges:**
- Stage 1: Days 1-10
- Stage 2: Days 11-18  
- Stage 3: Days 19-25
- Stage 4: Days 26-30

**Example:**
```javascript
const stage = getStageIdForDay(15); // Returns 2
```

### `milestoneMessage(pct)`

Generates encouraging message based on completion percentage.

**Parameters:**
- `pct: number` - Completion percentage (0-1)

**Returns:** `string` - Milestone message

**Messages:**
- 0%: "Let's begin"
- 25%: "Great start—25% complete!"
- 50%: "Halfway there—keep going!"
- 75%: "75% milestone—momentum is yours!"
- 100%: "Incredible! You finished. 🎉"

**Example:**
```javascript
const msg = milestoneMessage(0.6); // "Halfway there—keep going!"
```

### `suggestExpressionForLesson(day)`

Provides appropriate graphing expression based on lesson content.

**Parameters:**
- `day: number` - Lesson day number (1-30)

**Returns:** `string` - Mathematical expression

**Expression Selection:**
- Days 1-10: Linear expressions (`2x + 3`)
- Days 11-18: Trigonometric (`sin(x) + cos(2x)`)
- Days 19-25: Quadratic (`x^2 - 4x + 1`)
- Days 26-30: Advanced (`sqrt(x) * sin(x)`)

**Example:**
```javascript
const expr = suggestExpressionForLesson(12); // "sin(x) + cos(2x)"
```

---

## Flashcard System Functions

### `getCardStats(id)`

Retrieves or creates spaced repetition statistics for a flashcard.

**Parameters:**
- `id: string` - Unique card identifier

**Returns:** `CardStats` - Statistics object

**Default Stats:**
```javascript
{
  ease: 2.4,        // Difficulty factor (1.3-3.0)
  intervalDays: 0,  // Days until next review
  reps: 0,          // Successful repetitions
  due: Date.now(),  // Due timestamp
  lapse: 0          // Number of "again" ratings
}
```

**Example:**
```javascript
const stats = getCardStats('card_abc123');
console.log(`Due: ${new Date(stats.due)}, Ease: ${stats.ease}`);
```

### `scheduleCards()`

Builds priority queue of due flashcards and updates UI counters.

**Parameters:** None

**Returns:** `void`

**Scheduling Logic:**
1. Priority to cards with `due <= now`
2. Include up to 3 cards not yet due (nearest first)
3. Ensure at least one card available if deck not empty

**Side Effects:**
- Updates `dueQueue` array
- Resets `currentCardIndex` to 0
- Calls `updateFlashcardUI()`

**Example:**
```javascript
scheduleCards(); // Rebuilds queue after card rating
```

### `updateFlashcardUI()`

Refreshes flashcard display with current card content and progress indicators.

**Parameters:** None

**Returns:** `void`

**Updates:**
- Card front/back content with sanitization
- Due count and progress indicators
- Removes flip state for new card

**Example:**
```javascript
updateFlashcardUI(); // Shows next card in queue
```

### `rateCurrentCard(grade)`

Applies spaced repetition algorithm based on user's difficulty rating.

**Parameters:**
- `grade: number` - Difficulty rating (1-4)
  - 1: Again (failed, show in 10 minutes)
  - 2: Hard (reduce ease, shorter interval)
  - 3: Good (normal progression)
  - 4: Easy (accelerated progression)

**Returns:** `void`

**Algorithm Details:**
```javascript
// Grade 1: Again
stats.ease = Math.max(1.3, stats.ease - 0.2);
stats.reps = 0;
stats.due = now + 10 * 60 * 1000; // 10 minutes

// Grade 2: Hard
stats.ease = Math.max(1.3, stats.ease - 0.15);
stats.intervalDays = Math.max(1, Math.round(stats.intervalDays * 0.5));

// Grade 3: Good
stats.ease = Math.min(2.8, stats.ease + 0.1);
if (stats.reps === 1) stats.intervalDays = 1;
else if (stats.reps === 2) stats.intervalDays = 6;
else stats.intervalDays = Math.round(stats.intervalDays * stats.ease);

// Grade 4: Easy  
stats.ease = Math.min(3.0, stats.ease + 0.15);
// Similar to Good but with bonus multiplier
```

**Example:**
```javascript
rateCurrentCard(3); // Mark current card as "Good"
```

### `getDefaultDeck()`

Creates initial flashcard deck with cards covering all learning stages.

**Parameters:** None

**Returns:** `Flashcard[]` - Array of flashcard objects

**Card Structure:**
```javascript
{
  id: string,     // Unique identifier
  front: string,  // Question/prompt
  back: string,   // Answer/explanation
  tags: string[]  // Category tags
}
```

**Content Coverage:**
- Stage 1: PEMDAS, linear equations, slope
- Stage 2: Trigonometry, distance formula, quadratics
- Stage 3: Golden ratio, Fibonacci sequences
- Stage 4: Sacred geometry concepts

**Example:**
```javascript
const initialCards = getDefaultDeck();
console.log(`Created ${initialCards.length} default cards`);
```

### `sanitize(html)`

Basic HTML sanitization to prevent script injection in flashcard content.

**Parameters:**
- `html: string` - Raw HTML content

**Returns:** `string` - Sanitized HTML

**Sanitization:**
- Replaces `<script` tags with `&lt;script`
- Prevents code execution in user-generated cards

**Example:**
```javascript
const safe = sanitize('<script>alert("bad")</script>Hello');
// Returns: "&lt;script>alert("bad")&lt;/script>Hello"
```

---

## Geometry Functions

### `worldToScreen(point)`

Converts world coordinates to screen pixel coordinates accounting for zoom and pan.

**Parameters:**
- `point: {x: number, y: number}` - World coordinate point

**Returns:** `{x: number, y: number}` - Screen pixel coordinates

**Transformation:**
```javascript
const cx = canvas.width/2 + offset.x;
const cy = canvas.height/2 + offset.y;
return {
  x: cx + point.x * zoom,
  y: cy + point.y * zoom
};
```

**Example:**
```javascript
const screenPt = worldToScreen({x: 100, y: -50});
console.log(`Screen: (${screenPt.x}, ${screenPt.y})`);
```

### `screenToWorld(point)`

Converts screen pixel coordinates to world coordinates.

**Parameters:**
- `point: {x: number, y: number}` - Screen pixel coordinates

**Returns:** `{x: number, y: number}` - World coordinate point

**Example:**
```javascript
const worldPt = screenToWorld({x: 300, y: 200});
// Accounts for current zoom and pan state
```

### `drawGeometry()`

Main geometry rendering function that clears canvas and draws current shape.

**Parameters:** None

**Returns:** `void`

**Rendering Process:**
1. Clear canvas and draw coordinate axes
2. Render selected shape with styling
3. Draw interactive handles for manipulation
4. Update measurement display

**Example:**
```javascript
geomState.shape = 'triangle';
drawGeometry(); // Renders triangle with handles
```

### `drawTriangle()`, `drawRectangle()`, `drawCircle()`, `drawCube()`

Specialized rendering functions for each geometry type.

**Parameters:** None

**Returns:** `void`

**Shape-Specific Features:**
- **Triangle**: Three vertex handles, area via Heron's formula
- **Rectangle**: Four corner handles, width/height calculation
- **Circle**: Center and radius handles, circumference/area
- **Cube**: Isometric projection with rotation parameter

**Example:**
```javascript
geomState.shape = 'circle';
drawCircle(); // Renders circle with center and radius handles
```

### `updateGeomProps()`

Calculates and displays live measurements for the current geometric shape.

**Parameters:** None

**Returns:** `void`

**Calculations by Shape:**
- **Triangle**: Perimeter = sum of sides, Area = Heron's formula
- **Rectangle**: Perimeter = 2(w+h), Area = w×h
- **Circle**: Circumference = 2πr, Area = πr²
- **Cube**: Surface Area = 6s², Volume = s³

**Example:**
```javascript
updateGeomProps(); // Updates measurement display panel
```

### `pickHandleAt(x, y)`

Hit testing function to determine which interactive element is at screen coordinates.

**Parameters:**
- `x, y: number` - Screen pixel coordinates

**Returns:** `Object` - Hit result with type and optional index

**Return Types:**
- `{type: 'vertex', index: number}` - Triangle vertex
- `{type: 'rectCorner', index: number}` - Rectangle corner
- `{type: 'circleCenter'}` - Circle center
- `{type: 'circleEdge'}` - Circle radius handle
- `{type: 'cubeCenter'}` - Cube center
- `{type: 'shape'}` - General shape area (for panning)

**Example:**
```javascript
const hit = pickHandleAt(mouseX, mouseY);
if (hit.type === 'vertex') {
  console.log(`Clicked vertex ${hit.index}`);
}
```

### `drawHandle(point)`

Renders an interactive handle (small circle) at the given screen coordinates.

**Parameters:**
- `point: {x: number, y: number}` - Screen coordinates

**Returns:** `void`

**Styling:**
- White fill with semi-transparent black border
- 6px radius for good touch targets

**Example:**
```javascript
const screenPt = worldToScreen(vertex);
drawHandle(screenPt); // Draw vertex handle
```

### `resizeGeom()`

Responsive canvas sizing function that handles high DPI displays and triggers redraw.

**Parameters:** None

**Returns:** `void`

**Features:**
- Automatic devicePixelRatio scaling for crisp rendering
- ResizeObserver integration for container changes
- Maintains coordinate system during resize

**Example:**
```javascript
// Called automatically by ResizeObserver
resizeGeom(); // Adjusts canvas size and redraws
```

---

## Graphing & Parsing Functions

### `tokenize(expr)`

Converts mathematical expression string into array of tokens for parsing.

**Parameters:**
- `expr: string` - Mathematical expression (e.g., "2*sin(x) + 1")

**Returns:** `string[]` - Array of tokens

**Preprocessing:**
- Removes whitespace
- Inserts explicit multiplication: `2x` → `2*x`, `)(` → `)*(`
- Recognizes numbers, functions, operators, parentheses

**Example:**
```javascript
const tokens = tokenize("2sin(x) + pi");
// Returns: ['2', '*', 'sin', '(', 'x', ')', '+', 'pi']
```

### `toRPN(tokens)`

Converts infix token array to Reverse Polish Notation using Shunting-yard algorithm.

**Parameters:**
- `tokens: string[]` - Infix tokens from `tokenize()`

**Returns:** `string[]` - RPN token array

**Features:**
- Handles operator precedence: `^` > `*/` > `+-`
- Right-associative exponentiation
- Unary minus detection and handling
- Function and parentheses processing

**Example:**
```javascript
const tokens = ['2', '*', 'x', '+', '3'];
const rpn = toRPN(tokens);
// Returns: ['2', 'x', '*', '3', '+']
```

### `compile(expr)`

Complete parsing pipeline from expression string to evaluable RPN.

**Parameters:**
- `expr: string` - Mathematical expression

**Returns:** `string[]|null` - RPN array or null on parse error

**Error Handling:**
- Returns null for invalid syntax
- Logs warnings for debugging

**Example:**
```javascript
const compiled = compile("sin(x^2) + 3*x");
if (compiled) {
  // Expression parsed successfully
  const result = evalRPN(compiled, Math.PI);
}
```

### `evalRPN(rpn, x)`

Evaluates compiled RPN expression for given variable value.

**Parameters:**
- `rpn: string[]` - Compiled RPN expression
- `x: number` - Variable value

**Returns:** `number` - Calculated result

**Supported Operations:**
- Arithmetic: `+`, `-`, `*`, `/`, `^`
- Functions: `sin`, `cos`, `tan`, `sqrt`, `abs`, `ln`, `log`, `exp`
- Constants: `pi`, `e`
- Unary minus: `u-`

**Example:**
```javascript
const rpn = compile("x^2 + 2*x + 1");
const result = evalRPN(rpn, 3); // Returns 16 (3² + 2*3 + 1)
```

### `safeEval(rpn, x)`

Wrapper for `evalRPN` that handles exceptions and returns NaN for invalid results.

**Parameters:**
- `rpn: string[]` - Compiled RPN expression
- `x: number` - Variable value

**Returns:** `number` - Result or NaN if evaluation fails

**Example:**
```javascript
const result = safeEval(rpn, x);
if (isFinite(result)) {
  // Valid result
  drawPoint(x, result);
}
```

### `drawGraph()`

Main graph rendering function that draws coordinate plane and function curve.

**Parameters:** None

**Returns:** `void`

**Rendering Elements:**
- Adaptive grid with smart spacing
- Coordinate axes with numeric labels
- Function curve with anti-aliasing
- Intercept markers (red x-intercepts, green y-intercept)

**Example:**
```javascript
graphState.compiled = compile("sin(x)");
drawGraph(); // Renders sine curve with axes
```

### `findIntercepts(rpn)`

Analyzes function to find x and y intercepts within visible domain.

**Parameters:**
- `rpn: string[]` - Compiled expression

**Returns:** `{x: number[], y: number}` - Intercept coordinates

**Algorithm:**
- Scans domain for sign changes (x-intercepts)
- Refines roots using bisection method
- Evaluates f(0) for y-intercept

**Example:**
```javascript
const intercepts = findIntercepts(compile("x^2 - 4"));
console.log(`X-intercepts: ${intercepts.x}`); // [-2, 2]
console.log(`Y-intercept: ${intercepts.y}`);  // -4
```

### `refineRootBisection(rpn, a, b, iters)`

Uses bisection method to refine root location between two points.

**Parameters:**
- `rpn: string[]` - Compiled expression
- `a, b: number` - Bracket points with opposite signs
- `iters: number` - Maximum iterations (default 20)

**Returns:** `number` - Refined root location or NaN

**Convergence:**
- Stops when |f(x)| < 1e-6 or max iterations reached
- Requires f(a) and f(b) to have opposite signs

**Example:**
```javascript
const root = refineRootBisection(rpn, -1, 1, 20);
// Finds root near x=0 for functions crossing zero
```

### `trySymbolicSteps(expr, rpn)`

Attempts to detect function type and provide analytical solutions.

**Parameters:**
- `expr: string` - Original expression
- `rpn: string[]` - Compiled expression

**Returns:** `string[]|null` - Step-by-step solution or null

**Detection Methods:**
- **Linear**: Tests if second differences ≈ 0
- **Quadratic**: Tests if second differences are constant
- Provides appropriate formulas and solutions

**Example:**
```javascript
const steps = trySymbolicSteps("2*x + 3", rpn);
// Returns: ["Detected linear: f(x) ≈ 2.0000x + 3.0000", ...]
```

### `plotCurrentExpression()`

Parses input expression, compiles it, finds intercepts, and renders graph.

**Parameters:** None

**Returns:** `void`

**Process:**
1. Parse expression from input field
2. Compile to RPN and store in state
3. Find intercepts for current view
4. Generate step-by-step solution
5. Update graph and overlay displays

**Example:**
```javascript
exprInput.value = "x^2 - 2*x - 3";
plotCurrentExpression(); // Parses and graphs the quadratic
```

### `niceGridStep(scalePxPerUnit)`

Calculates visually appropriate grid spacing based on current zoom level.

**Parameters:**
- `scalePxPerUnit: number` - Current zoom scale

**Returns:** `number` - Grid step size in world units

**Algorithm:**
- Targets approximately 60px between grid lines
- Rounds to nice values: 1, 2, 5, 10 × 10^n

**Example:**
```javascript
const step = niceGridStep(30); // Returns appropriate step for 30px/unit
```

---

## Teach-Back Functions

### `populateTeachConcepts()`

Fills the concept selection dropdown with available mathematical topics.

**Parameters:** None

**Returns:** `void`

**Available Concepts:**
- Order of Operations (PEMDAS)
- Solving Linear Equations
- Right Triangle Trigonometry  
- Quadratics & Roots
- Area and Perimeter

**Example:**
```javascript
populateTeachConcepts(); // Populates <select> with options
```

### `randomPromptForConcept(concept)`

Generates random teaching prompt to encourage varied explanation approaches.

**Parameters:**
- `concept: Object` - Concept definition with name and hints

**Returns:** `string` - Random prompt text

**Prompt Variations:**
- "Explain this concept as if to a 12-year-old with one real-life example"
- "Teach it to a colleague returning to math after years; focus on confidence"  
- "Write a step-by-step method in under 6 sentences"
- "Describe a common mistake and how to avoid it"

**Example:**
```javascript
const prompt = randomPromptForConcept(concepts[0]);
console.log(prompt); // Random teaching challenge
```

### `updateTeachPrompt()`

Updates current teaching prompt and resets progressive hint system.

**Parameters:** None

**Returns:** `void`

**Process:**
1. Get selected concept from dropdown
2. Generate random prompt
3. Reset hints to show only first hint
4. Configure hint reveal buttons

**Example:**
```javascript
updateTeachPrompt(); // New prompt for current concept
```

### `addHint(text)`

Adds a hint item to the progressive hint list.

**Parameters:**
- `text: string` - Hint content to display

**Returns:** `void`

**Usage:**
- Called to reveal next hint in sequence
- Used to show full worked example

**Example:**
```javascript
addHint("Try rewriting 3 + 2 × 4 as 3 + (2 × 4)");
```

---

## UI & Navigation Functions

### `switchTab(id)`

Changes active application tab and initializes tab-specific functionality.

**Parameters:**
- `id: string` - Tab identifier ('progress', 'flash', 'geometry', 'graph', 'teach', 'settings')

**Returns:** `void`

**Side Effects:**
- Updates aria-current attributes on nav buttons
- Shows/hides appropriate content sections
- Triggers initialization for visual tabs (canvas rendering)

**Example:**
```javascript
switchTab('geometry'); // Switch to geometry lab
```

### `getActiveTabId()`

Determines which tab is currently visible.

**Parameters:** None

**Returns:** `string` - Active tab ID

**Logic:**
- Checks `hidden` attribute on each tab section
- Returns corresponding tab identifier

**Example:**
```javascript
const current = getActiveTabId();
if (current === 'tab-flash') {
  // Handle flashcard-specific keyboard events
}
```

### `applyContrastSetting()`

Applies high contrast visual theme based on user preference.

**Parameters:** None

**Returns:** `void`

**Theme Changes:**
- Toggles `high-contrast` class on document element
- Switches to black background with white text
- Updates all CSS custom properties for accessibility

**Example:**
```javascript
state.settings.highContrast = true;
applyContrastSetting(); // Apply high contrast theme
```

### `renderStages()`, `renderLessons()`

Generate and display stage completion cards and lesson grid.

**Parameters:** None

**Returns:** `void`

**Rendering:**
- **Stages**: Progress bars and completion percentages
- **Lessons**: Interactive cards with status indicators and click handlers

**Example:**
```javascript
renderStages();  // Update stage progress displays
renderLessons(); // Refresh lesson grid with current state
```

---

## Utility Functions

### `clamp(value, min, max)`

Constrains numeric value between specified bounds.

**Parameters:**
- `value: number` - Value to constrain
- `min: number` - Minimum allowed value
- `max: number` - Maximum allowed value

**Returns:** `number` - Clamped value

**Example:**
```javascript
const zoom = clamp(userZoom, 0.4, 5.0);    // Keep zoom reasonable
const alpha = clamp(transparency, 0, 1);    // Valid alpha range
```

### `lerp(a, b, t)`

Linear interpolation between two values.

**Parameters:**
- `a: number` - Start value
- `b: number` - End value
- `t: number` - Interpolation parameter (0 = a, 1 = b)

**Returns:** `number` - Interpolated value

**Example:**
```javascript
const midpoint = lerp(10, 20, 0.5);      // Returns 15
const quarter = lerp(0, 100, 0.25);      // Returns 25
```

### `formatPercent(decimal)`

Formats decimal value as rounded percentage string.

**Parameters:**
- `decimal: number` - Decimal value (0.75 = 75%)

**Returns:** `string` - Formatted percentage with % symbol

**Example:**
```javascript
const display = formatPercent(0.678);     // "68%"
const progress = formatPercent(23/30);    // "77%"
```

### `uid()`

Generates unique identifier combining random string and timestamp.

**Parameters:** None

**Returns:** `string` - Unique identifier

**Format:** Random base36 string + timestamp in base36

**Example:**
```javascript
const cardId = uid();                     // "abc123def456"
const uniqueKey = uid();                  // Always different
```

### `dedupeSorted(arr)`

Removes duplicate values from sorted numeric array within tolerance.

**Parameters:**
- `arr: number[]` - Array of numbers

**Returns:** `number[]` - Deduplicated array

**Tolerance:** 1e-3 (removes values closer than 0.001)

**Example:**
```javascript
const roots = [1.9999, 2.0001, 3.0];
const cleaned = dedupeSorted(roots);      // [2.0, 3.0]
```

---

## Event Handler Functions

### `geomPointerDown(e)`, `geomPointerMove(e)`, `geomPointerUp()`

Geometry canvas interaction handlers for mouse and touch events.

**Parameters:**
- `e: Event` - Mouse or touch event object

**Returns:** `void`

**Functionality:**
- **Down**: Hit testing and drag initialization
- **Move**: Shape manipulation based on drag type
- **Up**: Cleanup and drag termination

**Example:**
```javascript
geomCanvas.addEventListener('mousedown', geomPointerDown);
geomCanvas.addEventListener('mousemove', geomPointerMove);
geomCanvas.addEventListener('mouseup', geomPointerUp);
```

### Confetti Animation Functions

### `launchConfetti()`

Creates particle system for milestone celebrations.

**Parameters:** None

**Returns:** `void`

**Features:**
- 180 particles with random colors and physics
- Gravity and wind effects
- Automatic cleanup when particles expire

**Example:**
```javascript
launchConfetti(); // Triggered on lesson milestones
```

### `confettiLoop()`

Animation loop for updating and rendering confetti particles.

**Parameters:** None

**Returns:** `void`

**Physics:**
- Position updates with velocity
- Rotation animation
- Particle lifetime management

**Example:**
```javascript
// Called automatically by requestAnimationFrame
confettiLoop(); // Updates all active particles
```

### `resizeConfetti()`

Adjusts confetti canvas size to match window dimensions.

**Parameters:** None

**Returns:** `void`

**Example:**
```javascript
window.addEventListener('resize', resizeConfetti);
```

---

## Input Validation & Sanitization

### `sanitize(html)`

Basic HTML sanitization for user-generated content.

**Parameters:**
- `html: string` - Raw HTML content

**Returns:** `string` - Sanitized HTML

**Protection:**
- Escapes `<script` tags to prevent code execution
- Maintains other HTML formatting for flashcards

**Example:**
```javascript
const userInput = '<script>alert("bad")</script><b>Math</b>';
const safe = sanitize(userInput);
// Returns: "&lt;script>alert("bad")&lt;/script><b>Math</b>"
```

### Error Handling Patterns

Most functions include error handling appropriate to their context:

**State Functions:**
- localStorage errors fall back to default state
- JSON parse errors return initial values

**Canvas Functions:**
- Invalid coordinates are clamped to valid ranges
- Division by zero returns NaN for mathematical correctness

**Parser Functions:**
- Syntax errors return null for graceful degradation
- Mathematical errors (sqrt(-1)) return NaN

---

This comprehensive function reference provides detailed specifications for all JavaScript functions in the 30-Day Math Journey application, enabling effective customization and debugging.