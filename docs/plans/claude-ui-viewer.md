# Claude UI Viewer - Implementation Plan

## Problem Statement & Objectives

- Provide a lightweight React UI to browse `.claude/commands` and `.claude/agents` markdown files without modifying existing content
- Surface concise overviews plus detailed views for each command/agent to improve discoverability
- Ensure markdown parsing is reliable, rendering key metadata cleanly
- Keep setup friction minimal so contributors can run the UI locally with standard tooling

## Technical Approach & Architecture Decisions

- **Stack**: Vite + React + TypeScript for fast iteration and built-in JSX/TSX support
- **Routing**: React Router for clear separation between list and detail views
- **Markdown Processing**: Parse with `gray-matter` (frontmatter) and `remark/rehype` pipeline for HTML; sanitize output with `rehype-sanitize`
- **Data Model**: Shared data model for commands/agents; exposed through file-system-backed loader/hook
- **Styling**: CSS Modules or Tailwind (Phase 3 decision) to ensure scoped, maintainable styling

## Implementation Phases

### Phase 1 – Project Scaffold & Data Loading

**Description**: Set up the React project foundation with TypeScript and create the data loading infrastructure.

**Files to Create/Modify**:
- `package.json`
- `tsconfig.json`
- `vite.config.ts`
- `src/main.tsx`
- `src/App.tsx`
- `src/types/index.ts`

**Implementation Steps**:
1. Initialize Vite React-TS template; configure path aliases (e.g., `@/components`)
2. Define shared types (`CatalogItem`, `ItemKind`)
3. Create a loader utility to scan `.claude/commands` and `.claude/agents` using Node `fs/promises`
4. Wire loader into Vite dev server via `vite.config.ts` plugin or separate API route
5. Ensure dev server serves data as JSON; add npm scripts (`dev`, `build`, `preview`)

**Testing Approach**:
- Run `npm run dev` manually
- Add unit test for loader with Jest/Vitest

**Success Criteria**:
- `npm run dev` starts successfully
- Data endpoint returns combined catalog with correct structure

### Phase 2 – Command & Agent Listing UI

**Description**: Build the main list view showing all available commands and agents with basic routing.

**Files to Create/Modify**:
- `src/App.tsx`
- `src/components/CatalogList.tsx`
- `src/components/Layout.tsx`
- `src/hooks/useCatalog.ts`

**Implementation Steps**:
1. Implement `useCatalog` hook fetching JSON and exposing loading/error states
2. Create `Layout` with shared header, nav, and responsive container
3. Build `CatalogList` to render grouped lists (Commands, Agents) with concise metadata
4. Integrate React Router: `/` shows list view; set selected item via route params
5. Add empty-state and skeleton loaders

**Testing Approach**:
- Component snapshot/unit tests with Testing Library
- Verify fetch mocked correctly

**Success Criteria**:
- List view renders both categories
- Selecting an item updates URL and highlights active entry

### Phase 3 – Detail View & Markdown Rendering

**Description**: Implement the detail panel with full markdown rendering capabilities.

**Files to Create/Modify**:
- `src/components/ItemDetail.tsx`
- `src/utils/markdown.ts`
- `src/routes/ItemRoute.tsx`

**Implementation Steps**:
1. Extend data loader to include frontmatter and markdown body
2. Implement `markdown.ts` to convert markdown→sanitized HTML using remark pipeline
3. Build `ItemDetail` showing metadata plus rendered markdown
4. Add route guard for 404s; show "Select an item" message when none selected
5. Ensure keyboard navigation when switching items

**Testing Approach**:
- Rendering tests validating markdown conversion
- Regression test for sanitized HTML

**Success Criteria**:
- Detail pane displays parsed markdown accurately
- Handles headings, code blocks, and tables correctly

### Phase 4 – Styling, Responsiveness & Polish

**Description**: Apply consistent styling and ensure responsive, accessible UI.

**Files to Create/Modify**:
- `src/styles/global.css`
- Component-specific CSS modules or `tailwind.config.js`
- Update all components for styling

**Implementation Steps**:
1. Choose styling approach (CSS Modules vs Tailwind)
2. Create responsive split layout: sidebar list + detail panel
3. Add subtle theming (accent colors, hover states)
4. Improve accessibility (ARIA roles, focus outlines, semantic headings)
5. Update README with setup instructions and screenshots

**Testing Approach**:
- Use responsive dev tools
- Run axe accessibility checks
- Manual keyboard navigation verification

**Success Criteria**:
- UI adapts gracefully to mobile/desktop
- Passes automated accessibility linting
- README documents setup

### Phase 5 – Packaging & QA

**Description**: Final polish, testing setup, and documentation.

**Files to Create/Modify**:
- `package.json`
- `README.md`
- CI configuration (`.github/workflows/ci.yml`)
- Optional `scripts/postinstall`

**Implementation Steps**:
1. Add linting/prettier configs; integrate with `npm run lint`
2. Configure CI to run install, lint, tests, and build
3. Document testing strategy and deployment options
4. Review dependency tree for size/security
5. Prepare final PR with release notes

**Testing Approach**:
- CI pipeline green
- Run `npm run build` locally

**Success Criteria**:
- Automated checks succeed
- Documentation updated
- Project ready for deployment

## Potential Challenges & Solutions

- **Markdown Variability**: Define fallback logic and validate during load; optionally add schema validation with `zod`
- **File Watch in Dev Mode**: Configure `server.watch` to include `.claude` directory; document manual reload if needed
- **Security Risks from Markdown HTML**: Sanitize with strict allowlist via `rehype-sanitize`
- **Large Catalog Performance**: Memoize parsed markdown and leverage suspense/lazy loading
- **Environment Portability**: Normalize paths for cross-platform compatibility

## Code Examples

### Type Definitions
```typescript
// src/types/index.ts
export type ItemKind = 'command' | 'agent';

export interface CatalogItem {
  id: string;
  kind: ItemKind;
  title: string;
  summary: string;
  slug: string;
  path: string;
  metadata: Record<string, unknown>;
  body: string;
}
```

### Markdown Processing
```typescript
// src/utils/markdown.ts
import { unified } from 'unified';
import remarkParse from 'remark-parse';
import remarkGfm from 'remark-gfm';
import remarkRehype from 'remark-rehype';
import rehypeRaw from 'rehype-raw';
import rehypeSanitize from 'rehype-sanitize';
import rehypeStringify from 'rehype-stringify';

export async function renderMarkdown(markdown: string): Promise<string> {
  const file = await unified()
    .use(remarkParse)
    .use(remarkGfm)
    .use(remarkRehype, { allowDangerousHtml: false })
    .use(rehypeRaw)
    .use(rehypeSanitize)
    .use(rehypeStringify)
    .process(markdown);
  return String(file);
}
```

### Catalog List Component
```tsx
// src/components/CatalogList.tsx
import { CatalogItem } from '@/types';
import { NavLink } from 'react-router-dom';

interface Props {
  title: string;
  items: CatalogItem[];
}

export function CatalogList({ title, items }: Props) {
  return (
    <section aria-labelledby={`${title}-heading`}>
      <h2 id={`${title}-heading`}>{title}</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            <NavLink to={`/${item.kind}s/${item.slug}`}>
              <strong>{item.title}</strong>
              <p>{item.summary}</p>
            </NavLink>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

## Next Steps

- Confirm preferred styling system (CSS Modules vs Tailwind)
- Decide whether to deploy (e.g., static hosting) after Phase 5 or keep local-only
- Begin Phase 1 implementation with project scaffolding