---
name: react
description: Use when implementing React applications, components, Redux state, RTK Query services, routing, forms, or any React-related code following the established patterns
---

# React

## Overview

React applications built with Vite + TypeScript. State management with Redux Toolkit and RTK Query. Routing with React Router. Validation with Zod. Functional utilities with Ramda. Date handling with Day.js. Styling is project-dependent (Chakra UI, MUI, Tailwind, styled-components). Testing with Vitest and React Testing Library.

## Project Structure

The project follows a **modular architecture**: each feature is a self-contained folder with PascalCase naming. Shared infrastructure lives in `Core/`. Shared UI and utilities live in `Common/`.

```
src/
├── Core/                     # App-wide infrastructure
│   ├── Redux/
│   │   ├── Root.ts           # combineReducers
│   │   ├── Store.ts          # configureStore
│   │   └── middlewares/      # Custom Redux middlewares
│   ├── Services/
│   │   └── Api/index.ts      # RTK Query base API
│   └── Router.tsx            # Root router
├── Config/
│   └── Paths.ts              # Central URL definitions (Config.urls.*)
├── Theme/                    # Design tokens, theme config
├── Common/                   # Shared components, hooks, utils
├── Assets/                   # Static assets
├── <Module>/                 # Feature module (see below)
├── App.tsx
└── main.tsx
```

### Feature Module Structure

Every feature module follows the same internal layout:

```
Auth/
├── Components/               # Presentational components
├── Forms/                    # Form components (*Form.tsx)
├── Views/                    # Page-level container components (*View.tsx)
├── Guards/                   # Route guard HOFs or wrappers
├── Models/                   # Component display model definitions
├── Redux/
│   ├── index.ts              # Slice: state, reducers, actions, selectors
│   └── middlewares.ts        # Module-specific Redux middlewares
├── Schemas/                  # Zod validation schemas (*Schema.ts)
├── Services/
│   └── Api.ts                # RTK Query injected endpoints
├── Types/
│   └── index.ts              # TypeScript types and interfaces
├── Hooks.ts                  # Custom hooks for this module
├── Permissions.ts            # Permission constants
├── Router.tsx                # Module-level React Router routes
└── Utils.ts                  # Pure utility functions
```

Sub-features within a module get their own subfolder inside `Components/` or `Views/`, each self-contained with its own hooks, utils, and index.

## Naming Conventions

| File type | Convention | Example |
|-----------|------------|---------|
| Component | `PascalCase.tsx`, default export | `UserCard.tsx` |
| View (page) | `*View.tsx`, default export | `ProfileView.tsx` |
| Form | `*Form.tsx`, default export | `SignInForm.tsx` |
| Router | `*Router.tsx`, named export | `AuthRouter.tsx` |
| Schema | `*Schema.ts`, named export | `UserSchema.ts` |
| Hooks file | `Hooks.ts` or `use*.ts`, named exports | `Hooks.ts` |
| Types | `Types/index.ts`, named exports | `Types/index.ts` |
| Redux slice | `Redux/index.ts`, default export (reducer) | `Redux/index.ts` |
| API service | `Services/Api.ts`, named exports | `Services/Api.ts` |
| Utils | `Utils.ts`, named exports | `Utils.ts` |
| Folders | PascalCase | `Auth/`, `Common/`, `Statements/` |

**Export rules:**
- Default exports for components (Components, Views, Forms)
- Named exports for everything else (utils, hooks, types, schemas, actions, selectors)

## Redux Toolkit

### Slice pattern

Each module's `Redux/index.ts` defines the slice, exports actions by name, exports selectors by name, and exports the reducer as default.

```typescript
// Auth/Redux/index.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit'
import type { RootState } from 'Core/Redux/Root'
import type { AuthUser } from 'Auth/Types'

interface AuthState {
  user: AuthUser | null
}

const initialState: AuthState = {
  user: null,
}

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    setUser(state, { payload }: PayloadAction<AuthUser>) {
      state.user = payload
    },
    logout(state) {
      state.user = null
    },
  },
})

export const { setUser, logout } = authSlice.actions

export const selectUser = (state: RootState): AuthUser | null => state.auth.user

export default authSlice.reducer
```

### Root reducer

```typescript
// Core/Redux/Root.ts
import { combineReducers } from '@reduxjs/toolkit'
import { api } from 'Core/Services/Api'
import authReducer from 'Auth/Redux'
import uiReducer from 'Core/Redux/Ui'

const RootReducer = combineReducers({
  api: api.reducer,
  auth: authReducer,
  ui: uiReducer,
  // add module reducers here
})

export type RootState = ReturnType<typeof RootReducer>
export default RootReducer
```

### Selectors

Always name selectors `select*`. Keep them in the slice file.

```typescript
export const selectUser = (state: RootState) => state.auth.user
export const selectIsAuthenticated = (state: RootState) => state.auth.user !== null
```

## RTK Query

### Base API

Defined once in `Core/Services/Api/index.ts`. Uses `fetchBaseQuery` with automatic token injection from the Redux state.

```typescript
// Core/Services/Api/index.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react'
import type { RootState } from 'Core/Redux/Root'

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({
    baseUrl: import.meta.env.VITE_API_BASE_URL,
    prepareHeaders: (headers, { getState }) => {
      const token = (getState() as RootState).auth.token
      if (token) headers.set('Authorization', `Bearer ${token}`)
      return headers
    },
  }),
  endpoints: () => ({}),
})
```

### Injecting endpoints per module

Each module injects its own endpoints into the base API.

```typescript
// Auth/Services/Api.ts
import { api } from 'Core/Services/Api'
import type { AuthUser, SignInData, SignInResponse } from 'Auth/Types'

const extendedApi = api.injectEndpoints({
  endpoints: (builder) => ({
    currentUser: builder.query<AuthUser, void>({
      query: () => '/auth/me/',
    }),
    signIn: builder.mutation<SignInResponse, SignInData>({
      query: (body) => ({ url: '/auth/sign-in/', method: 'POST', body }),
    }),
  }),
})

export const { useCurrentUserQuery, useSignInMutation } = extendedApi
```

## Routing

Use the **object-based router API** (`createBrowserRouter` + `RouterProvider`). It has a more complete API than the JSX `<Routes>` approach and enables lazy loading and per-route error boundaries.

### URL configuration

All paths are defined centrally in `Config/Paths.ts` and accessed via `Config.urls.*`. Never hardcode path strings in components or routers.

```typescript
// Config/Paths.ts
export const urls = {
  home: '/',
  signIn: '/sign-in',
  profile: '/profile',
  admin: {
    users: '/admin/users',
    roles: '/admin/roles',
  },
  statements: {
    root: '/statements',
    input: '/statements/input',
    schemas: '/statements/schemas',
  },
}
```

### Root router

Defined once in `Core/Router.tsx`. Module routes are imported as `RouteObject[]` arrays and spread into `children`. Mount with `RouterProvider` in `main.tsx`.

```typescript
// Core/Router.tsx
import { createBrowserRouter } from 'react-router-dom'
import { makePrivate } from 'Auth/Guards/PrivateRoute'
import { statementsRoutes } from 'Statements/Router'
import Config from 'Config'

export const router = createBrowserRouter([
  // Public routes
  { path: Config.urls.signIn, element: <SignInView /> },
  { path: '/reset-password', element: <ResetPasswordView /> },

  // Protected routes inside the app shell
  {
    path: '/',
    element: makePrivate(<BaseLayout />),
    children: [
      { index: true, element: <DashboardView /> },
      { path: Config.urls.profile, element: <ProfileView /> },
      ...statementsRoutes,
    ],
  },
])
```

```typescript
// main.tsx
import { RouterProvider } from 'react-router-dom'
import { router } from 'Core/Router'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <Provider store={store}>
    <RouterProvider router={router} />
  </Provider>
)
```

### Module routes

Each module exports a `RouteObject[]` array spread into the parent `children`.

```typescript
// Statements/Router.tsx
import type { RouteObject } from 'react-router-dom'
import { makePrivate } from 'Auth/Guards/PrivateRoute'
import Config from 'Config'

export const statementsRoutes: RouteObject[] = [
  { path: Config.urls.statements.input, element: makePrivate(<InputView />) },
  { path: Config.urls.statements.schemas, element: makePrivate(<SchemasView />) },
]
```

### Route guards

`PrivateRoute` wraps children, reads auth state directly from the Redux store (not hooks), and optionally accepts a `permFunction` to check fine-grained permissions. `makePrivate` is the HOF used inline in route definitions.

```typescript
// Auth/Guards/PrivateRoute.tsx
import { Navigate } from 'react-router-dom'
import type { PropsWithChildren, ReactNode } from 'react'
import store from 'Core/Redux/Store'
import type { AuthenticatedUser } from 'Auth/Types'
import Config from 'Config'

const PrivateRoute = ({
  children,
  permFunction,
}: PropsWithChildren<{ permFunction?: (user: AuthenticatedUser) => boolean }>) => {
  const state = store.getState()
  if (state.auth?.user && (!permFunction || permFunction(state.auth.user))) {
    return children
  }
  return (
    <Navigate
      to={{ pathname: Config.urls.signIn }}
      state={{ next: window.location.pathname }}
      replace
    />
  )
}

export const makePrivate = (
  View: ReactNode,
  props?: { permFunction?: (user: AuthenticatedUser) => boolean },
) => <PrivateRoute {...props}>{View}</PrivateRoute>

export default PrivateRoute
```

Use `permFunction` for permission-restricted routes:

```typescript
{
  path: Config.urls.admin.users,
  element: makePrivate(<UsersView />, { permFunction: (u) => u.isStaff }),
}
```

## Components

### Views (page-level containers)

Views connect to Redux, call RTK Query hooks, and coordinate sub-components. They do not contain markup beyond layout.

```typescript
// Auth/Views/UsersView.tsx
import { useUsersQuery } from 'Auth/Services/Api'
import UsersList from 'Auth/Components/UsersList'

const UsersView = () => {
  const { data: users = [], isLoading } = useUsersQuery()
  return <UsersList users={users} isLoading={isLoading} />
}

export default UsersView
```

### Presentational components

Receive all data via props. No Redux, no API calls.

```typescript
// Auth/Components/UsersList.tsx
import type { User } from 'Auth/Types'

interface Props {
  users: User[]
  isLoading: boolean
}

const UsersList = ({ users, isLoading }: Props) => {
  if (isLoading) return <Spinner />
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}

export default UsersList
```

### Custom hooks

Extract business logic from components into `use*` hooks. Keep them in `Hooks.ts` at the module level, or in a `Hooks/` subfolder for complex sub-features.

```typescript
// Statements/Hooks.ts
import { useSelector } from 'react-redux'
import { selectYear } from 'Statements/Redux'

export const useCurrentYear = () => useSelector(selectYear)

export const useStatementPermissions = () => {
  const user = useCurrentUser()
  return {
    canEdit: user?.permissions.includes('statements.change'),
    canDelete: user?.permissions.includes('statements.delete'),
  }
}
```

## Validation with Zod

Define one schema file per form/entity in `Schemas/`. Export the schema and its inferred type.

```typescript
// Auth/Schemas/UserSchema.ts
import { z } from 'zod'

export const UserSchema = z.object({
  name: z.string().min(1, 'Required'),
  email: z.string().email('Invalid email'),
  role: z.string().min(1, 'Required'),
})

export type UserFormData = z.infer<typeof UserSchema>
```

## Forms

Forms live in `Forms/`. Use Zod schemas for validation. Use React Hook Form or custom hooks depending on complexity.

```typescript
// Auth/Forms/UserForm.tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { UserSchema, type UserFormData } from 'Auth/Schemas/UserSchema'
import { useCreateUserMutation } from 'Auth/Services/Api'

const UserForm = () => {
  const [createUser] = useCreateUserMutation()
  const { register, handleSubmit, formState: { errors } } = useForm<UserFormData>({
    resolver: zodResolver(UserSchema),
  })

  const onSubmit = (data: UserFormData) => createUser(data)

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} />
      {errors.name && <span>{errors.name.message}</span>}
    </form>
  )
}

export default UserForm
```

## Utilities

### Ramda

Use Ramda for data transformation pipelines. Prefer `pipe` / `compose` over imperative chains.

```typescript
import { pipe, filter, map, sortBy, prop } from 'ramda'

const getActiveUserNames = pipe(
  filter<User>((u) => u.isActive),
  sortBy(prop('name')),
  map(prop('name')),
)
```

### Day.js

Use Day.js for all date formatting and manipulation. Import plugins as needed.

```typescript
import dayjs from 'dayjs'
import relativeTime from 'dayjs/plugin/relativeTime'

dayjs.extend(relativeTime)

const formatDate = (date: string) => dayjs(date).format('DD/MM/YYYY')
const fromNow = (date: string) => dayjs(date).fromNow()
```

## Types

Define all TypeScript types in `Types/index.ts` per module. Use `interface` for object shapes, `type` for unions and aliases.

```typescript
// Auth/Types/index.ts
export interface AuthUser {
  id: number
  name: string
  email: string
  permissions: string[]
}

export interface SignInData {
  email: string
  password: string
}

export type UserRole = 'admin' | 'editor' | 'viewer'
```

## Testing

Tests live in `Tests/` at the src level, or co-located in `__tests__/` inside the module. Use Vitest and React Testing Library. Test behavior, not implementation.

```typescript
// Tests/Auth/SignInForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import { describe, it, expect, vi } from 'vitest'
import { Provider } from 'react-redux'
import { store } from 'Core/Redux/Store'
import SignInForm from 'Auth/Forms/SignInForm'

describe('SignInForm', () => {
  it('shows validation errors on empty submit', async () => {
    render(
      <Provider store={store}>
        <SignInForm />
      </Provider>
    )
    fireEvent.click(screen.getByRole('button', { name: /sign in/i }))
    await waitFor(() => {
      expect(screen.getByText(/required/i)).toBeInTheDocument()
    })
  })
})
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Hardcoded route strings | Always use `Config.urls.*` |
| API logic in components | Use RTK Query hooks, keep components thin |
| Redux for server state | Server state goes in RTK Query cache, not slices |
| Default export for hooks/utils | Named exports only for non-component files |
| Business logic in Views | Extract to custom hooks in `Hooks.ts` |
| Inline date formatting | Use Day.js helpers from Utils.ts |
| Imperative array transforms | Use Ramda pipelines |

## Quick Reference

| Concern | Location | Pattern |
|---------|----------|---------|
| Global state | `<Module>/Redux/index.ts` | `createSlice`, selectors as `select*` |
| Server data | `<Module>/Services/Api.ts` | `api.injectEndpoints` |
| URL paths | `Config/Paths.ts` | `Config.urls.*` |
| Router | `Core/Router.tsx` | `createBrowserRouter` + `RouterProvider` |
| Module routes | `<Module>/Router.tsx` | `RouteObject[]` named export, spread into parent `children` |
| Route protection | `Auth/Guards/PrivateRoute.tsx` | `makePrivate(element, { permFunction })` |
| Validation | `<Module>/Schemas/*Schema.ts` | Zod + `z.infer<>` |
| Page components | `<Module>/Views/*View.tsx` | Default export, connects Redux/RTK |
| UI components | `<Module>/Components/*.tsx` | Default export, props only |
| Business logic | `<Module>/Hooks.ts` | Named exports, `use*` prefix |
| Shared types | `<Module>/Types/index.ts` | Named exports |
| Date formatting | Day.js | `dayjs(date).format(...)` |
| Data transforms | Ramda | `pipe(filter, map, sort)` |
