# Usage Examples and Integration Guide

This document provides practical examples and code snippets for using and extending the 30-Day Math Journey application.

## Table of Contents
1. [Basic Usage](#basic-usage)
2. [State Management Examples](#state-management-examples)
3. [Flashcard System Examples](#flashcard-system-examples)
4. [Geometry Examples](#geometry-examples)
5. [Graphing Examples](#graphing-examples)
6. [Teach-Back Examples](#teach-back-examples)
7. [Custom Extensions](#custom-extensions)
8. [Integration Patterns](#integration-patterns)
9. [Testing Examples](#testing-examples)

---

## Basic Usage

### Opening the Application

```bash
# Option 1: Direct file open
# Double-click index.html in file manager

# Option 2: Local server (optional)
python3 -m http.server 8000
# Open http://localhost:8000

# Option 3: Live Server (VS Code)
# Right-click index.html → "Open with Live Server"
```

### First Time Setup

```javascript
// Application initializes automatically on page load
window.addEventListener('load', init);

// init() performs:
// 1. Applies theme settings
// 2. Updates progress UI
// 3. Renders current lesson
// 4. Schedules flashcards
// 5. Resizes canvases
// 6. Populates teach concepts
```

---

## State Management Examples

### Reading Current State

```javascript
// Open browser console (F12)

// View entire state
console.log(state);

// Check completed lessons
console.log(state.progress.completed);
// [false, false, true, true, ...]

// Count completed
const completedCount = state.progress.completed.filter(Boolean).length;
console.log(`${completedCount}/30 lessons completed`);

// View current lesson
console.log(`Currently on lesson ${state.currentLessonIndex + 1}`);

// View flashcard deck size
console.log(`Deck has ${state.flashcards.deck.length} cards`);

// Check high contrast mode
console.log(`High contrast: ${state.settings.highContrast}`);
```

### Modifying State

```javascript
// Complete a lesson
state.progress.completed[0] = true;
saveState();
// Progress bar updates automatically

// Jump to a specific lesson
selectLesson(15);  // Jump to lesson 16

// Toggle high contrast
state.settings.highContrast = !state.settings.highContrast;
applyContrastSetting();
saveState();

// Mark multiple lessons complete
[0, 1, 2, 3, 4].forEach(i => {
  state.progress.completed[i] = true;
});
saveState();
celebrateIfMilestone(); // Confetti at lesson 5!
```

### State Persistence

```javascript
// Manually save state
saveState();
// Writes to localStorage['math_journey_v1']

// Manually load state
state = loadState();
// Reads from localStorage or creates initial state

// View raw localStorage
const raw = localStorage.getItem('math_journey_v1');
console.log(JSON.parse(raw));

// Clear state (reset app)
localStorage.removeItem('math_journey_v1');
location.reload();
```

### State Subscription

```javascript
// Subscribe to state changes
subscribe((newState) => {
  console.log('State changed!');
  console.log('Completed lessons:', 
    newState.progress.completed.filter(Boolean).length);
});

// Trigger notification
state.progress.completed[10] = true;
notify(); // Subscriber function called
```

---

## Flashcard System Examples

### Adding Custom Flashcards

#### Via UI
```javascript
// User clicks "Add Card" button
// Prompts appear for front and back
// Card automatically added to deck
```

#### Via Console
```javascript
// Add a single card
state.flashcards.deck.push({
  id: uid(),
  front: "What is the derivative of x²?",
  back: "2x",
  tags: ['calculus', 'custom']
});
saveState();
scheduleCards(); // Update due queue

// Add multiple cards
const newCards = [
  {
    front: "Factor: x² - 9",
    back: "(x + 3)(x - 3)"
  },
  {
    front: "Simplify: √16",
    back: "4"
  },
  {
    front: "What is 30% of 80?",
    back: "24"
  }
];

newCards.forEach(card => {
  state.flashcards.deck.push({
    id: uid(),
    front: card.front,
    back: card.back,
    tags: ['custom']
  });
});

saveState();
scheduleCards();
```

### Viewing Card Statistics

```javascript
// View stats for a specific card
const card = state.flashcards.deck[0];
const stats = getCardStats(card.id);

console.log(`Card: ${card.front}`);
console.log(`Ease: ${stats.ease}`);
console.log(`Interval: ${stats.intervalDays} days`);
console.log(`Reps: ${stats.reps}`);
console.log(`Lapses: ${stats.lapse}`);
console.log(`Due: ${new Date(stats.due)}`);

// Find all due cards
const now = Date.now();
const dueCards = state.flashcards.deck.filter(card => {
  const stats = getCardStats(card.id);
  return stats.due <= now;
});
console.log(`${dueCards.length} cards due now`);
```

### Manipulating Card Schedule

```javascript
// Mark all cards as due immediately
state.flashcards.deck.forEach(card => {
  const stats = getCardStats(card.id);
  stats.due = Date.now();
  stats.reps = 0;
  stats.intervalDays = 0;
});
saveState();
scheduleCards();

// Reset a card's statistics
const card = state.flashcards.deck[0];
state.flashcards.stats[card.id] = {
  ease: 2.4,
  intervalDays: 0,
  reps: 0,
  due: Date.now(),
  lapse: 0
};
saveState();

// Find difficult cards (high lapse count)
const difficultCards = state.flashcards.deck.filter(card => {
  const stats = getCardStats(card.id);
  return stats.lapse >= 3;
});
console.log(`${difficultCards.length} difficult cards`);
```

### Custom Deck Import

```javascript
// Import deck from JSON array
const deckData = [
  {
    front: "What is Euler's number?",
    back: "e ≈ 2.71828...",
    tags: ['constants']
  },
  {
    front: "Define limit",
    back: "The value a function approaches as input approaches a point",
    tags: ['calculus']
  }
];

deckData.forEach(cardData => {
  state.flashcards.deck.push({
    id: uid(),
    ...cardData
  });
});

saveState();
scheduleCards();
console.log('Imported', deckData.length, 'cards');
```

---

## Geometry Examples

### Programmatic Shape Manipulation

```javascript
// Switch to geometry tab
switchTab('geometry');

// Create an equilateral triangle
const side = 100;
const height = side * Math.sqrt(3) / 2;
geomState.vertices = [
  {x: -side/2, y: height/3},
  {x: 0, y: -2*height/3},
  {x: side/2, y: height/3}
];
geomState.shape = 'triangle';
drawGeometry();

// Create a golden rectangle
const phi = (1 + Math.sqrt(5)) / 2; // ~1.618
geomState.rect = {
  x: -100,
  y: -100/phi,
  w: 200,
  h: 200/phi
};
geomState.shape = 'rectangle';
drawGeometry();

// Create a perfect circle
geomState.circle = {
  x: 0,
  y: 0,
  r: 100
};
geomState.shape = 'circle';
drawGeometry();
```

### Reading Measurements

```javascript
// Get triangle measurements
if (geomState.shape === 'triangle') {
  const [A, B, C] = geomState.vertices;
  const side = (P, Q) => Math.hypot(P.x - Q.x, P.y - Q.y);
  const a = side(B, C);
  const b = side(A, C);
  const c = side(A, B);
  const perimeter = a + b + c;
  const s = perimeter / 2;
  const area = Math.sqrt(s * (s-a) * (s-b) * (s-c));
  
  console.log(`Triangle perimeter: ${perimeter.toFixed(2)}`);
  console.log(`Triangle area: ${area.toFixed(2)}`);
}

// Get circle measurements
if (geomState.shape === 'circle') {
  const circumference = 2 * Math.PI * geomState.circle.r;
  const area = Math.PI * Math.pow(geomState.circle.r, 2);
  
  console.log(`Circumference: ${circumference.toFixed(2)}`);
  console.log(`Area: ${area.toFixed(2)}`);
}
```

### Custom Shape Animations

```javascript
// Animate circle radius
let radius = 50;
let growing = true;

function animateCircle() {
  radius += growing ? 2 : -2;
  if (radius >= 150) growing = false;
  if (radius <= 50) growing = true;
  
  geomState.circle.r = radius;
  drawGeometry();
  
  requestAnimationFrame(animateCircle);
}

geomState.shape = 'circle';
animateCircle();
// Stop with: cancelAnimationFrame(animationId)
```

### Coordinate Transformations

```javascript
// Convert between world and screen coordinates
const worldPoint = {x: 50, y: 50};
const screenPoint = worldToScreen(worldPoint);
console.log(`World ${worldPoint.x},${worldPoint.y} → Screen ${screenPoint.x},${screenPoint.y}`);

const backToWorld = screenToWorld(screenPoint);
console.log(`Screen ${screenPoint.x},${screenPoint.y} → World ${backToWorld.x},${backToWorld.y}`);

// Test with different zoom levels
geomState.zoom = 2;
const zoomed = worldToScreen(worldPoint);
console.log(`2x zoom: ${zoomed.x},${zoomed.y}`);
```

---

## Graphing Examples

### Plotting Various Functions

```javascript
// Switch to graph tab
switchTab('graph');

// Linear function
exprInput.value = '2x + 3';
plotCurrentExpression();

// Quadratic
exprInput.value = 'x^2 - 4x + 3';
plotCurrentExpression();

// Trigonometric
exprInput.value = 'sin(x) + cos(2x)';
plotCurrentExpression();

// Exponential
exprInput.value = 'exp(-x^2)';  // Gaussian
plotCurrentExpression();

// Rational
exprInput.value = '1/x';
plotCurrentExpression();

// Composite
exprInput.value = 'sqrt(x) * ln(x)';
plotCurrentExpression();

// Piecewise approximation
exprInput.value = 'abs(x) - 2';
plotCurrentExpression();
```

### Finding Function Values

```javascript
// Compile an expression
const expr = 'x^2 - 4x + 3';
const compiled = compile(expr);

// Evaluate at specific points
console.log(`f(0) = ${safeEval(compiled, 0)}`);  // 3
console.log(`f(1) = ${safeEval(compiled, 1)}`);  // 0
console.log(`f(2) = ${safeEval(compiled, 2)}`);  // -1
console.log(`f(3) = ${safeEval(compiled, 3)}`);  // 0
console.log(`f(4) = ${safeEval(compiled, 4)}`);  // 3

// Find minimum by sampling
let minY = Infinity;
let minX = 0;
for (let x = -10; x <= 10; x += 0.1) {
  const y = safeEval(compiled, x);
  if (y < minY) {
    minY = y;
    minX = x;
  }
}
console.log(`Minimum near (${minX.toFixed(2)}, ${minY.toFixed(2)})`);
```

### Custom Intercept Finding

```javascript
// Find x-intercepts (roots)
const expr = 'x^2 - 4';
const compiled = compile(expr);
const intercepts = findIntercepts(compiled);

console.log(`X-intercepts: ${intercepts.x.join(', ')}`);
// [-2, 2]
console.log(`Y-intercept: ${intercepts.y}`);
// -4

// Verify roots
intercepts.x.forEach(root => {
  const value = safeEval(compiled, root);
  console.log(`f(${root}) = ${value.toFixed(6)}`);
  // Should be very close to 0
});
```

### Graph Navigation

```javascript
// Pan to specific location
graphState.origin.x = 100;  // Move origin 100px right
graphState.origin.y = -50;  // Move origin 50px up
drawGraph();

// Zoom in/out
graphState.zoom *= 1.5;  // 50% zoom in
drawGraph();

graphState.zoom *= 0.667; // Back to original
drawGraph();

// Reset view
graphState.zoom = 60;
graphState.origin = {x: 0, y: 0};
drawGraph();

// Focus on a specific point
function focusOn(x, y, zoomLevel = 100) {
  // Convert world coordinates to screen offset
  graphState.zoom = zoomLevel;
  graphState.origin.x = 0;
  graphState.origin.y = 0;
  drawGraph();
}

focusOn(2, 0);  // Focus on (2, 0)
```

### Custom Math Functions

```javascript
// Add a custom function
FUNCTIONS.sigmoid = (x) => 1 / (1 + Math.exp(-x));

// Use it
exprInput.value = 'sigmoid(x)';
plotCurrentExpression();

// Add secant
FUNCTIONS.sec = (x) => 1 / Math.cos(x);

// Add cosecant
FUNCTIONS.csc = (x) => 1 / Math.sin(x);

// Add cotangent
FUNCTIONS.cot = (x) => 1 / Math.tan(x);

// Now usable:
exprInput.value = 'sec(x) - csc(x)';
plotCurrentExpression();

// Add custom constant
CONSTANTS.phi = (1 + Math.sqrt(5)) / 2;

// Use it:
exprInput.value = 'phi * x';
plotCurrentExpression();
```

---

## Teach-Back Examples

### Adding Custom Concepts

```javascript
// Add a new concept
concepts.push({
  key: 'derivatives',
  name: 'Introduction to Derivatives',
  hints: [
    'A derivative measures the rate of change of a function.',
    'Think of it as the slope of the tangent line at a point.',
    'The power rule: d/dx(x^n) = n*x^(n-1)'
  ],
  example: 'For f(x) = x³, the derivative is f\'(x) = 3x². At x=2, the slope is 3(2²) = 12.'
});

// Refresh the concept selector
populateTeachConcepts();

// Now available in dropdown
```

### Programmatic Reflection Creation

```javascript
// Create a reflection entry
const reflection = {
  concept: 'Solving Linear Equations',
  prompt: 'Explain the process step-by-step',
  text: `To solve a linear equation:
1. Identify the variable to solve for
2. Move all terms with the variable to one side
3. Move all constants to the other side
4. Divide both sides by the variable's coefficient
Example: 3x + 5 = 14 → 3x = 9 → x = 3`,
  timestamp: Date.now(),
  selfScore: 3
};

state.teachBack.reflections.push(reflection);
saveState();

console.log(`Added reflection. Total: ${state.teachBack.reflections.length}`);
```

### Analyzing Reflections

```javascript
// View all reflections
console.table(state.teachBack.reflections.map(r => ({
  concept: r.concept,
  score: r.selfScore || 'N/A',
  date: new Date(r.timestamp).toLocaleDateString()
})));

// Calculate average self-score
const scores = state.teachBack.reflections
  .filter(r => r.selfScore)
  .map(r => r.selfScore);
const avgScore = scores.reduce((a, b) => a + b, 0) / scores.length;
console.log(`Average self-score: ${avgScore.toFixed(2)}`);

// Find concepts with low scores
const needsWork = state.teachBack.reflections.filter(r => 
  r.selfScore === 1
);
console.log('Concepts needing more practice:');
needsWork.forEach(r => console.log(`- ${r.concept}`));

// Export reflections to text
const exportText = state.teachBack.reflections.map(r => `
Concept: ${r.concept}
Date: ${new Date(r.timestamp).toLocaleDateString()}
Prompt: ${r.prompt}
Response: ${r.text}
Self-Score: ${r.selfScore || 'N/A'}
${'='.repeat(60)}
`).join('\n');

console.log(exportText);
// Can be copied and saved to a file
```

---

## Custom Extensions

### Adding a New Tab

```javascript
// 1. Add HTML section (in index.html)
// <section class="card" id="tab-custom" hidden>
//   <h2>Custom Tab</h2>
//   <div id="customContent"></div>
// </section>

// 2. Add navigation button (in header)
// <button class="tab-btn" data-tab="custom">Custom</button>

// 3. Update switchTab function
function switchTabExtended(id) {
  // ... existing tabs ...
  document.getElementById('tab-custom').hidden = id !== 'custom';
  
  if (id === 'custom') {
    // Initialize custom tab
    initCustomTab();
  }
}

// 4. Implement custom tab logic
function initCustomTab() {
  const content = document.getElementById('customContent');
  content.innerHTML = '<p>Welcome to custom tab!</p>';
  
  // Add interactivity
  const button = document.createElement('button');
  button.textContent = 'Click me';
  button.onclick = () => alert('Custom action!');
  content.appendChild(button);
}

// 5. Update getActiveTabId
function getActiveTabIdExtended() {
  if (!document.getElementById('tab-custom').hidden) return 'tab-custom';
  // ... rest of checks ...
}
```

### Custom Progress Tracking

```javascript
// Track time spent on each lesson
const lessonTimes = {};

function startLessonTimer(lessonIndex) {
  lessonTimes[lessonIndex] = lessonTimes[lessonIndex] || 0;
  const startTime = Date.now();
  
  // Return stop function
  return () => {
    const elapsed = Date.now() - startTime;
    lessonTimes[lessonIndex] += elapsed;
  };
}

// Usage
const stopTimer = startLessonTimer(0);
// ... user works on lesson ...
stopTimer();

console.log(`Lesson 1 time: ${lessonTimes[0] / 1000} seconds`);

// Save to state
if (!state.customData) state.customData = {};
state.customData.lessonTimes = lessonTimes;
saveState();
```

### Custom Celebration Effects

```javascript
// Add sound effect on milestone
function celebrateWithSound() {
  const audioContext = new (window.AudioContext || window.webkitAudioContext)();
  const oscillator = audioContext.createOscillator();
  const gainNode = audioContext.createGain();
  
  oscillator.connect(gainNode);
  gainNode.connect(audioContext.destination);
  
  oscillator.frequency.value = 440; // A note
  oscillator.type = 'sine';
  
  gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
  gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 1);
  
  oscillator.start(audioContext.currentTime);
  oscillator.stop(audioContext.currentTime + 1);
}

// Use in celebrateIfMilestone
function celebrateIfMilestoneExtended() {
  celebrateIfMilestone(); // Original confetti
  celebrateWithSound();   // Add sound
}
```

### Data Export to CSV

```javascript
// Export progress to CSV
function exportProgressCSV() {
  const headers = 'Day,Title,Completed,Stage\n';
  const rows = lessons.map((lesson, i) => {
    const completed = state.progress.completed[i] ? 'Yes' : 'No';
    const stage = getStageIdForDay(lesson.day);
    return `${lesson.day},"${lesson.title}",${completed},${stage}`;
  }).join('\n');
  
  const csv = headers + rows;
  const blob = new Blob([csv], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);
  
  const a = document.createElement('a');
  a.href = url;
  a.download = 'progress.csv';
  a.click();
  
  URL.revokeObjectURL(url);
}

// Export flashcard statistics to CSV
function exportFlashcardStatsCSV() {
  const headers = 'Front,Ease,Interval,Reps,Lapses\n';
  const rows = state.flashcards.deck.map(card => {
    const stats = getCardStats(card.id);
    return `"${card.front}",${stats.ease},${stats.intervalDays},${stats.reps},${stats.lapse}`;
  }).join('\n');
  
  const csv = headers + rows;
  const blob = new Blob([csv], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);
  
  const a = document.createElement('a');
  a.href = url;
  a.download = 'flashcard_stats.csv';
  a.click();
  
  URL.revokeObjectURL(url);
}
```

---

## Integration Patterns

### Embedding in Another Application

```html
<!-- In parent application -->
<!DOCTYPE html>
<html>
<head>
  <title>My Learning Platform</title>
</head>
<body>
  <h1>Welcome to My Platform</h1>
  
  <!-- Embed math journey in iframe -->
  <iframe src="index.html" 
          width="100%" 
          height="800px" 
          frameborder="0">
  </iframe>
</body>
</html>
```

### PostMessage Communication

```javascript
// In parent page
const iframe = document.querySelector('iframe');

// Send message to iframe
iframe.contentWindow.postMessage({
  type: 'SET_LESSON',
  lessonIndex: 5
}, '*');

// Listen for messages from iframe
window.addEventListener('message', (event) => {
  if (event.data.type === 'LESSON_COMPLETED') {
    console.log('Lesson completed:', event.data.lessonIndex);
    // Update external system
  }
});

// In index.html (add to existing code)
window.addEventListener('message', (event) => {
  if (event.data.type === 'SET_LESSON') {
    selectLesson(event.data.lessonIndex);
  }
});

// Send completion event
function notifyParentOfCompletion(lessonIndex) {
  if (window.parent !== window) {
    window.parent.postMessage({
      type: 'LESSON_COMPLETED',
      lessonIndex: lessonIndex
    }, '*');
  }
}
```

### REST API Integration

```javascript
// Sync progress to server
async function syncProgressToServer() {
  try {
    const response = await fetch('https://api.example.com/progress', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer YOUR_TOKEN'
      },
      body: JSON.stringify({
        userId: 'user123',
        progress: state.progress,
        timestamp: Date.now()
      })
    });
    
    const result = await response.json();
    console.log('Synced to server:', result);
  } catch (error) {
    console.error('Sync failed:', error);
  }
}

// Load progress from server
async function loadProgressFromServer() {
  try {
    const response = await fetch('https://api.example.com/progress/user123', {
      headers: {
        'Authorization': 'Bearer YOUR_TOKEN'
      }
    });
    
    const data = await response.json();
    state.progress = data.progress;
    saveState();
    updateProgressUI();
    
    console.log('Loaded from server:', data);
  } catch (error) {
    console.error('Load failed:', error);
  }
}

// Auto-sync on state changes
const originalSaveState = saveState;
saveState = function() {
  originalSaveState();
  syncProgressToServer(); // Sync after every save
};
```

### Analytics Integration

```javascript
// Track events with Google Analytics
function trackEvent(category, action, label, value) {
  if (typeof gtag !== 'undefined') {
    gtag('event', action, {
      'event_category': category,
      'event_label': label,
      'value': value
    });
  }
}

// Track lesson completion
const originalCelebrateIfMilestone = celebrateIfMilestone;
celebrateIfMilestone = function() {
  originalCelebrateIfMilestone();
  
  const completedCount = state.progress.completed.filter(Boolean).length;
  trackEvent('Progress', 'lesson_completed', `Lesson ${completedCount}`, completedCount);
};

// Track flashcard practice
const originalRateCurrentCard = rateCurrentCard;
rateCurrentCard = function(grade) {
  originalRateCurrentCard(grade);
  
  const ratings = ['', 'again', 'hard', 'good', 'easy'];
  trackEvent('Flashcards', 'card_rated', ratings[grade], grade);
};

// Track time spent
let sessionStart = Date.now();
window.addEventListener('beforeunload', () => {
  const sessionDuration = Date.now() - sessionStart;
  trackEvent('Engagement', 'session_duration', 'Total time', sessionDuration);
});
```

---

## Testing Examples

### Unit Testing with Jest

```javascript
// mathJourney.test.js

describe('Utility Functions', () => {
  test('clamp constrains values', () => {
    expect(clamp(5, 0, 10)).toBe(5);
    expect(clamp(-5, 0, 10)).toBe(0);
    expect(clamp(15, 0, 10)).toBe(10);
  });
  
  test('lerp interpolates correctly', () => {
    expect(lerp(0, 100, 0.5)).toBe(50);
    expect(lerp(10, 20, 0.25)).toBe(12.5);
  });
  
  test('formatPercent converts decimals', () => {
    expect(formatPercent(0.75)).toBe('75%');
    expect(formatPercent(0.3333)).toBe('33%');
  });
  
  test('uid generates unique IDs', () => {
    const id1 = uid();
    const id2 = uid();
    expect(id1).not.toBe(id2);
    expect(typeof id1).toBe('string');
  });
});

describe('Expression Parser', () => {
  test('tokenize handles basic expressions', () => {
    expect(tokenize('2 + 3')).toEqual(['2', '+', '3']);
    expect(tokenize('2x')).toEqual(['2', '*', 'x']);
    expect(tokenize('sin(x)')).toEqual(['sin', '(', 'x', ')']);
  });
  
  test('toRPN converts infix to postfix', () => {
    expect(toRPN(['2', '+', '3'])).toEqual(['2', '3', '+']);
    expect(toRPN(['2', '*', 'x', '+', '3'])).toEqual(['2', 'x', '*', '3', '+']);
  });
  
  test('evalRPN evaluates correctly', () => {
    expect(evalRPN(['2', '3', '+'], 0)).toBe(5);
    expect(evalRPN(['2', 'x', '*', '3', '+'], 5)).toBe(13);
  });
});

describe('Flashcard System', () => {
  test('getCardStats creates default stats', () => {
    const stats = getCardStats('new_card_id');
    expect(stats.ease).toBe(2.4);
    expect(stats.reps).toBe(0);
    expect(stats.intervalDays).toBe(0);
  });
  
  test('rateCurrentCard updates stats correctly', () => {
    // Setup
    state.flashcards.deck = [{ id: 'test1', front: 'Q', back: 'A', tags: [] }];
    dueQueue = [state.flashcards.deck[0]];
    currentCardIndex = 0;
    
    // Rate as good
    rateCurrentCard(3);
    
    const stats = getCardStats('test1');
    expect(stats.reps).toBe(1);
    expect(stats.intervalDays).toBe(1);
  });
});
```

### Integration Testing with Puppeteer

```javascript
// integration.test.js

const puppeteer = require('puppeteer');

describe('Math Journey Integration Tests', () => {
  let browser;
  let page;
  
  beforeAll(async () => {
    browser = await puppeteer.launch();
    page = await browser.newPage();
    await page.goto('file:///path/to/index.html');
  });
  
  afterAll(async () => {
    await browser.close();
  });
  
  test('Page loads with correct title', async () => {
    const title = await page.title();
    expect(title).toContain('30-Day Math Journey');
  });
  
  test('Can complete a lesson', async () => {
    // Click first lesson
    await page.click('.lesson-card');
    
    // Wait for lesson detail to appear
    await page.waitForSelector('#markCompleteBtn');
    
    // Click mark complete
    await page.click('#markCompleteBtn');
    
    // Check progress updated
    const progressText = await page.$eval('#progressText', el => el.textContent);
    expect(progressText).toContain('1/30');
  });
  
  test('Can flip flashcard', async () => {
    // Switch to flashcards tab
    await page.click('[data-tab="flash"]');
    
    // Click flashcard
    await page.click('#flashcard');
    
    // Check for flipped class
    const isFlipped = await page.$eval('#flashcard', el => 
      el.classList.contains('flipped')
    );
    expect(isFlipped).toBe(true);
  });
  
  test('Can plot a function', async () => {
    // Switch to graph tab
    await page.click('[data-tab="graph"]');
    
    // Enter expression
    await page.type('#exprInput', 'x^2');
    
    // Click plot
    await page.click('#plotBtn');
    
    // Wait for graph to render
    await page.waitForTimeout(500);
    
    // Check overlay appears
    const overlayVisible = await page.$eval('#solveOverlay', el => 
      !el.hidden
    );
    expect(overlayVisible).toBe(true);
  });
});
```

### Visual Regression Testing

```javascript
// visual.test.js

const puppeteer = require('puppeteer');
const pixelmatch = require('pixelmatch');
const PNG = require('pngjs').PNG;
const fs = require('fs');

async function captureScreenshot(page, name) {
  await page.screenshot({ path: `screenshots/${name}.png` });
}

async function compareScreenshots(baseline, current) {
  const img1 = PNG.sync.read(fs.readFileSync(baseline));
  const img2 = PNG.sync.read(fs.readFileSync(current));
  const {width, height} = img1;
  const diff = new PNG({width, height});
  
  const numDiffPixels = pixelmatch(
    img1.data, img2.data, diff.data, 
    width, height, 
    {threshold: 0.1}
  );
  
  return numDiffPixels;
}

describe('Visual Regression Tests', () => {
  let browser, page;
  
  beforeAll(async () => {
    browser = await puppeteer.launch();
    page = await browser.newPage();
    await page.setViewport({ width: 1280, height: 800 });
    await page.goto('file:///path/to/index.html');
  });
  
  afterAll(async () => {
    await browser.close();
  });
  
  test('Progress tab appearance', async () => {
    await captureScreenshot(page, 'progress-tab');
    const diff = await compareScreenshots(
      'baselines/progress-tab.png',
      'screenshots/progress-tab.png'
    );
    expect(diff).toBeLessThan(100); // Allow minor differences
  });
  
  test('Flashcard appearance', async () => {
    await page.click('[data-tab="flash"]');
    await captureScreenshot(page, 'flashcard');
    const diff = await compareScreenshots(
      'baselines/flashcard.png',
      'screenshots/flashcard.png'
    );
    expect(diff).toBeLessThan(100);
  });
});
```

---

## Performance Testing

### Measuring Render Performance

```javascript
// Measure graph rendering time
function measureGraphPerformance() {
  const iterations = 100;
  const start = performance.now();
  
  for (let i = 0; i < iterations; i++) {
    drawGraph();
  }
  
  const end = performance.now();
  const avgTime = (end - start) / iterations;
  
  console.log(`Average graph render time: ${avgTime.toFixed(2)}ms`);
  console.log(`Target: <16.67ms for 60fps`);
}

// Measure expression evaluation
function measureExpressionPerformance() {
  const expr = 'sin(x) + cos(2*x) + x^2';
  const compiled = compile(expr);
  const iterations = 10000;
  
  const start = performance.now();
  for (let i = 0; i < iterations; i++) {
    safeEval(compiled, i * 0.1);
  }
  const end = performance.now();
  
  const avgTime = (end - start) / iterations;
  console.log(`Average evaluation time: ${avgTime.toFixed(4)}ms`);
}

// Measure state save/load
function measureStatePerformance() {
  const iterations = 1000;
  
  const saveStart = performance.now();
  for (let i = 0; i < iterations; i++) {
    localStorage.setItem('test', JSON.stringify(state));
  }
  const saveEnd = performance.now();
  
  const loadStart = performance.now();
  for (let i = 0; i < iterations; i++) {
    JSON.parse(localStorage.getItem('test'));
  }
  const loadEnd = performance.now();
  
  console.log(`Save time: ${((saveEnd - saveStart) / iterations).toFixed(4)}ms`);
  console.log(`Load time: ${((loadEnd - loadStart) / iterations).toFixed(4)}ms`);
  
  localStorage.removeItem('test');
}
```

---

## Debugging Tips

### Enable Verbose Logging

```javascript
// Add to beginning of script
const DEBUG = true;

function log(...args) {
  if (DEBUG) console.log('[DEBUG]', ...args);
}

// Use throughout code
function selectLesson(index) {
  log('Selecting lesson:', index);
  state.currentLessonIndex = index;
  log('New state:', state);
  saveState();
  renderCurrentLesson();
  updateProgressUI();
}
```

### State Inspector

```javascript
// Add to console
function inspectState() {
  console.group('State Inspector');
  console.log('Completed lessons:', state.progress.completed.filter(Boolean).length);
  console.log('Current lesson:', state.currentLessonIndex + 1);
  console.log('Flashcard deck size:', state.flashcards.deck.length);
  console.log('Due cards:', dueQueue.length);
  console.log('High contrast:', state.settings.highContrast);
  console.log('Reflections:', state.teachBack.reflections.length);
  console.groupEnd();
}

// Run periodically
setInterval(inspectState, 10000); // Every 10 seconds
```

### Canvas Debug Overlay

```javascript
// Draw debug info on canvas
function drawDebugOverlay() {
  const ctx = graphCanvas.getContext('2d');
  ctx.save();
  ctx.fillStyle = 'rgba(255, 255, 0, 0.8)';
  ctx.font = '12px monospace';
  ctx.fillText(`Zoom: ${graphState.zoom.toFixed(2)}`, 10, 20);
  ctx.fillText(`Origin: (${graphState.origin.x}, ${graphState.origin.y})`, 10, 40);
  ctx.fillText(`Expression: ${graphState.expr}`, 10, 60);
  ctx.restore();
}

// Call after drawGraph()
```

---

This examples document provides comprehensive code samples for using, extending, and integrating the 30-Day Math Journey application.
