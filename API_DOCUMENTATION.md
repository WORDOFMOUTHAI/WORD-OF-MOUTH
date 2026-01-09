# 30-Day Math Journey - Comprehensive API Documentation

## Table of Contents
1. [Overview](#overview)
2. [Application State Management](#application-state-management)
3. [Lesson and Stage System](#lesson-and-stage-system)
4. [Flashcard Engine](#flashcard-engine)
5. [Geometry Canvas System](#geometry-canvas-system)
6. [Function Graphing System](#function-graphing-system)
7. [Teach-Back Module](#teach-back-module)
8. [UI Components and Tab Management](#ui-components-and-tab-management)
9. [Utility Functions](#utility-functions)
10. [Event Handlers](#event-handlers)
11. [Usage Examples](#usage-examples)

## Overview

The 30-Day Math Journey is a comprehensive single-file HTML application designed for adults returning to mathematics. It features:

- **30 structured lessons** across 4 progressive stages
- **Spaced repetition flashcards** with SM-2 inspired algorithm
- **Interactive geometry canvas** with 2D shapes and isometric 3D
- **Function graphing system** with expression parsing and step-by-step solutions
- **Teach-back module** for concept explanation and self-assessment
- **Progress tracking** with milestone celebrations
- **Accessibility features** including high contrast mode and keyboard navigation

## Application State Management

### State Structure

The application uses a centralized state object stored in localStorage with the following structure:

```javascript
const state = {
  progress: {
    completed: boolean[30]  // Array tracking completion of 30 lessons
  },
  currentLessonIndex: number,  // Currently selected lesson (0-29)
  flashcards: {
    deck: Flashcard[],  // Array of flashcard objects
    stats: Record<id, CardStats>  // Performance statistics per card
  },
  settings: {
    highContrast: boolean  // Accessibility setting
  },
  teachBack: {
    reflections: Array<{concept:string, prompt:string, text:string, timestamp:number, selfScore?:number}>
  }
}
```

### Core State Functions

#### `createInitialState()`
Creates a fresh state object with default values.

**Returns:** `Object` - Initial state object

**Example:**
```javascript
const initialState = createInitialState();
console.log(initialState.progress.completed.length); // 30
```

#### `loadState()`
Loads state from localStorage with fallback to initial state.

**Returns:** `Object` - Loaded or initial state

**Example:**
```javascript
const currentState = loadState();
console.log(`Completed lessons: ${currentState.progress.completed.filter(Boolean).length}`);
```

#### `saveState()`
Persists current state to localStorage and updates UI.

**Example:**
```javascript
state.progress.completed[0] = true;
saveState(); // Saves to localStorage and updates progress UI
```

### Pub/Sub System

#### `subscribe(listener)`
Registers a function to be called when state changes.

**Parameters:**
- `listener` (Function): Callback function to execute on state changes

**Example:**
```javascript
subscribe((newState) => {
  console.log('State updated:', newState);
});
```

#### `notify()`
Triggers all registered listeners with current state.

**Example:**
```javascript
// After modifying state
notify(); // Calls all subscribed listeners
```

## Lesson and Stage System

### Stage Definitions

The application defines 4 learning stages:

```javascript
const stages = [
  { id: 1, title: 'Stage 1: Foundation Reconstruction', range: [1, 10], color: '#7dd3fc' },
  { id: 2, title: 'Stage 2: Intermediate Integration', range: [11, 18], color: '#60a5fa' },
  { id: 3, title: 'Stage 3: Classical Mathematical Thinking', range: [19, 25], color: '#34d399' },
  { id: 4, title: 'Stage 4: Sacred Geometry Synthesis', range: [26, 30], color: '#f59e0b' }
];
```

### Lesson Structure

Each lesson contains:

```javascript
{
  day: number,        // Lesson number (1-30)
  title: string,      // Lesson title
  objective: string,   // Learning objective
  activity: string    // Quick practice activity
}
```

### Core Lesson Functions

#### `getStageIdForDay(day)`
Determines which stage a lesson belongs to.

**Parameters:**
- `day` (number): Lesson number (1-30)

**Returns:** `number` - Stage ID (1-4)

**Example:**
```javascript
const stageId = getStageIdForDay(15); // Returns 2
```

#### `getCurrentLessonIndex()`
Finds the index of the next incomplete lesson.

**Returns:** `number` - Index of current lesson (0-29)

**Example:**
```javascript
const currentIndex = getCurrentLessonIndex();
console.log(`Current lesson: ${lessons[currentIndex].title}`);
```

#### `selectLesson(index)`
Sets the currently selected lesson and updates UI.

**Parameters:**
- `index` (number): Lesson index (0-29)

**Example:**
```javascript
selectLesson(5); // Selects lesson 6
```

#### `renderStages()`
Updates the stages display with completion percentages.

**Example:**
```javascript
renderStages(); // Refreshes stage progress cards
```

#### `renderLessons()`
Updates the lesson grid with completion status and current lesson highlighting.

**Example:**
```javascript
renderLessons(); // Refreshes lesson cards
```

#### `renderCurrentLesson()`
Displays detailed information for the selected lesson.

**Example:**
```javascript
renderCurrentLesson(); // Shows lesson details and action buttons
```

#### `suggestExpressionForLesson(day)`
Provides a sample mathematical expression based on lesson content.

**Parameters:**
- `day` (number): Lesson number (1-30)

**Returns:** `string` - Suggested expression for graphing

**Example:**
```javascript
const expr = suggestExpressionForLesson(12); // Returns "sin(x) + cos(2x)"
```

## Flashcard Engine

### Card Structure

```javascript
{
  id: string,      // Unique identifier
  front: string,   // Question/prompt
  back: string,    // Answer/explanation
  tags: string[]   // Categories (e.g., ['stage1', 'trig'])
}
```

### Card Statistics

```javascript
{
  ease: number,        // Difficulty factor (1.3-3.0)
  intervalDays: number, // Days until next review
  reps: number,        // Number of successful reviews
  due: number,         // Timestamp when card is due
  lapse: number        // Number of failed reviews
}
```

### Core Flashcard Functions

#### `getDefaultDeck()`
Creates the initial set of flashcards covering all stages.

**Returns:** `Array<Flashcard>` - Default flashcard deck

**Example:**
```javascript
const deck = getDefaultDeck();
console.log(`Default deck has ${deck.length} cards`);
```

#### `getCardStats(id)`
Retrieves or creates statistics for a specific card.

**Parameters:**
- `id` (string): Card identifier

**Returns:** `CardStats` - Card performance statistics

**Example:**
```javascript
const stats = getCardStats('card123');
console.log(`Card ease factor: ${stats.ease}`);
```

#### `scheduleCards()`
Builds the review queue prioritizing due cards.

**Example:**
```javascript
scheduleCards(); // Updates dueQueue and currentCardIndex
```

#### `updateFlashcardUI()`
Refreshes the flashcard display with current card and progress.

**Example:**
```javascript
updateFlashcardUI(); // Shows current card and progress stats
```

#### `rateCurrentCard(grade)`
Processes a card rating and updates its schedule.

**Parameters:**
- `grade` (number): Rating (1=Again, 2=Hard, 3=Good, 4=Easy)

**Example:**
```javascript
rateCurrentCard(3); // Marks current card as "Good"
```

### Spaced Repetition Algorithm

The system uses a simplified SM-2 algorithm:

- **Again (1):** Reset to 10 minutes, decrease ease
- **Hard (2):** Halve interval, decrease ease slightly
- **Good (3):** Normal progression, slight ease increase
- **Easy (4):** Accelerated progression, larger ease increase

## Geometry Canvas System

### Canvas State

```javascript
const geomState = {
  shape: string,           // 'triangle', 'rectangle', 'circle', 'cube'
  zoom: number,            // Zoom level (0.4-5)
  offset: { x: number, y: number }, // Pan offset
  dragging: Object|null,   // Current drag operation
  vertices: Array<{x:number, y:number}>, // Triangle vertices
  rect: { x:number, y:number, w:number, h:number }, // Rectangle
  circle: { x:number, y:number, r:number }, // Circle
  cube: { center:{x:number, y:number}, size:number, rot:number } // Cube
};
```

### Core Geometry Functions

#### `resizeGeom()`
Adjusts canvas size to match container dimensions.

**Example:**
```javascript
resizeGeom(); // Resizes canvas and redraws
```

#### `worldToScreen(p)`
Converts world coordinates to screen coordinates.

**Parameters:**
- `p` (Object): Point with x, y coordinates

**Returns:** `Object` - Screen coordinates

**Example:**
```javascript
const screenPoint = worldToScreen({x: 100, y: 50});
```

#### `screenToWorld(p)`
Converts screen coordinates to world coordinates.

**Parameters:**
- `p` (Object): Point with x, y coordinates

**Returns:** `Object` - World coordinates

**Example:**
```javascript
const worldPoint = screenToWorld({x: 300, y: 200});
```

#### `drawGeometry()`
Renders the current shape with measurements and handles.

**Example:**
```javascript
drawGeometry(); // Redraws entire geometry canvas
```

#### `drawTriangle()`
Renders triangle with draggable vertices.

**Example:**
```javascript
drawTriangle(); // Draws triangle with vertex handles
```

#### `drawRectangle()`
Renders rectangle with corner handles.

**Example:**
```javascript
drawRectangle(); // Draws rectangle with corner handles
```

#### `drawCircle()`
Renders circle with center and edge handles.

**Example:**
```javascript
drawCircle(); // Draws circle with radius handle
```

#### `drawCube()`
Renders isometric cube with center handle.

**Example:**
```javascript
drawCube(); // Draws 3D cube projection
```

#### `updateGeomProps()`
Calculates and displays shape measurements.

**Example:**
```javascript
updateGeomProps(); // Updates measurements display
```

#### `pickHandleAt(x, y)`
Determines which handle is at the given screen coordinates.

**Parameters:**
- `x` (number): Screen X coordinate
- `y` (number): Screen Y coordinate

**Returns:** `Object` - Handle information

**Example:**
```javascript
const handle = pickHandleAt(150, 200);
if (handle.type === 'vertex') {
  console.log(`Vertex ${handle.index} selected`);
}
```

## Function Graphing System

### Graph State

```javascript
const graphState = {
  expr: string,                    // Current expression
  compiled: Array|null,            // Parsed RPN tokens
  zoom: number,                    // Pixels per unit (20-400)
  origin: { x: number, y: number }, // Pan offset
  dragging: boolean,               // Pan state
  dragStart: { x: number, y: number }, // Drag start point
  lastPan: { x: number, y: number },   // Last pan position
  intercepts: { x: Array<number>, y: number }, // Function intercepts
  stepsHtml: string                // Solution steps HTML
};
```

### Expression Parser

#### `tokenize(expr)`
Converts mathematical expression to tokens.

**Parameters:**
- `expr` (string): Mathematical expression

**Returns:** `Array<string>` - Token array

**Example:**
```javascript
const tokens = tokenize("2x + sin(x)");
// Returns: ["2", "*", "x", "+", "sin", "(", "x", ")"]
```

#### `toRPN(tokens)`
Converts infix tokens to Reverse Polish Notation.

**Parameters:**
- `tokens` (Array<string>): Token array

**Returns:** `Array<string>` - RPN token array

**Example:**
```javascript
const rpn = toRPN(["2", "*", "x", "+", "1"]);
// Returns: ["2", "x", "*", "1", "+"]
```

#### `compile(expr)`
Compiles expression to executable RPN form.

**Parameters:**
- `expr` (string): Mathematical expression

**Returns:** `Array<string>|null` - Compiled RPN or null if error

**Example:**
```javascript
const compiled = compile("x^2 + 2x - 1");
if (compiled) {
  console.log("Expression compiled successfully");
}
```

#### `evalRPN(rpn, x)`
Evaluates RPN expression for given x value.

**Parameters:**
- `rpn` (Array<string>): Compiled RPN tokens
- `x` (number): Input value

**Returns:** `number` - Function result

**Example:**
```javascript
const result = evalRPN(compiled, 3);
console.log(`f(3) = ${result}`);
```

#### `safeEval(rpn, x)`
Safely evaluates expression with error handling.

**Parameters:**
- `rpn` (Array<string>): Compiled RPN tokens
- `x` (number): Input value

**Returns:** `number` - Function result or NaN

**Example:**
```javascript
const result = safeEval(compiled, 0);
if (isFinite(result)) {
  console.log(`f(0) = ${result}`);
}
```

### Graphing Functions

#### `resizeGraph()`
Adjusts graph canvas size to match container.

**Example:**
```javascript
resizeGraph(); // Resizes canvas and redraws
```

#### `drawGraph()`
Renders the complete graph with axes, grid, and function.

**Example:**
```javascript
drawGraph(); // Redraws entire graph
```

#### `niceGridStep(scalePxPerUnit)`
Calculates appropriate grid spacing.

**Parameters:**
- `scalePxPerUnit` (number): Current zoom level

**Returns:** `number` - Grid step size

**Example:**
```javascript
const step = niceGridStep(60); // Returns appropriate grid spacing
```

#### `drawGrid(ctx, w, h, scale, step)`
Draws grid lines on canvas.

**Parameters:**
- `ctx` (CanvasRenderingContext2D): Canvas context
- `w` (number): Canvas width
- `h` (number): Canvas height
- `scale` (number): Pixels per unit
- `step` (number): Grid spacing

**Example:**
```javascript
drawGrid(g2, 800, 600, 60, 1); // Draws grid with 1-unit spacing
```

#### `findIntercepts(rpn)`
Finds x-intercepts and y-intercept of function.

**Parameters:**
- `rpn` (Array<string>): Compiled expression

**Returns:** `Object` - Intercept data

**Example:**
```javascript
const intercepts = findIntercepts(compiled);
console.log(`X-intercepts: ${intercepts.x.length}`);
console.log(`Y-intercept: ${intercepts.y}`);
```

#### `refineRootBisection(rpn, a, b, iters)`
Refines root using bisection method.

**Parameters:**
- `rpn` (Array<string>): Compiled expression
- `a` (number): Left bound
- `b` (number): Right bound
- `iters` (number): Iteration limit

**Returns:** `number` - Refined root or NaN

**Example:**
```javascript
const root = refineRootBisection(compiled, 0, 2, 20);
console.log(`Root: ${root}`);
```

#### `plotCurrentExpression()`
Plots the current expression and shows solution steps.

**Example:**
```javascript
plotCurrentExpression(); // Plots exprInput.value
```

#### `trySymbolicSteps(expr, rpn)`
Attempts to provide symbolic solution steps.

**Parameters:**
- `expr` (string): Original expression
- `rpn` (Array<string>): Compiled expression

**Returns:** `Array<string>|null` - Solution steps or null

**Example:**
```javascript
const steps = trySymbolicSteps("2x + 3", compiled);
if (steps) {
  console.log("Symbolic solution found");
}
```

## Teach-Back Module

### Concept Structure

```javascript
{
  key: string,           // Unique identifier
  name: string,          // Display name
  hints: Array<string>,  // Progressive hints
  example: string        // Example explanation
}
```

### Core Teach-Back Functions

#### `populateTeachConcepts()`
Fills the concept selector with available topics.

**Example:**
```javascript
populateTeachConcepts(); // Updates concept dropdown
```

#### `randomPromptForConcept(concept)`
Generates a random teaching prompt for a concept.

**Parameters:**
- `concept` (Object): Concept object

**Returns:** `string` - Teaching prompt

**Example:**
```javascript
const prompt = randomPromptForConcept(concepts[0]);
console.log(prompt); // "Explain this concept as if to a 12-year-old..."
```

#### `updateTeachPrompt()`
Updates the current teaching prompt and resets hints.

**Example:**
```javascript
updateTeachPrompt(); // Refreshes prompt and hint system
```

#### `addHint(text)`
Adds a hint to the hint list.

**Parameters:**
- `text` (string): Hint text

**Example:**
```javascript
addHint("Try breaking this into smaller steps");
```

## UI Components and Tab Management

### Tab System

#### `switchTab(id)`
Switches to the specified tab and updates UI.

**Parameters:**
- `id` (string): Tab identifier ('progress', 'flash', 'geometry', 'graph', 'teach', 'settings')

**Example:**
```javascript
switchTab('flash'); // Switches to flashcards tab
```

#### `getActiveTabId()`
Determines which tab is currently active.

**Returns:** `string` - Active tab ID

**Example:**
```javascript
const activeTab = getActiveTabId();
console.log(`Current tab: ${activeTab}`);
```

### Settings Functions

#### `applyContrastSetting()`
Applies high contrast mode based on settings.

**Example:**
```javascript
applyContrastSetting(); // Toggles high contrast class
```

## Utility Functions

### Mathematical Utilities

#### `clamp(v, min, max)`
Constrains value between min and max.

**Parameters:**
- `v` (number): Value to clamp
- `min` (number): Minimum value
- `max` (number): Maximum value

**Returns:** `number` - Clamped value

**Example:**
```javascript
const clamped = clamp(15, 0, 10); // Returns 10
```

#### `lerp(a, b, t)`
Linear interpolation between two values.

**Parameters:**
- `a` (number): Start value
- `b` (number): End value
- `t` (number): Interpolation factor (0-1)

**Returns:** `number` - Interpolated value

**Example:**
```javascript
const interpolated = lerp(0, 100, 0.5); // Returns 50
```

#### `formatPercent(n)`
Formats number as percentage string.

**Parameters:**
- `n` (number): Decimal value (0-1)

**Returns:** `string` - Percentage string

**Example:**
```javascript
const percent = formatPercent(0.75); // Returns "75%"
```

#### `uid()`
Generates unique identifier.

**Returns:** `string` - Unique ID

**Example:**
```javascript
const id = uid(); // Returns unique string
```

### Content Utilities

#### `sanitize(html)`
Basic HTML sanitization to prevent script injection.

**Parameters:**
- `html` (string): HTML string to sanitize

**Returns:** `string` - Sanitized HTML

**Example:**
```javascript
const safe = sanitize("<script>alert('xss')</script>");
// Returns "&lt;script&gt;alert('xss')&lt;/script&gt;"
```

## Event Handlers

### Keyboard Navigation

The application supports comprehensive keyboard navigation:

- **Tab Navigation:** Arrow keys for tab switching
- **Flashcards:** Enter (flip), 1-4 (rate), S (shuffle)
- **Graphing:** +/- (zoom), 0 (reset), Arrow keys (pan)
- **Global:** Shift+? (help)

### Mouse/Touch Events

- **Canvas Interactions:** Drag to pan, wheel to zoom
- **Geometry:** Drag vertices/shapes, scroll to zoom
- **Touch Support:** Pinch to zoom, drag to pan

## Usage Examples

### Basic Application Setup

```javascript
// Initialize the application
function init() {
  applyContrastSetting();
  updateProgressUI();
  renderCurrentLesson();
  scheduleCards();
  resizeGeom();
  resizeGraph();
  populateTeachConcepts();
  updateTeachPrompt();
}

// Load and start
window.addEventListener('load', init);
```

### Adding Custom Flashcards

```javascript
// Add a new flashcard
function addCustomCard(front, back, tags = ['custom']) {
  const newCard = {
    id: uid(),
    front: front,
    back: back,
    tags: tags
  };
  
  state.flashcards.deck.push(newCard);
  saveState();
  scheduleCards();
}

// Example usage
addCustomCard(
  "What is the derivative of x²?",
  "2x",
  ['calculus', 'derivatives']
);
```

### Custom Expression Plotting

```javascript
// Plot a custom function
function plotFunction(expression) {
  exprInput.value = expression;
  plotCurrentExpression();
}

// Example usage
plotFunction("sin(x) * cos(x)");
```

### Progress Tracking

```javascript
// Mark lesson as complete
function completeLesson(lessonIndex) {
  if (!state.progress.completed[lessonIndex]) {
    state.progress.completed[lessonIndex] = true;
    saveState();
    celebrateIfMilestone();
  }
}

// Get completion statistics
function getProgressStats() {
  const completed = state.progress.completed.filter(Boolean).length;
  const total = lessons.length;
  const percentage = (completed / total) * 100;
  
  return {
    completed,
    total,
    percentage: Math.round(percentage)
  };
}
```

### Geometry Shape Manipulation

```javascript
// Set triangle vertices
function setTriangleVertices(v1, v2, v3) {
  geomState.vertices = [v1, v2, v3];
  drawGeometry();
}

// Example usage
setTriangleVertices(
  {x: -50, y: 50},
  {x: 0, y: -50},
  {x: 50, y: 50}
);
```

### Teach-Back Reflection

```javascript
// Save a teach-back reflection
function saveReflection(concept, prompt, response, selfScore) {
  const reflection = {
    concept: concept,
    prompt: prompt,
    text: response,
    timestamp: Date.now(),
    selfScore: selfScore
  };
  
  state.teachBack.reflections.push(reflection);
  saveState();
}

// Example usage
saveReflection(
  "Linear Equations",
  "Explain solving 2x + 3 = 7",
  "First subtract 3 from both sides...",
  3
);
```

### Data Export/Import

```javascript
// Export user data
function exportData() {
  const blob = new Blob([JSON.stringify(state, null, 2)], {
    type: 'application/json'
  });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'math_journey_export.json';
  a.click();
  URL.revokeObjectURL(url);
}

// Import user data
function importData(jsonString) {
  try {
    const data = JSON.parse(jsonString);
    state = { ...createInitialState(), ...data };
    saveState();
    applyContrastSetting();
    updateProgressUI();
    renderCurrentLesson();
    scheduleCards();
    drawGeometry();
    drawGraph();
    return true;
  } catch (error) {
    console.error('Import failed:', error);
    return false;
  }
}
```

## Browser Compatibility

The application is designed to work in modern browsers with support for:

- **ES6+ features** (arrow functions, const/let, template literals)
- **Canvas API** for graphics rendering
- **localStorage** for data persistence
- **CSS Grid and Flexbox** for layout
- **Touch events** for mobile support

## Performance Considerations

- **Canvas rendering** is optimized with device pixel ratio scaling
- **State updates** are batched to minimize localStorage writes
- **Event listeners** use passive options where appropriate
- **Memory management** includes cleanup for animation loops

## Accessibility Features

- **High contrast mode** for visual accessibility
- **Keyboard navigation** for all interactive elements
- **ARIA labels** and semantic HTML structure
- **Focus management** with visible focus indicators
- **Screen reader compatibility** with proper roles and labels

---

This documentation covers all public APIs, functions, and components of the 30-Day Math Journey application. For implementation details or advanced usage patterns, refer to the source code comments and examples provided above.