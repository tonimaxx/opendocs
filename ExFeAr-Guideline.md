# ExFeAr Guidelines - Ultimate Edition

**Version:** 2.0 (Production Ready)
**Last Updated:** 2025-11-14
**Project:** Kanojo (Expo 54 + Expo Router)

---

## Table of Contents

1. [Introduction](#introduction)
2. [Core Architecture](#core-architecture)
3. [Project Structure](#project-structure)
4. [File Placement Rules](#file-placement-rules)
5. [Naming Conventions](#naming-conventions)
6. [Import Patterns](#import-patterns)
7. [Feature Development Guide](#feature-development-guide)
8. [Testing Strategy](#testing-strategy)
9. [Migration Guide](#migration-guide)
10. [Tooling & Configuration](#tooling--configuration)
11. [Best Practices](#best-practices)
12. [Common Patterns](#common-patterns)
13. [Troubleshooting](#troubleshooting)
14. [Quick Reference](#quick-reference)

---

## Introduction

### What is ExFeAr?

**ExFeAr** (ExpoRouter FeatureSlices Architecture) is a scalable, modular architecture pattern for React Native applications built with Expo Router. It organizes code by **domain features** rather than technical layers, making features portable, testable, and easy to reason about.

### Why ExFeAr?

Traditional folder structures organize code by type (`screens/`, `components/`, `services/`), which leads to:

- Scattered code across multiple directories for a single feature
- Difficulty finding all code related to one domain
- Hard-to-track dependencies
- Challenges in code reuse across apps
- Poor AI assistant comprehension

ExFeAr solves this by organizing code into **three clear layers**.

### Core Philosophy

> **"Routes map to features. Features use shared infrastructure. Infrastructure never depends on features."**

---

## Core Architecture

### Three-Layer Model

ExFeAr consists of three distinct layers with clear boundaries:

```
┌─────────────────────────────────────┐
│      ROUTING LAYER (app/)           │
│  - File-based routes                │
│  - Thin wrappers only               │
│  - No business logic                │
└──────────────┬──────────────────────┘
               │ imports screens
               ▼
┌─────────────────────────────────────┐
│   FEATURESLICES LAYER (features/)   │
│  - Domain-specific logic            │
│  - Self-contained modules           │
│  - Screens, components, hooks       │
└──────────────┬──────────────────────┘
               │ uses infrastructure
               ▼
┌─────────────────────────────────────┐
│  SHARED ROOT INFRASTRUCTURE         │
│  - components/, hooks/, services/   │
│  - utils/, types/, contexts/        │
│  - Reusable across all features     │
└─────────────────────────────────────┘
```

### Dependency Rules

**ALLOWED:**

✅ Routes → FeatureSlices
✅ Routes → Shared Infrastructure
✅ FeatureSlices → Shared Infrastructure
✅ FeatureSlices → Other FeatureSlices (with caution, prefer shared)

**FORBIDDEN:**

❌ Shared Infrastructure → FeatureSlices
❌ Shared Infrastructure → Routes
❌ Circular dependencies between features

---

## Project Structure

### Current Kanojo Structure

```
kanojo/
├── app/                          # Routing Layer (Expo Router)
│   ├── _layout.tsx              # Root layout with providers
│   ├── (tabs)/                  # Tab-based navigation
│   │   ├── _layout.tsx
│   │   ├── index.tsx            → features/knoww/home/
│   │   ├── chat.tsx             → features/knoww/chat/
│   │   ├── settings.tsx         → features/knoww/settings/
│   │   ├── utility.tsx
│   │   └── test.tsx
│   ├── settings/                # Settings sub-routes
│   │   ├── font-demo.tsx
│   │   └── component-showcase.tsx
│   ├── note.tsx
│   ├── notes.tsx
│   ├── camera.tsx
│   ├── modal.tsx
│   └── +not-found.tsx
│
├── features/                     # FeatureSlices Layer
│   ├── shared/                  # Cross-feature utilities
│   │   └── (shared helpers used by multiple features)
│   └── knoww/                   # Knoww app-specific features
│       ├── home/
│       ├── chat/
│       ├── settings/
│       │   └── SettingsScreen.tsx
│       ├── preferences/
│       └── notes/
│
├── components/                   # Shared Root Infrastructure
│   ├── AutoControl.tsx          # Generic auto-setting controls
│   ├── AutoSettingsScreen.tsx   # Base settings screen
│   ├── ComponentDocs.tsx        # Documentation component
│   ├── ComponentPreview.tsx     # Preview wrapper
│   ├── FABMenu.tsx              # Floating action button
│   ├── ThemeSelector.tsx        # Theme picker
│   ├── Typography.tsx           # Typography wrapper
│   ├── Themed.tsx               # Theme-aware components
│   ├── ExternalLink.tsx         # Link helper
│   └── __tests__/               # Component tests
│
├── contexts/                     # Global React Contexts
│   ├── AppConfigContext.tsx     # App-wide configuration
│   ├── ThemeContext.tsx         # Theme management
│   ├── TypographyContext.tsx    # Typography settings
│   └── PreferencesContext.tsx   # User preferences
│
├── services/                     # External Integrations
│   ├── DifyApiService.ts        # Dify API client
│   └── DifyDirectService.ts     # Direct Dify integration
│
├── hooks/                        # Shared Custom Hooks
│   └── (currently in components/, to be moved)
│
├── utils/                        # Pure Helper Functions
│   ├── fontLoader.ts            # Font loading utilities
│   ├── themeHelper.ts           # Theme manipulation
│   └── typographyScale.ts       # Typography calculations
│
├── types/                        # Global TypeScript Types
│   ├── note.ts                  # Note-related types
│   ├── settings.ts              # Settings types
│   └── config.ts                # Configuration types
│
├── constants/                    # App Constants
│   ├── fonts/                   # Font configurations
│   │   └── environments/
│   └── (other constants)
│
├── themes/                       # Design System
│   └── (theme configurations)
│
├── assets/                       # Static Assets
│   ├── fonts/                   # Font files
│   ├── images/                  # Images
│   └── (other assets)
│
├── screens/                      # LEGACY - Being phased out
│   └── (old screens to migrate to features/)
│
└── note/                         # Documentation
    ├── ExFeAr.md
    ├── ExFeAr-ClaudeReviewed.md
    └── ExFeAr-GuideLine.md      ← You are here
```

---

## File Placement Rules

### Decision Tree: Where Should My File Go?

Use this flowchart to determine file placement:

```
┌─────────────────────────────────┐
│   Is this a route/navigation?   │
│                                 │
└────────┬───────────┬────────────┘
         │ YES       │ NO
         ▼           ▼
    app/         ┌──────────────────────────────────┐
                 │  Is this domain-specific to      │
                 │  one feature area?               │
                 └────────┬───────────┬─────────────┘
                          │ YES       │ NO
                          ▼           ▼
                  features/       ┌────────────────────────────┐
                                  │ Is it used by 2+ features  │
                                  │ but not generic enough     │
                                  │ for root?                  │
                                  └────┬──────────┬────────────┘
                                       │ YES      │ NO
                                       ▼          ▼
                               features/shared/   Root folders
                                                 (components/,
                                                  services/,
                                                  utils/, etc.)
```

### Root Folders - What Goes Where

#### `components/`

**Purpose:** Shared, reusable UI primitives and layout components.

**Belongs here:**

- Generic UI elements (buttons, cards, inputs)
- Layout wrappers (ScreenContainer, Section)
- Typography wrappers
- Theme-aware components
- Generic form controls

**Examples:**

- ✅ `AutoControl.tsx` - Generic auto-setting control
- ✅ `FABMenu.tsx` - Floating action button
- ✅ `ThemeSelector.tsx` - Theme picker
- ✅ `Typography.tsx` - Typography wrapper
- ❌ `CourseCard.tsx` - Domain-specific (belongs in features/knoww/courses/)
- ❌ `ChatMessage.tsx` - Domain-specific (belongs in features/knoww/chat/)

**Naming:** PascalCase, descriptive, prefixed with `App` or generic name.

#### `contexts/`

**Purpose:** App-wide React contexts and providers.

**Belongs here:**

- Global state management (theme, auth, config)
- Providers used by multiple features
- App-level configuration

**Examples:**

- ✅ `ThemeContext.tsx` - App-wide theme
- ✅ `AppConfigContext.tsx` - App configuration
- ✅ `PreferencesContext.tsx` - User preferences
- ❌ `CourseContext.tsx` - Feature-specific (belongs in features/knoww/courses/)

**Rule:** If a context is used by 3+ features OR is truly app-wide, it belongs here.

#### `services/`

**Purpose:** External integrations, API clients, third-party SDKs.

**Belongs here:**

- HTTP clients (axios, fetch wrappers)
- Firebase/Supabase clients
- Analytics services
- Storage services
- Authentication clients
- Push notification services

**Examples:**

- ✅ `DifyApiService.ts` - Dify API client
- ✅ `apiClient.ts` - Generic HTTP client
- ✅ `firebaseClient.ts` - Firebase initialization
- ✅ `storageService.ts` - AsyncStorage wrapper
- ❌ `fetchCourses()` - Domain operation (belongs in features/knoww/courses/courses.service.ts)

**Pattern:**

```typescript
// services/apiClient.ts - Generic HTTP client
export class ApiClient {
  async get(url: string) { /* ... */ }
  async post(url: string, data: any) { /* ... */ }
}

export const apiClient = new ApiClient();
```

#### `hooks/`

**Purpose:** Shared custom hooks with no domain-specific logic.

**Belongs here:**

- Utility hooks (useDebounce, useBoolean)
- Platform detection hooks
- Lifecycle hooks
- Generic state management hooks

**Examples:**

- ✅ `useDebounce.ts`
- ✅ `useBoolean.ts`
- ✅ `useInterval.ts`
- ✅ `useKeyboardHeight.ts`
- ❌ `useCourses.ts` - Domain hook (belongs in features/knoww/courses/)

**Note:** Currently, some hooks are in `components/` (useColorScheme, useClientOnlyValue). These should be moved to `hooks/`.

#### `utils/`

**Purpose:** Pure helper functions with no side effects.

**Belongs here:**

- Formatters (dates, numbers, currency)
- Validators
- Transformers
- Constants helpers
- Math utilities

**Examples:**

- ✅ `fontLoader.ts` - Font loading utilities
- ✅ `themeHelper.ts` - Theme manipulation
- ✅ `typographyScale.ts` - Typography calculations
- ✅ `formatDate.ts`
- ✅ `validateEmail.ts`
- ❌ `calculateCourseProgress.ts` - Domain logic (belongs in features/knoww/courses/)

**Rule:** If it has side effects (API calls, storage, etc.), it's NOT a utility.

#### `types/`

**Purpose:** Global TypeScript types and interfaces used across the app.

**Belongs here:**

- API response types
- Global entity types (User, App, Config)
- Generic utility types
- Shared enums

**Examples:**

- ✅ `config.ts` - App configuration types
- ✅ `settings.ts` - Settings types
- ✅ `api.types.ts` - API response types
- ❌ `Course.ts` - Domain type (belongs in features/knoww/courses/courses.types.ts unless used by 3+ features)

**Rule:** If a type is used by only one feature, keep it in that feature. Promote to root when needed by 2+ features.

#### `constants/`

**Purpose:** Global configuration constants and static values.

**Belongs here:**

- Environment configuration
- API endpoints
- Feature flags
- Route names
- Theme constants

**Examples:**

- ✅ `fonts/` - Font configurations
- ✅ `routes.ts` - Route constants
- ✅ `config.ts` - App constants
- ❌ `courseCategories.ts` - Domain constant (belongs in features/knoww/courses/)

#### `themes/`

**Purpose:** Design system, color palettes, typography scales.

**Belongs here:**

- Light/dark theme definitions
- Color tokens
- Spacing scales
- Shadow definitions
- Typography configurations

### FeatureSlices - Structure

#### Feature Folder Structure

Each feature folder should be **flat** and self-contained:

```
features/knoww/courses/
├── CoursesScreen.tsx           # Main screen (list view)
├── CourseDetailScreen.tsx      # Detail screen
├── CourseCard.tsx              # UI component
├── ChapterListItem.tsx         # UI component
├── courses.hooks.ts            # useCourses, useCourse
├── courses.service.ts          # API calls
├── courses.types.ts            # Course, Chapter types
├── courses.utils.ts            # Course-specific helpers
└── __tests__/                  # Tests
    ├── CoursesScreen.test.tsx
    └── courses.service.test.ts
```

#### What Goes in a Feature

**Screens:**

- Primary UI for the feature
- Named `*Screen.tsx`
- Example: `CoursesScreen.tsx`, `CourseDetailScreen.tsx`

**Components:**

- UI elements specific to this feature
- Named descriptively
- Example: `CourseCard.tsx`, `ChapterListItem.tsx`

**Hooks:**

- State management for the feature
- Named `[feature].hooks.ts` or `use[Feature].ts`
- Example: `courses.hooks.ts` containing `useCourses()`, `useCourse(id)`

**Services:**

- Feature-specific API operations
- Wraps shared `services/`
- Named `[feature].service.ts`
- Example: `courses.service.ts`

**Types:**

- Feature-specific TypeScript types
- Named `[feature].types.ts`
- Example: `courses.types.ts`

**Utils:**

- Domain-specific helper functions
- Named `[feature].utils.ts`
- Example: `courses.utils.ts`

#### Feature Naming

**Convention:** `features/[app]/[domain]/`

- **app:** Application namespace (e.g., `knoww`, `knowwKids`)
- **domain:** Feature domain (e.g., `courses`, `chat`, `settings`)

**Examples:**

- `features/knoww/courses/` - Knoww courses feature
- `features/knoww/chat/` - Knoww chat feature
- `features/knoww/settings/` - Knoww settings feature
- `features/shared/` - Cross-feature utilities
- `features/auth/` - Generic authentication (no app namespace)

#### `features/shared/` - Special Case

Use `features/shared/` for code that:

- Is used by 2+ features within the same app
- Contains domain logic (not generic enough for root)
- Doesn't fit in root infrastructure

**Examples:**

- Feature-domain parsers used by lessons AND quiz
- Cross-feature types (shared between courses and lessons but not app-wide)
- Feature-level utilities

**When to use:**

```
Generic for all apps → Root folders (components/, utils/, etc.)
Used by 2+ features → features/shared/
Used by 1 feature → That feature's folder
```

---

## Naming Conventions

### Files

| Type | Convention | Example |
|------|-----------|---------|
| Screens | `[Name]Screen.tsx` | `CoursesScreen.tsx`, `SettingsScreen.tsx` |
| Components | `PascalCase.tsx` | `CourseCard.tsx`, `ThemeSelector.tsx` |
| Hooks | `[feature].hooks.ts` or `use[Name].ts` | `courses.hooks.ts`, `useDebounce.ts` |
| Services | `[Name]Service.ts` or `[feature].service.ts` | `DifyApiService.ts`, `courses.service.ts` |
| Utils | `camelCase.ts` or `[feature].utils.ts` | `fontLoader.ts`, `courses.utils.ts` |
| Types | `[name].types.ts` or `[domain].ts` | `courses.types.ts`, `config.ts` |
| Contexts | `[Name]Context.tsx` | `ThemeContext.tsx`, `AppConfigContext.tsx` |
| Constants | `UPPER_SNAKE_CASE` or `camelCase.ts` | `API_ENDPOINTS.ts`, `routes.ts` |

### Folders

- **Root folders:** lowercase plural (`components/`, `utils/`, `services/`)
- **Feature folders:** lowercase singular (`features/knoww/courses/`, not `coursess/`)
- **Test folders:** `__tests__/`

### Exports

**Prefer named exports over default exports:**

```typescript
// ✅ Good - Named export
export function useCourses() { /* ... */ }
export const courseService = { /* ... */ };

// ❌ Avoid - Default export (except for screens)
export default function useCourses() { /* ... */ }

// ✅ Exception - Screens use default export (Expo Router convention)
export default function CoursesScreen() { /* ... */ }
```

---

## Import Patterns

### Path Aliases

**Current configuration:**

```json
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

### Import Standards

**ALWAYS use `@/` alias for absolute imports:**

```typescript
// ✅ CORRECT - Use @ alias
import { ThemeSelector } from '@/components/ThemeSelector';
import { useAppConfig } from '@/contexts/AppConfigContext';
import { resetThemeToDefault } from '@/utils/themeHelper';
import { DifyApiService } from '@/services/DifyApiService';

// ❌ WRONG - Relative paths from features
import ThemeSelector from '../../../components/ThemeSelector';
import { useAppConfig } from '../../../contexts/AppConfigContext';
```

### Import Order

Organize imports in this order:

```typescript
// 1. React and external libraries
import { useState, useEffect } from 'react';
import { View, StyleSheet } from 'react-native';
import { Button, Card } from 'react-native-paper';
import { router } from 'expo-router';

// 2. Shared root infrastructure (@ alias)
import { AutoControl } from '@/components/AutoControl';
import { useTheme } from '@/contexts/ThemeContext';
import { apiClient } from '@/services/apiClient';
import { formatDate } from '@/utils/formatDate';
import type { Config } from '@/types/config';

// 3. Feature-level imports (relative within feature, @ alias for cross-feature)
import { useCourses } from './courses.hooks';
import { courseService } from './courses.service';
import type { Course } from './courses.types';

// 4. Assets
import logo from '@/assets/images/logo.png';
```

### Barrel Exports

**Use index files sparingly:**

```typescript
// features/knoww/courses/index.ts
export { default as CoursesScreen } from './CoursesScreen';
export { default as CourseDetailScreen } from './CourseDetailScreen';
export { useCourses, useCourse } from './courses.hooks';
export { courseService } from './courses.service';
export type { Course, Chapter } from './courses.types';

// Then import in route:
import { CoursesScreen } from '@/features/knoww/courses';
```

**⚠️ Warning:** Barrel exports can cause circular dependency issues and increase bundle size. Use with caution.

---

## Feature Development Guide

### Creating a New Feature

#### Step 1: Plan the Feature

Answer these questions:

1. **What domain does this belong to?** (e.g., courses, chat, settings)
2. **Which app?** (e.g., knoww, knowwKids, or generic)
3. **What screens are needed?**
4. **What data operations are required?**
5. **What shared components will be reused?**
6. **What new types are needed?**

#### Step 2: Create Feature Folder

```bash
# For app-specific feature
mkdir -p features/knoww/[feature-name]

# For generic feature
mkdir -p features/[feature-name]
```

#### Step 3: Create Feature Files

**Minimum viable feature:**

```
features/knoww/courses/
├── CoursesScreen.tsx        # Main screen
├── courses.types.ts         # Types
└── courses.service.ts       # API operations (optional)
```

**Full feature structure:**

```
features/knoww/courses/
├── CoursesScreen.tsx        # List screen
├── CourseDetailScreen.tsx   # Detail screen
├── CourseCard.tsx           # UI component
├── courses.hooks.ts         # Custom hooks
├── courses.service.ts       # API wrapper
├── courses.types.ts         # TypeScript types
├── courses.utils.ts         # Helpers
└── __tests__/               # Tests
    ├── CoursesScreen.test.tsx
    └── courses.service.test.ts
```

#### Step 4: Implement Feature Service

**Pattern for feature services:**

```typescript
// features/knoww/courses/courses.service.ts
import { apiClient } from '@/services/DifyApiService';
import type { Course, CreateCourseDTO } from './courses.types';

export const courseService = {
  /**
   * Fetch all courses
   */
  async fetchCourses(): Promise<Course[]> {
    const response = await apiClient.get('/courses');
    return response.data;
  },

  /**
   * Fetch single course by ID
   */
  async fetchCourse(id: string): Promise<Course> {
    const response = await apiClient.get(`/courses/${id}`);
    return response.data;
  },

  /**
   * Create a new course
   */
  async createCourse(data: CreateCourseDTO): Promise<Course> {
    const response = await apiClient.post('/courses', data);
    return response.data;
  },

  /**
   * Update course
   */
  async updateCourse(id: string, data: Partial<Course>): Promise<Course> {
    const response = await apiClient.patch(`/courses/${id}`, data);
    return response.data;
  },

  /**
   * Delete course
   */
  async deleteCourse(id: string): Promise<void> {
    await apiClient.delete(`/courses/${id}`);
  },
};
```

#### Step 5: Implement Feature Hooks

**Pattern for feature hooks:**

```typescript
// features/knoww/courses/courses.hooks.ts
import { useState, useEffect } from 'react';
import { courseService } from './courses.service';
import type { Course } from './courses.types';

/**
 * Hook to fetch and manage list of courses
 */
export function useCourses() {
  const [courses, setCourses] = useState<Course[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetchCourses();
  }, []);

  const fetchCourses = async () => {
    try {
      setLoading(true);
      setError(null);
      const data = await courseService.fetchCourses();
      setCourses(data);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  };

  const refresh = () => fetchCourses();

  return {
    courses,
    loading,
    error,
    refresh,
  };
}

/**
 * Hook to fetch and manage single course
 */
export function useCourse(courseId: string) {
  const [course, setCourse] = useState<Course | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    if (courseId) {
      fetchCourse();
    }
  }, [courseId]);

  const fetchCourse = async () => {
    try {
      setLoading(true);
      setError(null);
      const data = await courseService.fetchCourse(courseId);
      setCourse(data);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  };

  return {
    course,
    loading,
    error,
    refresh: fetchCourse,
  };
}
```

#### Step 6: Create Screen Component

```typescript
// features/knoww/courses/CoursesScreen.tsx
import { View, FlatList, StyleSheet } from 'react-native';
import { ActivityIndicator, Text } from 'react-native-paper';
import { CourseCard } from './CourseCard';
import { useCourses } from './courses.hooks';

export default function CoursesScreen() {
  const { courses, loading, error, refresh } = useCourses();

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.center}>
        <Text>Error: {error.message}</Text>
      </View>
    );
  }

  return (
    <FlatList
      data={courses}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <CourseCard course={item} />}
      onRefresh={refresh}
      refreshing={loading}
      contentContainerStyle={styles.list}
    />
  );
}

const styles = StyleSheet.create({
  center: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  list: {
    padding: 16,
  },
});
```

#### Step 7: Create Route

```typescript
// app/(tabs)/courses.tsx
import CoursesScreen from '@/features/knoww/courses/CoursesScreen';

export default function CoursesRoute() {
  return <CoursesScreen />;
}
```

---

## Testing Strategy

### Test File Placement

**Option 1: Colocated tests (RECOMMENDED)**

```
features/knoww/courses/
├── CoursesScreen.tsx
├── CoursesScreen.test.tsx     ← Test next to implementation
├── courses.service.ts
├── courses.service.test.ts    ← Test next to implementation
└── courses.hooks.ts
    └── courses.hooks.test.ts
```

**Option 2: `__tests__` folder**

```
features/knoww/courses/
├── __tests__/
│   ├── CoursesScreen.test.tsx
│   ├── courses.service.test.ts
│   └── courses.hooks.test.ts
├── CoursesScreen.tsx
├── courses.service.ts
└── courses.hooks.ts
```

### Testing Patterns

#### Screen Tests

```typescript
// features/knoww/courses/CoursesScreen.test.tsx
import { render, screen, waitFor } from '@testing-library/react-native';
import CoursesScreen from './CoursesScreen';
import { courseService } from './courses.service';

jest.mock('./courses.service');

describe('CoursesScreen', () => {
  it('displays loading state', () => {
    render(<CoursesScreen />);
    expect(screen.getByTestId('loading-indicator')).toBeTruthy();
  });

  it('displays courses after loading', async () => {
    const mockCourses = [
      { id: '1', title: 'Course 1' },
      { id: '2', title: 'Course 2' },
    ];

    (courseService.fetchCourses as jest.Mock).mockResolvedValue(mockCourses);

    render(<CoursesScreen />);

    await waitFor(() => {
      expect(screen.getByText('Course 1')).toBeTruthy();
      expect(screen.getByText('Course 2')).toBeTruthy();
    });
  });
});
```

#### Service Tests

```typescript
// features/knoww/courses/courses.service.test.ts
import { courseService } from './courses.service';
import { apiClient } from '@/services/DifyApiService';

jest.mock('@/services/DifyApiService');

describe('courseService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('fetchCourses', () => {
    it('fetches courses successfully', async () => {
      const mockData = [{ id: '1', title: 'Test Course' }];
      (apiClient.get as jest.Mock).mockResolvedValue({ data: mockData });

      const result = await courseService.fetchCourses();

      expect(apiClient.get).toHaveBeenCalledWith('/courses');
      expect(result).toEqual(mockData);
    });

    it('handles errors', async () => {
      const mockError = new Error('Network error');
      (apiClient.get as jest.Mock).mockRejectedValue(mockError);

      await expect(courseService.fetchCourses()).rejects.toThrow('Network error');
    });
  });
});
```

#### Hook Tests

```typescript
// features/knoww/courses/courses.hooks.test.ts
import { renderHook, waitFor } from '@testing-library/react-native';
import { useCourses } from './courses.hooks';
import { courseService } from './courses.service';

jest.mock('./courses.service');

describe('useCourses', () => {
  it('fetches courses on mount', async () => {
    const mockCourses = [{ id: '1', title: 'Test' }];
    (courseService.fetchCourses as jest.Mock).mockResolvedValue(mockCourses);

    const { result } = renderHook(() => useCourses());

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
      expect(result.current.courses).toEqual(mockCourses);
    });
  });
});
```

### Test Coverage Guidelines

**Minimum coverage:**

- Services: 80%+
- Hooks: 70%+
- Screens: 60%+
- Utils: 90%+

**Run tests:**

```bash
npm test                    # Run all tests
npm run test:watch          # Watch mode
npm run test:coverage       # Coverage report
```

---

## Migration Guide

### Migrating from Legacy `screens/` to `features/`

#### Assessment Phase

1. **Identify screens to migrate:**

   ```bash
   find screens/ -name "*.tsx" -o -name "*.ts"
   ```

2. **Group by domain:**

   - Which screens belong together?
   - What's the business domain?
   - What data do they share?

3. **Prioritize:**

   - Start with small, self-contained features
   - New features first
   - Most-changed screens next
   - Rarely-touched screens last

#### Migration Steps

**Example: Migrating Settings Screen**

**Before (Legacy):**

```
screens/
└── SettingsScreen.tsx

components/
├── ThemeSelector.tsx
└── Typography.tsx

contexts/
├── ThemeContext.tsx
└── AppConfigContext.tsx

app/(tabs)/
└── settings.tsx (imports from screens/)
```

**After (ExFeAr):**

```
features/knoww/settings/
└── SettingsScreen.tsx

components/            # Unchanged - shared components
├── ThemeSelector.tsx
└── Typography.tsx

contexts/              # Unchanged - global contexts
├── ThemeContext.tsx
└── AppConfigContext.tsx

app/(tabs)/
└── settings.tsx (imports from features/)
```

**Step-by-step:**

1. **Create feature folder:**

   ```bash
   mkdir -p features/knoww/settings
   ```

2. **Move screen file:**

   ```bash
   mv screens/SettingsScreen.tsx features/knoww/settings/
   ```

3. **Update imports in screen:**

   ```typescript
   // Before (relative paths)
   import ThemeSelector from '../../../components/ThemeSelector';
   import { useTheme } from '../../../contexts/ThemeContext';

   // After (@ alias)
   import { ThemeSelector } from '@/components/ThemeSelector';
   import { useTheme } from '@/contexts/ThemeContext';
   ```

4. **Update route import:**

   ```typescript
   // app/(tabs)/settings.tsx

   // Before
   import SettingsScreen from '@/screens/SettingsScreen';

   // After
   import SettingsScreen from '@/features/knoww/settings/SettingsScreen';
   ```

5. **Extract feature-specific code:**

   If the screen has domain logic, extract it:

   ```bash
   # Create feature service
   touch features/knoww/settings/settings.service.ts

   # Create feature hooks
   touch features/knoww/settings/settings.hooks.ts

   # Create feature types
   touch features/knoww/settings/settings.types.ts
   ```

6. **Update tests:**

   ```bash
   mv screens/__tests__/SettingsScreen.test.tsx features/knoww/settings/
   ```

7. **Verify:**

   - Check imports are correct
   - Run tests: `npm test`
   - Build app: `npx expo start`
   - Test feature functionality

8. **Clean up:**

   ```bash
   # Remove empty legacy folders
   rmdir screens/ (if empty)
   ```

#### Migration Checklist

- [ ] Identify feature domain and app namespace
- [ ] Create feature folder: `features/[app]/[domain]/`
- [ ] Move screen file(s)
- [ ] Update all imports to use `@/` alias
- [ ] Extract domain logic to services
- [ ] Extract state logic to hooks
- [ ] Create types file
- [ ] Update route imports
- [ ] Move/create tests
- [ ] Run tests and verify
- [ ] Update documentation

---

## Tooling & Configuration

### TypeScript Configuration

**Current `tsconfig.json`:**

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": [
    "**/*.ts",
    "**/*.tsx",
    ".expo/types/**/*.ts",
    "expo-env.d.ts"
  ]
}
```

**✅ Already configured correctly!**

### ESLint Rules (Recommended)

Create `.eslintrc.js`:

```javascript
module.exports = {
  extends: ['expo', 'prettier'],
  plugins: ['import'],
  rules: {
    // Enforce @ alias usage
    'no-restricted-imports': [
      'error',
      {
        patterns: [
          {
            group: ['../**/components/*', '../../components/*', '../../../components/*'],
            message: 'Use @ alias instead: @/components/*',
          },
          {
            group: ['../**/services/*', '../../services/*', '../../../services/*'],
            message: 'Use @ alias instead: @/services/*',
          },
          {
            group: ['../**/contexts/*', '../../contexts/*', '../../../contexts/*'],
            message: 'Use @ alias instead: @/contexts/*',
          },
          {
            group: ['../**/utils/*', '../../utils/*', '../../../utils/*'],
            message: 'Use @ alias instead: @/utils/*',
          },
        ],
      },
    ],

    // Enforce import order
    'import/order': [
      'error',
      {
        groups: [
          'builtin',
          'external',
          'internal',
          'parent',
          'sibling',
          'index',
        ],
        pathGroups: [
          {
            pattern: '@/**',
            group: 'internal',
          },
        ],
        'newlines-between': 'always',
        alphabetize: {
          order: 'asc',
          caseInsensitive: true,
        },
      },
    ],

    // Prefer named exports
    'import/prefer-default-export': 'off',
    'import/no-default-export': ['error', {
      // Allow default export for screens (Expo Router convention)
      allow: ['**/*Screen.tsx', 'app/**/*.tsx'],
    }],
  },
};
```

### VS Code Settings

Create `.vscode/settings.json`:

```json
{
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "editor.formatOnSave": true,
  "files.associations": {
    "*.tsx": "typescriptreact"
  },
  "search.exclude": {
    "**/node_modules": true,
    "**/screens": true
  },
  "files.exclude": {
    "**/screens": true
  }
}
```

### Feature Generator Script (Optional)

Create `scripts/create-feature.js`:

```javascript
#!/usr/bin/env node
const fs = require('fs');
const path = require('path');

const [,, app, domain] = process.argv;

if (!app || !domain) {
  console.error('Usage: node scripts/create-feature.js <app> <domain>');
  console.error('Example: node scripts/create-feature.js knoww courses');
  process.exit(1);
}

const featurePath = path.join('features', app, domain);

// Create directories
fs.mkdirSync(featurePath, { recursive: true });
fs.mkdirSync(path.join(featurePath, '__tests__'), { recursive: true });

// Create files
const screenName = domain.charAt(0).toUpperCase() + domain.slice(1);

const files = {
  [`${screenName}Screen.tsx`]: `
import { View, StyleSheet } from 'react-native';
import { Text } from 'react-native-paper';

export default function ${screenName}Screen() {
  return (
    <View style={styles.container}>
      <Text>${screenName} Screen</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});
`.trim(),

  [`${domain}.types.ts`]: `
export interface ${screenName} {
  id: string;
  // Add your properties here
}
`.trim(),

  [`${domain}.service.ts`]: `
import { apiClient } from '@/services/DifyApiService';
import type { ${screenName} } from './${domain}.types';

export const ${domain}Service = {
  async fetch(): Promise<${screenName}[]> {
    const response = await apiClient.get('/${domain}');
    return response.data;
  },
};
`.trim(),

  [`${domain}.hooks.ts`]: `
import { useState, useEffect } from 'react';
import { ${domain}Service } from './${domain}.service';
import type { ${screenName} } from './${domain}.types';

export function use${screenName}s() {
  const [data, setData] = useState<${screenName}[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetchData();
  }, []);

  const fetchData = async () => {
    try {
      setLoading(true);
      const result = await ${domain}Service.fetch();
      setData(result);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  };

  return { data, loading, error, refresh: fetchData };
}
`.trim(),
};

// Write files
Object.entries(files).forEach(([filename, content]) => {
  fs.writeFileSync(path.join(featurePath, filename), content);
  console.log(`✅ Created ${filename}`);
});

console.log(`\n🎉 Feature created at ${featurePath}\n`);
console.log('Next steps:');
console.log(`1. Implement your feature in ${featurePath}/`);
console.log(`2. Create a route in app/ that imports ${screenName}Screen`);
console.log(`3. Add tests in ${featurePath}/__tests__/\n`);
```

**Usage:**

```bash
node scripts/create-feature.js knoww courses
```

---

## Best Practices

### 1. Keep Routes Thin

**❌ Bad:**

```typescript
// app/(tabs)/courses.tsx
export default function CoursesRoute() {
  const [courses, setCourses] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchCourses();
  }, []);

  const fetchCourses = async () => {
    // Complex logic here...
  };

  return (
    <View>
      {/* Complex UI here... */}
    </View>
  );
}
```

**✅ Good:**

```typescript
// app/(tabs)/courses.tsx
import CoursesScreen from '@/features/knoww/courses/CoursesScreen';

export default function CoursesRoute() {
  return <CoursesScreen />;
}
```

### 2. Use Path Aliases Consistently

**❌ Bad:**

```typescript
import ThemeSelector from '../../../components/ThemeSelector';
import { useTheme } from '../../../contexts/ThemeContext';
```

**✅ Good:**

```typescript
import { ThemeSelector } from '@/components/ThemeSelector';
import { useTheme } from '@/contexts/ThemeContext';
```

### 3. Colocate Related Code

**❌ Bad:**

```
features/knoww/courses/CoursesScreen.tsx
features/knoww/courses/hooks/useCourses.ts
features/knoww/courses/services/courseService.ts
features/knoww/courses/types/course.ts
```

**✅ Good:**

```
features/knoww/courses/CoursesScreen.tsx
features/knoww/courses/courses.hooks.ts
features/knoww/courses/courses.service.ts
features/knoww/courses/courses.types.ts
```

### 4. Prefer Named Exports

**❌ Bad:**

```typescript
// useCourses.ts
export default function useCourses() { /* ... */ }

// Import
import useCourses from './useCourses';
```

**✅ Good:**

```typescript
// courses.hooks.ts
export function useCourses() { /* ... */ }
export function useCourse(id: string) { /* ... */ }

// Import
import { useCourses, useCourse } from './courses.hooks';
```

### 5. Promote Types When Needed

**Scenario:** `Course` type initially in `features/knoww/courses/courses.types.ts` is now needed by `features/knoww/lessons/`.

**Solution:**

1. Move type to `types/courses.ts`
2. Update imports in both features
3. Keep feature-specific types in features

**Example:**

```typescript
// types/courses.ts (promoted)
export interface Course {
  id: string;
  title: string;
  // ...
}

// features/knoww/courses/courses.types.ts
export interface CourseFilters {  // Feature-specific
  category?: string;
  level?: string;
}

// features/knoww/lessons/lessons.types.ts
import type { Course } from '@/types/courses';  // Import shared type

export interface Lesson {
  id: string;
  course: Course;  // Use shared type
  // ...
}
```

### 6. Document Feature Dependencies

Create `features/knoww/courses/README.md`:

```markdown
# Courses Feature

## Overview
Manages course listing, detail views, and enrollment.

## Dependencies

### Shared Infrastructure
- `@/components/` - Uses AutoControl, FABMenu
- `@/contexts/AppConfigContext` - For app configuration
- `@/services/DifyApiService` - For API calls

### Feature Dependencies
- `features/shared/parsers` - For content parsing

## Screens
- `CoursesScreen` - Course list
- `CourseDetailScreen` - Course detail

## API Endpoints
- `GET /courses` - List courses
- `GET /courses/:id` - Get course detail
- `POST /courses/:id/enroll` - Enroll in course

## Tests
Run: `npm test -- courses`
```

### 7. Handle Route Parameters Properly

**Pattern:**

```typescript
// app/course/[courseId].tsx
import { useLocalSearchParams } from 'expo-router';
import CourseDetailScreen from '@/features/knoww/courses/CourseDetailScreen';

export default function CourseDetailRoute() {
  const { courseId } = useLocalSearchParams<{ courseId: string }>();

  return <CourseDetailScreen courseId={courseId} />;
}

// features/knoww/courses/CourseDetailScreen.tsx
interface CourseDetailScreenProps {
  courseId: string;
}

export default function CourseDetailScreen({ courseId }: CourseDetailScreenProps) {
  const { course, loading, error } = useCourse(courseId);
  // ...
}
```

### 8. Error Boundaries

**Create error boundary:**

```typescript
// components/ErrorBoundary.tsx
import React from 'react';
import { View, Text, Button } from 'react-native';

interface Props {
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <View>
          <Text>Something went wrong</Text>
          <Button title="Try again" onPress={() => this.setState({ hasError: false })} />
        </View>
      );
    }

    return this.props.children;
  }
}
```

**Use in layout:**

```typescript
// app/_layout.tsx
import { ErrorBoundary } from '@/components/ErrorBoundary';

export default function RootLayout() {
  return (
    <ErrorBoundary>
      <Stack />
    </ErrorBoundary>
  );
}
```

---

## Common Patterns

### Pattern 1: Feature with List + Detail

```
features/knoww/courses/
├── CoursesScreen.tsx          # List view
├── CourseDetailScreen.tsx     # Detail view
├── CourseCard.tsx             # List item component
├── courses.hooks.ts           # useCourses(), useCourse(id)
├── courses.service.ts         # API operations
└── courses.types.ts           # Course, CourseFilters types
```

### Pattern 2: Feature with Form

```
features/knoww/profile/
├── ProfileScreen.tsx          # View mode
├── EditProfileScreen.tsx      # Edit mode
├── ProfileForm.tsx            # Form component
├── profile.hooks.ts           # useProfile(), useUpdateProfile()
├── profile.service.ts         # API operations
├── profile.validation.ts      # Form validation
└── profile.types.ts           # Profile, UpdateProfileDTO types
```

### Pattern 3: Feature with Real-time Data

```
features/knoww/chat/
├── ChatScreen.tsx             # Main chat UI
├── MessageList.tsx            # Message list component
├── MessageInput.tsx           # Input component
├── chat.hooks.ts              # useMessages(), useSendMessage()
├── chat.service.ts            # WebSocket/API operations
└── chat.types.ts              # Message, Chat types
```

### Pattern 4: Settings/Preferences Feature

```
features/knoww/settings/
├── SettingsScreen.tsx         # Main settings screen
├── settings.hooks.ts          # useSettings()
└── (uses global contexts: PreferencesContext, ThemeContext)
```

### Pattern 5: Feature with Shared Logic

```
features/
├── shared/
│   ├── parsers/
│   │   └── contentParser.ts  # Used by lessons + quiz
│   └── validators/
│       └── answerValidator.ts
├── knoww/
│   ├── lessons/
│   │   └── (uses parsers/)
│   └── quiz/
│       └── (uses parsers/, validators/)
```

---

## Troubleshooting

### Import Errors

**Problem:** `Cannot find module '@/components/ThemeSelector'`

**Solution:**

1. Check `tsconfig.json` has correct path alias
2. Restart TypeScript server (VS Code: Cmd+Shift+P → "Restart TS Server")
3. Verify file exists at correct path
4. Clear Metro bundler cache: `npx expo start -c`

### Circular Dependencies

**Problem:** `Require cycle: features/A -> features/B -> features/A`

**Solutions:**

1. **Move shared logic to `features/shared/` or root:**

   ```typescript
   // ❌ Bad
   // features/A imports from features/B
   // features/B imports from features/A

   // ✅ Good
   // Both import from features/shared/ or root
   ```

2. **Use dependency injection:**

   ```typescript
   // Instead of importing directly
   export function ComponentA({ serviceB }: { serviceB: ServiceB }) {
     // ...
   }
   ```

3. **Lazy imports:**

   ```typescript
   const ModuleB = React.lazy(() => import('./ModuleB'));
   ```

### TypeScript Errors

**Problem:** TypeScript can't find types from features

**Solution:**

1. Ensure feature types are exported:

   ```typescript
   // courses.types.ts
   export interface Course { /* ... */ }  // Must be exported!
   ```

2. Import with correct path:

   ```typescript
   import type { Course } from '@/features/knoww/courses/courses.types';
   ```

### Build/Bundle Issues

**Problem:** Metro bundler errors or large bundle size

**Solutions:**

1. **Clear cache:**

   ```bash
   npx expo start -c
   ```

2. **Check for accidental imports:**

   ```bash
   # Find large imports
   npx expo-cli customize:web
   npx webpack-bundle-analyzer
   ```

3. **Review barrel exports:**

   Avoid `export * from` patterns that import everything.

---

## Quick Reference

### Cheat Sheet

```
┌─────────────────────────────────────────────────────────┐
│              ExFeAr Quick Reference                     │
├─────────────────────────────────────────────────────────┤
│ PLACEMENT RULES                                         │
├─────────────────────────────────────────────────────────┤
│ Routes only           → app/                            │
│ Domain-specific       → features/[app]/[domain]/        │
│ Used by 2+ features   → features/shared/                │
│ Generic reusable      → Root folders                    │
├─────────────────────────────────────────────────────────┤
│ IMPORT RULES                                            │
├─────────────────────────────────────────────────────────┤
│ ✅ Always use @ alias for root imports                  │
│ ✅ Relative imports OK within same feature              │
│ ❌ Never ../../../ for root folders                     │
├─────────────────────────────────────────────────────────┤
│ FEATURE STRUCTURE                                       │
├─────────────────────────────────────────────────────────┤
│ features/[app]/[domain]/                                │
│   ├── [Domain]Screen.tsx      # Screens                │
│   ├── [domain].hooks.ts       # Hooks                  │
│   ├── [domain].service.ts     # API calls              │
│   ├── [domain].types.ts       # Types                  │
│   ├── [domain].utils.ts       # Helpers                │
│   └── __tests__/              # Tests                  │
├─────────────────────────────────────────────────────────┤
│ NAMING CONVENTIONS                                      │
├─────────────────────────────────────────────────────────┤
│ Screens:     [Name]Screen.tsx                           │
│ Components:  PascalCase.tsx                             │
│ Hooks:       [feature].hooks.ts                         │
│ Services:    [feature].service.ts                       │
│ Types:       [feature].types.ts                         │
│ Utils:       [feature].utils.ts                         │
├─────────────────────────────────────────────────────────┤
│ COMMON COMMANDS                                         │
├─────────────────────────────────────────────────────────┤
│ npm test                 # Run tests                    │
│ npm run test:coverage    # Coverage report              │
│ npx expo start -c        # Clear cache                  │
│ node scripts/create-feature.js [app] [domain]           │
└─────────────────────────────────────────────────────────┘
```

### Decision Matrix

| Question | Yes | No |
|----------|-----|-----|
| Is it a route/screen entry? | `app/` | ↓ |
| Is it domain-specific to one feature? | `features/[app]/[domain]/` | ↓ |
| Used by 2+ features but not generic? | `features/shared/` | ↓ |
| Is it a UI component? | `components/` | ↓ |
| Is it an external integration? | `services/` | ↓ |
| Is it a pure helper function? | `utils/` | ↓ |
| Is it a global type? | `types/` | ↓ |
| Is it app-wide state? | `contexts/` | ↓ |
| Is it a constant? | `constants/` | ↓ |

### File Template Quick Access

**Create new screen:**

```typescript
// features/[app]/[domain]/[Domain]Screen.tsx
import { View, StyleSheet } from 'react-native';
import { Text } from 'react-native-paper';

export default function [Domain]Screen() {
  return (
    <View style={styles.container}>
      <Text>[Domain] Screen</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
});
```

**Create new service:**

```typescript
// features/[app]/[domain]/[domain].service.ts
import { apiClient } from '@/services/DifyApiService';
import type { [Type] } from './[domain].types';

export const [domain]Service = {
  async fetch[Items](): Promise<[Type][]> {
    const response = await apiClient.get('/[endpoint]');
    return response.data;
  },
};
```

**Create new hook:**

```typescript
// features/[app]/[domain]/[domain].hooks.ts
import { useState, useEffect } from 'react';
import { [domain]Service } from './[domain].service';
import type { [Type] } from './[domain].types';

export function use[Domain]s() {
  const [data, setData] = useState<[Type][]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetchData();
  }, []);

  const fetchData = async () => {
    try {
      setLoading(true);
      const result = await [domain]Service.fetch[Items]();
      setData(result);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  };

  return { data, loading, error, refresh: fetchData };
}
```

---

## Appendix

### Glossary

- **ExFeAr:** ExpoRouter FeatureSlices Architecture
- **FeatureSlice:** Self-contained vertical module for a domain feature
- **Routing Layer:** Expo Router's `app/` directory
- **Shared Root Infrastructure:** Root-level folders (components/, services/, etc.)
- **Domain:** Business area (courses, chat, settings, etc.)
- **App Namespace:** Application identifier (knoww, knowwKids, etc.)
- **Barrel Export:** index.ts file that re-exports from multiple files

### Resources

- [Expo Router Documentation](https://docs.expo.dev/router/introduction/)
- [Feature-Sliced Design](https://feature-sliced.design/)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)

### Version History

- **v2.0 (2025-11-14):** Production-ready guidelines based on Kanojo project
- **v1.0 (Initial):** Original ExFeAr specification

---

## Summary

ExFeAr provides a **clear, scalable, and maintainable** architecture for Expo Router applications by:

✅ **Organizing code by domain** (features) not file type
✅ **Keeping routes thin** and navigation pure
✅ **Making features portable** and reusable
✅ **Enforcing clean dependencies** (one-directional flow)
✅ **Improving AI comprehension** through clear structure
✅ **Enabling gradual migration** from legacy patterns
✅ **Supporting multi-app ecosystems** with namespacing

**Remember the golden rule:**

> **"If it's domain-specific → features/. If it's reusable → root folders. If it's a route → app/."**

---

**Questions or feedback?** Update this guide as the project evolves and patterns emerge.

**Last updated:** 2025-11-14
**Maintained by:** Development Team
**Version:** 2.0 Production
