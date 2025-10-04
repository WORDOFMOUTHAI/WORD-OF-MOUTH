# Progress System Documentation

## Overview

The Progress System manages a structured 30-day mathematics learning journey divided into 4 distinct stages. It provides visual progress tracking, lesson management, and milestone celebrations.

## Core Data Structures

### Stages
The learning journey is organized into 4 progressive stages:

```javascript
const stages = [
  {
    id: 1,
    title: 'Stage 1: Foundation Reconstruction',
    range: [1, 10], // Days 1-10
    color: '#7dd3fc' // Light blue
  },
  {
    id: 2,
    title: 'Stage 2: Intermediate Integration',
    range: [11, 18], // Days 11-18
    color: '#60a5fa' // Blue
  },
  {
    id: 3,
    title: 'Stage 3: Classical Mathematical Thinking',
    range: [19, 25], // Days 19-25
    color: '#34d399' // Green
  },
  {
    id: 4,
    title: 'Stage 4: Sacred Geometry Synthesis',
    range: [26, 30], // Days 26-30
    color: '#f59e0b' // Amber
  }
];
```

### Lessons
Each lesson contains structured learning objectives and activities:

```javascript
// Example lesson structure
{
  day: 1,
  title: 'PEMDAS Warm-up',
  objective: 'Rebuild order of operations intuition',
  activity: 'Evaluate: 3 + 6 × 2^2 − 4'
}
```

## API Reference

### Progress Tracking Functions

#### `getStageIdForDay(day: number) → number`
Determines which stage a given day belongs to.

**Parameters:**
- `day`: Day number (1-30)

**Returns:** Stage ID (1-4)

**Usage:**
```javascript
getStageIdForDay(15); // Returns 2 (Stage 2)
getStageIdForDay(25); // Returns 3 (Stage 3)
```

#### `getCurrentLessonIndex() → number`
Returns the index of the next lesson that should be completed.

**Returns:** Lesson index (0-29), or 29 if all lessons are complete

**Usage:**
```javascript
const nextLesson = getCurrentLessonIndex();
selectLesson(nextLesson); // Jump to next lesson
```

#### `milestoneMessage(percentage: number) → string`
Generates encouraging milestone messages based on completion percentage.

**Parameters:**
- `percentage`: Completion percentage (0-1)

**Returns:** Encouraging message string

**Usage:**
```javascript
milestoneMessage(0.25);  // "Great start—25% complete!"
milestoneMessage(0.75);  // "75% milestone—momentum is yours!"
milestoneMessage(1.0);    // "Incredible! You finished. 🎉"
```

### Lesson Management Functions

#### `selectLesson(index: number) → void`
Selects and displays a specific lesson.

**Parameters:**
- `index`: Lesson index (0-29)

**Side Effects:**
- Updates `state.currentLessonIndex`
- Calls `renderCurrentLesson()`
- Refreshes progress UI

**Usage:**
```javascript
// Select lesson 10 (Day 11)
selectLesson(10);

// The lesson is now displayed in the UI
```

#### `renderCurrentLesson() → void`
Renders the currently selected lesson with full details and action buttons.

**Generated HTML Structure:**
```html
<div class="current-lesson-detail">
  <div class="lesson-header">
    <strong>Day X: Lesson Title</strong>
    <div class="lesson-actions">
      <button id="markCompleteBtn">Mark Complete</button>
      <button id="jumpToFlashBtn">Practice Flashcards</button>
      <button id="openGraphWithSampleBtn">Graph Example</button>
    </div>
  </div>
  <div><em>Objective:</em> [Lesson objective]</div>
  <div><em>Quick Activity:</em> [Lesson activity]</div>
  <div class="tip">Tip: After completing, celebrate progress...</div>
</div>
```

### Progress UI Functions

#### `renderStages() → void`
Renders the stage progress cards in the progress tab.

**Generated Structure:**
- Creates stage cards showing completion percentage
- Displays progress bars for each stage
- Shows total lessons completed per stage

#### `renderLessons() → void`
Renders the lesson grid with completion status.

**Lesson Card States:**
- **Current**: Yellow highlight for next lesson to complete
- **Completed**: Green background with "Done" badge
- **Future**: Gray background with "Next" badge

**Usage:**
```javascript
// Re-render lesson grid after completion
state.progress.completed[5] = true;
saveState(); // Triggers UI update via saveState()
```

## Event Handlers

### Lesson Completion
```javascript
// Mark lesson as complete
document.getElementById('markCompleteBtn').addEventListener('click', () => {
  if (!state.progress.completed[currentIndex]) {
    state.progress.completed[currentIndex] = true;
    saveState();
    celebrateIfMilestone(); // Trigger celebration for milestones
  }
});
```

### Lesson Selection
```javascript
// Handle lesson card clicks
card.addEventListener('click', () => selectLesson(lessonIndex));
```

## Milestone Celebrations

### `celebrateIfMilestone() → void`
Checks if current completion count matches a milestone and triggers celebration.

**Milestones:** [1, 5, 10, 15, 20, 25, 30]

**Usage:**
```javascript
// Automatically called after lesson completion
state.progress.completed[9] = true; // 10th lesson completed
saveState(); // Triggers milestone celebration
```

### `launchConfetti() → void`
Creates animated confetti celebration using HTML5 Canvas.

**Animation Details:**
- 180 particles with random colors and velocities
- Gravity and rotation effects
- Automatic cleanup when animation completes

## Integration Examples

### Custom Progress Tracking
```javascript
// Add custom progress tracking
subscribe((state) => {
  const completed = state.progress.completed.filter(Boolean).length;
  console.log(`${completed}/30 lessons completed`);

  // Custom milestone at 50%
  if (completed >= 15 && !state.customMilestone) {
    state.customMilestone = true;
    launchConfetti();
  }
});
```

### Dynamic Lesson Modification
```javascript
// Modify lesson content before rendering
const originalRender = renderCurrentLesson;
renderCurrentLesson = function() {
  originalRender();
  // Add custom content to lesson display
  const detailEl = document.getElementById('currentLessonDetail');
  detailEl.innerHTML += '<div class="custom-note">Custom tip: Practice daily!</div>';
};
```

### Progress-Based Features
```javascript
// Unlock advanced features based on progress
function checkFeatureUnlock() {
  const completed = state.progress.completed.filter(Boolean).length;

  if (completed >= 10) {
    // Unlock advanced graphing features
    document.getElementById('advancedFeatures').hidden = false;
  }

  if (completed >= 20) {
    // Unlock sacred geometry features
    document.getElementById('sacredGeometry').hidden = false;
  }
}
```

## State Persistence

The progress system automatically persists to localStorage via the core state management:

```javascript
// Progress data structure in localStorage
{
  progress: {
    completed: [true, true, false, ...], // 30 boolean values
  },
  currentLessonIndex: 5 // Currently selected lesson
}
```

## Visual Indicators

### Progress Bar
- **Overall Progress**: Top progress bar showing total completion
- **Stage Progress**: Individual progress bars for each stage
- **Color Coding**: Each stage has a distinct color theme

### Lesson Status Pills
- **Green "Done"**: Completed lessons
- **Yellow "Now"**: Current lesson to complete
- **Gray "Next"**: Future lessons

### Milestone Messages
- Contextual encouragement based on completion percentage
- Special celebration messages for key milestones

This progress system provides a comprehensive framework for structured learning with clear visual feedback and motivational elements.