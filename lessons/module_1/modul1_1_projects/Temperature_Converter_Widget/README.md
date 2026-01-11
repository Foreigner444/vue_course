# Guided Project: Temperature Converter Widget

## Module 1: Core Fundamentals (Lessons 1-5)

### What You'll Build
A weather app widget that converts between Celsius and Fahrenheit in real-time as the user types. This project showcases `reactive()` for object state, two-way computed properties, and watchers for input validation.

### Real-World Scenario
Weather apps and scientific tools need to display temperatures in different units. Users in the US prefer Fahrenheit, while most of the world uses Celsius. Real-time conversion provides instant feedback without requiring a "Convert" button.

### Patterns You'll Practice
1. `reactive()` for managing related state as an object
2. `ref()` for simple UI state
3. `computed()` with getters AND setters (writable computed)
4. `watch()` for input validation and range warnings
5. Two-way binding with `v-model` on computed properties
6. Conditional styling and visual feedback

### Component Structure
```
src/
├── components/
│   └── TemperatureConverter.vue   # Main converter component
├── App.vue                        # Root component
├── main.ts                        # Application entry
└── style.css                      # Tailwind imports
```

### Key Concepts Demonstrated

| Concept | Implementation |
|:--------|:---------------|
| `reactive()` | `temperatures` object with celsius/fahrenheit |
| `ref()` | `activeInput`, `showFreezeWarning` |
| Writable `computed()` | `celsiusModel`, `fahrenheitModel` with get/set |
| `watch()` | Validates temperature ranges, triggers warnings |

### How to Run
```bash
cd temperature-converter
npm install
npm run dev
```

### Self-Assessment Checklist
- [ ] Component uses `<script setup>` with TypeScript
- [ ] `reactive()` correctly groups related state
- [ ] Writable `computed()` enables two-way binding
- [ ] Editing one input updates the other in real-time
- [ ] `watch()` validates and warns on extreme values
- [ ] UI reflects current state with appropriate styling
