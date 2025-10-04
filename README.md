# 30-Day Math Journey - Interactive Mathematics Learning Platform

A comprehensive single-file HTML application designed for adults returning to mathematics. This platform blends rigorous learning with an accessible, encouraging interface featuring spaced repetition flashcards, interactive geometry, function graphing, and teach-back exercises.

## 🌟 Features

### 📚 **Structured Learning Path**
- **30 carefully designed lessons** across 4 progressive stages
- **Stage 1**: Foundation Reconstruction (Days 1-10)
- **Stage 2**: Intermediate Integration (Days 11-18)  
- **Stage 3**: Classical Mathematical Thinking (Days 19-25)
- **Stage 4**: Sacred Geometry Synthesis (Days 26-30)

### 🧠 **Spaced Repetition Flashcards**
- **SM-2 inspired algorithm** for optimal review timing
- **Progressive difficulty adjustment** based on performance
- **Custom card creation** for personalized learning
- **Keyboard shortcuts** for efficient review sessions

### 📐 **Interactive Geometry Lab**
- **Draggable shapes** with live measurement updates
- **2D shapes**: Triangle, Rectangle, Circle with real-time calculations
- **3D visualization**: Isometric cube with rotation
- **Educational annotations** with formulas and concepts

### 📊 **Advanced Function Graphing**
- **Mathematical expression parser** supporting functions, operators, and constants
- **Interactive coordinate plane** with pan, zoom, and touch support
- **Automatic intercept detection** and root finding
- **Step-by-step solutions** for linear and quadratic functions
- **Newton's method visualization** for general functions

### 🎓 **Teach-Back System**
- **Self-explanation prompts** to consolidate understanding
- **Progressive hint system** for guided learning
- **Self-assessment tools** for reflection and growth
- **Concept library** covering key mathematical topics

### ♿ **Accessibility & Usability**
- **High contrast mode** for visual accessibility
- **Comprehensive keyboard navigation** with shortcuts
- **Screen reader support** with ARIA labels
- **Responsive design** for desktop and mobile
- **Touch-friendly interfaces** for mobile interaction

## 🚀 Quick Start

### Running the Application

1. **Download** the `index.html` file
2. **Open** it in any modern web browser (Chrome, Firefox, Safari, Edge)
3. **Start learning** - no installation or setup required!

### First Steps

1. **Explore the Progress tab** to see your learning journey
2. **Select a lesson** to view objectives and activities  
3. **Try the Flashcards** for spaced repetition practice
4. **Experiment with Geometry** by dragging shapes and vertices
5. **Graph functions** using the mathematical expression parser
6. **Use Teach-Back** to explain concepts in your own words

## 📖 Usage Instructions

### Progress Tracking

**Navigate Your Journey:**
- View completion status across all 30 lessons
- Track progress through 4 learning stages
- Mark lessons complete to unlock celebrations
- Jump to any lesson for flexible learning

**Visual Indicators:**
- 🟢 **Green**: Completed lessons
- 🟡 **Yellow**: Current lesson
- ⚫ **Gray**: Future lessons

### Flashcard System

**Review Process:**
1. **Read the front** of the flashcard (question/prompt)
2. **Think** about your answer
3. **Flip the card** (click or press Enter)
4. **Rate your performance** using 1-4 keys:
   - **1 (Again)**: Show again in 10 minutes
   - **2 (Hard)**: Reduce ease, shorter interval
   - **3 (Good)**: Normal progression
   - **4 (Easy)**: Extended interval

**Keyboard Shortcuts:**
- `Enter`: Flip card
- `1-4`: Rate card difficulty
- `S`: Shuffle deck

**Custom Cards:**
- Click **"Add Card"** to create personalized flashcards
- Enter question on front, answer on back
- Cards integrate into spaced repetition system

### Geometry Lab

**Shape Manipulation:**
- **Click and drag vertices** to modify shapes
- **Drag shapes** to move them around
- **Scroll wheel** to zoom in/out
- **Reset button** to restore default positions

**Available Shapes:**
- **Triangle**: Drag vertices, see live area and perimeter
- **Rectangle**: Drag corners, view dimensions
- **Circle**: Drag center or radius handle
- **Cube**: Isometric 3D view with interactive center

**Live Measurements:**
- **Formulas displayed** for educational reference
- **Real-time calculations** as you modify shapes
- **Unit labels** for clarity and precision

### Function Graphing

**Expression Entry:**
- Type mathematical expressions in the input field
- Press `Enter` or click **Plot** to graph
- Use standard mathematical notation

**Supported Syntax:**
```
Functions: sin, cos, tan, asin, acos, atan, sqrt, abs, ln, log, exp
Constants: pi, e  
Operators: +, -, *, /, ^ (with proper precedence)
Variable: x
Implicit multiplication: 2x, sin(x)cos(x)
Examples: sin(x), x^2 + 2*x - 1, sqrt(x) * cos(pi*x)
```

**Navigation:**
- **Drag**: Pan the coordinate plane
- **Mouse wheel**: Zoom in/out
- **Touch**: Pan and pinch-to-zoom on mobile
- **Keyboard**: `+`/`-` zoom, `0` reset, arrows pan

**Analysis Features:**
- **Red dots**: X-intercepts (roots)
- **Green dot**: Y-intercept
- **Solution steps**: Automatic detection of linear/quadratic functions
- **Newton's method**: Iterative root finding for general functions

### Teach-Back Learning

**Process:**
1. **Select a concept** from the dropdown menu
2. **Read the teaching prompt** (varies each time)
3. **Write your explanation** as if teaching someone else
4. **Use progressive hints** if you need guidance
5. **Save your reflection** for progress tracking
6. **Rate your explanation** honestly for self-assessment

**Effective Teach-Back Tips:**
- Use simple, clear language
- Include specific examples
- Explain the "why" behind concepts
- Connect to real-world applications
- Identify common mistakes and how to avoid them

### Settings & Data Management

**Accessibility Options:**
- **High Contrast Mode**: Toggle for better visibility
- **Keyboard Navigation**: Full tab-order support
- **Screen Reader**: ARIA labels and semantic markup

**Data Management:**
- **Export Progress**: Download complete state as JSON
- **Import Data**: Restore from previously exported file
- **Reset All**: Clear all progress (with confirmation)

**Keyboard Shortcuts:**
- `Tab`: Navigate between elements
- `Shift + ?`: Show help dialog
- Context-specific shortcuts shown in footer

## 🛠️ Technical Details

### Architecture

**Single-File Design:**
- Complete application in one HTML file
- No external dependencies or internet connection required
- Self-contained CSS and JavaScript
- Portable and easy to distribute

**State Management:**
- Persistent storage using localStorage
- Automatic save on state changes
- Schema versioning for future updates
- Graceful degradation if storage unavailable

**Performance Features:**
- Canvas rendering with requestAnimationFrame
- High DPI display support
- Efficient event handling
- Responsive design patterns

### Browser Compatibility

**Supported Browsers:**
- Chrome 80+
- Firefox 75+
- Safari 13+
- Edge 80+

**Required Features:**
- Canvas 2D API
- ES6 JavaScript features
- CSS Grid and Flexbox
- localStorage (optional, graceful degradation)

### Mobile Support

**Touch Interactions:**
- Pan and zoom gestures on canvases
- Touch-friendly button sizes
- Responsive layout adaptation
- Orientation change handling

## 📚 Documentation

### Comprehensive Documentation Suite

This project includes extensive documentation for developers and educators:

- **[API Documentation](API_DOCUMENTATION.md)**: Complete reference for all public APIs, functions, and state management
- **[Component Documentation](COMPONENT_DOCUMENTATION.md)**: Detailed guide to all UI components, styling, and interactions  
- **[Function Reference](FUNCTION_REFERENCE.md)**: Comprehensive function specifications with parameters, return values, and examples

### Key Documentation Highlights

**For Developers:**
- Function signatures and parameters
- State management patterns
- Event handling architecture
- Canvas rendering pipeline
- Mathematical parsing system

**For Educators:**
- Pedagogical approach and learning theory
- Lesson progression and scaffolding
- Assessment and feedback mechanisms
- Customization and adaptation strategies

**For Learners:**
- Step-by-step usage instructions
- Keyboard shortcuts and accessibility
- Learning strategies and best practices
- Troubleshooting and support

## 🎯 Learning Approach

### Pedagogical Foundation

**Adult Learning Principles:**
- **Respect for experience**: Builds on existing knowledge
- **Practical relevance**: Real-world applications and examples
- **Active engagement**: Interactive rather than passive consumption
- **Self-direction**: Flexible pacing and lesson selection
- **Reflection**: Teach-back system for deeper understanding

**Spaced Repetition Science:**
- Based on cognitive psychology research
- Optimizes long-term retention
- Adapts to individual learning patterns
- Prevents cramming and promotes understanding

**Multi-Modal Learning:**
- **Visual**: Interactive graphs and geometric manipulations
- **Kinesthetic**: Drag-and-drop interfaces
- **Auditory**: Self-explanation through teach-back
- **Analytical**: Step-by-step problem solving

### Learning Strategies

**For Maximum Effectiveness:**

1. **Consistent Daily Practice**: Even 15-20 minutes daily
2. **Active Engagement**: Use all interactive features
3. **Self-Explanation**: Complete teach-back exercises regularly
4. **Spaced Review**: Trust the flashcard algorithm
5. **Visual Exploration**: Manipulate geometric shapes
6. **Function Analysis**: Graph various mathematical expressions
7. **Reflection**: Use self-assessment honestly

## 🤝 Contributing & Customization

### Customization Options

**Easy Modifications:**
- **Color Themes**: Modify CSS custom properties
- **Lesson Content**: Update lesson arrays with new topics
- **Flashcard Decks**: Add custom cards via interface or code
- **Expression Library**: Extend parser with new functions

**Advanced Customization:**
- **New Geometric Shapes**: Extend drawing and interaction system
- **Alternative Algorithms**: Modify spaced repetition parameters
- **Additional Modules**: Add new learning activities
- **Assessment Tools**: Integrate with external systems

### Code Structure

**Modular Design:**
- State management functions
- Progress tracking system  
- Flashcard engine with SM-2 algorithm
- Geometry canvas with interactive elements
- Mathematical expression parser and grapher
- Teach-back and assessment tools
- UI navigation and settings

**Clean Separation:**
- CSS for styling and animations
- JavaScript for logic and interactions
- HTML for semantic structure
- No external dependencies

## 📄 License & Credits

### License

This project is released under the MIT License, allowing for:
- Free use for personal and educational purposes
- Modification and customization
- Distribution and sharing
- Commercial use with attribution

### Educational Philosophy

Designed with the belief that mathematics should be:
- **Accessible**: No barriers to entry or prerequisites
- **Encouraging**: Supportive rather than intimidating
- **Practical**: Connected to real-world applications  
- **Engaging**: Interactive and visually appealing
- **Reflective**: Promoting deep understanding over memorization

### Acknowledgments

Built with inspiration from:
- Spaced repetition research (Hermann Ebbinghaus, Piotr Wozniak)
- Adult learning theory (Malcolm Knowles)
- Mathematical visualization principles
- Accessibility and inclusive design standards

---

## 🚀 Get Started Today!

Ready to rebuild your mathematical confidence? Simply download the `index.html` file and open it in your browser. Your 30-day journey to mathematical fluency begins now!

**Remember**: Mathematics is not about being "naturally gifted" - it's about patient practice, visual understanding, and building connections. This platform provides the structure, encouragement, and tools you need to succeed.

*Welcome back to mathematics. You've got this!* 🎯📊✨
