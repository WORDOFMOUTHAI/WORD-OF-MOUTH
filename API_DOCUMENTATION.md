# 30-Day Math Journey - Comprehensive API Documentation

## Overview

The **30-Day Math Journey** is a comprehensive single-page educational application designed for adults returning to mathematics. Built as a single HTML file, it provides an interactive learning platform with progress tracking, spaced repetition flashcards, interactive geometry, function graphing, and reflective learning exercises.

## Architecture

The application is organized into several core modules:

- **Progress System**: 30-day structured learning journey across 4 stages
- **Flashcard Engine**: Spaced repetition system for concept reinforcement
- **Geometry Lab**: Interactive 2D and isometric 3D geometry canvas
- **Graphing System**: Mathematical function parser and visualizer
- **Teach-Back Module**: Reflective learning through explanation
- **State Management**: Local storage persistence and pub/sub updates

## Core APIs

### State Management

#### `loadState() → Object`
Loads the complete application state from localStorage.

**Returns:**
```javascript
{
  progress: {
    completed: boolean[30], // Array of lesson completion status
  },
  currentLessonIndex: number, // Currently selected lesson (0-29)
  flashcards: {
    deck: Flashcard[], // Array of flashcard objects
    stats: Record<string, CardStats> // Card statistics by ID
  },
  settings: {
    highContrast: boolean // Accessibility setting
  },
  teachBack: {
    reflections: Array<{concept:string, prompt:string, text:string, timestamp:number, selfScore?:number}>
  }
}
```

#### `saveState() → void`
Persists the current application state to localStorage and triggers UI updates.

#### `createInitialState() → Object`
Creates a fresh application state with default values and initial flashcard deck.

### Progress Management

#### `updateProgressUI() → void`
Updates all progress-related UI elements including progress bars, lesson cards, and milestone messages.

**Usage:**
```javascript
updateProgressUI(); // Refreshes progress display after state changes
```

#### `selectLesson(index: number) → void`
Selects and displays a specific lesson for study.

**Parameters:**
- `index`: Lesson index (0-29)

**Usage:**
```javascript
selectLesson(5); // Select and display lesson 6 (Day 6)
```

#### `renderCurrentLesson() → void`
Renders the currently selected lesson with objectives, activities, and action buttons.

### Flashcard System

#### `scheduleCards() → void`
Schedules cards for review using spaced repetition algorithm. Prioritizes due cards and sprinkles new cards.

**Usage:**
```javascript
scheduleCards(); // Initialize or refresh the review queue
```

#### `rateCurrentCard(grade: number) → void`
Rates the current flashcard using the SM-2 spaced repetition algorithm.

**Parameters:**
- `grade`: Rating from 1-4 (1=Again, 2=Hard, 3=Good, 4=Easy)

**Usage:**
```javascript
rateCurrentCard(3); // Rate current card as "Good"
```

#### `getCardStats(cardId: string) → CardStats`
Retrieves or creates statistics for a specific flashcard.

**Returns:**
```javascript
{
  ease: number,        // Difficulty factor (1.3-3.0)
  intervalDays: number, // Days until next review
  reps: number,        // Number of successful repetitions
  due: number,         // Next due timestamp
  lapse: number        // Number of times forgotten
}
```

### Geometry System

#### `drawGeometry() → void`
Renders the current geometric shape on the canvas with handles and measurements.

**Usage:**
```javascript
drawGeometry(); // Redraw the geometry canvas
```

#### `updateGeomProps() → void`
Updates the measurements and concept notes display for the current shape.

### Graphing System

#### `plotCurrentExpression() → void`
Parses and plots the mathematical expression entered in the input field.

**Usage:**
```javascript
exprInput.value = "sin(x) + cos(2*x)";
plotCurrentExpression(); // Parse and plot the expression
```

#### `compile(expr: string) → Array|null`
Compiles a mathematical expression string into Reverse Polish Notation for evaluation.

**Parameters:**
- `expr`: Mathematical expression string

**Returns:** Array of tokens in RPN format, or `null` if parsing fails

**Supported Operations:**
- Arithmetic: `+`, `-`, `*`, `/`, `^`
- Functions: `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `sqrt`, `abs`, `ln`, `log`, `exp`
- Constants: `pi`, `e`

**Usage:**
```javascript
const rpn = compile("sin(x) + x^2"); // Returns RPN token array
```

#### `safeEval(rpn: Array, x: number) → number`
Safely evaluates a compiled expression at a specific x value.

**Parameters:**
- `rpn`: Compiled expression in RPN format
- `x`: Value to evaluate at

**Usage:**
```javascript
const result = safeEval(compiled, 1.5); // Evaluate f(1.5)
```

### Teach-Back System

#### `updateTeachPrompt() → void`
Generates a new random prompt for the currently selected concept.

**Usage:**
```javascript
updateTeachPrompt(); // Generate new teach-back prompt
```

#### `populateTeachConcepts() → void`
Populates the concept selection dropdown with available topics.

**Available Concepts:**
- `pemdas`: Order of Operations
- `linear`: Solving Linear Equations
- `trig`: Right Triangle Trigonometry
- `quadratic`: Quadratics & Roots
- `geometry`: Area and Perimeter

### Settings & Data Management

#### `switchTab(tabId: string) → void`
Switches the active tab and updates UI visibility.

**Parameters:**
- `tabId`: One of `'progress'`, `'flash'`, `'geometry'`, `'graph'`, `'teach'`, `'settings'`

**Usage:**
```javascript
switchTab('flash'); // Switch to flashcards tab
```

#### `exportBtn` Event Handler
Exports complete application state as JSON file.

**Usage:**
```javascript
exportBtn.click(); // Triggers download of 'math_journey_export.json'
```

#### `importBtn` Event Handler
Imports application state from uploaded JSON file.

## Event System

The application uses a simple pub/sub system for state updates:

#### `subscribe(listener: Function) → void`
Subscribes a function to state change notifications.

**Parameters:**
- `listener`: Function that receives the current state object

**Usage:**
```javascript
subscribe((state) => {
  console.log('State updated:', state.progress.completed.length, 'lessons completed');
});
```

#### `notify() → void`
Triggers all subscribed listeners with the current state.

## Utility Functions

#### `clamp(value: number, min: number, max: number) → number`
Constrains a value between minimum and maximum bounds.

#### `lerp(a: number, b: number, t: number) → number`
Linear interpolation between two values.

#### `formatPercent(n: number) → string`
Formats a decimal as a percentage string.

#### `uid() → string`
Generates a unique identifier string.

## Keyboard Shortcuts

### Global
- `Shift + ?`: Display help dialog

### Flashcards Tab
- `Enter`: Flip current card
- `1-4`: Rate card (1=Again, 2=Hard, 3=Good, 4=Easy)
- `S`: Shuffle deck

### Graphing Tab
- `Drag`: Pan view
- `Wheel / + / -`: Zoom in/out
- `0`: Reset view
- `Arrow Keys`: Pan view

## Constants

### Application Key
```javascript
const APP_KEY = 'math_journey_v1'; // localStorage key for state persistence
```

### Default Flashcard Deck
The application includes a comprehensive set of flashcards covering:
- **Stage 1**: Order of operations, basic algebra, linear equations, coordinate geometry
- **Stage 2**: Trigonometry, distance formula, quadratic formula
- **Stage 3**: Golden ratio, Fibonacci sequences
- **Stage 4**: Sacred geometry concepts

## Error Handling

The application includes robust error handling for:
- **State loading**: Falls back to initial state on corruption
- **Expression parsing**: Returns null for invalid expressions
- **Mathematical evaluation**: Returns NaN for undefined operations
- **File import**: Validates JSON structure before applying

## Accessibility Features

- **High contrast mode**: Toggle via settings for improved visibility
- **Keyboard navigation**: Full keyboard support for all interactions
- **Screen reader support**: Proper ARIA labels and semantic HTML
- **Focus management**: Visible focus indicators and logical tab order

## Browser Compatibility

- **Modern browsers**: Chrome, Firefox, Safari, Edge (ES6+ features)
- **Canvas support**: Required for geometry and graphing features
- **localStorage**: Required for state persistence
- **Touch events**: Supported for mobile interactions

## Performance Considerations

- **Single-file architecture**: No external dependencies or network requests
- **Efficient rendering**: Canvas-based graphics with proper scaling
- **Memory management**: Automatic cleanup of event listeners and resources
- **Responsive design**: Optimized for various screen sizes

## Integration Examples

### Basic Usage
```javascript
// Initialize the application (called automatically on page load)
window.addEventListener('load', init);

// Access current state
console.log(state.progress.completed.filter(Boolean).length, 'lessons completed');

// Subscribe to state changes
subscribe((newState) => {
  if (newState.progress.completed[0]) {
    console.log('First lesson completed!');
  }
});
```

### Custom Lesson Addition
```javascript
// Add a custom lesson (modify the lessons array before init)
const customLesson = {
  day: 31,
  title: 'Advanced Integration',
  objective: 'Master integration by parts',
  activity: 'Integrate x*sin(x) dx'
};
lessons.push(customLesson);
```

### Custom Flashcard Addition
```javascript
// Add cards to the deck
state.flashcards.deck.push({
  id: uid(),
  front: 'What is the integral of x?',
  back: 'x²/2 + C',
  tags: ['calculus', 'custom']
});
saveState();
scheduleCards();
```

This documentation covers all public APIs and provides comprehensive usage examples for integrating with or extending the 30-Day Math Journey application.