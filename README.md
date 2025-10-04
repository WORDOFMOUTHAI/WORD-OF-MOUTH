# 30-Day Math Journey

A comprehensive, single-file interactive web application designed for adults returning to mathematics. This application combines rigorous visual learning with encouragement and reflection to rebuild mathematical confidence.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-yellow.svg)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![No Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen.svg)](package.json)

## ✨ Features

### 📚 Structured Learning Path
- **30 Lessons** organized into 4 progressive stages
- **Stage 1 (Days 1-10):** Foundation Reconstruction
- **Stage 2 (Days 11-18):** Intermediate Integration
- **Stage 3 (Days 19-25):** Classical Mathematical Thinking
- **Stage 4 (Days 26-30):** Sacred Geometry Synthesis

### 🎴 Spaced Repetition Flashcards
- **SM-2 Algorithm** for optimal retention
- **Progressive Difficulty** with ease factor adjustment
- **Custom Cards** - Add your own concepts
- **Keyboard Controls** for rapid practice
- **Due Queue** prioritizes cards that need review

### 📐 Interactive Geometry Lab
- **2D Shapes:** Triangle, Rectangle, Circle
- **Isometric 3D:** Rotatable cube visualization
- **Live Measurements:** Real-time area, perimeter, volume calculations
- **Draggable Vertices** for hands-on exploration
- **Pan & Zoom** for detailed inspection

### 📊 Function Graphing
- **Expression Parser** supporting standard math functions
- **Interactive Visualization:** Pan, zoom, and explore
- **Automatic Intercept Detection**
- **Solution Steps:** Linear/quadratic formulas or Newton's method
- **Implicit Multiplication** for natural input (e.g., `2x`, `sin(x)cos(x)`)

### 🎓 Teach-Back Learning
- **Concept Mastery** through explanation
- **Progressive Hints** for guided discovery
- **Self-Assessment** to track understanding
- **Reflection Journal** with persistent storage
- **Random Prompts** for varied practice

### ⚙️ Settings & Accessibility
- **High Contrast Mode** for visual accessibility
- **Keyboard Navigation** throughout application
- **Data Export/Import** for backup and transfer
- **Progress Tracking** with milestone celebrations
- **Responsive Design** for mobile and desktop

## 🚀 Getting Started

### Quick Start

1. **Open the application:**
   - Simply open `index.html` in any modern web browser
   - No server, build process, or installation required

2. **Start your journey:**
   - Progress tab shows your 30-lesson roadmap
   - Click any lesson to view objectives and activities
   - Mark lessons complete as you finish them

3. **Practice with flashcards:**
   - Switch to Flashcards tab
   - Review due cards using spaced repetition
   - Rate your recall: Again, Hard, Good, or Easy

4. **Explore geometry:**
   - Geometry tab offers interactive shape manipulation
   - Drag vertices and watch measurements update live
   - Switch between 2D shapes and 3D isometric cube

5. **Graph functions:**
   - Enter expressions like `sin(x)`, `x^2 - 4x + 3`, `sqrt(x) * cos(x)`
   - Pan by dragging, zoom with mouse wheel
   - View solution steps for linear and quadratic functions

6. **Teach to learn:**
   - Teach-Back tab generates prompts for explanation
   - Type your understanding in your own words
   - Reveal progressive hints if stuck
   - Self-assess your explanation quality

### System Requirements

- **Modern Web Browser:**
  - Chrome 90+
  - Firefox 88+
  - Safari 14+
  - Edge 90+

- **JavaScript:** Enabled (required)

- **Storage:** ~1 MB localStorage for progress persistence

- **Screen:** Optimized for 360px+ width (mobile through desktop)

## 📖 Documentation

- **[API.md](API.md)** - Complete API reference for all functions and components
- **[COMPONENTS.md](COMPONENTS.md)** - UI component architecture and interactions
- **[EXAMPLES.md](EXAMPLES.md)** - Usage examples and integration guides

## 🎯 Usage Examples

### Marking a Lesson Complete

```javascript
// User clicks "Mark Complete" button
state.progress.completed[5] = true;  // Mark lesson 6 as done
saveState();                          // Persist to localStorage
celebrateIfMilestone();              // Launch confetti if milestone
```

### Adding a Custom Flashcard

```javascript
// Via UI: Click "Add Card" button
// Via console:
state.flashcards.deck.push({
  id: uid(),
  front: "What is the quadratic formula?",
  back: "x = [-b ± √(b² - 4ac)] / 2a",
  tags: ['quadratic', 'custom']
});
saveState();
```

### Graphing a Function

```javascript
// User enters: "x^2 - 4x + 3"
// System compiles to RPN
const compiled = compile("x^2 - 4x + 3");

// Finds intercepts
const intercepts = findIntercepts(compiled);
// Returns: {x: [1, 3], y: 3}

// Displays solution steps
// "Detected quadratic: f(x) ≈ 1.0000x² + -4.0000x + 3.0000"
```

### Creating a Custom Shape

```javascript
// Modify geometry state
geomState.vertices = [
  {x: -100, y: 0},
  {x: 0, y: -100},
  {x: 100, y: 0}
];
drawGeometry();  // Renders equilateral-ish triangle
```

## 🎨 Customization

### Theming

The application uses CSS custom properties for easy theming:

```css
:root {
  --bg-gradient-start: #0e1f2f;
  --bg-gradient-end: #1b3a57;
  --surface: #0f2538;
  --text: #e8f1f8;
  --accent: #7dd3fc;
  --good: #34d399;
  --warn: #f59e0b;
  --danger: #ef4444;
}
```

### High Contrast Mode

Toggle via Settings tab or programmatically:

```javascript
state.settings.highContrast = true;
applyContrastSetting();
```

### Adding Lessons

Extend the `lessons` array:

```javascript
lessons.push({
  day: 31,
  title: "Advanced Calculus",
  objective: "Understand derivatives",
  activity: "Compute d/dx of x³ + 2x"
});
```

### Adding Math Functions

Extend the parser's function library:

```javascript
FUNCTIONS.sec = (x) => 1 / Math.cos(x);
FUNCTIONS.csc = (x) => 1 / Math.sin(x);
// Now usable: "sec(x) + csc(x)"
```

## 🔧 Technical Architecture

### State Management
- **Centralized State Object** with pub/sub pattern
- **localStorage Persistence** for progress continuity
- **Automatic Migration** on schema updates
- **Immutable Updates** with save batching

### Canvas Rendering
- **Device Pixel Ratio** scaling for retina displays
- **ResizeObserver** for responsive canvas sizing
- **Transform-based** coordinate systems
- **RequestAnimationFrame** for smooth animations

### Expression Parsing
- **Shunting-Yard Algorithm** for operator precedence
- **RPN Evaluation** for efficient computation
- **Implicit Multiplication** detection
- **Function Composition** support

### Spaced Repetition
- **SM-2 Algorithm** (simplified)
- **Ease Factor** between 1.3 and 3.0
- **Dynamic Intervals** based on performance
- **Due Queue** with priority scheduling

## 📊 Progress Tracking

### Milestones
Confetti celebrations trigger at:
- **Lesson 1:** Welcome to the journey!
- **Lesson 5:** First week milestone
- **Lesson 10:** Stage 1 complete
- **Lesson 15:** Halfway through Stage 2
- **Lesson 20:** Two-thirds complete
- **Lesson 25:** Stage 3 complete
- **Lesson 30:** Journey complete! 🎉

### Statistics Tracked
- **Lessons Completed:** Out of 30 total
- **Flashcard Statistics:** Ease, interval, reps, lapses
- **Teach-Back Reflections:** Concept, prompt, explanation, self-score
- **Current Lesson:** Automatically advances to next incomplete

## ⌨️ Keyboard Shortcuts

### Global
| Key | Action |
|-----|--------|
| <kbd>Shift</kbd> + <kbd>?</kbd> | Show help dialog |
| <kbd>Tab</kbd> | Navigate focusable elements |

### Flashcards
| Key | Action |
|-----|--------|
| <kbd>Enter</kbd> | Flip card |
| <kbd>1</kbd> | Rate as "Again" |
| <kbd>2</kbd> | Rate as "Hard" |
| <kbd>3</kbd> | Rate as "Good" |
| <kbd>4</kbd> | Rate as "Easy" |
| <kbd>S</kbd> | Shuffle deck |

### Graphing
| Key | Action |
|-----|--------|
| <kbd>Enter</kbd> | Plot expression (in input) |
| <kbd>+</kbd> | Zoom in |
| <kbd>-</kbd> | Zoom out |
| <kbd>0</kbd> | Reset view |
| <kbd>←</kbd> <kbd>→</kbd> <kbd>↑</kbd> <kbd>↓</kbd> | Pan view |

### Geometry
| Action | Input |
|--------|-------|
| Pan | Drag canvas |
| Zoom | Mouse wheel |
| Move vertex | Drag vertex handle |

## 🎯 Learning Philosophy

This application embodies several evidence-based learning principles:

### 1. **Spaced Repetition**
Research shows distributed practice significantly improves long-term retention. The flashcard system automatically schedules reviews at optimal intervals.

### 2. **Active Recall**
Rather than passive reading, users actively retrieve information from memory, strengthening neural pathways.

### 3. **Interleaving**
Mixing different types of content (geometry, algebra, graphing) enhances discrimination and transfer of knowledge.

### 4. **Visual Learning**
Interactive geometry and graphing provide multiple representations of mathematical concepts, supporting diverse learning styles.

### 5. **Metacognition**
Teach-back prompts and self-assessment encourage reflection on one's own understanding, a key component of expertise.

### 6. **Immediate Feedback**
Live measurements, instant graph updates, and flashcard ratings provide rapid feedback loops for correction.

### 7. **Progressive Difficulty**
Four stages scaffold learning from foundational to advanced topics, building confidence incrementally.

## 🔒 Privacy & Security

- **No Telemetry:** Zero tracking, analytics, or external requests
- **Local Storage Only:** All data stays in your browser
- **No Server Required:** Runs entirely client-side
- **XSS Protection:** Input sanitization prevents script injection
- **Open Source:** Fully auditable single-file design

## 🐛 Troubleshooting

### Progress Not Saving
1. Check that localStorage is enabled in browser settings
2. Ensure you're not in private/incognito mode
3. Try exporting data and clearing localStorage
4. Re-import exported data

### Canvas Not Rendering
1. Update to a modern browser version
2. Enable hardware acceleration
3. Check browser console for errors
4. Try resizing the window to trigger redraw

### Expression Parse Errors
1. Use `*` for explicit multiplication: `2*x` not `2 x`
2. Check function names: `sin`, `cos`, `tan`, `sqrt`, `ln`, `log`, `exp`, `abs`
3. Use `pi` and `e` for constants
4. Parentheses must be balanced

### Flashcards Not Appearing
1. Check that deck has cards: `state.flashcards.deck.length`
2. Force reschedule: `scheduleCards()`
3. Reset due times: Mark all as "Again" to put in immediate queue
4. Add custom cards if default deck is complete

## 🤝 Contributing

This is a single-file application designed for simplicity and portability. To contribute:

1. **Fork** the repository
2. **Modify** `index.html` with your changes
3. **Test** thoroughly across browsers
4. **Submit** a pull request with detailed description

### Guidelines
- Maintain single-file architecture
- Preserve zero-dependency design
- Ensure accessibility standards
- Add inline comments for complex logic
- Update documentation for API changes

## 📝 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Spaced Repetition:** Inspired by SuperMemo's SM-2 algorithm
- **Expression Parsing:** Based on Dijkstra's Shunting-yard algorithm
- **Sacred Geometry:** Inspired by classical geometric traditions
- **Design:** Modern glassmorphism with accessibility focus

## 📧 Support

For questions, issues, or suggestions:
- **GitHub Issues:** [Open an issue](../../issues)
- **Discussions:** [Join the conversation](../../discussions)

## 🗺️ Roadmap

Future enhancements under consideration:
- [ ] LaTeX rendering for complex equations
- [ ] 3D geometry with WebGL
- [ ] Calculus module with derivative visualization
- [ ] Statistics & probability section
- [ ] Linear algebra with matrix operations
- [ ] Export progress as PDF report
- [ ] Social sharing of achievements
- [ ] Multi-language support

## 📚 Related Resources

- **Khan Academy:** Supplementary video lessons
- **Brilliant.org:** Interactive problem solving
- **Desmos:** Advanced graphing calculator
- **GeoGebra:** Professional geometry software
- **Anki:** General-purpose spaced repetition

---

**Built with ❤️ for math learners everywhere**

*Rigor meets confidence. Welcome back to math.*
