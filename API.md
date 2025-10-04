# API Documentation

## Table of Contents
1. [State Management](#state-management)
2. [Progress System](#progress-system)
3. [Flashcard System](#flashcard-system)
4. [Geometry Module](#geometry-module)
5. [Graphing Module](#graphing-module)
6. [Teach-Back Module](#teach-back-module)
7. [Utility Functions](#utility-functions)
8. [Settings & Storage](#settings--storage)

---

## State Management

### Global State Object

The application uses a centralized state object persisted to localStorage.

```javascript
state = {
  progress: {
    completed: boolean[30]  // Array tracking completion of 30 lessons
  },
  currentLessonIndex: number,
  flashcards: {
    deck: Flashcard[],
    stats: Record<string, CardStats>
  },
  settings: {
    highContrast: boolean
  },
  teachBack: {
    reflections: Reflection[]
  }
}
```

### Core State Functions

#### `createInitialState()`
Creates and returns a fresh state object with default values.

**Returns:** `State` - Initial application state

**Example:**
```javascript
const freshState = createInitialState();
// Returns state with 30 incomplete lessons, default deck, etc.
```

---

#### `loadState()`
Loads persisted state from localStorage or creates initial state if none exists.

**Returns:** `State` - Loaded or initial state

**Example:**
```javascript
const state = loadState();
// Retrieves persisted state or creates new one
```

---

#### `saveState()`
Saves current state to localStorage and updates UI.

**Side Effects:**
- Writes to localStorage under `APP_KEY`
- Triggers `updateProgressUI()`

**Example:**
```javascript
state.progress.completed[0] = true;
saveState();
// Persists change and updates UI
```

---

#### `subscribe(listener)`
Adds a listener function that gets called when state changes.

**Parameters:**
- `listener: (state: State) => void` - Callback function

**Example:**
```javascript
subscribe((newState) => {
  console.log('State changed:', newState);
});
```

---

#### `notify()`
Notifies all subscribed listeners of state changes.

**Example:**
```javascript
state.currentLessonIndex = 5;
notify(); // All listeners are called
```

---

## Progress System

### Data Structures

#### `Stage`
```javascript
{
  id: number,           // Stage identifier (1-4)
  title: string,        // Display name
  range: [number, number], // [startDay, endDay]
  color: string         // Hex color code
}
```

#### `Lesson`
```javascript
{
  day: number,          // Day number (1-30)
  title: string,        // Lesson title
  objective: string,    // Learning objective
  activity: string      // Quick activity description
}
```

### Functions

#### `updateProgressUI()`
Updates all progress-related UI elements including progress bar, stage indicators, and lesson grid.

**Side Effects:**
- Updates progress bar fill and text
- Re-renders stage completion indicators
- Re-renders lesson cards
- Updates milestone message

**Example:**
```javascript
state.progress.completed[5] = true;
updateProgressUI();
// UI reflects lesson 6 as completed
```

---

#### `getCurrentLessonIndex()`
Returns the index of the first incomplete lesson.

**Returns:** `number` - Index of current lesson (0-29)

**Example:**
```javascript
const currentIdx = getCurrentLessonIndex();
// Returns 0 if no lessons completed, 5 if first 5 done, etc.
```

---

#### `selectLesson(index)`
Selects a lesson by index and updates the UI.

**Parameters:**
- `index: number` - Lesson index (0-29)

**Side Effects:**
- Updates `state.currentLessonIndex`
- Saves state
- Renders current lesson details
- Updates UI

**Example:**
```javascript
selectLesson(10);
// Displays lesson 11 details
```

---

#### `renderCurrentLesson()`
Renders the detailed view of the currently selected lesson.

**Side Effects:**
- Updates `currentLessonDetailEl` with lesson info
- Attaches event listeners to action buttons

**Example:**
```javascript
renderCurrentLesson();
// Displays current lesson with objectives and activities
```

---

#### `celebrateIfMilestone()`
Triggers confetti animation if a milestone lesson is completed.

**Milestones:** 1, 5, 10, 15, 20, 25, 30

**Side Effects:**
- Launches confetti animation if milestone reached

**Example:**
```javascript
state.progress.completed[9] = true; // Lesson 10
celebrateIfMilestone();
// Confetti animation plays
```

---

## Flashcard System

### Data Structures

#### `Flashcard`
```javascript
{
  id: string,           // Unique identifier
  front: string,        // Question/prompt (HTML allowed)
  back: string,         // Answer/explanation (HTML allowed)
  tags: string[]        // Category tags
}
```

#### `CardStats`
```javascript
{
  ease: number,         // Ease factor (1.3-3.0)
  intervalDays: number, // Days until next review
  reps: number,         // Number of successful reviews
  due: number,          // Due timestamp (ms)
  lapse: number         // Number of times marked "Again"
}
```

### Spaced Repetition Algorithm

The system uses a simplified SM-2 algorithm:

**Grade Scale:**
1. **Again** - Complete memory lapse
2. **Hard** - Incorrect but remembered with effort
3. **Good** - Correct with some difficulty
4. **Easy** - Perfect recall

### Functions

#### `getDefaultDeck()`
Returns the default flashcard deck with stage-appropriate cards.

**Returns:** `Flashcard[]` - Array of default flashcards

**Example:**
```javascript
const defaultCards = getDefaultDeck();
// Returns 10+ pre-made flashcards
```

---

#### `getCardStats(id)`
Retrieves or creates statistics for a flashcard.

**Parameters:**
- `id: string` - Card identifier

**Returns:** `CardStats` - Statistics object for the card

**Example:**
```javascript
const stats = getCardStats('card_123');
// Returns stats or creates new with default values
```

---

#### `scheduleCards()`
Builds the due queue of flashcards prioritizing overdue cards.

**Algorithm:**
1. Separates due cards (due <= now)
2. Sorts not-due cards by due date
3. Sprinkles 3 upcoming cards into queue
4. Resets current card index

**Side Effects:**
- Updates `dueQueue` array
- Resets `currentCardIndex` to 0
- Calls `updateFlashcardUI()`

**Example:**
```javascript
scheduleCards();
// Rebuilds queue with due cards first
```

---

#### `rateCurrentCard(grade)`
Applies spaced repetition algorithm to current card based on rating.

**Parameters:**
- `grade: 1 | 2 | 3 | 4` - Quality of recall

**Grade Effects:**

**Again (1):**
- Ease: max(1.3, ease - 0.2)
- Interval: 0 days
- Due: now + 10 minutes
- Resets reps to 0

**Hard (2):**
- Ease: max(1.3, ease - 0.15)
- Interval: max(1, round(interval × 0.5))
- Maintains reps

**Good (3):**
- Ease: min(2.8, ease + 0.1)
- Interval: 1 day (first), 6 days (second), interval × ease (subsequent)
- Increments reps

**Easy (4):**
- Ease: min(3.0, ease + 0.15)
- Interval: 2 days (first), 8 days (second), interval × (ease + 0.1) (subsequent)
- Increments reps

**Side Effects:**
- Updates card statistics
- Saves state
- Removes card from due queue
- Updates UI

**Example:**
```javascript
rateCurrentCard(3); // Mark as "Good"
// Card scheduled for review based on algorithm
```

---

#### `updateFlashcardUI()`
Updates flashcard display with current card and queue stats.

**Side Effects:**
- Updates due count display
- Updates card progress counter
- Displays current card front/back
- Resets flip state

**Example:**
```javascript
updateFlashcardUI();
// UI shows current card and queue status
```

---

## Geometry Module

### State Object

```javascript
geomState = {
  shape: 'triangle' | 'rectangle' | 'circle' | 'cube',
  zoom: number,         // Scale factor (0.4-5.0)
  offset: {x, y},       // Pan offset in pixels
  dragging: null | {
    type: string,       // 'vertex' | 'rectCorner' | 'circleCenter' | 'circleEdge' | 'cubeCenter' | 'shape'
    index?: number,     // Vertex/corner index if applicable
    start: {x, y}       // Drag start position
  },
  vertices: Point[3],   // Triangle vertices
  rect: {x, y, w, h},   // Rectangle definition
  circle: {x, y, r},    // Circle definition
  cube: {
    center: {x, y},
    size: number,
    rot: number         // Rotation angle
  }
}
```

### Functions

#### `worldToScreen(p)`
Converts world coordinates to screen coordinates.

**Parameters:**
- `p: {x: number, y: number}` - World coordinates

**Returns:** `{x: number, y: number}` - Screen coordinates

**Example:**
```javascript
const screenPos = worldToScreen({x: 50, y: 50});
// Returns pixel coordinates on canvas
```

---

#### `screenToWorld(p)`
Converts screen coordinates to world coordinates.

**Parameters:**
- `p: {x: number, y: number}` - Screen coordinates

**Returns:** `{x: number, y: number}` - World coordinates

**Example:**
```javascript
const worldPos = screenToWorld({x: 300, y: 200});
// Returns world coordinates considering zoom and pan
```

---

#### `drawGeometry()`
Renders the current geometry shape with measurements and interactive handles.

**Side Effects:**
- Clears and redraws canvas
- Draws axes, shape, and handles
- Updates measurement display
- Updates concept notes

**Example:**
```javascript
geomState.shape = 'circle';
drawGeometry();
// Renders circle with area/circumference
```

---

#### `drawTriangle()`
Renders a triangle with draggable vertices.

**Calculations:**
- Perimeter: sum of three side lengths
- Area: Heron's formula

**Side Effects:**
- Draws filled and stroked triangle
- Draws vertex handles

---

#### `drawRectangle()`
Renders a rectangle with draggable corners.

**Calculations:**
- Perimeter: 2(w + h)
- Area: w × h

**Side Effects:**
- Draws filled and stroked rectangle
- Draws corner handles

---

#### `drawCircle()`
Renders a circle with draggable center and radius handle.

**Calculations:**
- Circumference: 2πr
- Area: πr²

**Side Effects:**
- Draws filled and stroked circle
- Draws center and edge handles

---

#### `drawCube()`
Renders an isometric cube projection.

**Calculations:**
- Surface Area: 6s²
- Volume: s³

**Side Effects:**
- Draws isometric wireframe cube
- Draws center handle

---

#### `updateGeomProps()`
Updates the measurement display and concept notes based on current shape.

**Side Effects:**
- Updates `geomPropsEl` with measurements
- Updates `geomNotesEl` with formulas

**Example:**
```javascript
updateGeomProps();
// Displays "Perimeter = 24.00, Area = 20.00"
```

---

## Graphing Module

### State Object

```javascript
graphState = {
  expr: string,         // Current expression
  compiled: Token[],    // Compiled RPN tokens
  zoom: number,         // Pixels per unit (20-400)
  origin: {x, y},       // Pan offset in pixels
  dragging: boolean,
  dragStart: {x, y},
  lastPan: {x, y},
  intercepts: {
    x: number[],        // x-intercepts
    y: number           // y-intercept
  },
  stepsHtml: string     // Solution steps HTML
}
```

### Expression Parser

#### Supported Operators
- **Arithmetic:** `+`, `-`, `*`, `/`, `^`
- **Functions:** `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `sqrt`, `abs`, `ln`, `log`, `exp`
- **Constants:** `pi`, `e`
- **Variable:** `x`

#### Implicit Multiplication
Automatically inserts `*` for:
- `2x` → `2*x`
- `2(x+1)` → `2*(x+1)`
- `)(` → `)*(`

### Functions

#### `tokenize(expr)`
Tokenizes a mathematical expression string.

**Parameters:**
- `expr: string` - Expression like "2x + sin(x)"

**Returns:** `string[]` - Array of tokens

**Example:**
```javascript
const tokens = tokenize("2x + 3");
// Returns ['2', '*', 'x', '+', '3']
```

---

#### `toRPN(tokens)`
Converts infix tokens to Reverse Polish Notation using Shunting-yard algorithm.

**Parameters:**
- `tokens: string[]` - Tokenized expression

**Returns:** `string[]` - RPN token array

**Example:**
```javascript
const rpn = toRPN(['2', '*', 'x', '+', '3']);
// Returns ['2', 'x', '*', '3', '+']
```

---

#### `compile(expr)`
Compiles an expression string to RPN for evaluation.

**Parameters:**
- `expr: string` - Mathematical expression

**Returns:** `string[] | null` - RPN array or null if parse error

**Example:**
```javascript
const compiled = compile("sin(x) + x^2");
// Returns RPN token array
```

---

#### `evalRPN(rpn, x)`
Evaluates RPN token array at given x value.

**Parameters:**
- `rpn: string[]` - RPN tokens
- `x: number` - Value to evaluate at

**Returns:** `number` - Result

**Example:**
```javascript
const result = evalRPN(['2', 'x', '*', '3', '+'], 5);
// Returns 13 (2*5 + 3)
```

---

#### `safeEval(rpn, x)`
Safely evaluates expression, returning NaN on error.

**Parameters:**
- `rpn: string[]` - RPN tokens
- `x: number` - Value to evaluate at

**Returns:** `number` - Result or NaN

**Example:**
```javascript
const y = safeEval(compiled, 2.5);
// Returns value or NaN if error
```

---

#### `findIntercepts(rpn)`
Finds x and y intercepts of the graphed function.

**Parameters:**
- `rpn: string[]` - Compiled expression

**Returns:** `{x: number[], y: number}` - Intercept coordinates

**Algorithm:**
1. Scans domain for sign changes
2. Uses bisection to refine roots
3. Evaluates at x=0 for y-intercept
4. Deduplicates nearby roots

**Example:**
```javascript
const intercepts = findIntercepts(compiled);
// Returns {x: [-1.5, 2.3], y: 4.0}
```

---

#### `refineRootBisection(rpn, a, b, iters)`
Refines a root estimate using bisection method.

**Parameters:**
- `rpn: string[]` - Compiled expression
- `a: number` - Left bound
- `b: number` - Right bound
- `iters: number` - Maximum iterations (default: 20)

**Returns:** `number` - Refined root or NaN

**Example:**
```javascript
const root = refineRootBisection(compiled, -1, 0, 20);
// Returns precise x-intercept between a and b
```

---

#### `trySymbolicSteps(expr, rpn)`
Attempts to detect linear or quadratic patterns and provide symbolic solution steps.

**Parameters:**
- `expr: string` - Original expression
- `rpn: string[]` - Compiled expression

**Returns:** `string[] | null` - Array of step descriptions or null

**Detection:**
- **Linear:** Tests if second difference ≈ 0
- **Quadratic:** Tests if second difference is constant

**Example:**
```javascript
const steps = trySymbolicSteps("2x + 3", compiled);
// Returns ["Detected linear: f(x) ≈ 2x + 3", ...]
```

---

#### `plotCurrentExpression()`
Parses, compiles, and graphs the expression from input field.

**Side Effects:**
- Compiles expression
- Finds intercepts
- Generates solution steps
- Updates graph display
- Shows step overlay

**Example:**
```javascript
exprInput.value = "x^2 - 4";
plotCurrentExpression();
// Graphs parabola with intercepts highlighted
```

---

#### `drawGraph()`
Renders the graph canvas with axes, grid, and function plot.

**Features:**
- Dynamic grid sizing
- Axis labels at major gridlines
- Function curve rendering
- Intercept highlighting (red: x-intercepts, green: y-intercept)

**Side Effects:**
- Clears and redraws canvas
- Updates visual representation

**Example:**
```javascript
drawGraph();
// Renders current graphState
```

---

## Teach-Back Module

### Data Structures

#### `Concept`
```javascript
{
  key: string,          // Unique identifier
  name: string,         // Display name
  hints: string[],      // Progressive hint array
  example: string       // Full example answer
}
```

#### `Reflection`
```javascript
{
  concept: string,      // Concept name
  prompt: string,       // Generated prompt
  text: string,         // User's explanation
  timestamp: number,    // Creation time
  selfScore?: number    // Self-assessment (1-3)
}
```

### Concepts Array

Pre-defined concepts:
- **PEMDAS** - Order of Operations
- **Linear Equations** - Solving for x
- **Right Triangle Trig** - SOH-CAH-TOA
- **Quadratics** - Roots and discriminant
- **Geometry** - Area and perimeter

### Functions

#### `populateTeachConcepts()`
Populates the concept select dropdown.

**Side Effects:**
- Updates `teachConceptSelect` with options

---

#### `randomPromptForConcept(concept)`
Generates a random teaching prompt for a concept.

**Parameters:**
- `concept: Concept` - Target concept

**Returns:** `string` - Random prompt

**Prompts:**
- Explain to a 12-year-old with example
- Teach to colleague with confidence focus
- Write step-by-step in 6 sentences
- Describe common mistake and fix

**Example:**
```javascript
const prompt = randomPromptForConcept(concepts[0]);
// Returns "Explain this concept as if to a 12-year-old..."
```

---

#### `updateTeachPrompt()`
Updates the teach-back prompt display and resets hints.

**Side Effects:**
- Updates prompt text
- Resets hint list to first hint
- Sets up hint reveal buttons

**Example:**
```javascript
updateTeachPrompt();
// Displays new prompt with first hint
```

---

#### `addHint(text)`
Adds a hint to the hint list display.

**Parameters:**
- `text: string` - Hint text

**Side Effects:**
- Appends `<li>` to hint list

**Example:**
```javascript
addHint("Start by identifying the coefficient");
// Adds hint to visible list
```

---

## Utility Functions

#### `clamp(v, min, max)`
Constrains a value between min and max.

**Parameters:**
- `v: number` - Value to clamp
- `min: number` - Minimum bound
- `max: number` - Maximum bound

**Returns:** `number` - Clamped value

**Example:**
```javascript
clamp(15, 0, 10); // Returns 10
clamp(-5, 0, 10); // Returns 0
clamp(5, 0, 10);  // Returns 5
```

---

#### `lerp(a, b, t)`
Linear interpolation between two values.

**Parameters:**
- `a: number` - Start value
- `b: number` - End value
- `t: number` - Interpolation factor (0-1)

**Returns:** `number` - Interpolated value

**Example:**
```javascript
lerp(0, 100, 0.5);  // Returns 50
lerp(10, 20, 0.25); // Returns 12.5
```

---

#### `formatPercent(n)`
Formats a decimal as percentage string.

**Parameters:**
- `n: number` - Decimal value (0-1)

**Returns:** `string` - Formatted percentage

**Example:**
```javascript
formatPercent(0.75);   // Returns "75%"
formatPercent(0.3333); // Returns "33%"
```

---

#### `uid()`
Generates a unique identifier.

**Returns:** `string` - Unique ID

**Example:**
```javascript
const id = uid();
// Returns "abc123def456"
```

---

#### `sanitize(html)`
Light HTML sanitizer preventing script injection.

**Parameters:**
- `html: string` - HTML string

**Returns:** `string` - Sanitized HTML

**Example:**
```javascript
sanitize("<script>alert('xss')</script>text");
// Returns "&lt;script>alert('xss')&lt;/script>text"
```

---

## Settings & Storage

### Storage Key

```javascript
const APP_KEY = 'math_journey_v1';
```

All data is persisted to localStorage under this key.

### Functions

#### `applyContrastSetting()`
Applies high contrast mode based on settings.

**Side Effects:**
- Toggles `.high-contrast` class on `<html>`

**Example:**
```javascript
state.settings.highContrast = true;
applyContrastSetting();
// Applies high contrast theme
```

---

### Export/Import

#### Export Process
1. Serializes current state to JSON
2. Creates blob and download link
3. Downloads as `math_journey_export.json`

#### Import Process
1. Reads uploaded JSON file
2. Parses and validates data
3. Merges with initial state defaults
4. Saves and refreshes UI

---

## Keyboard Shortcuts

### Global
- **Shift + ?** - Show help dialog

### Flashcards Tab
- **Enter** - Flip card
- **1** - Rate as "Again"
- **2** - Rate as "Hard"
- **3** - Rate as "Good"
- **4** - Rate as "Easy"
- **S** - Shuffle deck

### Graphing Tab
- **Enter** (in input) - Plot expression
- **+** - Zoom in
- **-** - Zoom out
- **0** - Reset view
- **Arrow Keys** - Pan view
- **Drag** - Pan graph
- **Wheel** - Zoom

### Geometry Tab
- **Drag** - Move vertices/shape
- **Scroll** - Zoom

---

## Event System

### Tab Switching

#### `switchTab(id)`
Switches to the specified tab.

**Parameters:**
- `id: string` - Tab identifier ('progress' | 'flash' | 'geometry' | 'graph' | 'teach' | 'settings')

**Side Effects:**
- Updates aria-current attributes
- Shows/hides tab sections
- Reinitializes tab-specific features
- For flash: reschedules cards
- For geometry/graph: redraws canvas

**Example:**
```javascript
switchTab('flash');
// Shows flashcards tab and schedules due cards
```

---

#### `getActiveTabId()`
Returns the ID of the currently active tab.

**Returns:** `string` - Active tab ID

**Example:**
```javascript
const tab = getActiveTabId();
// Returns 'tab-progress' if on progress tab
```

---

## Celebration System

### Confetti Animation

#### `launchConfetti()`
Launches confetti particle animation.

**Configuration:**
- 180 particles
- Random colors: cyan, blue, green, amber, red
- Gravity simulation
- Auto-cleanup when particles exit

**Triggered by:** Milestone completion (lessons 1, 5, 10, 15, 20, 25, 30)

**Example:**
```javascript
launchConfetti();
// Particles rain down from top
```

---

#### `confettiLoop()`
Animation loop rendering confetti particles.

**Features:**
- Updates particle positions
- Applies rotation
- Fades and removes off-screen particles
- Uses requestAnimationFrame

---

## Initialization

### `init()`
Initializes the application on page load.

**Process:**
1. Applies contrast settings
2. Updates progress UI
3. Renders current lesson
4. Schedules flashcards
5. Resizes geometry canvas
6. Resizes graph canvas
7. Populates teach concepts
8. Updates teach prompt

**Example:**
```javascript
window.addEventListener('load', init);
// App initializes when DOM ready
```

---

## Canvas Rendering

### Device Pixel Ratio Handling

Both geometry and graph canvases handle high-DPI displays:

```javascript
canvas.width = rect.width * devicePixelRatio;
canvas.height = rect.height * devicePixelRatio;
ctx.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
```

This ensures crisp rendering on retina displays.

### ResizeObserver

Both canvases use ResizeObserver for responsive sizing:

```javascript
new ResizeObserver(resizeFunction).observe(wrapElement);
```

Automatically redraws when container size changes.

---

## Error Handling

### Expression Parser
- Returns `null` on parse error
- Logs warning to console
- Displays error message in UI

### Expression Evaluator
- Returns `NaN` on evaluation error
- Skips rendering invalid points
- Breaks curves at discontinuities

### State Loading
- Falls back to initial state on parse error
- Logs warning to console
- Shallow merges with defaults for migration safety

---

## Accessibility Features

### ARIA Labels
- Progress bar: `aria-label="Overall progress"`
- Flashcard: `aria-label="Flashcard. Press Enter to flip."`
- Canvases: `role="img"` with descriptive labels
- Tab buttons: `aria-current="page"` for active tab

### Keyboard Navigation
- All interactive elements focusable
- Visible focus indicators: `outline: 2px dashed var(--focus)`
- Keyboard shortcuts for major actions
- Tab navigation supported

### High Contrast Mode
- Toggle in settings
- Replaces colors with high-contrast alternatives
- Increases text brightness
- Enhances border visibility

---

## Performance Considerations

### Canvas Optimization
- Pixel-aligned rendering for crisp lines
- Request animation frame for smooth animations
- Clears only affected regions when possible
- Device pixel ratio scaling

### State Management
- Batched UI updates
- Debounced save operations via localStorage
- Lazy evaluation of complex calculations

### Expression Evaluation
- Compiled RPN for fast repeated evaluation
- Adaptive sampling based on zoom
- Early termination for invalid values
- Bisection with iteration limits

---

## Browser Compatibility

### Required Features
- ES6+ JavaScript
- localStorage
- Canvas 2D context
- ResizeObserver
- Touch events (mobile)
- CSS custom properties
- CSS Grid & Flexbox

### Supported Browsers
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

---

## Security

### XSS Prevention
- `sanitize()` function prevents script injection in flashcards
- No use of `eval()` or `innerHTML` with untrusted content
- Expression parser uses controlled token evaluation

### Data Privacy
- All data stored locally in localStorage
- No external API calls
- No telemetry or tracking
- Export/import uses client-side only

---

## Extension Points

### Adding Custom Flashcards
```javascript
state.flashcards.deck.push({
  id: uid(),
  front: "Your question",
  back: "Your answer",
  tags: ['custom']
});
saveState();
```

### Adding Custom Lessons
```javascript
lessons.push({
  day: 31,
  title: "Custom Lesson",
  objective: "Learn something new",
  activity: "Practice activity"
});
```

### Adding Custom Math Functions
```javascript
FUNCTIONS.customFn = (x) => x * 2 + 1;
// Now usable in expressions: "customFn(x)"
```

### Adding Custom Teach-Back Concepts
```javascript
concepts.push({
  key: 'custom',
  name: 'Custom Concept',
  hints: ['Hint 1', 'Hint 2', 'Hint 3'],
  example: 'Full example explanation'
});
```
