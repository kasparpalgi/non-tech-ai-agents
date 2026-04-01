# About Project

Textbook about Claude Code, Gemini CLI, OpenClaw and other AI agents for non-tech people. Learn create own apps with AI agents. Setup AI agents. Use boilerplates. Also about scraping web.

The whole book is in /README.md

1. Read `README.md` for current state and outline.
2. Edit `README.md` directly — no build step needed.
3. Add images to `images/` if needed, reference them in the text.
4. Add code examples to `code/` if needed.

## About authoring the textbook

- Write for non-technical readers — no jargon without explanation.
- Use plain analogies (e.g. "like a more advanced Notepad").
- Keep sections practical: explain → show → outcome.
- Images go in `images/` and are referenced as `![Alt](images/filename.png)`.
- Code examples go in `code/` and are shown inline as fenced code blocks.
- Each section should end with an `Outcome:` line summarising what the reader gains.
- Tone: friendly, encouraging, direct. Not condescending.

# Sample Vibe Coding Stack Context

IMPORTANT: develop in the main branch and do not commit your changes (I'll review and decide). For any other tasks do not ask for permissions.

## MCP Servers (Pre-configured)

### Verify MCP Configuration
```bash
claude mcp list

# Should show (✓ Connected):
# - playwright
# - sequential-thinking
# - filesystem (optional but recommended)
# - context7 (optional)
```

If any are missing, see setup instructions in README.md.

### MCP Usage
- **Playwright**: Browser testing, console logs, UI snapshots
- **Sequential Thinking**: Complex planning, architecture decisions
- **Filesystem**: Bulk file operations (optional)
- **Context7**: for Svelte 5, Hasura etc. documentation

## Tech Stack

**Frontend**: Svelte 5, SvelteKit, TypeScript, Tailwind, shadcn-svelte, Neodrag  
**Backend**: Hasura GraphQL, PostgreSQL, Auth.js  
**Testing**: Playwright (E2E), Vitest (unit/component)

## Development Workflow

### 1. Plan
- In the existing task file in `todo/`. Leave the existing original requirement at top
- Use Sequential Thinking MCP for complex features
- Read relevant files for context

### 2. Implement
- Add structured console.logs: `console.log('[Component.method]', data)`
- Follow store factory pattern
- Always follow the best practices and act as a top senior programmer. But also, keep everything that is possible simple. Golden rule: simplicity is GENIUS.
- Keep file sizes around 100 lines no more than 200. For better readability also rather smaller functions (I like at most cases functional programming) but not as crazy small like Uncle Bob teaches
- Extract repetitive code patterns into separate, reusable components
- Separate business logic, utilities, and data fetching into dedicated files
- Use optimistic updates for mutations
- Use dynamic imports for components below the fold: `const Component = await import('./Heavy.svelte')`
- Prefer lightweight alternatives to large libraries
- Do not run any commands that will change the formatting in all of the files, such as `prettier --write .`

### 3. Verify
- **Playwright MCP**: Test in browser, capture console logs
- **Hasura Console**: Verify database changes (see below)
- Check terminal output

### 4. Test (MANDATORY)
- Write unit tests for stores
- Write component tests for UI
- Write E2E tests with Playwright
- Run `npm run check` (must pass)
- Run `npm test` (must pass)

### 5. Clean Up Once All Works
- Remove debug console logs
- Use `loggingStore` for production logs
- Update task file with results

## Hasura Database Verification

### Access Hasura Console
```bash
cd hasura
hasura console
# Opens browser at http://localhost:9695
```

### Common Hasura CLI Commands
```bash
# Apply metadata changes
hasura metadata apply

# Apply migrations
hasura migrate apply --all-databases

# Create new migration
hasura migrate create "migration_name" --from-server

# Reload metadata
hasura metadata reload

# Check status
hasura migrate status
```

### Verify Database Changes
```graphql
# In Hasura Console API Explorer
query VerifyData {
  todos(limit: 5, order_by: { created_at: desc }) {
    id
    title
    list_id
    order
  }
  boards { id name alias }
}
```

---

## Store Pattern (CRITICAL)

```typescript
import { browser } from '$app/environment';

function createStore() {
  const state = $state({
    items: [],
    loading: false,
    error: null
  });

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
    loadItems
  };
}

export const store = createStore();
```

**Rules**:
- Single `$state` object
- Browser guard: `if (!browser) return;`
- Loading in `finally` block
- Getters prevent external mutation
- Return `{ success, message, data? }`

---

## Optimistic Updates

```typescript
async function updateItem(id, updates) {
  const idx = state.items.findIndex(i => i.id === id);
  if (idx === -1) return { success: false, message: 'Not found' };
  
  const original = { ...state.items[idx] };
  state.items[idx] = { ...original, ...updates }; // Optimistic
  
  try {
    const data = await request(UPDATE_ITEM, { id, updates });
    const updated = data.update_items?.returning?.[0];
    if (!updated) throw new Error('Update failed');
    
    state.items[idx] = updated; // Server data
    return { success: true, message: 'Updated', data: updated };
  } catch (error) {
    state.items[idx] = original; // Rollback
    return { success: false, message: error.message };
  }
}
```

---

## GraphQL Workflow

1. Add query/mutation to `src/lib/graphql/documents.ts`
2. Run `npm run generate`
3. Import types from `src/lib/graphql/generated.ts`
4. Use `request()` from `src/lib/graphql/client.ts`
5. Verify in Hasura Console

```typescript
import { request } from '$lib/graphql/client';
import { GET_TABLENAME } from '$lib/graphql/documents';
import type { GetBoardsQuery } from '$lib/graphql/generated';

const data: GetBoardsQuery = await request(GET_BOARDS, { user_id });
```

---

## Logging

```typescript
import { loggingStore } from '$lib/stores/logging.svelte';

// Production logs (persisted to DB)
loggingStore.error('Component', 'Error msg', { error });
loggingStore.warn('Component', 'Warning', { context });
loggingStore.info('Component', 'Info', { data });

// Dev only (not persisted)
loggingStore.debug('Component', 'Debug', { data });
```

**Auto-redacts**: passwords, tokens, API keys  
**View logs**: `/[lang]/logs`

---

## User Feedback

```typescript
import { displayMessage } from '$lib/stores/errorSuccess.svelte';

displayMessage('Error occurred'); // Error, 7s
displayMessage('Success!', 1500, true); // Success, 1.5s
```

---

## Directory Structure

```
src/
├── routes/[lang]/      # Language-based routing
├── lib/
│   ├── components/
│   │   ├── subcomponent1/  # Todo components
│   │   ├── subcomponent2/
│   │   └── ui/             # Shared UI (shadcn)
│   ├── stores/         # State (factory pattern)
│   ├── graphql/
│   │   ├── client.ts
│   │   ├── documents.ts  # ALL queries/mutations
│   │   └── generated.ts  # Auto-generated types
│   └── locales/        # i18n translations
hasura/
├── metadata/           # GraphQL schema, permissions
├── migrations/         # DB migrations
└── seeds/              # Test data
tests/
├── e2e/                # Playwright E2E
└── unit/               # Unit tests
todo/                   # Task docs (ALWAYS update)
```

---

## Common Commands

```bash
# Dev
npm run dev

# Quality
npm run check          # MANDATORY before finalize
npm test               # MANDATORY before finalize
npm run generate       # After GraphQL changes
```

---

## Critical Rules

### Browser Safety
```typescript
if (!browser) return; // ALWAYS check first
if (browser) localStorage.setItem('key', 'value');
```

### Security
- Never store sensitive data in localStorage
- Database is single source of truth
- JWT tokens for API auth
- Validate inputs with Zod

### GraphQL
- All operations in `documents.ts`
- Run `npm run generate` after changes
- Use `request()` exclusively
- Import generated types

### State Management
- Factory pattern for stores
- Single `$state` object
- Expose via getters
- Check `states.svelte.ts` for global state

---

## Task Documentation Template

```markdown
# Task Name

## Original Requirement
[User's request - NEVER REMOVE]

## Analysis
- Affected files: [list]
- MCP needed: [playwright/sequential-thinking/filesystem]

## Implementation Plan
1. [Steps with file refs]
2. [Testing strategy]
3. [Verification approach]

## Changes
- `file.ts`: [description]

## Verification
- [ ] Playwright MCP: Browser tested, console verified
- [ ] Hasura Console: DB changes verified
- [ ] Tests written and passing
- [ ] `npm run check` passed
- [ ] `npm test` passed

## Test Coverage
- Store: X% line coverage
- Component: X% line coverage
- E2E: X workflows covered

## Results
- What works: [list]
- Known issues: [list]
```

## Multi-Terminal Development

### Patterns
- **Dev + Tester**: Terminal 1 codes, Terminal 2 writes tests
- **Dev + Reviewer**: Terminal 1 codes, Terminal 2 reviews
- **Parallel Features**: Each terminal on different feature branch

## Key Features

- TODO

## Test Credentials

**Dev only** (works only when .env='testing'):
- Email: `test@test.com`
- Access: TODO 

## Performance Tips

- Optimistic updates for instant feedback
- PostgreSQL functions where possible for better performance
- `$derived` for computed values
- Lazy load components when needed

## Common Patterns

### New Store
1. Create `src/lib/stores/feature.svelte.ts`
2. Follow factory pattern
3. Add browser guards
4. Implement optimistic updates
5. Write tests
6. Verify with Hasura Console

### New GraphQL Operation
1. Add to `documents.ts`
2. Run `npm run generate`
3. Import types
4. Use `request()`
5. Verify in Hasura Console

### New Component
1. Place in `src/lib/components` folder in `todo/`, `listBoard/`, `auth`, `editor`, `github`, `settings` or `ui/` folder wherever appropriate or create a new. Also in `src/lib/components` some common components.
2. Import stores for state and we prefer to do most of the DB operations in stores.
3. Use `$t()` for i18n
4. Tailwind utilities only
5. Write tests
6. Verify with Playwright MCP