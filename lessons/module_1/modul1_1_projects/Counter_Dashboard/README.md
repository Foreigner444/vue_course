# Guided Project: Counter Dashboard

## Module 1: Core Fundamentals (Lessons 1-5)

### What You'll Build
A fitness app dashboard that displays daily step count with increment/decrement controls and calculates progress percentage toward a daily goal. This project synthesizes all reactive primitives from Module 1.

### Real-World Scenario
Fitness apps like Fitbit, Apple Health, and Google Fit display step counts with visual progress indicators. Users can manually adjust counts (for corrections) and see real-time feedback on their daily goals.

### Patterns You'll Practice
1. `ref()` for primitive reactive state (step count, goal)
2. `computed()` for derived state (progress percentage, status message)
3. `watch()` for side effects (celebration when goal reached)
4. Template binding with `{{ }}` and `:class`
5. Event handling with `@click`
6. Conditional styling based on reactive state

### Component Structure
```
src/
├── components/
│   └── StepCounter.vue       # Main counter component
├── App.vue                   # Root component
├── main.ts                   # Application entry
└── style.css                 # Tailwind imports
```

### Key Concepts Demonstrated

| Concept | Implementation |
|:--------|:---------------|
| `ref()` | `stepCount`, `dailyGoal` |
| `computed()` | `progressPercentage`, `statusMessage`, `progressColor` |
| `watch()` | Triggers celebration effect when goal is reached |
| Template Binding | Dynamic text and class bindings |

### How to Run
```bash
cd counter-dashboard
npm install
npm run dev
```

### Self-Assessment Checklist
- [ ] Component uses `<script setup>` with TypeScript
- [ ] `ref()` correctly wraps primitive values
- [ ] `computed()` derives values without side effects
- [ ] `watch()` triggers only when condition is met
- [ ] UI updates reactively on state changes
- [ ] Tailwind/DaisyUI styling is applied correctly
