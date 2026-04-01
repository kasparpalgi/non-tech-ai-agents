# Sample CLAUDE.md — Svelte 5 / Hasura / SvelteKit Stack

This is an example `CLAUDE.md` file for a vibe coding project using the stack taught in this course.
Copy it into the `.claude/` folder of your own project and customise it.

---

# About Project

[Describe your project here in 1–2 sentences.]

IMPORTANT: develop in the main branch and do not commit your changes (I'll review and decide). For any other tasks do not ask for permissions.

## MCP Servers (Pre-configured)

```bash
claude mcp list
# Should show (✓ Connected):
# - playwright
# - sequential-thinking
# - filesystem (optional)
# - context7 (optional)
```

- **Playwright**: Browser testing, console logs, UI snapshots
- **Sequential Thinking**: Complex planning, architecture decisions
- **Context7**: Svelte 5, Hasura etc. documentation

## Tech Stack

**Frontend**: Svelte 5, SvelteKit, TypeScript, Tailwind, shadcn-svelte  
**Backend**: Hasura GraphQL, PostgreSQL, Auth.js  
**Testing**: Playwright (E2E), Vitest (unit/component)

## Development Workflow

### 1. Plan
- Use the task file in `todo/`. Leave the original requirement at top.
- Use Sequential Thinking MCP for complex features.

### 2. Implement
- Follow store factory pattern (see below).
- Golden rule: simplicity is GENIUS.
- Keep files ~100 lines, max 200.
- No global formatters (`prettier --write .` is banned).

### 3. Verify
- Playwright MCP: test in browser, capture console logs.
- Hasura Console: verify DB changes.

### 4. Test (MANDATORY)
- Unit tests for stores, component tests for UI, E2E with Playwright.
- `npm run check` must pass. `npm test` must pass.

### 5. Clean Up
- Remove debug console logs.
- Use `loggingStore` for production logs.
- Update task file with results.

---

## Hasura Console

```bash
cd hasura && hasura console   # http://localhost:9695
hasura metadata apply
hasura migrate apply --all-databases
```

---

## Store Pattern (CRITICAL)

```typescript
import { browser } from '$app/environment';

function createStore() {
  const state = $state({ items: [], loading: false, error: null });

  async function loadItems() {
    if (!browser) return;
    state.loading = true;
    state.error = null;
    try {
      const data = await request(GET_ITEMS, {});
      state.items = data.items || [];
    } catch (error) {
      state.error = error.message;
    } finally {
      state.loading = false;
    }
  }

  const sorted = $derived([...state.items].sort((a, b) => a.order - b.order));

  return {
    get items() { return state.items; },
    get loading() { return state.loading; },
    get error() { return state.error; },
    get sorted() { return sorted; },
    loadItems,
  };
}

export const store = createStore();
```

Rules: single `$state` object · browser guard · loading reset in `finally` · getters prevent external mutation · return `{ success, message, data? }`.

---

## Optimistic Updates

```typescript
async function updateItem(id, updates) {
  const idx = state.items.findIndex(i => i.id === id);
  if (idx === -1) return { success: false, message: 'Not found' };
  const original = { ...state.items[idx] };
  state.items[idx] = { ...original, ...updates }; // optimistic
  try {
    const data = await request(UPDATE_ITEM, { id, updates });
    const updated = data.update_items?.returning?.[0];
    if (!updated) throw new Error('Update failed');
    state.items[idx] = updated;
    return { success: true, message: 'Updated', data: updated };
  } catch (error) {
    state.items[idx] = original; // rollback
    return { success: false, message: error.message };
  }
}
```

---

## GraphQL Workflow

1. Add operation to `src/lib/graphql/documents.ts`
2. `npm run generate`
3. Import types from `src/lib/graphql/generated.ts`
4. Use `request()` from `src/lib/graphql/client.ts`
5. Verify in Hasura Console

---

## Logging

```typescript
import { loggingStore } from '$lib/stores/logging.svelte';
loggingStore.error('Component', 'msg', { error });  // persisted to DB
loggingStore.debug('Component', 'msg', { data });   // dev only
```

---

## User Feedback

```typescript
import { displayMessage } from '$lib/stores/errorSuccess.svelte';
displayMessage('Error occurred');          // error, 7 s
displayMessage('Saved!', 1500, true);      // success, 1.5 s
```

---

## Directory Structure

```
src/
├── routes/[lang]/
├── lib/
│   ├── components/
│   ├── stores/
│   ├── graphql/
│   │   ├── client.ts
│   │   ├── documents.ts   # ALL queries/mutations
│   │   └── generated.ts   # auto-generated types
│   └── locales/
hasura/
├── metadata/
├── migrations/
└── seeds/
tests/
├── e2e/
└── unit/
todo/
```

---

## Critical Rules

- `if (!browser) return;` — always check before DOM/localStorage access.
- Never store sensitive data in localStorage.
- All GraphQL in `documents.ts`; run `npm run generate` after changes.
- Factory pattern for all stores; single `$state` object; expose via getters.

---

## Task Template

```markdown
# Task Name

## Original Requirement
[NEVER REMOVE]

## Analysis
- Affected files:
- MCP needed:

## Implementation Plan
1.

## Changes
- `file.ts`:

## Verification
- [ ] Browser tested (Playwright MCP)
- [ ] DB verified (Hasura Console)
- [ ] Tests passing
- [ ] `npm run check` passed
- [ ] `npm test` passed
```
