# 30-Day Math Journey - Comprehensive API Documentation

## Overview

The 30-Day Math Journey is a single-file HTML application designed for adults returning to mathematics. It provides a structured learning experience with interactive tools including flashcards, geometry visualization, function graphing, and teach-back exercises.

## Table of Contents

1. [Core Application APIs](#core-application-apis)
2. [Lesson and Stage Management](#lesson-and-stage-management)
3. [Flashcard Engine](#flashcard-engine)
4. [Geometry Canvas System](#geometry-canvas-system)
5. [Function Graphing System](#function-graphing-system)
6. [Teach-Back Module](#teach-back-module)
7. [UI Components and Tab Management](#ui-components-and-tab-management)
8. [Utility Functions](#utility-functions)
9. [Usage Examples](#usage-examples)

---

## Core Application APIs

### State Management

#### `createInitialState()`
Creates the initial application state with default values.

**Returns:** `Object` - Complete application state object

**State Schema:**
```javascript
{
  progress: {
    completed: boolean[30]  // Array of 30 boolean values for lesson completion
  },
  currentLessonIndex: number,  // Index of currently selected lesson
  flashcards: {
    deck: Flashcard[],  // Array of flashcard objects
    stats: Record<id, CardStats>  // Statistics for each card
  },
  settings: {
    highContrast: boolean  // Accessibility setting
  },
  teachBack: {
    reflections: Array<{concept:string, prompt:string, text:string, timestamp:number, selfScore?:number}>
  }
}
```

#### `loadState()`
Loads application state from localStorage with error handling and schema migration.

**Returns:** `Object` - Application state object

**Example:**
```javascript
const state = loadState();
console.log(`Completed lessons: ${state.progress.completed.filter(Boolean).length}`);
```

#### `saveState()`
Persists current state to localStorage and updates UI.

**Example:**
```javascript
// Mark a lesson as complete
state.progress.completed[5] = true;
saveState(); // Automatically updates UI and persists to localStorage
```

### Pub/Sub System

#### `subscribe(listener)`
Subscribes a function to state change notifications.

**Parameters:**
- `listener` (Function): Callback function to execute on state changes

**Example:**
```javascript
subscribe((newState) => {
  console.log('State updated:', newState);
});
```

#### `notify()`
Notifies all subscribed listeners of state changes.

---

## Lesson and Stage Management

### Stage Definitions

#### `stages`
Array containing stage configuration objects.

**Structure:**
```javascript
[
  { 
    id: 1, 
    title: 'Stage 1: Foundation Reconstruction', 
    range: [1, 10], 
    color: '#7dd3fc' 
  },
  // ... additional stages
]
```

### Lesson Management Functions

#### `getStageIdForDay(day)`
Determines which stage a lesson day belongs to.

**Parameters:**
- `day` (number): Lesson day number (1-30)

**Returns:** `number` - Stage ID (1-4)

**Example:**
```javascript
const stageId = getStageIdForDay(15); // Returns 2
```

#### `getCurrentLessonIndex()`
Finds the index of the current (first incomplete) lesson.

**Returns:** `number` - Index of current lesson

#### `selectLesson(index)`
Sets the currently selected lesson and updates UI.

**Parameters:**
- `index` (number): Lesson index to select

**Example:**
```javascript
selectLesson(10); // Selects lesson 11 (0-indexed)
```

#### `suggestExpressionForLesson(day)`
Suggests a mathematical expression for graphing based on lesson day.

**Parameters:**
- `day` (number): Lesson day number

**Returns:** `string` - Suggested expression

**Example:**
```javascript
const expr = suggestExpressionForLesson(5); // Returns "2x + 3"
```

### Progress Tracking

#### `updateProgressUI()`
Updates all progress-related UI elements including progress bar, statistics, and lesson grid.

#### `milestoneMessage(pct)`
Generates milestone messages based on completion percentage.

**Parameters:**
- `pct` (number): Completion percentage (0-1)

**Returns:** `string` - Milestone message

---

## Flashcard Engine

### Spaced Repetition Algorithm

The flashcard system implements a simplified SM-2 algorithm for spaced repetition learning.

#### Card Structure

```javascript
{
  id: string,        // Unique identifier
  front: string,     // Question/prompt
  back: string,      // Answer/explanation
  tags: string[]     // Categorization tags
}
```

#### Card Statistics Structure

```javascript
{
  ease: number,           // Difficulty factor (1.3-3.0)
  intervalDays: number,   // Days until next review
  reps: number,          // Number of successful repetitions
  due: number,           // Timestamp when card is due
  lapse: number          // Number of times card was failed
}
```

### Core Functions

#### `getDefaultDeck()`
Creates the initial flashcard deck with pre-populated cards.

**Returns:** `Array<Flashcard>` - Default flashcard deck

#### `getCardStats(id)`
Retrieves or creates statistics for a specific card.

**Parameters:**
- `id` (string): Card identifier

**Returns:** `Object` - Card statistics object

#### `scheduleCards()`
Builds the review queue prioritizing due cards and sprinkling in new cards.

**Example:**
```javascript
scheduleCards(); // Updates dueQueue and currentCardIndex
```

#### `rateCurrentCard(grade)`
Processes a card rating and updates its schedule.

**Parameters:**
- `grade` (number): Rating (1=Again, 2=Hard, 3=Good, 4=Easy)

**Algorithm:**
- **Again (1):** Reset repetitions, reduce ease, schedule for 10 minutes
- **Hard (2):** Reduce ease, halve interval
- **Good (3):** Increase ease, progress interval normally
- **Easy (4):** Increase ease significantly, accelerate interval

**Example:**
```javascript
rateCurrentCard(3); // Mark current card as "Good"
```

#### `updateFlashcardUI()`
Updates flashcard display and progress indicators.

---

## Geometry Canvas System

### Canvas State Management

#### `geomState`
Global state object for geometry canvas.

```javascript
{
  shape: string,           // Current shape type
  zoom: number,           // Zoom level
  offset: {x: number, y: number}, // Pan offset
  dragging: Object|null,  // Current drag operation
  vertices: Array,        // Triangle vertices
  rect: Object,          // Rectangle properties
  circle: Object,        // Circle properties
  cube: Object          // Cube properties
}
```

### Core Functions

#### `resizeGeom()`
Handles canvas resizing and redraws geometry.

#### `worldToScreen(p)`
Converts world coordinates to screen coordinates.

**Parameters:**
- `p` (Object): Point with x, y coordinates

**Returns:** `Object` - Screen coordinates

#### `screenToWorld(p)`
Converts screen coordinates to world coordinates.

**Parameters:**
- `p` (Object): Point with x, y coordinates

**Returns:** `Object` - World coordinates

#### `drawGeometry()`
Main rendering function for all geometry shapes.

#### `updateGeomProps()`
Calculates and displays measurements for the current shape.

**Supported Shapes:**
- **Triangle:** Perimeter and area using Heron's formula
- **Rectangle:** Perimeter and area
- **Circle:** Circumference and area
- **Cube:** Edge length, surface area, and volume

### Shape-Specific Functions

#### `drawTriangle()`
Renders triangle with draggable vertices.

#### `drawRectangle()`
Renders rectangle with draggable corner handles.

#### `drawCircle()`
Renders circle with draggable center and radius handle.

#### `drawCube()`
Renders isometric cube projection.

### Interaction Functions

#### `pickHandleAt(x, y)`
Determines which handle is under the cursor.

**Parameters:**
- `x`, `y` (number): Screen coordinates

**Returns:** `Object` - Handle information

#### `geomPointerDown(e)`
Handles mouse/touch down events for dragging.

#### `geomPointerMove(e)`
Handles mouse/touch move events during dragging.

#### `geomPointerUp()`
Handles mouse/touch up events to end dragging.

---

## Function Graphing System

### Expression Parser

The graphing system includes a complete mathematical expression parser supporting:
- Basic operations: `+`, `-`, `*`, `/`, `^`
- Functions: `sin`, `cos`, `tan`, `sqrt`, `abs`, `ln`, `log`, `exp`
- Constants: `pi`, `e`
- Variables: `x`

### Core Functions

#### `tokenize(expr)`
Converts mathematical expression string into tokens.

**Parameters:**
- `expr` (string): Mathematical expression

**Returns:** `Array<string>` - Token array

**Example:**
```javascript
const tokens = tokenize("2x + sin(x)");
// Returns: ["2", "*", "x", "+", "sin", "(", "x", ")"]
```

#### `toRPN(tokens)`
Converts infix notation to Reverse Polish Notation.

**Parameters:**
- `tokens` (Array<string>): Token array

**Returns:** `Array<string>` - RPN token array

#### `compile(expr)`
Compiles expression string into executable RPN.

**Parameters:**
- `expr` (string): Mathematical expression

**Returns:** `Array<string>|null` - Compiled RPN or null on error

**Example:**
```javascript
const compiled = compile("x^2 + 1");
if (compiled) {
  const result = evalRPN(compiled, 3); // Returns 10
}
```

#### `evalRPN(rpn, x)`
Evaluates compiled RPN expression for given x value.

**Parameters:**
- `rpn` (Array<string>): Compiled RPN
- `x` (number): Input value

**Returns:** `number` - Function result

#### `safeEval(rpn, x)`
Safely evaluates RPN with error handling.

**Parameters:**
- `rpn` (Array<string>): Compiled RPN
- `x` (number): Input value

**Returns:** `number` - Function result or NaN on error

### Graphing Functions

#### `drawGraph()`
Main rendering function for the graph canvas.

#### `findIntercepts(rpn)`
Finds x-intercepts and y-intercept of function.

**Parameters:**
- `rpn` (Array<string>): Compiled RPN

**Returns:** `Object` - `{x: number[], y: number}`

#### `plotCurrentExpression()`
Plots the current expression and updates UI.

**Example:**
```javascript
exprInput.value = "sin(x) + cos(2x)";
plotCurrentExpression();
```

#### `trySymbolicSteps(expr, rpn)`
Attempts to detect linear/quadratic functions and show symbolic solutions.

**Parameters:**
- `expr` (string): Original expression
- `rpn` (Array<string>): Compiled RPN

**Returns:** `Array<string>|null` - Solution steps or null

### Graph State Management

#### `graphState`
Global state object for graphing system.

```javascript
{
  expr: string,              // Current expression
  compiled: Array<string>,   // Compiled RPN
  zoom: number,             // Zoom level (px per unit)
  origin: {x: number, y: number}, // Pan offset
  dragging: boolean,        // Drag state
  intercepts: Object,       // Function intercepts
  stepsHtml: string        // Solution steps HTML
}
```

---

## Teach-Back Module

### Concept Management

#### `concepts`
Array of available teaching concepts with hints and examples.

**Structure:**
```javascript
[
  {
    key: string,        // Unique identifier
    name: string,       // Display name
    hints: string[],    // Progressive hints
    example: string     // Example explanation
  }
]
```

### Core Functions

#### `populateTeachConcepts()`
Populates the concept selection dropdown.

#### `randomPromptForConcept(c)`
Generates random teaching prompts for a concept.

**Parameters:**
- `c` (Object): Concept object

**Returns:** `string` - Random prompt

#### `updateTeachPrompt()`
Updates the current teaching prompt and resets hints.

#### `addHint(text)`
Adds a hint to the hint list.

**Parameters:**
- `text` (string): Hint text

### Self-Assessment

#### Self-Assessment Buttons
Three assessment levels:
- **"Nailed it" (3):** Excellent understanding
- **"Almost" (2):** Good but needs refinement
- **"Need practice" (1):** Requires more study

---

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

### Settings Management

#### `applyContrastSetting()`
Applies high contrast mode to the document.

#### `exportBtn` Event Handler
Exports current state to JSON file.

#### `importBtn` Event Handler
Imports state from JSON file with validation.

#### `resetAllBtn` Event Handler
Resets all progress and settings with confirmation.

---

## Utility Functions

### Mathematical Utilities

#### `clamp(v, min, max)`
Constrains a value between minimum and maximum.

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

#### `formatPercent(n)`
Formats a decimal as a percentage string.

**Parameters:**
- `n` (number): Decimal value (0-1)

**Returns:** `string` - Formatted percentage

**Example:**
```javascript
const pct = formatPercent(0.75); // Returns "75%"
```

#### `uid()`
Generates a unique identifier.

**Returns:** `string` - Unique ID

### DOM Utilities

#### `sanitize(html)`
Basic HTML sanitization to prevent script injection.

**Parameters:**
- `html` (string): HTML string to sanitize

**Returns:** `string` - Sanitized HTML

---

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

// Start the application
window.addEventListener('load', init);
```

### Working with Lessons

```javascript
// Get current lesson information
const currentIndex = getCurrentLessonIndex();
const lesson = lessons[currentIndex];
console.log(`Current lesson: Day ${lesson.day} - ${lesson.title}`);

// Mark a lesson as complete
state.progress.completed[currentIndex] = true;
saveState();

// Switch to a specific lesson
selectLesson(5); // Selects lesson 6 (0-indexed)
```

### Flashcard Management

```javascript
// Add a custom flashcard
const newCard = {
  id: uid(),
  front: "What is the derivative of x²?",
  back: "2x",
  tags: ['calculus', 'custom']
};
state.flashcards.deck.push(newCard);
saveState();

// Rate a card
rateCurrentCard(3); // Mark as "Good"

// Get card statistics
const stats = getCardStats(cardId);
console.log(`Card ease factor: ${stats.ease}`);
```

### Geometry Canvas Usage

```javascript
// Change shape type
geomState.shape = 'circle';
drawGeometry();

// Update circle properties
geomState.circle.x = 50;
geomState.circle.y = -30;
geomState.circle.r = 100;
drawGeometry();

// Reset geometry
resetGeometryBtn.click();
```

### Function Graphing

```javascript
// Plot a function
exprInput.value = "x^2 - 4x + 3";
plotCurrentExpression();

// Compile and evaluate manually
const compiled = compile("sin(x) + cos(x)");
if (compiled) {
  const result = safeEval(compiled, Math.PI/4);
  console.log(`Result: ${result}`); // Should be approximately √2
}

// Find intercepts
const intercepts = findIntercepts(compiled);
console.log(`X-intercepts: ${intercepts.x}`);
console.log(`Y-intercept: ${intercepts.y}`);
```

### Teach-Back Exercises

```javascript
// Generate a new prompt
updateTeachPrompt();

// Add a custom hint
addHint("Remember to use real-world examples!");

// Save a reflection
const reflection = {
  concept: "Linear Equations",
  prompt: "Explain solving linear equations",
  text: "To solve linear equations, isolate the variable...",
  timestamp: Date.now()
};
state.teachBack.reflections.push(reflection);
saveState();
```

### State Management

```javascript
// Subscribe to state changes
subscribe((newState) => {
  console.log('Progress updated:', 
    newState.progress.completed.filter(Boolean).length);
});

// Export state
const exportData = JSON.stringify(state, null, 2);
const blob = new Blob([exportData], { type: 'application/json' });

// Import state
const importedState = JSON.parse(importData);
state = { ...createInitialState(), ...importedState };
saveState();
```

### Keyboard Shortcuts

The application supports various keyboard shortcuts:

- **Flashcards Tab:**
  - `Enter`: Flip card
  - `1-4`: Rate card (Again, Hard, Good, Easy)
  - `S`: Shuffle deck

- **Graphing Tab:**
  - `+/-`: Zoom in/out
  - `0`: Reset view
  - Arrow keys: Pan graph

- **Global:**
  - `Shift + ?`: Show help

### Accessibility Features

```javascript
// Toggle high contrast mode
state.settings.highContrast = !state.settings.highContrast;
applyContrastSetting();
saveState();

// The application includes:
// - ARIA labels for screen readers
// - Keyboard navigation support
// - High contrast mode
// - Focus indicators
```

---

## Error Handling

The application includes comprehensive error handling:

- **State Loading:** Graceful fallback to initial state on parse errors
- **Expression Parsing:** Safe evaluation with NaN returns for invalid expressions
- **Canvas Operations:** Error boundaries for drawing operations
- **File Operations:** Validation and user feedback for import/export

## Browser Compatibility

The application is designed to work in modern browsers with support for:
- ES6+ JavaScript features
- Canvas API
- localStorage
- CSS Grid and Flexbox
- Touch events for mobile devices

## Performance Considerations

- **Canvas Rendering:** Optimized drawing with device pixel ratio scaling
- **State Management:** Efficient localStorage operations with debouncing
- **Memory Management:** Proper cleanup of event listeners and canvas contexts
- **Responsive Design:** CSS-based responsive layout with minimal JavaScript

---

This documentation covers all public APIs, functions, and components in the 30-Day Math Journey application. Each function includes parameter descriptions, return values, and practical usage examples to help developers understand and extend the application.