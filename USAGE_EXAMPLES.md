# Usage Examples & Integration Guide

## Overview

This guide provides practical examples for using the 30-Day Math Journey APIs, from basic usage to advanced customization and integration scenarios.

## Basic Usage

### Initializing the Application

The application initializes automatically when the page loads:

```javascript
window.addEventListener('load', init);
```

However, you can also initialize it manually:

```javascript
// Manual initialization
applyContrastSetting();
updateProgressUI();
renderCurrentLesson();
scheduleCards();
resizeGeom();
resizeGraph();
populateTeachConcepts();
updateTeachPrompt();
```

### Accessing Application State

```javascript
// Access current state
console.log('Completed lessons:', state.progress.completed.filter(Boolean).length);
console.log('Current lesson:', lessons[state.currentLessonIndex].title);
console.log('Flashcards due:', state.flashcards.deck.filter(card =>
  getCardStats(card.id).due <= Date.now()
).length);
```

## Progress System Integration

### Tracking Progress Programmatically

```javascript
// Subscribe to progress updates
subscribe((newState) => {
  const completed = newState.progress.completed.filter(Boolean).length;
  const total = newState.progress.completed.length;
  const percentage = (completed / total) * 100;

  // Update external progress indicator
  document.getElementById('external-progress').textContent =
    `Math Journey: ${completed}/${total} (${Math.round(percentage)}%)`;

  // Trigger external achievements
  if (completed === 5) {
    unlockAchievement('First Milestone');
  }
});
```

### Custom Lesson Integration

```javascript
// Add external lesson content
const originalRenderCurrentLesson = renderCurrentLesson;

renderCurrentLesson = function() {
  originalRenderCurrentLesson();

  // Add external resource links
  const lessonEl = document.getElementById('currentLessonDetail');
  const currentLesson = lessons[state.currentLessonIndex];

  if (currentLesson.day <= 10) {
    lessonEl.innerHTML += `
      <div class="external-resources">
        <h4>Additional Resources</h4>
        <a href="https://example.com/algebra-basics" target="_blank">
          Khan Academy: Basic Algebra
        </a>
      </div>
    `;
  }
};
```

### Progress-Based Feature Unlocking

```javascript
// Unlock features based on progress
function updateFeatureAvailability() {
  const completedLessons = state.progress.completed.filter(Boolean).length;

  // Unlock advanced graphing after 10 lessons
  if (completedLessons >= 10) {
    document.getElementById('advanced-graph-features').hidden = false;
    exprInput.placeholder = 'Enter f(x), e.g., sin(x) + x^2 - 3 or use advanced functions';
  }

  // Unlock sacred geometry after 20 lessons
  if (completedLessons >= 20) {
    document.getElementById('sacred-geometry-tools').hidden = false;
  }

  // Enable custom flashcards after 5 lessons
  if (completedLessons >= 5) {
    document.getElementById('custom-cards-section').hidden = false;
  }
}

// Call on state changes
subscribe(updateFeatureAvailability);
```

## Flashcard System Integration

### Custom Study Sessions

```javascript
// Create topic-specific study sessions
function startTopicStudy(topicTag) {
  const topicCards = state.flashcards.deck.filter(card =>
    card.tags.includes(topicTag)
  );

  // Temporarily override due queue
  const originalDueQueue = dueQueue;
  dueQueue = topicCards.filter(card => {
    const stats = getCardStats(card.id);
    return stats.due <= Date.now() || stats.reps < 3; // Include new cards
  });

  updateFlashcardUI();

  // Restore original queue when done
  function restoreQueue() {
    dueQueue = originalDueQueue;
    updateFlashcardUI();
    document.removeEventListener('customStudyEnd', restoreQueue);
  }

  document.addEventListener('customStudyEnd', restoreQueue);
}

// Usage
startTopicStudy('stage1'); // Study only Stage 1 cards
```

### Performance Analytics

```javascript
// Analyze flashcard performance
function generateStudyReport() {
  const stats = Object.values(state.flashcards.stats);
  const cards = state.flashcards.deck;

  return {
    totalCards: cards.length,
    averageEase: stats.reduce((sum, s) => sum + s.ease, 0) / stats.length,
    averageInterval: stats.reduce((sum, s) => sum + s.intervalDays, 0) / stats.length,
    totalReviews: stats.reduce((sum, s) => sum + s.reps, 0),
    cardsDueToday: stats.filter(s => s.due <= Date.now()).length,
    cardsDueTomorrow: stats.filter(s =>
      s.due > Date.now() && s.due <= Date.now() + 24 * 60 * 60 * 1000
    ).length,
    strugglingCards: stats.filter(s => s.ease < 1.8 && s.lapse > 0).length
  };
}

// Display report
const report = generateStudyReport();
console.log('Study Report:', report);
```

### Bulk Card Operations

```javascript
// Add multiple cards at once
function addCardBulk(cardData) {
  cardData.forEach(card => {
    state.flashcards.deck.push({
      id: uid(),
      front: card.front,
      back: card.back,
      tags: card.tags || []
    });
  });
  saveState();
  scheduleCards();
}

// Usage
addCardBulk([
  { front: 'What is 2+2?', back: '4', tags: ['basic', 'addition'] },
  { front: 'Pythagorean theorem?', back: 'a² + b² = c²', tags: ['geometry'] }
]);
```

## Geometry System Integration

### Custom Shape Tools

```javascript
// Add measurement tools
function addMeasurementTool() {
  const tool = document.createElement('div');
  tool.className = 'canvas-toolbar';
  tool.innerHTML = `
    <button id="measureDistance">Measure Distance</button>
    <button id="measureAngle">Measure Angle</button>
    <span id="measurementResult"></span>
  `;

  geomWrap.querySelector('.canvas-toolbar').appendChild(tool);

  // Distance measurement
  let measuringDistance = false;
  let measureStart = null;

  document.getElementById('measureDistance').addEventListener('click', () => {
    measuringDistance = !measuringDistance;
    if (measuringDistance) {
      geomCanvas.style.cursor = 'crosshair';
    } else {
      geomCanvas.style.cursor = 'default';
    }
  });

  geomCanvas.addEventListener('click', (e) => {
    if (!measuringDistance) return;

    const rect = geomCanvas.getBoundingClientRect();
    const worldPoint = screenToWorld({
      x: e.clientX - rect.left,
      y: e.clientY - rect.top
    });

    if (!measureStart) {
      measureStart = worldPoint;
    } else {
      const distance = Math.hypot(
        worldPoint.x - measureStart.x,
        worldPoint.y - measureStart.y
      );
      document.getElementById('measurementResult').textContent =
        `Distance: ${distance.toFixed(2)} units`;
      measureStart = null;
      measuringDistance = false;
      geomCanvas.style.cursor = 'default';
    }
  });
}
```

### Animation Integration

```javascript
// Add animated demonstrations
function demonstrateConcept(concept) {
  switch (concept) {
    case 'trig':
      animateTrigDemo();
      break;
    case 'quadratic':
      animateQuadraticDemo();
      break;
  }
}

function animateTrigDemo() {
  let angle = 0;
  const interval = setInterval(() => {
    angle += 0.1;

    // Update triangle vertices to show angle
    const radius = 80;
    geomState.vertices = [
      { x: 0, y: 0 },
      { x: radius * Math.cos(angle), y: radius * Math.sin(angle) },
      { x: radius * Math.cos(angle + Math.PI/3), y: radius * Math.sin(angle + Math.PI/3) }
    ];

    drawGeometry();

    if (angle >= 2 * Math.PI) {
      clearInterval(interval);
    }
  }, 100);
}
```

## Graphing System Integration

### External Data Plotting

```javascript
// Plot data from external source
function plotExternalData(dataPoints) {
  // Switch to graph tab
  switchTab('graph');

  // Clear current expression
  exprInput.value = '';

  // Plot data points as a custom function
  const points = dataPoints.map((y, x) => `(${x},${y})`).join(',');
  exprInput.value = `points(${points})`;

  // Note: Would need to implement points() function in parser
  plotCurrentExpression();
}
```

### Mathematical Analysis Tools

```javascript
// Add calculus tools
function addCalculusTools() {
  const toolbar = document.createElement('div');
  toolbar.innerHTML = `
    <button id="showDerivative">Show Derivative</button>
    <button id="showIntegral">Show Integral</button>
    <button id="findRoots">Find Roots</button>
  `;

  graphWrap.querySelector('.canvas-toolbar').appendChild(toolbar);

  // Derivative visualization
  document.getElementById('showDerivative').addEventListener('click', () => {
    if (!graphState.compiled) return;

    const h = 1e-4;
    const derivative = (x) => {
      return (safeEval(graphState.compiled, x + h) - safeEval(graphState.compiled, x - h)) / (2 * h);
    };

    // Plot derivative (simplified implementation)
    plotFunction(derivative, '#ef4444', 'derivative');
  });
}
```

### Interactive Graphing Widgets

```javascript
// Add draggable points for function construction
function addInteractivePoints() {
  const points = [
    { x: -2, y: 1 },
    { x: 0, y: -1 },
    { x: 2, y: 1 }
  ];

  points.forEach((point, index) => {
    const handle = document.createElement('div');
    handle.className = 'graph-handle';
    handle.style.left = `${50 + (point.x * graphState.zoom)}px`;
    handle.style.top = `${50 - (point.y * graphState.zoom)}px`;

    // Make draggable (implementation would need drag logic)
    graphWrap.appendChild(handle);
  });
}
```

## Teach-Back System Integration

### External Assessment Integration

```javascript
// Connect to external learning management system
function submitReflectionToLMS(reflection) {
  const lmsData = {
    studentId: getCurrentUserId(),
    concept: reflection.concept,
    explanation: reflection.text,
    selfScore: reflection.selfScore,
    timestamp: reflection.timestamp,
    wordCount: reflection.text.split(' ').length
  };

  // Send to external API
  fetch('/api/lms/submit-reflection', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(lmsData)
  }).then(response => {
    if (response.ok) {
      showNotification('Reflection submitted to LMS');
    }
  });
}

// Override save handler
const originalSaveHandler = saveTeachBtn.onclick;
saveTeachBtn.addEventListener('click', () => {
  originalSaveHandler();
  const lastReflection = state.teachBack.reflections[state.teachBack.reflections.length - 1];
  if (lastReflection) {
    submitReflectionToLMS(lastReflection);
  }
});
```

### Peer Review Integration

```javascript
// Add peer review functionality
function addPeerReview() {
  const reviewBtn = document.createElement('button');
  reviewBtn.textContent = 'Submit for Peer Review';
  reviewBtn.addEventListener('click', () => {
    const lastReflection = state.teachBack.reflections[state.teachBack.reflections.length - 1];
    if (lastReflection) {
      // Submit for peer review
      fetch('/api/peer-review/submit', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(lastReflection)
      });
    }
  });

  document.getElementById('teachResponse').parentNode.appendChild(reviewBtn);
}
```

## Complete Integration Example

### Educational Platform Integration

```javascript
// Integrate with educational platform
class MathJourneyIntegration {
  constructor(platform) {
    this.platform = platform;
    this.initializeEventListeners();
    this.setupDataSync();
  }

  initializeEventListeners() {
    // Listen for lesson completions
    subscribe((state) => {
      const completedLessons = state.progress.completed;
      completedLessons.forEach((completed, index) => {
        if (completed && !this.platform.isLessonCompleted(index)) {
          this.platform.markLessonComplete(index);
          this.unlockNextContent(index);
        }
      });
    });

    // Listen for flashcard progress
    const originalRateCard = rateCurrentCard;
    rateCurrentCard = (grade) => {
      originalRateCard(grade);
      this.platform.updateCardStats(dueQueue[currentCardIndex], grade);
    };
  }

  setupDataSync() {
    // Sync progress with platform
    setInterval(() => {
      if (this.hasUnsyncedData()) {
        this.syncToPlatform();
      }
    }, 30000); // Sync every 30 seconds
  }

  unlockNextContent(completedLessonIndex) {
    const nextLessons = this.platform.getLockedLessons();
    nextLessons.forEach(lesson => {
      if (lesson.prerequisites.includes(completedLessonIndex)) {
        this.platform.unlockLesson(lesson.id);
      }
    });
  }

  hasUnsyncedData() {
    return this.platform.lastSync < state.lastModified;
  }

  syncToPlatform() {
    const dataToSync = {
      progress: state.progress,
      flashcards: state.flashcards,
      reflections: state.teachBack.reflections,
      timestamp: Date.now()
    };

    this.platform.syncData(dataToSync);
  }
}

// Usage
const integration = new MathJourneyIntegration(educationalPlatform);
```

### Custom Dashboard Integration

```javascript
// Create custom learning dashboard
function createLearningDashboard() {
  const dashboard = document.createElement('div');
  dashboard.className = 'learning-dashboard';
  dashboard.innerHTML = `
    <div class="dashboard-section">
      <h3>Progress Overview</h3>
      <div id="dashboard-progress"></div>
    </div>
    <div class="dashboard-section">
      <h3>Study Streak</h3>
      <div id="dashboard-streak"></div>
    </div>
    <div class="dashboard-section">
      <h3>Recent Reflections</h3>
      <div id="dashboard-reflections"></div>
    </div>
  `;

  document.body.appendChild(dashboard);

  // Update dashboard
  subscribe((state) => {
    updateDashboardProgress(state);
    updateDashboardStreak(state);
    updateDashboardReflections(state);
  });
}

function updateDashboardProgress(state) {
  const completed = state.progress.completed.filter(Boolean).length;
  const percentage = (completed / 30) * 100;

  document.getElementById('dashboard-progress').innerHTML = `
    <div class="progress-ring" style="--percentage: ${percentage}%">
      <span>${Math.round(percentage)}%</span>
    </div>
  `;
}
```

## Error Handling and Debugging

### Debug Mode

```javascript
// Enable debug logging
const DEBUG = true;

function debugLog(message, data) {
  if (DEBUG) {
    console.log(`[MathJourney] ${message}`, data);
  }
}

// Enhanced state functions
const originalSaveState = saveState;
saveState = function() {
  debugLog('Saving state', state);
  originalSaveState();
};

const originalLoadState = loadState;
loadState = function() {
  const loadedState = originalLoadState();
  debugLog('Loaded state', loadedState);
  return loadedState;
};
```

### Error Boundaries

```javascript
// Wrap critical functions with error handling
function safeExecute(fn, fallback) {
  try {
    return fn();
  } catch (error) {
    console.error('Error in MathJourney:', error);
    if (fallback) return fallback();
    return null;
  }
}

// Usage
const safePlot = () => safeExecute(plotCurrentExpression, () => {
  alert('Error plotting expression. Please check your input.');
});
```

## Performance Optimization

### Lazy Loading

```javascript
// Load heavy features only when needed
function lazyLoadFeature(featureName) {
  switch (featureName) {
    case 'advanced-graphing':
      if (!window.AdvancedGraphing) {
        import('./advanced-graphing.js').then(module => {
          window.AdvancedGraphing = module;
          module.init();
        });
      }
      break;
  }
}

// Load when tab is activated
const originalSwitchTab = switchTab;
switchTab = function(tabId) {
  originalSwitchTab(tabId);

  if (tabId === 'graph') {
    lazyLoadFeature('advanced-graphing');
  }
};
```

### Memory Management

```javascript
// Clean up resources
function cleanup() {
  // Clear any running animations
  if (window.currentAnimation) {
    cancelAnimationFrame(window.currentAnimation);
  }

  // Clear event listeners
  document.removeEventListener('custom-events');

  // Reset large data structures if needed
  if (memoryUsage() > MAX_MEMORY) {
    compactData();
  }
}
```

This integration guide demonstrates how to extend and customize the 30-Day Math Journey application for various educational platforms and use cases.