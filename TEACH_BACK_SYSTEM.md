# Teach-Back System Documentation

## Overview

The Teach-Back System implements a reflective learning methodology where students explain concepts in their own words to reinforce understanding. It includes progressive hint systems, self-assessment, and structured reflection prompts.

## Core Concepts

### Learning Through Explanation
The system is based on the principle that explaining a concept to others (or oneself) significantly improves retention and understanding. Users are prompted to teach concepts as if explaining to a curious friend.

### Progressive Disclosure
Hints and guidance are revealed progressively to support learning without giving away answers immediately.

## Data Structures

### Concept Definition
```javascript
{
  key: 'pemdas',           // Unique identifier
  name: 'Order of Operations (PEMDAS)', // Display name
  hints: [                 // Progressive hints array
    'Which operation should be handled first in 3 + 2 × 4?',
    'Try rewriting 3 + 2 × 4 as 3 + (2 × 4). What next?',
    'Multiplication comes before addition, so 3 + (8) = 11.'
  ],
  example: 'PEMDAS organizes what to do first: parentheses, exponents, multiplication/division, then addition/subtraction.'
}
```

### Reflection Entry
```javascript
{
  concept: string,         // Concept name
  prompt: string,          // Generated prompt text
  text: string,            // User's explanation
  timestamp: number,       // Creation timestamp
  selfScore?: number       // Optional self-assessment (1-3)
}
```

### Available Concepts
- **pemdas**: Order of Operations (PEMDAS)
- **linear**: Solving Linear Equations
- **trig**: Right Triangle Trigonometry
- **quadratic**: Quadratics & Roots
- **geometry**: Area and Perimeter

## API Reference

### Concept Management

#### `populateTeachConcepts() → void`
Populates the concept selection dropdown with available topics.

**Usage:**
```javascript
populateTeachConcepts(); // Initialize concept dropdown
```

#### `concepts` Array
Global array containing all available concepts with their hints and examples.

**Structure:**
```javascript
const concepts = [
  {
    key: 'pemdas',
    name: 'Order of Operations (PEMDAS)',
    hints: [/* progressive hints */],
    example: '/* complete example explanation */'
  },
  // ... more concepts
];
```

### Prompt Generation

#### `randomPromptForConcept(concept: Object) → string`
Generates a random prompt for teaching a specific concept.

**Available Prompt Types:**
- "Explain this concept as if to a 12-year-old with one real-life example."
- "Teach it to a colleague returning to math after years; focus on confidence."
- "Write a step-by-step method in under 6 sentences."
- "Describe a common mistake and how to avoid it."

**Usage:**
```javascript
const concept = concepts.find(c => c.key === 'linear');
const prompt = randomPromptForConcept(concept);
// Returns: "Explain linear equations as if to a 12-year-old..."
```

#### `updateTeachPrompt() → void`
Updates the current prompt display and resets hint progression.

**Functionality:**
- Selects random prompt for current concept
- Resets hint list to first hint only
- Updates hint reveal button functionality
- Updates full answer button functionality

### Hint System

#### `addHint(text: string) → void`
Adds a hint to the progressive hint list.

**Parameters:**
- `text`: Hint text to display

**Usage:**
```javascript
addHint('Start with the parentheses first.');
addHint('Then handle any exponents.');
```

#### Progressive Hint Reveal
```javascript
revealNextHintBtn.onclick = () => {
  const shown = hintList.querySelectorAll('li').length;
  if (shown < concept.hints.length) {
    addHint(concept.hints[shown]);
  }
};
```

#### Full Answer Display
```javascript
showFullAnswerBtn.onclick = () => {
  addHint('Full example: ' + concept.example);
};
```

## Reflection Management

### `saveTeachBtn` Event Handler
Saves the user's explanation to the application state.

**Process:**
```javascript
saveTeachBtn.addEventListener('click', () => {
  const concept = concepts.find(c => c.key === teachConceptSelect.value);
  const entry = {
    concept: concept.name,
    prompt: teachPromptEl.textContent,
    text: teachResponse.value,
    timestamp: Date.now()
  };
  state.teachBack.reflections.push(entry);
  saveState();

  // Show success message
  teachSaveMsg.textContent = 'Saved! Reflecting builds mastery.';
  setTimeout(() => teachSaveMsg.textContent = '', 2000);

  // Clear input
  teachResponse.value = '';
});
```

### State Storage
Reflections are persisted with the application state:

```javascript
{
  teachBack: {
    reflections: [
      {
        concept: "Order of Operations (PEMDAS)",
        prompt: "Explain this concept as if to a 12-year-old...",
        text: "PEMDAS is like a recipe for solving math problems...",
        timestamp: 1703123456789,
        selfScore: 3
      }
      // ... more reflections
    ]
  }
}
```

## Self-Assessment System

### Rating Scale
- **3 (Nailed it)**: Complete understanding, confident explanation
- **2 (Almost)**: Good understanding with minor gaps
- **1 (Need practice)**: Significant gaps in understanding

### Assessment Interface
```javascript
document.querySelectorAll('.selfBtn').forEach(btn => {
  btn.addEventListener('click', () => {
    const score = Number(btn.dataset.score);
    const last = state.teachBack.reflections[state.teachBack.reflections.length - 1];

    if (last) {
      last.selfScore = score;
      saveState();

      // Show contextual feedback
      const messages = {
        3: 'Excellent clarity.',
        2: 'Good! Try one more example.',
        1: 'Great effort. Revisit hints and try again.'
      };

      selfMsg.textContent = messages[score];
      setTimeout(() => selfMsg.textContent = '', 2500);
    }
  });
});
```

## UI Components

### Concept Selection
```html
<select id="teachConceptSelect">
  <option value="pemdas">Order of Operations (PEMDAS)</option>
  <option value="linear">Solving Linear Equations</option>
  <option value="trig">Right Triangle Trigonometry</option>
  <option value="quadratic">Quadratics & Roots</option>
  <option value="geometry">Area and Perimeter</option>
</select>
```

### Prompt Display
```html
<div class="card">
  <div id="teachPrompt">
    [Selected Concept]: [Random prompt text]
  </div>
</div>
```

### Response Area
```html
<div class="card">
  <label for="teachResponse"><strong>Your explanation</strong></label>
  <textarea id="teachResponse" placeholder="Explain as if to a curious friend. Use examples."></textarea>
  <div class="button-group">
    <button id="saveTeachBtn">Save Reflection</button>
    <span id="teachSaveMsg"></span>
  </div>
</div>
```

### Hint System
```html
<div class="card">
  <h3>Progressive Hints</h3>
  <ol id="hintList"></ol>
  <button id="revealNextHintBtn">Reveal next hint</button>
  <button id="showFullAnswerBtn">Show full example answer</button>
</div>
```

### Self-Assessment
```html
<div class="card">
  <h3>Self-Assessment</h3>
  <div class="rating-buttons">
    <button class="selfBtn" data-score="3">Nailed it</button>
    <button class="selfBtn" data-score="2">Almost</button>
    <button class="selfBtn" data-score="1">Need practice</button>
    <span id="selfMsg"></span>
  </div>
</div>
```

## Event Handlers

### Concept Selection
```javascript
teachConceptSelect.addEventListener('change', updateTeachPrompt);
```

### New Prompt Generation
```javascript
newPromptBtn.addEventListener('click', updateTeachPrompt);
```

### Reflection Saving
```javascript
saveTeachBtn.addEventListener('click', saveReflection);
```

### Hint Progression
```javascript
revealNextHintBtn.addEventListener('click', revealNextHint);
showFullAnswerBtn.addEventListener('click', showFullAnswer);
```

## Integration Examples

### Custom Concept Addition
```javascript
// Add a new concept to the system
const newConcept = {
  key: 'calculus',
  name: 'Basic Calculus',
  hints: [
    'What does the derivative represent?',
    'Think about slope as an instantaneous rate of change.',
    'The derivative f\'(x) gives the slope of the tangent line at x.'
  ],
  example: 'If position s(t) = t², then velocity v(t) = 2t is the derivative.'
};

concepts.push(newConcept);
populateTeachConcepts();
```

### Reflection Analytics
```javascript
// Analyze reflection patterns
function analyzeReflections() {
  const reflections = state.teachBack.reflections;

  return {
    totalReflections: reflections.length,
    averageScore: reflections
      .filter(r => r.selfScore)
      .reduce((sum, r) => sum + r.selfScore, 0) / reflections.filter(r => r.selfScore).length,
    conceptsPracticed: [...new Set(reflections.map(r => r.concept))],
    improvementOverTime: calculateImprovementTrend(reflections)
  };
}
```

### Export Reflections
```javascript
// Export reflections for external analysis
function exportReflections() {
  const data = state.teachBack.reflections.map(r => ({
    concept: r.concept,
    date: new Date(r.timestamp).toISOString(),
    length: r.text.length,
    score: r.selfScore || 'Not rated'
  }));

  const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'math_reflections.json';
  a.click();
  URL.revokeObjectURL(url);
}
```

## Pedagogical Features

### Prompt Variety
The system uses varied prompts to encourage different explanation styles:
- **Age-appropriate**: Explaining to a 12-year-old
- **Professional**: Teaching returning students
- **Concise**: Step-by-step in limited sentences
- **Error-focused**: Common mistakes and prevention

### Hint Progression
Hints are designed to scaffold learning:
1. **Initial Prompt**: Gentle guidance toward the concept
2. **Intermediate Hints**: Break down the concept into steps
3. **Final Hint**: Complete the thought process

### Self-Assessment Integration
The system encourages metacognition by asking users to rate their own understanding, promoting self-awareness of learning progress.

## Accessibility Features

- **Keyboard Navigation**: Full keyboard support for all interactions
- **Screen Reader Support**: Proper ARIA labels and semantic structure
- **High Contrast Mode**: Automatic adaptation to contrast settings
- **Responsive Design**: Works on various screen sizes

This teach-back system provides a comprehensive framework for reflective learning, encouraging students to engage deeply with mathematical concepts through explanation and self-assessment.