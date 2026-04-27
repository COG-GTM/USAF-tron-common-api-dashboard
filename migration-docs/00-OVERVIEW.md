# React to Blazor Migration Overview

This folder contains all migration documentation and mapping artifacts for converting the TRON Common API Dashboard from React 17 + TypeScript to Blazor Web (.NET 9).

## Document Index

- **00-OVERVIEW.md** - This file, navigation hub ✅
- **01-POC-REQUIREMENTS.md** - POC success criteria and scope ✅
- **02-SERVICE-MAPPING.md** - React service → C# service mapping ✅
- **03-COMPONENT-MAPPING.md** - React component → Blazor component mapping ✅
- **04-ROUTE-MAPPING.md** - URL routes inventory ✅
- **05-STATE-MIGRATION.md** - Hookstate → Blazor state patterns ✅
- **06-VALIDATION-RULES.md** - Validation logic extraction ✅
- **07-API-CLIENT-SETUP.md** - NSwag configuration ✅
- **08-TEST-STRATEGY.md** - Test plan details ✅
- **POC-QUICKSTART.md** - 3-week POC implementation guide ✅

## Quick Reference

### Current Application Stats
- **Pages**: 17 (6 in DocumentSpace alone)
- **State Modules**: 22 domain services
- **Service Classes**: 33 service files
- **Components**: 41 reusable components
- **Routes**: 20+ with nested children
- **Test Files**: 20+ Cypress tests, multiple Jest tests

### Migration Approach
**Proof-of-Concept First** → Incremental Migration → Full Cutover

### POC Target
**DashboardUser CRUD Page** - Full CRUD operations with grid, forms, validation, authorization

### Technology Stack
- **Framework**: Blazor Server (.NET 9)
- **UI Library**: MudBlazor + Radzen DataGrid
- **API Client**: NSwag-generated from OpenAPI spec
- **Testing**: xUnit + bUnit + Playwright

## Migration Status Tracking

### Phase 0: Foundation & POC 🚀 READY TO START
- [ ] Project setup (See POC-QUICKSTART.md)
- [ ] NSwag API client generation (See 07-API-CLIENT-SETUP.md)
- [ ] Authentication infrastructure (See POC-QUICKSTART.md)
- [ ] Base components (See 03-COMPONENT-MAPPING.md)
- [ ] DashboardUser POC (See 01-POC-REQUIREMENTS.md)
- [ ] POC testing (See 08-TEST-STRATEGY.md)
- [ ] Go/No-Go decision

### Phase 1: Core Pages
- [ ] 7 core pages migrated
- [ ] Service layer ported
- [ ] Testing infrastructure

### Phase 2: Advanced Features
- [ ] 6 advanced pages
- [ ] Charts integration

### Phase 3: Document Space
- [ ] 4 main pages
- [ ] 20+ child components

### Phase 4: Metrics
- [ ] AppSource metrics pages

### Phase 5: Testing & Quality
- [ ] Test coverage goals met

### Phase 6: Deployment
- [ ] Production cutover

## Key Contacts & Resources

- **Migration Plan**: `~/.windsurf/plans/blazor-migration-plan-273994.md`
- **React Source**: `src/` directory
- **Blazor Target**: `TronDashboard.Blazor/` (to be created)
- **OpenAPI Spec**: `resources/tron-common-api.json`

## Critical Decisions Log

| Decision | Status | Resolution | Date |
|----------|--------|------------|------|
| Blazor rendering mode | 🟡 Pending | TBD - Recommend Server | - |
| UI component library | 🟡 Pending | TBD - Recommend MudBlazor | - |
| Authentication mechanism | 🟡 Pending | TBD - Need backend clarification | - |
| OpenAPI generator | 🟢 Decided | NSwag | - |
| State management pattern | 🟢 Decided | Custom services (matches current) | - |
| Deployment strategy | 🟢 Decided | Side-by-side dual-run | - |

## Getting Started

1. Review this overview
2. Read POC requirements (01-POC-REQUIREMENTS.md)
3. Review service mapping (02-SERVICE-MAPPING.md)
4. Set up development environment
5. Begin POC implementation

---

Last Updated: 2026-04-27
