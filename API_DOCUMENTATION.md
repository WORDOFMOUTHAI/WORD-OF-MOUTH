# 30-Day Math Journey - API Documentation

## Overview

The 30-Day Math Journey is a comprehensive single-file HTML application designed for adults returning to mathematics. It combines rigorous learning with an accessible, encouraging interface featuring spaced repetition flashcards, interactive geometry, function graphing, and teach-back exercises.

## Table of Contents

1. [Application State Management](#application-state-management)
2. [Progress Tracking System](#progress-tracking-system)
3. [Flashcard Engine](#flashcard-engine)
4. [Geometry Canvas](#geometry-canvas)
5. [Graphing System](#graphing-system)
6. [Teach-Back Module](#teach-back-module)
7. [UI Navigation & Settings](#ui-navigation--settings)
8. [Utility Functions](#utility-functions)

---

## Application State Management

### State Structure

The application maintains its state in a centralized object with the following schema:

```javascript
state = {
  progress: {
    completed: boolean[30]  // Array tracking completion of 30 lessons
  },
  currentLessonIndex: number,
  flashcards: {
    deck: Flashcard[],
    stats: Record<cardId, CardStats>
  },
  settings: {
    highContrast: boolean
  },
  teachBack: {
    reflections: TeachBackEntry[]
  }
}
```

### Core State Functions

#### `createInitialState()`
Creates a fresh application state with default values.

**Returns:** `Object` - Initial state object
**Example:**
```javascript
const initialState = createInitialState();
// Returns state with empty progress, default flashcards, etc.
```

#### `loadState()`
Loads application state from localStorage with schema migration safety.

**Returns:** `Object` - Loaded or fresh state
**Example:**
```javascript
const currentState = loadState();
```

#### `saveState()`
Persists current state to localStorage and triggers UI updates.

**Side Effects:** 
- Saves to localStorage
- Calls `updateProgressUI()`

**Example:**
```javascript
// After modifying state
state.progress.completed[0] = true;
saveState(); // Persists and updates UI
```

### Pub/Sub System

#### `subscribe(listener)`
Adds a listener function to be called when state changes.

**Parameters:**
- `listener: Function` - Callback function that receives the state

**Example:**
```javascript
subscribe((newState) => {
  console.log('State updated:', newState);
});
```

#### `notify()`
Triggers all subscribed listeners with the current state.

---

## Progress Tracking System

### Lesson and Stage Structure

The application organizes 30 lessons into 4 stages:

```javascript
const stages = [
  { id: 1, title: 'Stage 1: Foundation Reconstruction', range: [1, 10], color: '#7dd3fc' },
  { id: 2, title: 'Stage 2: Intermediate Integration', range: [11, 18], color: '#60a5fa' },
  { id: 3, title: 'Stage 3: Classical Mathematical Thinking', range: [19, 25], color: '#34d399' },
  { id: 4, title: 'Stage 4: Sacred Geometry Synthesis', range: [26, 30], color: '#f59e0b' }
];
```

### Progress Functions

#### `updateProgressUI()`
Refreshes all progress-related UI elements including the header progress bar, milestone text, stages, and lessons.

**Side Effects:**
- Updates progress bar width and text
- Renders stage completion percentages
- Updates lesson grid with current/completed states

#### `getCurrentLessonIndex()`
Determines the index of the current lesson based on completion status.

**Returns:** `number` - Index of first incomplete lesson, or last lesson if all complete

**Example:**
```javascript
const currentLesson = getCurrentLessonIndex();
console.log(`User is on lesson ${currentLesson + 1}`);
```

#### `selectLesson(index)`
Sets a specific lesson as the current lesson and updates the UI.

**Parameters:**
- `index: number` - Zero-based lesson index (0-29)

**Example:**
```javascript
selectLesson(5); // Jump to lesson 6
```

#### `celebrateIfMilestone()`
Triggers confetti animation if the user has reached specific completion milestones (1, 5, 10, 15, 20, 25, 30 lessons).

**Example:**
```javascript
// Called automatically after marking lessons complete
state.progress.completed[4] = true; // Complete lesson 5
celebrateIfMilestone(); // Triggers confetti
```

---

## Flashcard Engine

The flashcard system implements spaced repetition using an SM-2 inspired algorithm.

### Flashcard Data Structure

```javascript
// Card structure
{
  id: string,
  front: string,
  back: string,
  tags: string[]
}

// Stats structure  
{
  ease: number,        // Difficulty multiplier (1.3-3.0)
  intervalDays: number, // Days until next review
  reps: number,        // Number of successful reviews
  due: number,         // Timestamp when card is due
  lapse: number        // Number of times marked as "again"
}
```

### Flashcard Functions

#### `getCardStats(id)`
Retrieves or creates statistics for a flashcard.

**Parameters:**
- `id: string` - Unique card identifier

**Returns:** `CardStats` - Statistics object for the card

**Example:**
```javascript
const stats = getCardStats('card_123');
console.log(`Card ease: ${stats.ease}, due: ${new Date(stats.due)}`);
```

#### `scheduleCards()`
Creates a priority queue of due cards and updates the flashcard UI.

**Algorithm:**
1. Prioritize cards that are due (due <= now)
2. Include up to 3 cards that aren't due yet
3. Ensure at least one card is always available

**Example:**
```javascript
scheduleCards(); // Refreshes the due queue and UI
```

#### `rateCurrentCard(grade)`
Applies spaced repetition algorithm based on user's performance rating.

**Parameters:**
- `grade: number` - Rating from 1-4
  - 1 (Again): Reset progress, show in 10 minutes
  - 2 (Hard): Reduce ease, shorter interval
  - 3 (Good): Normal progression
  - 4 (Easy): Accelerated progression

**Algorithm Details:**
```javascript
// Grade 1 (Again)
ease = max(1.3, ease - 0.2)
reps = 0
intervalDays = 0
due = now + 10 minutes

// Grade 2 (Hard)  
ease = max(1.3, ease - 0.15)
intervalDays = max(1, round(intervalDays * 0.5))

// Grade 3 (Good)
ease = min(2.8, ease + 0.1)
reps += 1
if (reps == 1) intervalDays = 1
else if (reps == 2) intervalDays = 6  
else intervalDays = round(intervalDays * ease)

// Grade 4 (Easy)
ease = min(3.0, ease + 0.15)
// Similar to Good but with bonus multiplier
```

**Example:**
```javascript
rateCurrentCard(3); // Mark current card as "Good"
```

#### `getDefaultDeck()`
Creates the initial set of flashcards covering all learning stages.

**Returns:** `Flashcard[]` - Array of default flashcard objects

---

## Geometry Canvas

Interactive 2D and pseudo-3D geometry system with draggable elements.

### Geometry State

```javascript
const geomState = {
  shape: string,           // 'triangle', 'rectangle', 'circle', 'cube'
  zoom: number,           // Zoom level (0.4-5.0)
  offset: {x, y},         // Pan offset in pixels
  dragging: object|null,  // Current drag operation
  vertices: Point[],      // Triangle vertices
  rect: {x, y, w, h},    // Rectangle bounds
  circle: {x, y, r},     // Circle center and radius
  cube: {center, size, rot} // Cube parameters
}
```

### Coordinate Functions

#### `worldToScreen(point)`
Converts world coordinates to screen pixel coordinates.

**Parameters:**
- `point: {x: number, y: number}` - World coordinate point

**Returns:** `{x: number, y: number}` - Screen pixel coordinates

**Example:**
```javascript
const screenPoint = worldToScreen({x: 100, y: -50});
// Accounts for zoom, pan, and canvas center
```

#### `screenToWorld(point)`
Converts screen pixel coordinates to world coordinates.

**Parameters:**
- `point: {x: number, y: number}` - Screen pixel coordinates

**Returns:** `{x: number, y: number}` - World coordinate point

### Drawing Functions

#### `drawGeometry()`
Main rendering function that clears canvas and draws the current shape with interactive handles.

**Renders:**
- Coordinate axes
- Selected shape with styling
- Interactive handles for manipulation
- Live measurements

#### `drawTriangle()`, `drawRectangle()`, `drawCircle()`, `drawCube()`
Specialized drawing functions for each geometry type.

**Features:**
- Interactive vertex/corner handles
- Fill and stroke styling with transparency
- Live area/perimeter calculations

### Interaction Functions  

#### `pickHandleAt(x, y)`
Determines what element (if any) is at the given screen coordinates for interaction.

**Parameters:**
- `x, y: number` - Screen coordinates

**Returns:** `Object` - Hit test result with type and index

**Example:**
```javascript
const hit = pickHandleAt(150, 200);
// Returns: {type: 'vertex', index: 1} or {type: 'shape'}
```

#### `updateGeomProps()`
Calculates and displays live measurements for the current shape.

**Measurements by Shape:**
- **Triangle**: Perimeter, Area (Heron's formula)
- **Rectangle**: Perimeter, Area  
- **Circle**: Circumference, Area
- **Cube**: Surface Area, Volume

---

## Graphing System

Advanced function plotting with expression parsing and analysis.

### Expression Parser

The graphing system includes a complete mathematical expression parser supporting:

- **Operators**: `+`, `-`, `*`, `/`, `^` (with proper precedence)
- **Functions**: `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `sqrt`, `abs`, `ln`, `log`, `exp`
- **Constants**: `pi`, `e`
- **Variables**: `x`
- **Implicit multiplication**: `2x`, `sin(x)cos(x)`
- **Unary operators**: `-x`

### Parser Functions

#### `tokenize(expr)`
Converts a mathematical expression string into tokens.

**Parameters:**
- `expr: string` - Mathematical expression like "2*sin(x) + 1"

**Returns:** `string[]` - Array of tokens

**Example:**
```javascript
const tokens = tokenize("2*sin(x) + 1");
// Returns: ['2', '*', 'sin', '(', 'x', ')', '+', '1']
```

#### `toRPN(tokens)`  
Converts tokens to Reverse Polish Notation using the Shunting-yard algorithm.

**Parameters:**
- `tokens: string[]` - Token array from tokenize()

**Returns:** `string[]` - RPN token array

#### `compile(expr)`
Full parsing pipeline from expression string to evaluable RPN.

**Parameters:**
- `expr: string` - Mathematical expression

**Returns:** `string[]|null` - RPN array or null if parse error

**Example:**
```javascript
const compiled = compile("sin(x^2) + 3*x");
if (compiled) {
  const result = evalRPN(compiled, Math.PI);
}
```

#### `evalRPN(rpn, x)`
Evaluates a compiled RPN expression for a given x value.

**Parameters:**
- `rpn: string[]` - Compiled RPN expression
- `x: number` - Variable value

**Returns:** `number` - Calculated result

### Graph Functions

#### `drawGraph()`
Main rendering function for the coordinate plane and function plot.

**Renders:**
- Adaptive grid with smart spacing
- Coordinate axes with labels  
- Function curve with sampling
- X-intercepts (red dots) and Y-intercept (green dot)

#### `findIntercepts(rpn)`
Analyzes the function to find x and y intercepts.

**Parameters:**
- `rpn: string[]` - Compiled expression

**Returns:** `{x: number[], y: number}` - Intercept coordinates

**Algorithm:**
- Scans visible domain for sign changes
- Uses bisection method to refine roots
- Evaluates f(0) for y-intercept

#### `trySymbolicSteps(expr, rpn)`
Attempts to detect linear or quadratic functions and provide analytical solutions.

**Returns:** `string[]|null` - Step-by-step solution or null

**Detection:**
- **Linear**: Tests if second differences ≈ 0
- **Quadratic**: Tests if second differences are constant
- **Fallback**: Newton's method iterations

### Navigation

The graph supports full interactive navigation:
- **Pan**: Drag to move viewport
- **Zoom**: Mouse wheel, touch pinch, or +/- keys  
- **Reset**: '0' key or reset button
- **Touch**: Full mobile support with pan and pinch

---

## Teach-Back Module

Self-explanation system with progressive hints and self-assessment.

### Concept Structure

```javascript
const concept = {
  key: string,         // Unique identifier
  name: string,        // Display name
  hints: string[],     // Progressive hint sequence
  example: string      // Full worked example
}
```

### Teach-Back Functions

#### `populateTeachConcepts()`
Populates the concept selection dropdown with available topics.

**Concepts Include:**
- Order of Operations (PEMDAS)
- Solving Linear Equations  
- Right Triangle Trigonometry
- Quadratics & Roots
- Area and Perimeter

#### `randomPromptForConcept(concept)`
Generates a random teaching prompt for the given concept.

**Parameters:**
- `concept: Object` - Concept definition

**Returns:** `string` - Random prompt text

**Example Prompts:**
- "Explain this concept as if to a 12-year-old with one real-life example"
- "Teach it to a colleague returning to math after years; focus on confidence"
- "Write a step-by-step method in under 6 sentences"
- "Describe a common mistake and how to avoid it"

#### `updateTeachPrompt()`
Updates the current teaching prompt and resets the hint system.

**Side Effects:**
- Updates prompt display
- Resets hints to show only the first hint
- Configures hint progression buttons

### Self-Assessment System

The teach-back module includes a 3-point self-assessment scale:
- **3 (Nailed it)**: Excellent understanding and explanation
- **2 (Almost)**: Good grasp with minor gaps  
- **1 (Need practice)**: Requires more study

Assessments are stored with reflections for progress tracking.

---

## UI Navigation & Settings

### Tab System

#### `switchTab(id)`
Changes the active application tab and initializes tab-specific functionality.

**Parameters:**
- `id: string` - Tab identifier ('progress', 'flash', 'geometry', 'graph', 'teach', 'settings')

**Side Effects:**
- Shows/hides appropriate content sections
- Updates navigation aria-current attributes
- Initializes canvas rendering for visual tabs

**Example:**
```javascript
switchTab('geometry'); // Switch to geometry lab
```

#### `getActiveTabId()`
Returns the currently active tab identifier.

**Returns:** `string` - Current tab ID

### Settings Functions

#### `applyContrastSetting()`
Applies the high contrast theme based on user preference.

**Side Effects:**
- Toggles 'high-contrast' CSS class on document element
- Updates all color variables to high contrast values

#### Export/Import System

The application supports full data export and import:

**Export**: Downloads complete state as JSON file
**Import**: Restores state from uploaded JSON file  
**Reset**: Clears all data and returns to initial state

---

## Utility Functions

### Mathematical Utilities

#### `clamp(value, min, max)`
Constrains a value between minimum and maximum bounds.

**Parameters:**
- `value: number` - Value to constrain
- `min: number` - Minimum allowed value  
- `max: number` - Maximum allowed value

**Returns:** `number` - Clamped value

**Example:**
```javascript
const zoom = clamp(newZoom, 0.4, 5.0); // Keep zoom in valid range
```

#### `lerp(a, b, t)`
Linear interpolation between two values.

**Parameters:**
- `a: number` - Start value
- `b: number` - End value  
- `t: number` - Interpolation factor (0-1)

**Returns:** `number` - Interpolated value

**Example:**
```javascript
const midpoint = lerp(10, 20, 0.5); // Returns 15
```

#### `formatPercent(decimal)`
Formats a decimal as a rounded percentage string.

**Parameters:**
- `decimal: number` - Decimal value (0.75 = 75%)

**Returns:** `string` - Formatted percentage

**Example:**
```javascript
const display = formatPercent(0.67); // Returns "67%"
```

#### `uid()`
Generates a unique identifier for new entities.

**Returns:** `string` - Unique ID combining random string and timestamp

### Canvas Utilities

#### `resizeGeom()`, `resizeGraph()`
Responsive canvas sizing functions that handle high DPI displays.

**Features:**
- Automatic devicePixelRatio scaling
- ResizeObserver integration  
- Maintains aspect ratio and clarity

#### `niceGridStep(scalePxPerUnit)`
Calculates visually appealing grid spacing for the graph canvas.

**Parameters:**
- `scalePxPerUnit: number` - Current zoom scale

**Returns:** `number` - Optimal grid step size

**Algorithm:**
- Targets ~60px spacing between grid lines
- Rounds to nice values (1, 2, 5, 10 × powers of 10)

---

## Event Handling

### Keyboard Shortcuts

The application provides comprehensive keyboard support:

**Global:**
- `Tab`: Navigate between elements
- `Shift+?`: Show help dialog

**Flashcards Tab:**
- `Enter`: Flip card
- `1-4`: Rate card quality  
- `S`: Shuffle deck

**Graphing Tab:**
- `+/-`: Zoom in/out
- `0`: Reset view
- `Arrow keys`: Pan viewport
- `Enter` (in expression field): Plot function

**Geometry Tab:**
- `Mouse wheel`: Zoom
- `Drag`: Move shapes/vertices

### Touch Support

Full touch and mobile support includes:
- **Pan gestures**: Single finger drag
- **Pinch zoom**: Two finger zoom (graphs and geometry)
- **Touch targets**: Appropriately sized for mobile interaction
- **Responsive design**: Adapts to mobile viewports

---

## Performance Considerations

### Rendering Optimization

- **RequestAnimationFrame**: Used for smooth animations (confetti)
- **Canvas efficiency**: Only redraws when necessary
- **Event throttling**: Prevents excessive updates during interactions

### Memory Management

- **State persistence**: Automatic localStorage with error handling
- **Event cleanup**: Proper listener management
- **Canvas reuse**: Single canvas instances per module

### Browser Compatibility

The application uses modern web APIs but includes fallbacks:
- **Canvas 2D**: Universal support
- **LocalStorage**: Graceful degradation if unavailable
- **Touch events**: Progressive enhancement
- **CSS Grid/Flexbox**: Modern layout with fallbacks

---

This API documentation covers all public interfaces and core functionality of the 30-Day Math Journey application. Each function includes parameter specifications, return values, usage examples, and implementation notes to facilitate understanding and potential customization.