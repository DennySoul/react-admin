# CLAUDE.md

This file provides guidance for Claude Code instances working in the react-admin codebase.

## Project Overview

React Admin is a frontend framework for building admin applications in React. This is a monorepo managed with Lerna, containing core packages, UI components, data providers, examples, and documentation.

## Development Commands

### Setup
```bash
make install    # or yarn install
```

### Building
```bash
make build      # Build all packages
yarn build      # Build all packages
yarn watch      # Build and watch for changes
```

### Running Examples
```bash
make run-simple    # Run simple example
make run-demo      # Run demo application
make run-crm       # Run CRM example
```

### Testing
```bash
make test              # Run all tests
make test-unit         # Run unit tests
make test-e2e          # Run E2E tests
make test-unit-watch   # Run unit tests in watch mode
yarn test-unit <path>  # Run specific unit test
```

### Linting & Formatting
```bash
make lint      # Run ESLint
make prettier  # Format code with Prettier
```

### Storybook
```bash
yarn storybook    # or make storybook
```

### Documentation
```bash
make doc    # Build documentation
```

## Monorepo Structure

- **packages/** - Core library packages
  - `ra-core` - Core logic without UI dependencies
  - `ra-ui-materialui` - Material UI component implementations
  - `react-admin` - Main package combining core + UI
  - `ra-data-*` - Data provider adapters for various backends
  - `ra-language-*` - Internationalization packages
- **examples/** - Demo applications (simple, demo, crm, tutorial)
- **cypress/** - E2E test suite

## Architecture Overview

### Provider-Based Layered Architecture

React Admin uses a provider-based architecture where the `<Admin>` component establishes a context hierarchy:

**Admin → AdminContext → Provider hierarchy → AdminUI**

### Core Providers

The framework is built around these core providers:
- **Auth** - Authentication and authorization
- **Data** - API communication abstraction
- **i18n** - Internationalization
- **Router** - Routing abstraction
- **Store** - Client-side storage
- **Notification** - User notifications

### State Management

- **Server State**: React Query (TanStack Query)
- **App State**: React Context API

### Key Modules in ra-core

- `/core` - Main admin context, routing, and resource registry
- `/auth` - Authentication hooks and components
- `/dataProvider` - API abstraction with Proxy pattern for CRUD operations
- `/routing` - Router abstraction layer (supports react-router, next.js)
- `/form` - Form handling built on react-hook-form
- `/store` - Client-side storage hooks
- `/controller` - Page-level logic controllers (list, edit, create, show)

### Hook-Based API

Most functionality is exposed through custom hooks following the pattern:
- Controllers: `useListController`, `useEditController`, etc.
- Data: `useGetList`, `useGetOne`, `useUpdate`, etc.
- Auth: `usePermissions`, `useAuthenticated`, etc.

### Adapter Pattern

Swappable implementations for:
- Routers (react-router, Next.js)
- Data providers (REST, GraphQL, custom backends)
- i18n providers (polyglot, i18next)

## Code Conventions & Important Rules

### CRITICAL: Import Rules for Bundle Size

**MUI Material Imports**
```typescript
// ✓ CORRECT
import { makeStyles } from '@mui/material/styles';
import { createTheme } from '@mui/material/styles';

// ✗ WRONG - increases bundle size
import { makeStyles, createTheme } from '@mui/material';
```

**MUI Icons Imports**
```typescript
// ✓ CORRECT - use default imports
import DeleteIcon from '@mui/icons-material/Delete';
import EditIcon from '@mui/icons-material/Edit';

// ✗ WRONG - significantly increases bundle size
import { Delete, Edit } from '@mui/icons-material';
```

**Lodash Imports**
```typescript
// ✓ CORRECT - use default imports with .js extension
import get from 'lodash/get.js';
import set from 'lodash/set.js';

// ✗ WRONG - imports entire lodash library
import { get, set } from 'lodash';
```

### Code Style (Prettier)

- **Indentation**: 4 spaces
- **Line Width**: 80 characters
- **Arrow Parens**: Avoid when possible (`x => x` not `(x) => x`)
- **Quotes**: Single quotes for JS, double quotes for JSX
- **Semicolons**: Required
- **Trailing Commas**: ES5 style

### TypeScript

- Target: ES2020
- Generates declaration files (`.d.ts`)
- Unused variables prefixed with `_` are allowed

## Testing Guidelines

- **Unit tests required** for new features or modifications
- **Storybook stories** encouraged when applicable
- Test pattern: `**/*.spec.*`
- Stories pattern: `**/*.stories.*`
- E2E tests in `cypress/` directory

## Pull Request Workflow

### Branch Targeting

- **Target `master`**: Bug fixes, hotfixes
- **Target `next`**: New features, breaking changes

### PR Prefixes

- `[WIP]` - Work in Progress
- `[RFR]` - Ready for Review
- `[RFC]` - Request for Comments

### Required in PR

- Problem description and context
- Solution explanation
- Testing instructions
- Documentation updates (if applicable)
- Unit tests
- Examples or Storybook stories (when relevant)

### Keep PRs Small and Focused

One feature or fix per PR for easier review and maintenance.

## Key Entry Points for Navigation

- `packages/react-admin/src/Admin.tsx` - Main `<Admin>` component entry point
- `packages/react-admin/src/index.ts` - Main package exports
- `packages/ra-core/src/core/CoreAdminContext.tsx` - Provider setup and context composition
- `packages/ra-core/src/core/CoreAdminRoutes.tsx` - Routing structure and resource rendering
- `packages/ra-core/src/dataProvider/` - Data provider abstraction and hooks
- `packages/ra-ui-materialui/src/` - Material UI component implementations
