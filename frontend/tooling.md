# Development Tooling

## Overview

This guide covers the development tooling configuration for consistent code formatting, linting, and editor setup across the project.

## Prettier

Prettier is configured at the project root to ensure consistent code formatting.

### Configuration

`.prettierrc`:

```json
{
  "singleQuote": false,
  "trailingComma": "all",
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "printWidth": 100,
  "arrowParens": "avoid"
}
```

### npm Scripts

```json
{
  "scripts": {
    "format": "prettier --write \"src/**/*.{ts,tsx,json}\" \"*.{ts,js,json}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,json}\" \"*.{ts,js,json}\""
  }
}
```

**Usage:**
- `npm run format` - Format all files
- `npm run format:check` - Check formatting without modifying files

## ESLint

ESLint is configured with Prettier integration and custom rules for the recursive architecture.

### Configuration

`eslint.config.js`:

```javascript
import js from "@eslint/js";
import prettierPlugin from "eslint-plugin-prettier/recommended";
import reactHooks from "eslint-plugin-react-hooks";
import reactRefresh from "eslint-plugin-react-refresh";
import { defineConfig, globalIgnores } from "eslint/config";
import globals from "globals";
import tseslint from "typescript-eslint";

export default defineConfig([
  globalIgnores(["dist"]),
  prettierPlugin,
  {
    files: ["**/*.{ts,tsx}"],
    extends: [
      js.configs.recommended,
      tseslint.configs.recommended,
      reactHooks.configs["recommended-latest"],
      reactRefresh.configs.vite,
    ],
    languageOptions: {
      ecmaVersion: 2020,
      globals: globals.browser,
    },
    rules: {
      "@typescript-eslint/no-restricted-imports": [
        "error",
        {
          patterns: [
            {
              group: ["../*", "../**/*"],
              message:
                "Relative imports going up directories are not allowed. " +
                "In our recursive architecture, components can only import from: " +
                "1) their own subcomponents (in /components), or 2) global components (via @/ alias). " +
                "This ensures clear dependency direction and prevents sibling dependencies.",
            },
          ],
        },
      ],
    },
  },
]);
```

### Key Features

1. **Prettier Integration**: `eslint-plugin-prettier` runs Prettier as an ESLint rule, catching formatting issues during linting
2. **No Relative Imports Up**: Custom rule prevents `../` imports to enforce recursive architecture
3. **TypeScript Support**: Full TypeScript linting with `typescript-eslint`

### npm Scripts

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix"
  }
}
```

**Usage:**
- `npm run lint` - Check for linting errors
- `npm run lint -- --fix` - Auto-fix linting errors

## EditorConfig

`.editorconfig` ensures consistent editor settings across different IDEs:

```
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 2
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false
```

## VSCode/Cursor Settings

`.vscode/settings.json` configures the editor for optimal development experience:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.organizeImports": "explicit",
    "source.fixAll.eslint": "explicit"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit",
      "source.fixAll.eslint": "explicit"
    }
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit",
      "source.fixAll.eslint": "explicit"
    }
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit",
      "source.fixAll.eslint": "explicit"
    }
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit",
      "source.fixAll.eslint": "explicit"
    }
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[jsonc]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

### Required Extensions

Install these extensions in VSCode/Cursor:

1. **Prettier - Code formatter** (`esbenp.prettier-vscode`)
   - Provides Prettier formatting support

2. **ESLint** (`dbaeumer.vscode-eslint`)
   - Provides ESLint integration (usually included by default)

### What Happens on Save

When you save a file:
1. **Prettier formats** the code according to `.prettierrc`
2. **Imports are organized** automatically
3. **ESLint fixes** are applied automatically

## Import Path Rules

ESLint enforces the recursive architecture by prohibiting relative imports going up directories:

```typescript
// ✅ Correct - using @/ alias
import { client } from "@/client/api-client";
import authService from "@/services/auth.service";

// ❌ Wrong - ESLint will error
import { client } from "../client/api-client";
```

This ensures components can only import:
1. Their own subcomponents (in `/components`)
2. Global components/services (via `@/` alias)

For more details, see the [Architecture Guide](./architecture.md).

## Project Structure

Tooling configuration files:

```
project-root/
├── .prettierrc              # Prettier configuration
├── .editorconfig            # Editor configuration
├── .vscode/
│   └── settings.json        # VSCode/Cursor settings
└── front/
    └── eslint.config.js     # ESLint configuration
```

## Troubleshooting

### Prettier not formatting on save

1. Ensure Prettier extension is installed
2. Check that `editor.formatOnSave` is `true` in settings
3. Verify `editor.defaultFormatter` is set to `esbenp.prettier-vscode`
4. Reload VSCode/Cursor window

### ESLint not detecting formatting errors

1. Ensure `eslint-plugin-prettier` is installed
2. Verify `prettierPlugin` is included in `eslint.config.js`
3. Run `npm run lint` to see if errors appear in terminal

### Import errors not being caught

1. Verify the `@typescript-eslint/no-restricted-imports` rule is in `eslint.config.js`
2. Check that TypeScript path aliases are configured in `tsconfig.json`
3. Ensure ESLint extension is enabled

## Related Documentation

- [Architecture Guide](./architecture.md) - Why relative imports are prohibited
- [React Components Guide](./react.md) - Component structure and patterns
- [API Client Guide](./api-client.md) - Example of using @/ alias




