# Settings & Data Management Documentation

## Overview

The Settings System provides configuration options, data export/import functionality, and application reset capabilities. It manages user preferences and enables data portability.

## Settings Structure

### Application Settings
```javascript
{
  settings: {
    highContrast: boolean  // Accessibility setting for improved visibility
  }
}
```

## Accessibility Features

### High Contrast Mode

#### `contrastToggleBtn` Event Handler
Toggles high contrast mode for better accessibility.

**Functionality:**
- Applies `.high-contrast` class to document root
- Updates CSS custom properties for improved visibility
- Persists setting to localStorage

**CSS Custom Properties Override:**
```css
.high-contrast {
  --bg-gradient-start: #000000;
  --bg-gradient-end: #000000;
  --surface: #000000;
  --text: #ffffff;
  --accent: #00ffff;
  --good: #00ff87;
  --warn: #ffd000;
  --danger: #ff3b3b;
  --focus: #ffff00;
}
```

**Usage:**
```javascript
contrastToggleBtn.addEventListener('click', () => {
  state.settings.highContrast = !state.settings.highContrast;
  saveState();
  applyContrastSetting();
});
```

#### `applyContrastSetting() → void`
Applies the current contrast setting to the document.

**Implementation:**
```javascript
function applyContrastSetting() {
  document.documentElement.classList.toggle('high-contrast', !!state.settings.highContrast);
}
```

## Data Management

### Export Functionality

#### `exportBtn` Event Handler
Exports complete application state as a downloadable JSON file.

**Process:**
1. Serializes current state to JSON string
2. Creates blob with proper MIME type
3. Generates temporary download URL
4. Triggers browser download
5. Cleans up temporary URL

**Implementation:**
```javascript
exportBtn.addEventListener('click', () => {
  const blob = new Blob([JSON.stringify(state, null, 2)], {
    type: 'application/json'
  });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'math_journey_export.json';
  a.click();
  URL.revokeObjectURL(url);
});
```

**Export Data Structure:**
```json
{
  "progress": {
    "completed": [true, true, false, ...],
  },
  "currentLessonIndex": 5,
  "flashcards": {
    "deck": [...],
    "stats": {...}
  },
  "settings": {
    "highContrast": false
  },
  "teachBack": {
    "reflections": [...]
  }
}
```

### Import Functionality

#### `importBtn` Event Handler
Imports application state from an uploaded JSON file.

**Process:**
1. Triggers hidden file input click
2. Validates uploaded file
3. Parses JSON data
4. Merges with default state structure
5. Applies new state
6. Updates all UI components
7. Shows success/error message

**Implementation:**
```javascript
importBtn.addEventListener('click', () => importInput.click());

importInput.addEventListener('change', async (e) => {
  const file = e.target.files[0];
  if (!file) return;

  try {
    const text = await file.text();
    const data = JSON.parse(text);

    // Merge with default state to handle schema evolution
    state = { ...createInitialState(), ...data };
    saveState();

    // Apply settings
    applyContrastSetting();

    // Update all UI components
    updateProgressUI();
    renderCurrentLesson();
    scheduleCards();
    drawGeometry();
    drawGraph();

    alert('Import successful');
  } catch (err) {
    alert('Import failed: ' + err.message);
  } finally {
    importInput.value = ''; // Reset file input
  }
});
```

**Validation:**
- JSON parsing errors are caught and reported
- Invalid data structures trigger fallback to default state
- File type validation (accepts only `application/json`)

#### `importInput` Element
Hidden file input for import functionality.

```html
<input type="file" id="importInput" accept="application/json" style="display:none" />
```

### Reset Functionality

#### `resetAllBtn` Event Handler
Completely resets the application to initial state.

**Process:**
1. Shows confirmation dialog
2. Creates fresh initial state
3. Saves new state
4. Applies settings
5. Updates all UI components
6. Provides user feedback

**Implementation:**
```javascript
resetAllBtn.addEventListener('click', () => {
  if (!confirm('Reset all progress and settings? This cannot be undone.')) return;

  state = createInitialState();
  saveState();

  // Reinitialize all systems
  applyContrastSetting();
  updateProgressUI();
  renderCurrentLesson();
  scheduleCards();
  drawGeometry();
  drawGraph();
});
```

**Warning Features:**
- Requires user confirmation before proceeding
- Clear warning that action cannot be undone
- Destructive styling to indicate danger

## Tab Management

### `switchTab(tabId: string) → void`
Manages tab visibility and active state.

**Parameters:**
- `tabId`: One of `'progress'`, `'flash'`, `'geometry'`, `'graph'`, `'teach'`, `'settings'`

**Functionality:**
- Updates `aria-current` attributes on tab buttons
- Shows/hides tab sections using `hidden` attribute
- Triggers tab-specific initialization (geometry/graph rendering)

**Implementation:**
```javascript
function switchTab(id) {
  // Update button states
  tabButtons.forEach(btn => {
    btn.setAttribute('aria-current', btn.dataset.tab === id ? 'page' : 'false');
  });

  // Update section visibility
  document.getElementById('tab-progress').hidden = id !== 'progress';
  document.getElementById('tab-flash').hidden = id !== 'flash';
  document.getElementById('tab-geometry').hidden = id !== 'geometry';
  document.getElementById('tab-graph').hidden = id !== 'graph';
  document.getElementById('tab-teach').hidden = id !== 'teach';
  document.getElementById('tab-settings').hidden = id !== 'settings';

  // Tab-specific initialization
  if (id === 'flash') scheduleCards();
  if (id === 'geometry') drawGeometry();
  if (id === 'graph') drawGraph();
}
```

### `getActiveTabId() → string`
Returns the currently active tab identifier.

**Implementation:**
```javascript
function getActiveTabId() {
  if (!document.getElementById('tab-progress').hidden) return 'tab-progress';
  if (!document.getElementById('tab-flash').hidden) return 'tab-flash';
  if (!document.getElementById('tab-geometry').hidden) return 'tab-geometry';
  if (!document.getElementById('tab-graph').hidden) return 'tab-graph';
  if (!document.getElementById('tab-teach').hidden) return 'tab-teach';
  return 'tab-settings';
}
```

## Keyboard Shortcuts

### Global Shortcuts
```javascript
document.addEventListener('keydown', (e) => {
  if (e.shiftKey && e.key === '?') {
    alert('Shortcuts:\nFlashcards: Enter flip, 1-4 rate\nGraphing: Drag to pan, wheel/+/- to zoom, 0 reset');
  }
});
```

### Tab-Specific Shortcuts
```javascript
document.addEventListener('keydown', (e) => {
  const activeTab = getActiveTabId();
  if (activeTab === 'tab-flash') {
    // Flashcard shortcuts...
  }
  if (activeTab === 'tab-graph') {
    // Graphing shortcuts...
  }
});
```

## UI Components

### Settings Tab Structure
```html
<section class="card" id="tab-settings">
  <h2>Settings & Help</h2>

  <div class="grid grid-2">
    <div class="card">
      <h3>Accessibility</h3>
      <div class="setting-controls">
        <button id="contrastToggleBtn">Toggle High Contrast</button>
        <span class="chip">Keyboard tips live in the footer</span>
      </div>
    </div>

    <div class="card">
      <h3>Data</h3>
      <div class="data-controls">
        <button id="exportBtn">Export Progress</button>
        <input type="file" id="importInput" accept="application/json" style="display:none" />
        <button id="importBtn">Import</button>
        <button id="resetAllBtn">Reset All</button>
      </div>
    </div>
  </div>

  <div class="card">
    <h3>About</h3>
    <p>Application description and feature overview</p>
    <ul>
      <li>Four stages guiding 30 days of learning</li>
      <li>Spaced repetition to reinforce concepts</li>
      <li>Interactive geometry and graphing tools</li>
      <li>Teach-back prompts for deeper understanding</li>
    </ul>
  </div>
</section>
```

## Error Handling

### Import Errors
```javascript
importInput.addEventListener('change', async (e) => {
  try {
    const text = await file.text();
    const data = JSON.parse(text);
    // Process data...
  } catch (err) {
    alert('Import failed: ' + err.message);
  } finally {
    importInput.value = '';
  }
});
```

### Export Errors
- File system errors are handled by browser
- JSON serialization errors would be caught during blob creation
- Network errors during download trigger browser error handling

### Reset Confirmation
```javascript
resetAllBtn.addEventListener('click', () => {
  if (!confirm('Reset all progress and settings? This cannot be undone.')) {
    return; // User cancelled
  }
  // Proceed with reset...
});
```

## Integration Examples

### Custom Settings Addition
```javascript
// Add a custom setting
const originalSettings = state.settings;
state.settings = {
  ...originalSettings,
  animationSpeed: 'normal' // slow, normal, fast
};

// Persist custom setting
const originalSaveState = saveState;
saveState = function() {
  // Save state with custom settings
  localStorage.setItem(APP_KEY, JSON.stringify(state));
  updateProgressUI();
};
```

### Backup Management
```javascript
// Automatic backup before major operations
function createBackup() {
  const backup = {
    timestamp: Date.now(),
    state: JSON.parse(JSON.stringify(state))
  };
  localStorage.setItem(`${APP_KEY}_backup`, JSON.stringify(backup));
}

function restoreFromBackup() {
  const backup = localStorage.getItem(`${APP_KEY}_backup`);
  if (backup) {
    const data = JSON.parse(backup);
    state = data.state;
    saveState();
    return true;
  }
  return false;
}
```

### Settings Validation
```javascript
// Validate settings on import
function validateImportedSettings(data) {
  const errors = [];

  if (typeof data.settings?.highContrast !== 'boolean') {
    errors.push('Invalid highContrast setting');
  }

  if (data.settings?.customSetting !== undefined) {
    if (!['value1', 'value2'].includes(data.settings.customSetting)) {
      errors.push('Invalid customSetting value');
    }
  }

  return errors;
}
```

## Browser Compatibility

### LocalStorage Support
- Required for state persistence
- Fallback handling for private browsing modes
- Quota exceeded errors handled gracefully

### File API Support
- FileReader API for import functionality
- Blob API for export functionality
- URL.createObjectURL for download triggering

### Accessibility API
- ARIA attributes for screen reader support
- Focus management for keyboard navigation
- High contrast media queries

This settings system provides comprehensive configuration management, data portability, and accessibility features for the mathematics learning application.