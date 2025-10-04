# API Reference

This document describes all public APIs, functions, and interactive components exposed by the single-file application in `index.html`. It is organized by feature area with example usage.

- Audience: Developers embedding or extending this app
- Runtime: Browser (no external dependencies)
- Global Scope: All functions and state live in the page scope

## Conventions
- Functions are listed with purpose, params, returns, and notes.
- UI components list relevant elements and events.
- Example snippets assume `index.html` context and browser console access.

---

## App Lifecycle

- `init(): void`  
  Bootstraps the application on `window.load`. It applies contrast settings, renders progress and current lesson, schedules flashcards, and sizes canvases.

- Boot hooks (invoked by `init()`):
  - `applyContrastSetting()`
  - `updateProgressUI()`
  - `renderCurrentLesson()`
  - `scheduleCards()`
  - `resizeGeom()` and `resizeGraph()`
  - `populateTeachConcepts()` and `updateTeachPrompt()`

Example:
```js
// Manually re-run initialization after major state import
init();
```

---

## Application State

### Constants
- `APP_KEY: string`
  - LocalStorage key name for persisted state.

### State Shape
- `state: object`
  - Persisted app state. See `createInitialState()` for full structure.

```js
state = {
  progress: { completed: boolean[30] },
  currentLessonIndex: number,
  flashcards: { deck: Flashcard[], stats: Record<string, CardStats> },
  settings: { highContrast: boolean },
  teachBack: { reflections: Array<{concept:string, prompt:string, text:string, timestamp:number, selfScore?:number}> },
}
```

### Creation & Persistence
- `createInitialState(): State`
- `loadState(): State`
- `saveState(): void`

Example:
```js
// Reset app completely
state = createInitialState();
saveState();
```

### Utilities
- `clamp(v:number, min:number, max:number): number`
- `lerp(a:number, b:number, t:number): number`
- `formatPercent(n:number): string`  // e.g., 0.42 -> "42%"
- `uid(): string`  // random id

### Pub/Sub
- `subscribe(listener: (state)=>void): void`
- `notify(): void`  // manually trigger update to subscribers

---

## Progress Module
Manages lesson stages, current lesson, and progress UI.

### Data
- `stages: Array<{ id:number, title:string, range:[number,number], color:string }>`
- `lessons: Array<{ day:number, title:string, objective:string, activity:string }>`

### DOM Refs
- `progressFillEl, progressTextEl, milestoneTextEl`
- `stagesContainerEl, lessonGridEl, currentLessonDetailEl`

### Functions
- `updateProgressUI(): void`  
  Recomputes progress stats and renders stages/lessons.

- `milestoneMessage(pct:number): string`  
  Returns a celebratory string based on completion percentage.

- `renderStages(): void`  
  Builds stage cards and fill bars.

- `renderLessons(): void`  
  Renders lesson cards with status pills.

- `getStageIdForDay(day:number): number`

- `getCurrentLessonIndex(): number`

- `selectLesson(index:number): void`  
  Updates `state.currentLessonIndex`, persists, rerenders.

- `renderCurrentLesson(): void`  
  Populates current lesson detail and wires buttons.

- `suggestExpressionForLesson(day:number): string`

- `celebrateIfMilestone(): void`  
  Launches confetti at milestone counts.

Example:
```js
// Mark current lesson complete and refresh UI
const i = getCurrentLessonIndex();
state.progress.completed[i] = true;
saveState();
updateProgressUI();
```

---

## Celebration (Confetti)
Canvas confetti shown at milestones.

### Elements
- `confettiCanvas: HTMLCanvasElement`

### Functions
- `resizeConfetti(): void`
- `launchConfetti(): void`
- `confettiLoop(): void`  // animation frame loop

Example:
```js
launchConfetti();
```

---

## Flashcards Module (Spaced Repetition)
SM-2–inspired scheduler, deck management, and study UI.

### Types
- `Flashcard: { id:string, front:string, back:string, tags:string[] }`
- `CardStats: { ease:number, intervalDays:number, reps:number, due:number, lapse:number }`

### DOM Refs
- `flashcardEl, flashFrontEl, flashBackEl`
- `flipBtn, shuffleDeckBtn, addCardBtn`
- `againBtn, hardBtn, goodBtn, easyBtn`
- `dueCountEl, cardProgressEl`

### Functions
- `getDefaultDeck(): Flashcard[]`
- `getCardStats(id:string): CardStats`
- `scheduleCards(): void`  
  Builds `dueQueue` from `state.flashcards.deck` and stats.

- `updateFlashcardUI(): void`
- `sanitize(html:string): string`
- `rateCurrentCard(grade: 1|2|3|4): void`  
  Mutates stats, reschedules, and updates UI.

### State
- `currentCardIndex: number`
- `dueQueue: Flashcard[]`

Example:
```js
// Add a custom card and reschedule
state.flashcards.deck.push({ id: uid(), front: 'What is π?', back: '≈ 3.14159', tags:['custom'] });
saveState();
scheduleCards();
```

---

## Geometry Module
Interactive canvas for 2D shapes and isometric cube.

### DOM Refs
- `geomCanvas, geomWrap, shapeSelect, resetGeometryBtn`
- `geomPropsEl, geomNotesEl`

### Drawing & Transforms
- `worldToScreen(p:{x:number,y:number}): {x:number,y:number}`
- `screenToWorld(p:{x:number,y:number}): {x:number,y:number}`
- `drawHandle(p:{x:number,y:number}): void`
- `drawGeometry(): void`
- `drawTriangle(): void`
- `drawRectangle(): void`
- `drawCircle(): void`
- `drawCube(): void`

### Measurements
- `updateGeomProps(): void`  
  Computes numeric properties and concept notes per shape.

### Interaction
- `pickHandleAt(x:number, y:number): { type:string, index?:number }`
- `geomPointerDown(e:MouseEventLike): void`
- `geomPointerMove(e:MouseEventLike): void`
- `geomPointerUp(): void`

### State
- `geomState: { shape:'triangle'|'rectangle'|'circle'|'cube', zoom:number, offset:{x:number,y:number}, dragging:any, vertices:Point[], rect:{x,y,w,h}, circle:{x,y,r}, cube:{center:{x,y}, size:number, rot:number} }`

Example:
```js
// Switch to circle and enlarge radius
geomState.shape = 'circle';
geomState.circle.r = 140;
drawGeometry();
```

---

## Graphing Module
Expression parser, function plotting, intercepts, Newton steps.

### DOM Refs
- `graphCanvas, graphWrap, exprInput, plotBtn, resetViewBtn, solveOverlay`

### Graph State
- `graphState: { expr:string, compiled:any, zoom:number, origin:{x:number,y:number}, dragging:boolean, intercepts:{x:number[], y:number}, stepsHtml:string }`

### Rendering
- `resizeGraph(): void`
- `drawGraph(): void`
- `niceGridStep(scalePxPerUnit:number): number`
- `drawGrid(ctx, w, h, scale, step): void`
- `drawAxisLabels(ctx, w, h, scale, step): void`

### Parsing & Evaluation
- `FUNCTIONS: Record<string,(n:number)=>number>`  // sin, cos, tan, ln, log, sqrt, abs, exp
- `CONSTANTS: { pi:number, e:number }`
- `tokenize(expr:string): string[]`
- `toRPN(tokens:string[]): string[]`
- `compile(expr:string): string[]|null`
- `evalRPN(rpn:string[], x:number): number`
- `safeEval(rpn:string[], x:number): number`

### Analysis
- `findIntercepts(rpn): { x:number[], y:number }`
- `refineRootBisection(rpn, a:number, b:number, iters?:number): number`
- `dedupeSorted(arr:number[]): number[]`
- `trySymbolicSteps(expr:string, rpn): string[]|null`

### Actions
- `plotCurrentExpression(): void`

Example:
```js
// Plot a custom function via console
exprInput.value = 'sin(x) + 0.3x';
plotCurrentExpression();
```

---

## Teach-Back Module
Prompt generation, hints progression, reflections, and self-assessment.

### DOM Refs
- `teachConceptSelect, teachPromptEl, newPromptBtn`
- `teachResponse, saveTeachBtn, teachSaveMsg`
- `hintList, revealNextHintBtn, showFullAnswerBtn`
- `selfMsg`

### Data
- `concepts: Array<{ key:string, name:string, hints:string[], example:string }>`

### Functions
- `populateTeachConcepts(): void`
- `randomPromptForConcept(c): string`
- `updateTeachPrompt(): void`
- `addHint(text:string): void`

### Persistence
Reflections are stored in `state.teachBack.reflections` via `saveTeachBtn` handler.

Example:
```js
// Programmatically save a reflection
state.teachBack.reflections.push({
  concept: 'Custom',
  prompt: 'Explain derivatives',
  text: 'Rate of change...',
  timestamp: Date.now(),
  selfScore: 2,
});
saveState();
```

---

## Tabs & Settings

### Tabs
- `switchTab(id: 'progress'|'flash'|'geometry'|'graph'|'teach'|'settings'): void`
- `getActiveTabId(): 'tab-progress'|'tab-flash'|'tab-geometry'|'tab-graph'|'tab-teach'|'tab-settings'`

### Settings
- `applyContrastSetting(): void`

Example:
```js
// Toggle high contrast
state.settings.highContrast = !state.settings.highContrast;
saveState();
applyContrastSetting();
```

---

## Keyboard Shortcuts
- Global help: `Shift + ?`
- Flashcards: `Enter` (flip), `1-4` (Again/Hard/Good/Easy)
- Graphing: Drag (pan), Wheel or `+`/`-` (zoom), `0` (reset)

---

## Extending the App
- All functions are available in the page scope; call from DevTools.
- Prefer calling documented functions over mutating DOM directly.
- After mutating `state`, call `saveState()` and relevant renderers.

---

## Changelog
- v1: Initial export of APIs from single-file app.
