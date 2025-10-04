# Flashcard System Documentation

## Overview

The Flashcard System implements a sophisticated spaced repetition algorithm based on the SM-2 (SuperMemo-2) method. It manages a deck of flashcards with intelligent scheduling, statistics tracking, and user-friendly interaction.

## Core Data Structures

### Flashcard Object
```javascript
{
  id: string,        // Unique identifier (UUID)
  front: string,     // Question/prompt text
  back: string,      // Answer/explanation text
  tags: string[]     // Categorization tags (e.g., ['stage1', 'trig'])
}
```

### Card Statistics
```javascript
{
  ease: number,        // Difficulty factor (1.3-3.0)
  intervalDays: number, // Days until next review (0 = today)
  reps: number,        // Number of successful repetitions
  due: number,         // Next review timestamp (milliseconds)
  lapse: number        // Number of times forgotten
}
```

## Spaced Repetition Algorithm

### Rating System
The system uses a 1-4 rating scale:
- **1 (Again)**: Complete blackout, couldn't recall
- **2 (Hard)**: Difficult to recall, but got it right
- **3 (Good)**: Correct with some hesitation
- **4 (Easy)**: Perfect recall, no hesitation

### SM-2 Algorithm Implementation

#### `rateCurrentCard(grade: number) → void`
Processes a card rating and updates its statistics according to SM-2 rules.

**Algorithm Details:**

**For "Again" (1):**
```javascript
stats.ease = Math.max(1.3, stats.ease - 0.2);
stats.reps = 0;
stats.intervalDays = 0;
stats.lapse += 1;
stats.due = now + 10 * 60 * 1000; // 10 minutes from now
```

**For "Hard" (2):**
```javascript
stats.ease = Math.max(1.3, stats.ease - 0.15);
stats.reps = Math.max(1, stats.reps);
stats.intervalDays = Math.max(1, Math.round(Math.max(1, stats.intervalDays) * 0.5));
stats.due = now + stats.intervalDays * 24 * 60 * 60 * 1000;
```

**For "Good" (3):**
```javascript
stats.ease = Math.min(2.8, stats.ease + 0.1);
stats.reps += 1;
if (stats.reps === 1) stats.intervalDays = 1;
else if (stats.reps === 2) stats.intervalDays = 6;
else stats.intervalDays = Math.round(stats.intervalDays * stats.ease);
stats.due = now + stats.intervalDays * 24 * 60 * 60 * 1000;
```

**For "Easy" (4):**
```javascript
stats.ease = Math.min(3.0, stats.ease + 0.15);
stats.reps += 1;
if (stats.reps === 1) stats.intervalDays = 2;
else if (stats.reps === 2) stats.intervalDays = 8;
else stats.intervalDays = Math.round(stats.intervalDays * (stats.ease + 0.1));
stats.due = now + stats.intervalDays * 24 * 60 * 60 * 1000;
```

## Scheduling Algorithm

### `scheduleCards() → void`
Implements intelligent card scheduling with the following priorities:

1. **Due Cards**: Cards with `due <= now` (highest priority)
2. **Near-Future Cards**: Next 3 cards due soonest (sprinkled in)
3. **New Cards**: If no due cards exist, shows new cards

**Queue Construction:**
```javascript
const now = Date.now();
const dueCards = deck.filter(card => getCardStats(card.id).due <= now);
const notDueCards = deck.filter(card => getCardStats(card.id).due > now)
                       .sort((a,b) => getCardStats(a.id).due - getCardStats(b.id).due);
const sprinkleCount = Math.min(3, notDueCards.length);

dueQueue = [...dueCards, ...notDueCards.slice(0, sprinkleCount)];
```

## API Reference

### Card Management Functions

#### `getCardStats(cardId: string) → CardStats`
Retrieves or creates statistics for a flashcard.

**Parameters:**
- `cardId`: Unique card identifier

**Returns:** Card statistics object

**Usage:**
```javascript
const stats = getCardStats('card-123');
console.log(`Next review in ${stats.intervalDays} days`);
```

#### `getDefaultDeck() → Flashcard[]`
Returns the initial flashcard deck with comprehensive coverage of mathematical concepts.

**Categories:**
- **Stage 1**: Order of operations, basic algebra, linear equations, coordinate geometry
- **Stage 2**: Trigonometry, distance formula, quadratic formula
- **Stage 3**: Golden ratio, Fibonacci sequences
- **Stage 4**: Sacred geometry concepts

**Usage:**
```javascript
const initialDeck = getDefaultDeck();
console.log(`${initialDeck.length} cards in default deck`);
```

### User Interaction Functions

#### `updateFlashcardUI() → void`
Updates the flashcard display and progress indicators.

**Updates:**
- Card content (front/back)
- Progress counter (current/total)
- Due count display
- Card flip state

#### `sanitize(html: string) → string`
Basic HTML sanitization for card content.

**Removes:** `<script>` tags (prevents XSS)

## Event Handlers

### Card Interaction
```javascript
// Card flip on click/tap
flashcardEl.addEventListener('click', () => {
  flashcardEl.classList.toggle('flipped');
});

// Keyboard flip
document.addEventListener('keydown', (e) => {
  if (e.key === 'Enter' && activeTab === 'flash') {
    flashcardEl.classList.toggle('flipped');
  }
});
```

### Rating Buttons
```javascript
// Rate card buttons
againBtn.addEventListener('click', () => rateCurrentCard(1));
hardBtn.addEventListener('click', () => rateCurrentCard(2));
goodBtn.addEventListener('click', () => rateCurrentCard(3));
easyBtn.addEventListener('click', () => rateCurrentCard(4));

// Keyboard ratings
document.addEventListener('keydown', (e) => {
  if (activeTab === 'flash' && ['1','2','3','4'].includes(e.key)) {
    rateCurrentCard(parseInt(e.key));
  }
});
```

### Deck Management
```javascript
// Shuffle deck
shuffleDeckBtn.addEventListener('click', () => {
  state.flashcards.deck.sort(() => Math.random() - 0.5);
  saveState();
  scheduleCards();
});

// Add new card
addCardBtn.addEventListener('click', () => {
  const front = prompt('New card front (question/prompt):');
  const back = prompt('New card back (answer/explanation):') || '';
  if (front) {
    state.flashcards.deck.push({
      id: uid(),
      front,
      back,
      tags: ['custom']
    });
    saveState();
    scheduleCards();
  }
});
```

## Keyboard Shortcuts

### Flashcard Controls
- `Enter`: Flip card between front and back
- `1`: Rate as "Again" (forgot)
- `2`: Rate as "Hard" (difficult)
- `3`: Rate as "Good" (correct with hesitation)
- `4`: Rate as "Easy" (perfect recall)
- `S`: Shuffle the entire deck

## Visual Interface

### Card Display
```html
<div class="flashcard" tabindex="0" aria-label="Flashcard. Press Enter to flip.">
  <div class="face front">
    <div id="flashFront">[Question text]</div>
  </div>
  <div class="face back">
    <div id="flashBack">[Answer text]</div>
  </div>
</div>
```

### Rating Controls
```html
<div class="flash-controls">
  <button id="flipBtn">Flip (Enter)</button>
  <button id="againBtn">Again (1)</button>
  <button id="hardBtn">Hard (2)</button>
  <button id="goodBtn">Good (3)</button>
  <button id="easyBtn">Easy (4)</button>
</div>
```

### Progress Indicators
```html
<span class="chip" id="dueCount">Due: 5</span>
<span class="chip" id="cardProgress">3/8</span>
```

## Integration Examples

### Custom Card Statistics
```javascript
// Add custom statistics tracking
const originalRateCard = rateCurrentCard;
rateCurrentCard = function(grade) {
  originalRateCard(grade);

  // Track average rating per tag
  const card = dueQueue[currentCardIndex];
  if (card && !cardStats[card.id].customTracking) {
    cardStats[card.id].customTracking = {
      totalRatings: 0,
      averageRating: 0
    };
  }

  const stats = cardStats[card.id].customTracking;
  stats.totalRatings++;
  stats.averageRating = (stats.averageRating * (stats.totalRatings - 1) + grade) / stats.totalRatings;
};
```

### Tag-Based Filtering
```javascript
// Filter cards by tags
function getCardsByTag(tag) {
  return state.flashcards.deck.filter(card => card.tags.includes(tag));
}

function studyCardsByTag(tag) {
  const tagCards = getCardsByTag(tag);
  // Custom scheduling for tag-specific study
  dueQueue = tagCards.filter(card => getCardStats(card.id).due <= Date.now());
  updateFlashcardUI();
}
```

### Performance Analytics
```javascript
// Analyze learning performance
function getPerformanceStats() {
  const stats = Object.values(state.flashcards.stats);
  return {
    averageEase: stats.reduce((sum, s) => sum + s.ease, 0) / stats.length,
    averageInterval: stats.reduce((sum, s) => sum + s.intervalDays, 0) / stats.length,
    totalReviews: stats.reduce((sum, s) => sum + s.reps, 0),
    lapseRate: stats.reduce((sum, s) => sum + s.lapse, 0) / stats.length
  };
}
```

## State Persistence

Flashcard data is automatically persisted with the application state:

```javascript
{
  flashcards: {
    deck: [/* Flashcard objects */],
    stats: {
      "card-id-1": { ease: 2.4, intervalDays: 3, reps: 5, due: 1234567890, lapse: 0 },
      "card-id-2": { /* ... */ }
    }
  }
}
```

## Error Handling

The system includes robust error handling:
- **Invalid ratings**: Silently ignored
- **Missing cards**: Falls back to showing no cards due
- **Corrupted stats**: Creates new statistics objects
- **Scheduling errors**: Uses fallback scheduling

This flashcard system provides a complete spaced repetition learning experience with intelligent scheduling, comprehensive statistics, and flexible card management.