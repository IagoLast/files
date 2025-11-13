# Frontend Development Guidelines

This directory contains comprehensive guidelines for frontend development in TrainerStudio. These guidelines ensure consistency, maintainability, and scalability across our React applications.

## 📁 Available Guidelines

### [Architecture](./architecture.md)

**Recursive Architecture with Clean Code Principles**

- Core architectural layers (Core, Application, Presentation)
- Recursive component structure with strict dependency rules
- Separation of concerns between entities, services, repositories, and views
- Controller pattern for managing component logic
- Folder structure conventions that scale
- Note: HTTP client configuration is documented in [API Client](./api-client.md)

### [React Components](./react.md)

**React Best Practices & Component Patterns**

- Function components over class components
- Separation of presentation and business logic
- Controller hooks for stateful logic
- Query/mutation naming conventions (`useListUsersQuery`, `useCreateUserMutation`)
- Component structure and organization
- Services as collections of pure functions

### [Routing](./routing.md)

**React Router Architecture with Authentication**

- Hierarchical routing with nested routes
- Centralized route constants in `/constants/routes.ts`
- Loader-based authentication and data fetching
- Translation namespace preloading
- Protected routes pattern
- File organization: `*.route.tsx` files

### [Internationalization](./i18n.md)

**i18n with i18next Integration**

- Translation keys as English text (no abstract keys)
- Namespace constants in `/constants/namespaces.ts`
- File naming: `ns-[feature-name].json`
- Route-level namespace loading
- Direct `t` function import from i18next

### [File Naming](./filenames.md)

**Consistent File Naming Conventions**

- Snake case (dash-case) for all filenames
- Descriptive file type suffixes:
  - `component-name.view.tsx` - UI components
  - `component-name.controller.ts` - Business logic
  - `component-name.service.ts` - Pure functions
  - `component-name.repository.ts` - Data access
  - `component-name.spec.tsx` - Tests

### [Testing](./frontend-testing.md)

**Testing Strategy & Best Practices**

- Test real use cases, not implementation details
- Mock only server calls to avoid side effects
- Render the entire app for integration tests
- Descriptive test descriptions
- User-centric assertions

### [API Client](./api-client.md)

**HTTP Client Configuration**

- Axios singleton setup and configuration
- Request/response interceptors
- Automatic authentication token handling
- Separation of concerns: auth service vs HTTP client
- Usage patterns in repositories

### [Forms](./forms.md)

**Form Handling with react-hook-form + zod**

- Form validation with zod schemas
- Controller pattern for form logic
- View components for form rendering
- Error handling and display
- Type-safe form data with TypeScript

### [Tooling](./tooling.md)

**Development Tooling Configuration**

- Prettier setup and configuration
- ESLint integration with Prettier
- Import path rules and restrictions
- EditorConfig for consistency
- VSCode/Cursor settings for format on save

## 🏗️ Architecture Overview

Our frontend follows a **Recursive Architecture** pattern based on Clean Architecture principles:

```
/src
├── client/           # HTTP client configuration
├── components/       # Global reusable components
├── constants/        # Business constants & route definitions
├── pages/           # Route-based entry points
├── queries/         # TanStack Query hooks
├── repositories/    # Data access layer
├── services/        # Pure business logic functions
└── types/           # TypeScript interfaces
```

## 🔄 Component Structure

Every component follows this recursive pattern:

```
component-name/
├── component-name.view.tsx        # UI presentation
├── component-name.controller.ts   # Business logic hook
├── component-name.spec.tsx        # Tests
└── components/                    # Sub-components
    ├── sub-component-a/
    └── sub-component-b/
```

## 🚀 Key Principles

1. **Separation of Concerns**: Views handle presentation, controllers manage logic
2. **Dependency Rule**: Dependencies flow inward (UI → Application → Core)
3. **Recursive Structure**: Same patterns at every level
4. **Pure Functions**: Services contain only pure, testable functions
5. **English Keys**: Translation keys are actual English text
6. **Route Protection**: Authentication handled in route loaders
7. **Test Real Usage**: Integration tests over unit tests

## 🛠️ Development Workflow

1. **Plan**: Use recursive architecture to structure new features
2. **Routes**: Define route constants and configure loaders
3. **Components**: Create view + controller pairs
4. **Logic**: Extract business logic into services
5. **Data**: Use repositories for API interactions
6. **Test**: Write integration tests that simulate real usage
7. **i18n**: Add translations with English keys

## 📚 Quick Reference

- **Components**: Always function components with PascalCase names
- **Hooks**: `use[Action][Resource][Type]` (e.g., `useListUsersQuery`)
- **Files**: dash-case with descriptive suffixes
- **Routes**: Constants in `/constants/routes.ts`
- **i18n**: Namespaces in `/constants/namespaces.ts`
- **Tests**: `.spec.tsx` extension with descriptive names

These guidelines work together to create a maintainable, scalable, and developer-friendly frontend architecture. Start with the [Architecture](./architecture.md) guide for the foundational concepts, then explore specific areas as needed.
