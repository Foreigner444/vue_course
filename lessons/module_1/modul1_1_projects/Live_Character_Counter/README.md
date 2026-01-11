# Guided Project: Live_Character_Counter

## Module 1: Core Fundamentals (Lessons 1-5)

### What You'll Build
A social media post composer that shows remaining characters in real-time and provides visual warnings when approaching the character limit. This project emphasizes reactive state and watchers for validation feedback.

### Real-World Scenario
Social media platforms like Twitter/X (280 chars), LinkedIn posts, and SMS messages have character limits. Users need immediate visual feedback as they type to know how much space remains.

### Patterns You'll Practice
1. `ref()` for text input state and limit configuration
2. `computed()` for character count, remaining count, and validation state
3. `watch()` for triggering warnings and auto-save behavior
4. Two-way binding with `v-model`
5. Dynamic class binding with `:class`
6. Conditional rendering with `v-if`

### Component Structure
```
src/
├── components/
│   └── CharacterCounter.vue   # Main composer component
├── App.vue                    # Root component
├── main.ts                    # Application entry
└── style.css                  # Tailwind imports
```

### Key Concepts Demonstrated

| Concept | Implementation |
|:--------|:---------------|
| `ref()` | `message`, `maxLength`, `showWarning` |
| `computed()` | `characterCount`, `remainingChars`, `isOverLimit`, `progressColor` |
| `watch()` | Triggers warning animation, logs drafts |
| `v-model` | Binds textarea to reactive `message` ref |

### How to Run
```bash
cd character-counter
npm install
npm run dev
```

### Self-Assessment Checklist
- [ ] Component uses `<script setup>` with TypeScript
- [ ] `v-model` correctly binds to `ref()`
- [ ] `computed()` values update as user types
- [ ] `watch()` triggers at appropriate thresholds
- [ ] Visual feedback (colors, warnings) updates reactively
- [ ] Edge cases handled (0 chars, over limit)
