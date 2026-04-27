# Migration Documentation

Complete documentation for migrating TRON Common API Dashboard from React + TypeScript to Blazor Web (.NET 9).

## 🚀 Quick Start

**New to this migration?** Start here:

1. Read **00-OVERVIEW.md** for the big picture
2. Review **01-POC-REQUIREMENTS.md** to understand the POC scope
3. Follow **POC-QUICKSTART.md** for step-by-step 3-week implementation

## 📚 Documentation Structure

### Planning Documents
- **`00-OVERVIEW.md`** - Executive summary, navigation, status tracking
- **`01-POC-REQUIREMENTS.md`** - POC definition, success criteria, scope

### Technical Mapping Documents
- **`02-SERVICE-MAPPING.md`** - Maps 33 TypeScript services → C# services
- **`03-COMPONENT-MAPPING.md`** - Maps 41 React components → Blazor components
- **`04-ROUTE-MAPPING.md`** - Maps 20+ routes + authorization policies
- **`05-STATE-MIGRATION.md`** - Hookstate → Blazor state patterns
- **`06-VALIDATION-RULES.md`** - Extracts 9 validation functions → DataAnnotations
- **`07-API-CLIENT-SETUP.md`** - NSwag configuration for OpenAPI client generation
- **`08-TEST-STRATEGY.md`** - Jest/Cypress → xUnit/bUnit/Playwright migration

### Implementation Guide
- **`POC-QUICKSTART.md`** - Day-by-day 3-week POC implementation guide

## 🎯 POC Overview

**Target:** DashboardUser CRUD page (full CRUD operations)

**Timeline:** 3 weeks
- Week 1: Foundation & infrastructure
- Week 2: UI implementation
- Week 3: Testing & review

**Success Criteria:**
- ✅ Full CRUD operations work
- ✅ 80%+ service test coverage
- ✅ 50%+ component test coverage
- ✅ 1 E2E test passes
- ✅ Team consensus: approach is viable

## 🛠️ Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Framework | Blazor Server (.NET 9) | Web UI framework |
| UI Components | MudBlazor 7.x | Component library |
| Data Grid | Radzen DataGrid | Grid component |
| API Client | NSwag 14.x | OpenAPI C# client generator |
| State Management | Scoped Services | Per-user session state |
| Validation | DataAnnotations | Form validation |
| Unit Testing | xUnit 2.x | Test framework |
| Component Testing | bUnit 1.x | Blazor component tests |
| E2E Testing | Playwright for .NET | End-to-end tests |
| Mocking | Moq 4.x | Test mocking |

## 📊 Migration Scope

### Current Application
- 17 pages (6 in DocumentSpace alone)
- 22 domain state modules
- 33 service files
- 41 reusable components
- 20+ routes with nested children
- 20+ Cypress E2E tests

### Migration Phases
1. **POC** (3 weeks) - DashboardUser page
2. **Phase 1** (5 weeks) - 7 core pages
3. **Phase 2** (4 weeks) - 6 advanced pages
4. **Phase 3** (4 weeks) - Document Space (complex)
5. **Phase 4** (2 weeks) - Metrics & charts
6. **Phase 5** (2 weeks) - Testing & quality
7. **Phase 6** (2 weeks) - Deployment & cutover

**Total Estimated Timeline:** 22 weeks (5.5 months)

## 🔑 Key Decisions Needed

Before starting POC, decide:

| Decision | Options | Recommendation |
|----------|---------|----------------|
| Blazor rendering mode | Server / WebAssembly / Auto | **Server** (simpler, smaller bundle) |
| UI component library | MudBlazor / Radzen / Other | **MudBlazor** (comprehensive, free) |
| OpenAPI generator | NSwag / Kiota | **NSwag** (mature, proven) |
| State management | Services / Fluxor / Other | **Custom Services** (matches current pattern) |
| Authentication | Determine backend mechanism | **TBD** - Need backend team input |

## 📂 Folder Structure After Migration

```
USAF-tron-common-api-dashboard/
├── src/                         # Existing React app (unchanged during POC)
├── resources/                    # OpenAPI spec (shared)
│   └── tron-common-api.json
├── migration-docs/              # This folder
│   ├── 00-OVERVIEW.md
│   ├── ... (8 mapping docs)
│   └── POC-QUICKSTART.md
└── TronDashboard.Blazor/        # New Blazor solution
    ├── TronDashboard.sln
    ├── TronDashboard.Web/       # Blazor Server project
    │   ├── Pages/
    │   │   └── DashboardUser/
    │   ├── Components/
    │   └── Program.cs
    ├── TronDashboard.Core/      # Shared business logic
    │   ├── Services/
    │   ├── Models/
    │   ├── ApiClient/           # Generated from OpenAPI
    │   └── Validation/
    ├── TronDashboard.Tests/     # xUnit + bUnit tests
    └── TronDashboard.E2E/       # Playwright E2E tests
```

## 🚦 Migration Status

### POC Phase (Weeks 1-3)
- [x] Migration documentation complete
- [ ] Blazor solution created
- [ ] NSwag API client generated
- [ ] Authentication infrastructure
- [ ] Base components built
- [ ] DashboardUser page implemented
- [ ] Tests written and passing
- [ ] Go/No-Go decision

### Phase 1-6
- [ ] Not started (pending POC completion)

## ⚠️ Critical Risks

1. **Grid Feature Parity** - AG Grid → Blazor grid may lack features
   - **Mitigation:** POC validates grid capabilities early
   
2. **File Upload/Download** - Document Space complexity
   - **Mitigation:** Defer to Phase 3, spike early
   
3. **Authentication Integration** - Backend mechanism unclear
   - **Mitigation:** Clarify with backend team before POC
   
4. **WebSocket Infrastructure** - Blazor Server requires WebSockets
   - **Mitigation:** Verify infrastructure supports WebSockets
   
5. **Developer Learning Curve** - Team .NET experience
   - **Mitigation:** Training sessions, pair programming

## 📖 Reading Order

### For Developers Starting POC
1. `POC-QUICKSTART.md` - Follow this day-by-day
2. `02-SERVICE-MAPPING.md` - Reference when implementing services
3. `03-COMPONENT-MAPPING.md` - Reference when building components
4. `06-VALIDATION-RULES.md` - Reference when adding validation
5. `08-TEST-STRATEGY.md` - Reference when writing tests

### For Project Managers
1. `00-OVERVIEW.md` - High-level summary
2. `01-POC-REQUIREMENTS.md` - POC scope and success criteria
3. Review the main plan: `~/.windsurf/plans/blazor-migration-plan-273994.md`

### For Architects
1. Read all mapping documents (02-08)
2. Review technology decisions
3. Validate with infrastructure team

## 🤝 Contributing

As migration progresses:
- Update status in `00-OVERVIEW.md`
- Document deviations in respective mapping docs
- Add lessons learned to `POC-QUICKSTART.md`
- Update success criteria if needed

## 📞 Support

Questions or blockers?
- Check mapping docs first
- Review main migration plan
- Escalate to architecture team if needed

---

**Current Status:** ✅ **Documentation Complete - Ready to Begin POC**

**Next Action:** Follow `POC-QUICKSTART.md` Day 1 instructions to create Blazor solution structure.

**Last Updated:** 2026-04-27
