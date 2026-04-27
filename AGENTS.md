# TRON Common API Dashboard – Agent Guide

## 1. What this app is

This repository contains the React frontend dashboard for the tron-common-api backend. It provides administrative and operational views over app clients, app sources, document spaces, metrics, logs, KPIs, and related data exposed by the tron-common-api service.

## 2. Tech stack

- **React**: `react@^17.0.1`, `react-dom@^17.0.1` (from `package.json`)
- **TypeScript**: `typescript@^4.1.3`
- **State management**: `@hookstate/core@^3.0.6` (+ `@hookstate/initial@^3.0.0`, `@hookstate/touched@^3.0.0`, `@hookstate/validation@^3.0.0`)
- **Router**: `react-router-dom@^5.2.0`
- **HTTP client**: `axios@^0.21.4`
- **OpenAPI client generator**: `@openapitools/openapi-generator-cli@^2.1.22` with `typescript-axios` generator (see `package.json` scripts and `resources/tron-common-api.json`)
- **UI libraries**: `@trussworks/react-uswds@^1.12.2`, `bootstrap@^4.5.3`, `semantic-ui-react@^2.0.4`, `semantic-ui-css@^2.4.1`, `bootstrap-icons@^1.3.0`, `uswds@^2.8.0`, `react-modal@^3.12.1`, `react-toastify@^7.0.4`
- **Charts and data viz**: `apexcharts@^3.26.0`, `react-apexcharts@^1.3.9`, `@visx/*@^2.1.x`, `d3-array@^3.0.4`
- **Grid**: `ag-grid-community@^25.0.1`, `ag-grid-react@^25.0.1`
- **Testing (unit/integration)**: `@testing-library/react@^11.2.3`, `@testing-library/jest-dom@^5.11.9`, `@testing-library/user-event@^12.6.0`, `jest` (via `react-scripts@4.0.3`), `msw@^0.26.1`, `chai@^4.2.0`
- **Testing (e2e)**: `cypress@^7.6.0`, `cypress-file-upload@^5.0.8`
- **Build tool / dev server**: `react-scripts@4.0.3` (Create React App)

## 3. Repo layout (`src/` to two levels)

- **`src/App.tsx`** – Top-level app component and routing wrapper.
- **`src/App.scss`** – Global app styles.
- **`src/App.test.tsx`** – App-level tests.
- **`src/api/`** – API utilities and health/logfile wrappers.
  - **`src/api/config.ts`** – Environment-based API URL configuration.
  - **`src/api/http-client.ts`** – Base Axios HTTP client.
  - **`src/api/health/`** – Health-check API wrappers.
  - **`src/api/logfile/`** – Logfile API wrappers.
- **`src/api.json`** – Processed OpenAPI metadata.
- **`src/components/`** – Reusable presentational and layout components.
  - **`src/components/Accordion/`** – Accordion UI.
  - **`src/components/ApiSpecCellRenderer/`** – API spec cell renderer components.
  - **`src/components/AppInfoTag/`** – App info tag display.
  - **`src/components/BackdropOverlay/`** – Backdrop overlay UI.
  - **`src/components/BreadCrumbTrail/`** – Breadcrumb navigation.
  - **`src/components/Button/`** – Button components.
  - **`src/components/Card/`** – Card layout components.
  - **`src/components/Chart/`** – Chart wrapper components.
  - **`src/components/CheckboxCellRenderer/`** – Grid checkbox cell renderer.
  - **`src/components/ComboBoxCellRenderer/`** – Grid combo box cell renderer.
  - **`src/components/CopyToClipboard/`** – Copy-to-clipboard helpers.
  - **`src/components/DataCrudFormPage/`** – Generic CRUD form page components and tests.
  - **`src/components/DeleteCellRenderer/`** – Grid delete action cell renderer.
  - **`src/components/DocSpaceItemRenderer/`** – Document space item renderers.
  - **`src/components/DocumentRowActionCellRenderer/`** – Document row action renderer.
  - **`src/components/DropDown/`** – Dropdown control.
  - **`src/components/ErrorBoundary/`** – Error boundary component.
  - **`src/components/GenericDialog/`** – Generic modal dialog.
  - **`src/components/Grid/`** – AG Grid-related components/helpers.
  - **`src/components/HeaderUserInfo/`** – Header user info and editor components.
  - **`src/components/InfoNotice/`** – Informational notices.
  - **`src/components/ItemChooser/`** – Item chooser UI.
  - **`src/components/KpiMiniChart/`** – KPI mini chart components.
  - **`src/components/LoadingCellRenderer/`** – Loading cell renderers.
  - **`src/components/MetricCellRenderer/`** – Metric cell renderers.
  - **`src/components/MiniChart/`** – Small chart components.
  - **`src/components/Modal/`** – Modal dialog components.
  - **`src/components/NestedSidebarNav/`** – Nested sidebar navigation.
  - **`src/components/PageFormat/`** – Page layout wrapper components.
  - **`src/components/PageTitle/`** – Page title header components.
  - **`src/components/PrivilegeCellRenderer/`** – Privilege cell renderers.
  - **`src/components/ProtectedRoute/`** – Auth-aware route wrapper.
  - **`src/components/SideDrawer/`** – Side drawer UI.
  - **`src/components/Sidebar/`** – Sidebar navigation components.
  - **`src/components/Spinner/`** – Loading spinner.
  - **`src/components/StatusCard/`** – Status/KPI cards.
  - **`src/components/TabBar/`** – Tab bar UI.
  - **`src/components/Toast/`** – Toast notification components.
  - **`src/components/UnusedEndpointCellRenderer/`** – Unused endpoint cell renderer.
  - **`src/components/documentspace/`** – Document-space-specific components (actions, upload, etc.).
  - **`src/components/forms/`** – Shared form components and helpers.
- **`src/containers/`** – Container components.
  - **`src/containers/HeaderUserInfoContainer.tsx`** – Header user info container.
- **`src/hocs/`** – Higher-order components.
  - **`src/hocs/UseLoading/`** – Loading HOC (`WithLoading`).
- **`src/hooks/`** – Custom hooks.
- **`src/icons/`** – Icon components used in routes/navigation.
- **`src/index.tsx`** – React entry point, wraps `App` in `BrowserRouter`.
- **`src/pages/`** – Route-level pages grouped by domain.
  - **`src/pages/ApiTest/`** – API test page and tests.
  - **`src/pages/AppClient/`** – App client listing, forms, and tests.
  - **`src/pages/AppSource/`** – App source management and metrics pages.
  - **`src/pages/AuditLog/`** – Audit log listing and details.
  - **`src/pages/DashboardUser/`** – Dashboard user management.
  - **`src/pages/DocumentSpace/`** – Document space pages (spaces, recents, favorites, memberships, etc.).
  - **`src/pages/Health/`** – Health check page.
  - **`src/pages/Home/`** – Home dashboard page.
  - **`src/pages/Kpi/`** – KPI pages.
  - **`src/pages/Logfile/`** – Log file viewer pages.
  - **`src/pages/MyDigitizeApps/`** – My Digitize Apps pages.
  - **`src/pages/NotAuthorized/`** – Not-authorized page.
  - **`src/pages/NotFound/`** – 404 page.
  - **`src/pages/Organization/`** – Organization pages.
  - **`src/pages/Person/`** – Person pages.
  - **`src/pages/PubSub/`** – Pub/Sub pages.
  - **`src/pages/ScratchStorage/`** – Scratch storage pages.
- **`src/openapi/`** – Generated OpenAPI TypeScript client (models, apis, configuration).
- **`src/react-app-env.d.ts`** – CRA React/TS environment declarations.
- **`src/reportWebVitals.ts`** – Web Vitals reporting helper.
- **`src/routes.ts`** – Central route and navigation configuration.
- **`src/setupTests.ts`** – Jest and MSW setup for tests.
- **`src/state/`** – Global state modules and services (Hookstate-based).
  - **`src/state/app-clients/`** – App client state and service.
  - **`src/state/app-info/`** – App info state.
  - **`src/state/app-source/`** – App source state and service.
  - **`src/state/audit-log/`** – Audit log state.
  - **`src/state/authorized-user/`** – Authorized user state and service.
  - **`src/state/crud-page/`** – Generic CRUD page state helpers.
  - **`src/state/dashboard-user/`** – Dashboard user state.
  - **`src/state/data-service/`** – Generic data service abstractions.
  - **`src/state/document-space/`** – Document space state and services.
  - **`src/state/event-request-log/`** – Event request log state.
  - **`src/state/global-service/`** – Abstract global state service helpers.
  - **`src/state/health/`** – Health state.
  - **`src/state/kpi/`** – KPI state.
  - **`src/state/logfile/`** – Logfile state.
  - **`src/state/metrics/`** – Metrics-related state.
  - **`src/state/my-digitize-apps/`** – My Digitize Apps state.
  - **`src/state/organization/`** – Organization state.
  - **`src/state/person/`** – Person state.
  - **`src/state/privilege/`** – Privilege state and types.
  - **`src/state/pub-sub/`** – Pub/Sub state.
  - **`src/state/scratch-storage/`** – Scratch storage state.
  - **`src/state/user/`** – User info state.
- **`src/styles/`** – Shared SCSS/CSS styles.
- **`src/utils/`** – Utility functions (validation, document space helpers, etc.).

## 4. Routing map (top-level routes)

Routing is configured in `src/routes.ts` and rendered via `ProtectedRoute` in `src/App.tsx`. The top-level (non-nested) routes and their page components are:

| Route path | Route key | Component (from `src/pages/...`) |
| --- | --- | --- |
| `/` | `RoutePath.HOME` | `HomePage` (`src/pages/Home/HomePage.tsx`) |
| `/health` | `RoutePath.HEALTH` | `HealthPage` (`src/pages/Health/HealthPage.tsx`) |
| `/person` | `RoutePath.PERSON` | `PersonPage` (`src/pages/Person/PersonPage.tsx`) |
| `/app-clients` | `RoutePath.APP_CLIENT` | `AppClientPage` (`src/pages/AppClient/AppClientPage.tsx`) |
| `/organization` | `RoutePath.ORGANIZATION` | `OrganizationPage` (`src/pages/Organization/OrganizationPage.tsx`) |
| `/logfile` | `RoutePath.LOGFILE` | `LogfilePage` (`src/pages/Logfile/LogfilePage.tsx`) |
| `/dashboard-user` | `RoutePath.DASHBOARD_USER` | `DashboardUserPage` (`src/pages/DashboardUser/DashboardUserPage.tsx`) |
| `/scratch-storage` | `RoutePath.SCRATCH_STORAGE` | `ScratchStoragePage` (`src/pages/ScratchStorage/ScratchStoragePage.tsx`) |
| `/app-source` | `RoutePath.APP_SOURCE` | `AppSourcePage` (`src/pages/AppSource/AppSourcePage.tsx`) |
| `/digitize-apps` | `RoutePath.MY_DIGITIZE_APPS` | `MyDigitizeAppsPage` (`src/pages/MyDigitizeApps/MyDigitizeAppsPage.tsx`) |
| `/pubsub` | `RoutePath.PUB_SUB` | `PubSubPage` (`src/pages/PubSub/PubSubPage.tsx`) |
| `/audit-log` | `RoutePath.AUDIT_LOG` | `AuditLogPage` (`src/pages/AuditLog/AuditLogPage.tsx`) |
| `/kpi` | `RoutePath.KPI` | `KpiPage` (`src/pages/Kpi/KpiPage.tsx`) |
| `/document-space/spaces` | `RoutePath.DOCUMENT_SPACE_SPACES` | `DocumentSpacePage` (`src/pages/DocumentSpace/DocumentSpacePage.tsx`) |
| `/document-space/recents` | `RoutePath.DOCUMENT_SPACE_RECENTS` | `DocumentSpaceRecentsPage` (`src/pages/DocumentSpace/Recents/DocumentSpaceRecentsPage.tsx`) |
| `/document-space/favorites` | `RoutePath.DOCUMENT_SPACE_FAVORITES` | `DocumentSpaceFavoritesPage` (`src/pages/DocumentSpace/Favorites/DocumentSpaceFavoritesPage.tsx`) |
| `/document-space/archived` | `RoutePath.DOCUMENT_SPACE_ARCHIVED` | `DocumentSpaceArchivedItemsPage` (`src/pages/DocumentSpace/DocumentSpaceArchivedItemsPage.tsx`) |
| `/app-source/:id/metrics/:type/:name/:method?` | `RoutePath.APP_SOURCE_METRIC` | `MetricPageProtectedWrapper` (`src/pages/AppSource/Metrics/MetricPageProtectedWrapper.tsx`) |
| `/app-api/:apiId` | `RoutePath.API_TEST` | `ApiTestPage` (`src/pages/ApiTest/ApiTestPage.tsx`) |
| `/not-authorized` | `RoutePath.NOT_AUTHORIZED` | `NotAuthorizedPage` (`src/pages/NotAuthorized/NotAuthorizedPage.tsx`) |
| (fallback) | `RoutePath.NOT_FOUND` / default | `NotFoundPage` (`src/pages/NotFound/NotFoundPage.tsx`) |

## 5. State management (Hookstate)

Global state is implemented with Hookstate atoms created via `createState` and accessed via typed service classes plus `useState` wrappers.

A canonical example is **Document Space state** in `src/state/document-space/document-space-state.ts`:

- **Global atoms** are defined at module scope, e.g. `spacesState`, `privilegeState`, `clipBoardState`, `recentsPageState`, `spacesPageState`, `membershipsPageState`, and `globalDocumentSpaceState` using `createState` from `@hookstate/core`.
- **Service wrappers** such as `DocumentSpaceService`, `DocumentSpaceMembershipService`, `DocumentSpacePrivilegeService`, `RecentsPageService`, `SpacesPageService`, and `DocumentSpaceGlobalService` encapsulate business logic and API calls.
- **Accessors** follow a consistent pattern:
  - `useXxxState` returns a service instance configured with `useState(...)` for React components.
  - `accessXxxState` returns a service instance using the raw global state for non-component code.

For example (from `src/state/document-space/document-space-state.ts`):

- `useDocumentSpaceState()` and `accessDocumentSpaceState()` wrap `spacesState` and an injected `DocumentSpaceControllerApi` created with the global OpenAPI configuration.
- `useDocumentSpacePageState(...)` composes multiple Hookstate atoms (`spacesPageState`, `globalDocumentSpaceState`, `spacesState`, `privilegeState`, `clipBoardState`) and service classes into a single page-level service.

Other domains (e.g. `src/state/app-clients/app-clients-state.ts`, `src/state/organization/organization-state.ts`, `src/state/authorized-user/authorized-user-state.ts`) follow this same pattern: domain-specific Hookstate atoms + a domain service class + `useXxxState` / `accessXxxState` accessors.

## 6. API integration (OpenAPI-generated client)

The OpenAPI-generated client is produced from `resources/tron-common-api.json` via the `generate-api-client` npm script, which writes TypeScript into `src/openapi/`.

**Configuration and base URL**

- `src/api/config.ts` reads environment variables:
  - `REACT_APP_API_BASE_URL`
  - `REACT_APP_API_PATH_PREFIX`
  - `REACT_APP_API_VERSION_PREFIX`
  - `REACT_APP_API_VERSION_2_PREFIX`
  - `REACT_APP_TEST_USER_TOKEN` (for test token access)
- It exposes:
  - `Config.API_URL` (v1) – `API_BASE_URL + API_PATH_PREFIX + "/" + API_VERSION_PREFIX + "/"`
  - `Config.API_URL_V2` – `API_BASE_URL + API_PATH_PREFIX + "/" + API_VERSION_2_PREFIX + "/"`
  - `Config.ACTUATOR_URL` – `API_BASE_URL + API_PATH_PREFIX + "/actuator/"`

**OpenAPI client wiring**

- `src/api/openapi-config.ts` (not yet inspected here) defines `globalOpenapiConfig`, which is imported in state modules such as `src/state/document-space/document-space-state.ts` and passed into generated API classes like `DocumentSpaceControllerApi` from `src/openapi`.
- In `src/state/document-space/document-space-state.ts`, `globalOpenapiConfig` provides the configuration and basePath used when instantiating `DocumentSpaceControllerApi`, ensuring all Document Space operations use the centralized OpenAPI/axios configuration.

**Axios instance and auth headers**

- `src/api/http-client.ts` defines an abstract `HttpClient` that wraps an `AxiosInstance` created with a given `baseURL` and attaches response interceptors from `src/api/openapi-config`:
  - `axiosAuthSuccessResponseInterceptor`
  - `axiosAuthErrorResponseInterceptor`
- These interceptors are responsible for global auth handling (e.g. attaching/refreshing auth tokens, handling auth errors). They are applied to the axios instance used by generated API classes or custom API wrappers.
- In tests, `src/setupTests.ts` mocks `globalOpenapiConfig` to return a `Configuration` from `src/openapi/configuration.ts` and uses `msw` (`setupServer`, `rest`) to stub backend endpoints such as `/api/v2/userinfo`, `/api/v2/document-space/...`, `/api/v2/privilege`, and others.

Overall, the app talks to the tron-common-api backend via the generated OpenAPI client in `src/openapi/`, configured by `globalOpenapiConfig` and the environment-driven URLs in `src/api/config.ts`, with axios interceptors from `src/api/openapi-config.ts` handling auth concerns.

## 7. How to run it locally

From `README.md` (`/Users/jakecosme/Documents/GitHub/USAF-tron-common-api-dashboard/README.md`) and `package.json`:

- **Node version**: Node v16 or later is recommended.
- **Install dependencies**:

  ```bash
  npm install
  ```

- **Run the development server** (Create React App, `react-scripts start`):

  ```bash
  npm start
  ```

- **Default port**: CRA defaults to `http://localhost:3000`.

- **Backend communication**:
  - The frontend expects the tron-common-api backend to be running and exposes an OpenAPI spec used to generate the client (`resources/tron-common-api.json`).
  - API base URLs and paths are configured via environment variables read in `src/api/config.ts`:
    - `REACT_APP_API_BASE_URL`
    - `REACT_APP_API_PATH_PREFIX`
    - `REACT_APP_API_VERSION_PREFIX`
    - `REACT_APP_API_VERSION_2_PREFIX`
  - For local development, set these in `.env` or `.env.development` at the repo root (e.g. `REACT_APP_API_BASE_URL=http://localhost:8088` as implied by the README’s example URL `http://localhost:8088/api/api-docs/dashboard-api-v2`).

- **Generating/updating the OpenAPI client** (when the tron-common-api spec changes):
  - Update `resources/tron-common-api.json` using the backend’s `/api/api-docs/dashboard-api-v2` endpoint (see `README.md`).
  - Run:

    ```bash
    npm run generate-api-client
    ```

  - This runs the following scripts from `package.json`:
    - `generate-api-client:code` – uses `openapi-generator-cli` to generate `src/openapi`.
    - `generate-api-client:guards` / `generate-api-client:guards-windows` – generates JSON schema guards into `src/api/model-types.json`.
    - `generate-api-client:post` – post-processes the JSON via `process-json.js`.

## 8. How to run the tests

**Unit / integration tests (Jest + React Testing Library)**

- General test command (from `package.json`):

  ```bash
  npm test
  ```

  This runs `npx react-scripts test` in watch mode.

- CI-style/unit tests with coverage (non-watch):

  ```bash
  npm run test:unit
  ```

  This runs `react-scripts test --coverage --watchAll=false --runInBand` after ensuring dependencies are installed.

- Test setup:
  - `src/setupTests.ts` configures Jest DOM, MSW (`setupServer`), and mocks `globalOpenapiConfig` for the OpenAPI client.
  - Tests live co-located with components and pages in `__tests__` directories under `src/components/`, `src/pages/`, and in root-level files such as `src/App.test.tsx`.

**End-to-end tests (Cypress)**

- Open Cypress UI against local dev server (configured with `cypress-all.json` and local env vars):

  ```bash
  npm run cypress:open
  ```

- Run local e2e tests (headless, minimal env):

  ```bash
  npm run test:e2e-local
  ```

- Run CI-style e2e tests against local backend endpoints with full config file:

  ```bash
  npm run test:e2e-ci-local
  ```

- Run CI-style e2e tests against staging backend (`https://tron-common-api.staging.dso.mil`):

  ```bash
  npm run test:e2e-ci
  ```

Cypress integration tests are located under `cypress/integration/` and use helpers from `cypress/support/`.

## 9. Conventions an agent should follow

- **Component and page structure**
  - Route-level screens live under `src/pages/<Domain>/` (e.g. `src/pages/DocumentSpace/DocumentSpacePage.tsx`).
  - Reusable components live under `src/components/<ComponentName>/` with related tests in `__tests__` subdirectories (e.g. `src/components/Sidebar/Sidebar.tsx`, `src/components/Sidebar/__tests__/Sidebar.test.tsx`).
  - Global state for a domain lives under `src/state/<domain>/` split into `*-state.ts` (Hookstate atoms and accessors) and `*-service.ts` (business logic and API calls), e.g. `src/state/document-space/document-space-state.ts` and `src/state/document-space/document-space-service.ts`.

- **Naming conventions**
  - React components use PascalCase filenames and exports, e.g. `HomePage`, `DocumentSpacePage`, `ProtectedRoute`.
  - Hookstate accessors follow `useXxxState` / `accessXxxState` naming, where `useXxxState` is hook-safe (uses `useState`) and `accessXxxState` is non-hook global access.
  - Services are suffixed with `Service`, e.g. `DocumentSpaceService`, `AuthorizedUserService`.
  - Route paths and keys are centralized in the `RoutePath` enum and `routes` array in `src/routes.ts`.

- **Styling approach**
  - Global styles in `src/App.scss`, plus additional SCSS/CSS in `src/styles/`.
  - Bootstrap and Semantic UI are imported globally in `src/App.tsx`:
    - `import 'bootstrap/dist/css/bootstrap.min.css';`
    - `import 'semantic-ui-css/semantic.min.css';`
  - Components generally use className-based styling in combination with these frameworks.

- **Forms and CRUD patterns**
  - Many domain pages use shared CRUD components from `src/components/DataCrudFormPage/`.
  - Hookstate is used to manage form state (fields, touched/validation) alongside the Hookstate plugins `@hookstate/initial`, `@hookstate/touched`, and `@hookstate/validation` (see various form components under `src/components/forms/` and state modules under `src/state/*`).
  - Validation utilities and common form helpers are centralized under `src/utils/` (e.g. `src/utils/validation-utils.ts`).

- **Routing and authorization**
  - All protected routes should be registered in `src/routes.ts` and rendered via `ProtectedRoute` in `src/App.tsx`.
  - Route entries specify `requiredPrivileges: PrivilegeType[]` from `src/state/privilege/privilege-type.ts`.
  - When adding new routes, follow the existing pattern: add an entry to `RoutePath`, create the page under `src/pages/<Domain>/`, add a `RouteItem` with `requiredPrivileges`, and use a domain-specific icon from `src/icons/` if it should appear in the sidebar.

- **API usage**
  - Prefer using generated OpenAPI API classes from `src/openapi` (e.g. `DocumentSpaceControllerApi`) instead of ad-hoc axios calls.
  - Instantiate OpenAPI API classes using `globalOpenapiConfig` from `src/api/openapi-config.ts` so they share base URL, configuration, and axios instance/interceptors.
  - Domain services (e.g. `src/state/document-space/document-space-service.ts`) are the place to call into these OpenAPI clients; components should use the service methods exposed via Hookstate accessors instead of calling APIs directly.

Following these patterns will keep new code consistent with the existing tron-common-api dashboard frontend and ensure that routing, state, styling, and API calls behave as expected.
