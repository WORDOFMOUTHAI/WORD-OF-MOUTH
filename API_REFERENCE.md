# Complete API Reference

## Overview

This reference documents all public APIs, functions, and components in the 30-Day Math Journey application, including parameters, return values, and detailed descriptions.

## Core Application APIs

### State Management

#### `loadState() → Object`
Loads the complete application state from localStorage.

**Returns:**
```javascript
{
  progress: {
    completed: boolean[] // Array of 30 lesson completion booleans
  },
  currentLessonIndex: number, // Currently selected lesson (0-29)
  flashcards: {
    deck: Flashcard[], // Array of flashcard objects
    stats: Record<string, CardStats> // Card statistics by card ID
  },
  settings: {
    highContrast: boolean // Accessibility setting
  },
  teachBack: {
    reflections: Array<{
      concept: string,
      prompt: string,
      text: string,
      timestamp: number,
      selfScore?: number
    }>
  }
}
```

#### `saveState() → void`
Persists the current application state to localStorage and triggers UI updates.

#### `createInitialState() → Object`
Creates a fresh application state with default values and initial flashcard deck.

### Progress System APIs

#### `updateProgressUI() → void`
Updates all progress-related UI elements including progress bars, lesson cards, and milestone messages.

#### `selectLesson(index: number) → void`
Selects and displays a specific lesson for study.

**Parameters:**
- `index: number` - Lesson index (0-29)

#### `renderCurrentLesson() → void`
Renders the currently selected lesson with objectives, activities, and action buttons.

#### `getStageIdForDay(day: number) → number`
Determines which stage a given day belongs to.

**Parameters:**
- `day: number` - Day number (1-30)

**Returns:** `number` - Stage ID (1-4)

#### `getCurrentLessonIndex() → number`
Returns the index of the next lesson that should be completed.

**Returns:** `number` - Lesson index (0-29), or 29 if all lessons are complete

#### `milestoneMessage(percentage: number) → string`
Generates encouraging milestone messages based on completion percentage.

**Parameters:**
- `percentage: number` - Completion percentage (0-1)

**Returns:** `string` - Encouraging message string

#### `renderStages() → void`
Renders the stage progress cards in the progress tab.

#### `renderLessons() → void`
Renders the lesson grid with completion status.

#### `suggestExpressionForLesson(day: number) → string`
Suggests a mathematical expression for graphing based on the lesson day.

**Parameters:**
- `day: number` - Lesson day number

**Returns:** `string` - Suggested expression string

### Flashcard System APIs

#### `scheduleCards() → void`
Schedules cards for review using spaced repetition algorithm.

#### `rateCurrentCard(grade: number) → void`
Rates the current flashcard using the SM-2 spaced repetition algorithm.

**Parameters:**
- `grade: number` - Rating from 1-4 (1=Again, 2=Hard, 3=Good, 4=Easy)

#### `getCardStats(cardId: string) → CardStats`
Retrieves or creates statistics for a specific flashcard.

**Parameters:**
- `cardId: string` - Unique card identifier

**Returns:**
```javascript
{
  ease: number,        // Difficulty factor (1.3-3.0)
  intervalDays: number, // Days until next review (0 = today)
  reps: number,        // Number of successful repetitions
  due: number,         // Next review timestamp (milliseconds)
  lapse: number        // Number of times forgotten
}
```

#### `getDefaultDeck() → Flashcard[]`
Returns the initial flashcard deck with comprehensive coverage of mathematical concepts.

**Returns:** `Flashcard[]` - Array of flashcard objects

#### `updateFlashcardUI() → void`
Updates the flashcard display and progress indicators.

#### `sanitize(html: string) → string`
Basic HTML sanitization for card content.

**Parameters:**
- `html: string` - HTML string to sanitize

**Returns:** `string` - Sanitized string (script tags removed)

### Geometry System APIs

#### `drawGeometry() → void`
Renders the current geometric shape on the canvas with handles and measurements.

#### `updateGeomProps() → void`
Calculates and displays live measurements for the current shape.

#### `resizeGeom() → void`
Resizes the geometry canvas to fit its container.

#### `worldToScreen(point: Point) → Point`
Converts world coordinates to screen coordinates.

**Parameters:**
- `point: Point` - World coordinate point

**Returns:** `Point` - Screen coordinate point

#### `screenToWorld(point: Point) → Point`
Converts screen coordinates to world coordinates.

**Parameters:**
- `point: Point` - Screen coordinate point

**Returns:** `Point` - World coordinate point

#### `drawHandle(point: Point) → void`
Draws a draggable handle at the specified screen coordinates.

**Parameters:**
- `point: Point` - Screen coordinate point

#### `pickHandleAt(x: number, y: number) → HandleInfo`
Determines which handle (if any) is at the given screen coordinates.

**Parameters:**
- `x: number` - Screen X coordinate
- `y: number` - Screen Y coordinate

**Returns:**
```javascript
{
  type: string,  // 'vertex', 'rectCorner', 'circleCenter', 'circleEdge', 'cubeCenter', or 'shape'
  index?: number // Handle index (for multi-handle shapes)
}
```

#### `geomPointerDown(e: MouseEvent) → void`
Handles mouse/touch down events for geometry interaction.

**Parameters:**
- `e: MouseEvent` - Mouse event object

#### `geomPointerMove(e: MouseEvent) → void`
Handles mouse/touch move events for geometry interaction.

**Parameters:**
- `e: MouseEvent` - Mouse event object

#### `geomPointerUp() → void`
Handles mouse/touch up events for geometry interaction.

### Graphing System APIs

#### `plotCurrentExpression() → void`
Parses and plots the mathematical expression entered in the input field.

#### `compile(expr: string) → string[]|null`
Compiles a mathematical expression string into Reverse Polish Notation for evaluation.

**Parameters:**
- `expr: string` - Mathematical expression string

**Returns:** `string[]|null` - Array of RPN tokens, or null if parsing fails

#### `evalRPN(rpn: string[], x: number) → number`
Evaluates an RPN expression at a specific x value.

**Parameters:**
- `rpn: string[]` - RPN token array
- `x: number` - Value to evaluate at

**Returns:** `number` - Evaluated result

#### `safeEval(rpn: string[], x: number) → number`
Safely evaluates an RPN expression with error handling.

**Parameters:**
- `rpn: string[]` - RPN token array
- `x: number` - Value to evaluate at

**Returns:** `number` - Evaluated result or NaN if error

#### `tokenize(expr: string) → string[]`
Converts expression string into tokens, handling implicit multiplication.

**Parameters:**
- `expr: string` - Mathematical expression

**Returns:** `string[]` - Array of tokens

#### `toRPN(tokens: string[]) → string[]`
Converts infix tokens to Reverse Polish Notation.

**Parameters:**
- `tokens: string[]` - Infix token array

**Returns:** `string[]` - RPN token array

#### `findIntercepts(rpn: string[]) → Intercepts`
Finds x and y intercepts of a function.

**Parameters:**
- `rpn: string[]` - Compiled RPN expression

**Returns:**
```javascript
{
  x: number[],  // Array of x-intercepts
  y: number     // Y-intercept value
}
```

#### `refineRootBisection(rpn: string[], a: number, b: number, iterations?: number) → number`
Refines root finding using bisection method.

**Parameters:**
- `rpn: string[]` - Compiled RPN expression
- `a: number` - Lower bound
- `b: number` - Upper bound
- `iterations?: number` - Maximum iterations (default: 20)

**Returns:** `number` - Root approximation or NaN if not found

#### `trySymbolicSteps(expr: string, rpn: string[]) → string[]|null`
Attempts to detect linear or quadratic patterns and generate solution steps.

**Parameters:**
- `expr: string` - Original expression string
- `rpn: string[]` - Compiled RPN expression

**Returns:** `string[]|null` - Array of solution steps or null if not detected

#### `drawGraph() → void`
Renders the graph canvas with grid, axes, and plotted function.

#### `resizeGraph() → void`
Resizes the graph canvas to fit its container.

#### `niceGridStep(scalePxPerUnit: number) → number`
Calculates optimal grid step size for given scale.

**Parameters:**
- `scalePxPerUnit: number` - Pixels per world unit

**Returns:** `number` - Optimal step size

#### `drawGrid(ctx: CanvasRenderingContext2D, width: number, height: number, scale: number, step: number) → void`
Draws grid lines on the canvas.

**Parameters:**
- `ctx: CanvasRenderingContext2D` - Canvas context
- `width: number` - Canvas width
- `height: number` - Canvas height
- `scale: number` - Scale factor
- `step: number` - Grid step size

#### `drawAxisLabels(ctx: CanvasRenderingContext2D, width: number, height: number, scale: number, step: number) → void`
Draws axis labels on the graph.

**Parameters:**
- `ctx: CanvasRenderingContext2D` - Canvas context
- `width: number` - Canvas width
- `height: number` - Canvas height
- `scale: number` - Scale factor
- `step: number` - Label step size

#### `dedupeSorted(arr: number[]) → number[]`
Removes duplicate values from sorted array within tolerance.

**Parameters:**
- `arr: number[]` - Sorted array of numbers

**Returns:** `number[]` - Array with duplicates removed

### Teach-Back System APIs

#### `updateTeachPrompt() → void`
Generates a new random prompt for the currently selected concept.

#### `populateTeachConcepts() → void`
Populates the concept selection dropdown with available topics.

#### `randomPromptForConcept(concept: Object) → string`
Generates a random prompt for teaching a specific concept.

**Parameters:**
- `concept: Object` - Concept object with name and hints

**Returns:** `string` - Generated prompt text

#### `addHint(text: string) → void`
Adds a hint to the progressive hint list.

**Parameters:**
- `text: string` - Hint text to display

### Settings & Data Management APIs

#### `switchTab(tabId: string) → void`
Switches the active tab and updates UI visibility.

**Parameters:**
- `tabId: string` - Tab identifier ('progress', 'flash', 'geometry', 'graph', 'teach', 'settings')

#### `getActiveTabId() → string`
Returns the currently active tab identifier.

**Returns:** `string` - Active tab ID

#### `applyContrastSetting() → void`
Applies the current contrast setting to the document.

#### `launchConfetti() → void`
Creates animated confetti celebration.

#### `resizeConfetti() → void`
Resizes the confetti canvas to fit the window.

#### `confettiLoop() → void`
Animation loop for confetti particles.

#### `celebrateIfMilestone() → void`
Checks if current completion count matches a milestone and triggers celebration.

### Utility Functions

#### `clamp(value: number, min: number, max: number) → number`
Constrains a value between minimum and maximum bounds.

**Parameters:**
- `value: number` - Value to clamp
- `min: number` - Minimum bound
- `max: number` - Maximum bound

**Returns:** `number` - Clamped value

#### `lerp(a: number, b: number, t: number) → number`
Linear interpolation between two values.

**Parameters:**
- `a: number` - Start value
- `b: number` - End value
- `t: number` - Interpolation factor (0-1)

**Returns:** `number` - Interpolated value

#### `formatPercent(n: number) → string`
Formats a decimal as a percentage string.

**Parameters:**
- `n: number` - Decimal value (0-1)

**Returns:** `string` - Formatted percentage (e.g., "75%")

#### `uid() → string`
Generates a unique identifier string.

**Returns:** `string` - Unique ID string

### Event System APIs

#### `subscribe(listener: Function) → void`
Subscribes a function to state change notifications.

**Parameters:**
- `listener: Function` - Function that receives the current state object

#### `notify() → void`
Triggers all subscribed listeners with the current state.

### Constants

#### `APP_KEY = 'math_journey_v1'`
localStorage key for state persistence.

#### `FUNCTIONS` Object
Mathematical functions available in expression parser:
- `sin`, `cos`, `tan` - Trigonometric functions
- `asin`, `acos`, `atan` - Inverse trigonometric functions
- `sqrt` - Square root
- `abs` - Absolute value
- `ln` - Natural logarithm
- `log` - Base-10 logarithm
- `exp` - Exponential function

#### `CONSTANTS` Object
Mathematical constants:
- `pi` - π (3.14159...)
- `e` - Euler's number (2.71828...)

### Data Types

#### `Point`
```javascript
{
  x: number,  // X coordinate
  y: number   // Y coordinate
}
```

#### `Flashcard`
```javascript
{
  id: string,      // Unique identifier
  front: string,   // Question/prompt text
  back: string,    // Answer/explanation text
  tags: string[]   // Categorization tags
}
```

#### `CardStats`
```javascript
{
  ease: number,        // Difficulty factor (1.3-3.0)
  intervalDays: number, // Days until next review
  reps: number,        // Successful repetitions
  due: number,         // Next due timestamp
  lapse: number        // Times forgotten
}
```

#### `Intercepts`
```javascript
{
  x: number[],  // X-intercepts
  y: number     // Y-intercept
}
```

#### `HandleInfo`
```javascript
{
  type: string,   // Handle type
  index?: number  // Handle index (for multi-handle shapes)
}
```

## Global Variables

#### `state: Object`
Current application state object.

#### `lessons: Array`
Array of lesson objects defining the 30-day curriculum.

#### `stages: Array`
Array of stage objects defining the 4 learning stages.

#### `concepts: Array`
Array of teach-back concept objects.

#### `dueQueue: Flashcard[]`
Current queue of flashcards scheduled for review.

#### `currentCardIndex: number`
Index of the currently displayed flashcard in the due queue.

## DOM Elements

### Progress Tab
- `progressFillEl` - Progress bar fill element
- `progressTextEl` - Progress text display
- `milestoneTextEl` - Milestone message display
- `stagesContainerEl` - Stages container
- `lessonGridEl` - Lesson grid container
- `currentLessonDetailEl` - Current lesson details

### Flashcard Tab
- `flashcardEl` - Main flashcard element
- `flashFrontEl` - Front face content
- `flashBackEl` - Back face content
- `flipBtn` - Flip button
- `againBtn` - "Again" rating button
- `hardBtn` - "Hard" rating button
- `goodBtn` - "Good" rating button
- `easyBtn` - "Easy" rating button
- `dueCountEl` - Due cards counter
- `cardProgressEl` - Card progress display

### Geometry Tab
- `geomCanvas` - Geometry canvas element
- `geomWrap` - Canvas container
- `shapeSelect` - Shape selection dropdown
- `resetGeometryBtn` - Reset button
- `geomPropsEl` - Measurements display
- `geomNotesEl` - Concept notes display

### Graphing Tab
- `graphCanvas` - Graph canvas element
- `graphWrap` - Canvas container
- `exprInput` - Expression input field
- `plotBtn` - Plot button
- `resetViewBtn` - Reset view button
- `solveOverlay` - Solution steps overlay

### Teach-Back Tab
- `teachConceptSelect` - Concept selection dropdown
- `teachPromptEl` - Prompt display
- `newPromptBtn` - New prompt button
- `teachResponse` - Response textarea
- `saveTeachBtn` - Save button
- `teachSaveMsg` - Save message display
- `hintList` - Hints list
- `revealNextHintBtn` - Reveal hint button
- `showFullAnswerBtn` - Show answer button
- `selfMsg` - Self-assessment message

### Settings Tab
- `contrastToggleBtn` - High contrast toggle
- `exportBtn` - Export button
- `importBtn` - Import button
- `importInput` - Hidden file input
- `resetAllBtn` - Reset all button

### Global Elements
- `app` - Main application container
- `confettiCanvas` - Celebration canvas

This API reference provides complete documentation for all public interfaces in the 30-Day Math Journey application.