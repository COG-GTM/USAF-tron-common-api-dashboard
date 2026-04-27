---
description: Frontend development workflow for TRON Common API Dashboard
---

# Frontend Development Skills

## 1. Local Dev Loop

**Install & Start:**
```bash
npm install
npm start
```
Runs on `http://localhost:3000` with hot-reload enabled (React Scripts).

**Point at Different Backend:**

Edit `.env` (copy from `.env.development.example`):
```bash
REACT_APP_API_BASE_URL=http://localhost:9000/  # Local backend
# OR
REACT_APP_API_BASE_URL=https://tron-common-api.staging.dso.mil/  # Remote
REACT_APP_API_VERSION_PREFIX=v1
REACT_APP_API_VERSION_2_PREFIX=v2
REACT_APP_API_PATH_PREFIX=api
REACT_APP_TEST_USER_TOKEN=  # Optional: bypass auth locally
```

Backend URL construction: `src/api/config.ts`

---

## 2. Adding a New Page

**Trace: Person Page Example**

1. **Component**: `src/pages/Person/PersonPage.tsx`
   - Uses `<DataCrudFormPage>` wrapper with grid columns + edit form
   - Invokes `usePersonState()` hook for data/API

2. **Route Registration**: `src/routes.ts`
   - Add enum: `RoutePath.PERSON = '/person'`
   - Add route config with `component`, `requiredPrivileges`
   - Can nest under `childRoutes` for sidebar grouping

3. **State Hook**: `src/state/person/person-state.ts`
   - Creates global state with `@hookstate/core`
   - Instantiates API client from `src/openapi/apis`
   - Wraps in service (`PersonService`)

4. **Service**: `src/state/person/person-service.ts` (assumed)
   - Extends base data service
   - Implements CRUD operations calling API methods

5. **Wire in App.tsx**: `src/App.tsx`
   - Routes auto-loaded from `routes` array
   - Wrapped in `<ProtectedRoute>` with privilege checks

---

## 3. Adding a New Form

**Pattern: Hookstate + Validation**

Reference: `src/pages/Person/PersonEditForm.tsx`

**Setup:**
```tsx
import { useState } from '@hookstate/core';
import { Validation } from '@hookstate/validation';
import { Touched } from '@hookstate/touched';
import { Initial } from '@hookstate/initial';

const formState = useState({ ...props.data });
formState.attach(Validation);
formState.attach(Initial);
formState.attach(Touched);
```

**Validation:**
```tsx
Validation(formState.firstName).validate(
  validateRequiredString, 
  validationErrors.requiredText, 
  'error'
);
```
Validators in: `src/utils/validation-utils.ts`

**Field Binding:**
```tsx
<TextInput 
  id="firstName"
  defaultValue={props.data?.firstName || ''}
  error={failsHookstateValidation(formState.firstName)}
  onChange={(event) => formState.firstName.set(event.target.value)}
/>
```

**Submit:**
```tsx
const submitForm = (event: FormEvent<HTMLFormElement>) => {
  event.preventDefault();
  props.onSubmit(formState.get());  // Extract plain object
}
```

**Check Modified:**
```tsx
isFormModified(): boolean {
  return Initial(formState).modified();
}
```

**Complex State (OrganizationEditForm):**
- Use nested state for arrays/related entities
- `useHookstate<T[]>([])` for multi-select lists
- `formState.members.merge(newMembers)` to append
- `formState.members.find(...).set(none)` to remove

---

## 4. Regenerating API Client

**Command:**
```bash
npm run generate-api-client
```

**Steps (automated):**
1. Clears `src/openapi/` 
2. Runs OpenAPI Generator on `resources/tron-common-api.json`
3. Generates TS models + API clients (axios-based)
4. Creates JSON schema guards
5. Post-processes output

**Update OpenAPI Spec:**
1. Run Common API backend locally (port 8088 dev / 8080 prod)
2. Fetch: `http://localhost:8088/api/api-docs/dashboard-api-v2`
3. Save JSON to `resources/tron-common-api.json`
4. Run `npm run generate-api-client`

**Generated Files:**
- `src/openapi/models/` — DTOs
- `src/openapi/apis/` — Controller API clients
- `src/api/model-types.json` — JSON schemas

---

## 5. Common Gotchas

**Auth Flow:**
- Local dev: Set `REACT_APP_TEST_USER_TOKEN` in `.env` to bypass
- Prod: Expects JWT in request headers via `globalOpenapiConfig.axios` interceptor
- User state: `src/state/authorized-user/authorized-user-state.ts`
- Privileges checked in routes (`requiredPrivileges`) & components

**Env Vars:**
- Must prefix with `REACT_APP_` to be accessible
- Restart dev server after `.env` changes
- Not tracked in git (only `.env.development.example`)

**CORS:**
- Backend must allow `http://localhost:3000` origin
- If testing remote backend, ensure CORS headers permit it
- Common issue: credential/cookie mismatch

**State Mutation:**
- Never mutate Hookstate directly: `state.value = x` ❌
- Use `.set()`, `.merge()`, `.set(none)` ✅
- For arrays: `state[index].set(value)` or `state.merge([item])`

**Grid Data:**
- AG Grid requires `rowData` as plain objects
- Use `state.attach(Downgraded).get()` to strip Hookstate proxy

**Hot Reload:**
- Sometimes breaks on deep state changes
- Refresh browser if HMR fails

---

## 6. Useful One-Liners

**Find component usage:**
```bash
grep -r "PersonEditForm" src/
```

**Find all pages:**
```bash
fd -e tsx "*Page.tsx" src/pages/
```

**Find all state hooks:**
```bash
fd -e ts "*-state.ts" src/state/
```

**Find validation utilities:**
```bash
grep -r "validateRequiredString\|validateEmail" src/utils/
```

**Run tests:**
```bash
npm test                    # Interactive
npm run test:unit           # CI mode with coverage
```

**Lint:**
```bash
npm run lint                # Check
npm run lint:fix            # Auto-fix
```

**Search for Hookstate usage:**
```bash
grep -r "useHookstate\|useState" src/ --include="*.tsx"
```

**Find DTOs:**
```bash
ls src/openapi/models/ | grep -i "dto"
```

**Check env vars in use:**
```bash
grep -r "process.env.REACT_APP" src/
```

**Clear node_modules & reinstall:**
```bash
rm -rf node_modules package-lock.json && npm install
```

**Cypress E2E tests:**
```bash
npm run cypress:open        # Interactive
npm run test:e2e-local      # Headless vs localhost
```

---

## Quick Reference Paths

| Concern | Path |
|---------|------|
| Routes | `src/routes.ts` |
| API Config | `src/api/config.ts`, `src/api/openapi-config.ts` |
| OpenAPI Spec | `resources/tron-common-api.json` |
| Generated Client | `src/openapi/` |
| State Hooks | `src/state/*/` |
| Validation | `src/utils/validation-utils.ts` |
| Env Example | `.env.development.example` |
| Entry Point | `src/index.tsx`, `src/App.tsx` |
