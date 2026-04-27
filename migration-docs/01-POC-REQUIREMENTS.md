# POC Requirements: DashboardUser CRUD Page

## Why DashboardUser Page?

Selected for POC because it represents **ideal complexity** for validating migration approach:

✅ **Representative Features**
- Full CRUD operations (Create, Read, Update, Delete)
- Data grid with filtering and sorting
- Form with multiple input types
- Validation (email, required fields)
- Authorization checks (privilege-based)
- Service layer abstraction

✅ **Not Too Simple**
- More than just static content
- Requires state management
- Has business logic

✅ **Not Too Complex**
- Single entity (DashboardUser)
- No file uploads
- No charts/metrics
- No complex relationships

✅ **Well-Tested**
- Existing Cypress tests to port
- Clear acceptance criteria

## POC Success Criteria

The POC must demonstrate that the following are **viable and acceptable**:

### 1. Service Layer Pattern ✅
- C# service classes map cleanly from TypeScript
- Interface-driven design works
- DTO conversions are straightforward
- API client integration is clean

### 2. State Management ✅
- Scoped services pattern works for shared state
- Component reactivity is acceptable
- State updates propagate correctly

### 3. Form Handling ✅
- EditForm + EditContext replaces Hookstate forms
- Validation works equivalently to React
- Form state tracking (modified, touched) works
- Error display is intuitive

### 4. Grid Functionality ✅
- Selected grid library (Radzen or MudBlazor) has feature parity
- Filtering and sorting work
- Custom cell rendering works
- Performance is acceptable (1000+ rows)

### 5. Authorization Flow ✅
- AuthenticationStateProvider works correctly
- Privilege checks function properly
- Protected pages enforce authorization

### 6. Testing Approach ✅
- xUnit tests for service layer are straightforward
- bUnit tests for components are viable
- Playwright tests replicate Cypress functionality

### 7. Developer Experience ✅
- Hot reload works consistently
- Build times are acceptable (<30s incremental)
- Debugging experience is satisfactory
- Code patterns feel maintainable

## POC Scope (In Scope)

### Feature Requirements

#### **Grid View (List Page)**
- Display all dashboard users in a data grid
- Columns: UUID, Email, Dashboard Admin (checkbox)
- Filtering by email (text search)
- Sorting by any column
- Row selection/highlighting
- Click row to edit
- Delete button per row
- Add new user button

#### **Create Form**
- Modal or page-based form
- Fields:
  - Email (text input, required, email validation, 1-255 chars)
  - Dashboard Admin (checkbox)
- Validation messages display
- Submit button (disabled until valid)
- Cancel button
- Success message on create
- Error handling (validation errors, API errors)

#### **Edit Form**
- Load existing user data
- Same fields as create form
- UUID displayed (read-only)
- Track form modifications (disable submit if unchanged)
- Success message on update
- Error handling

#### **Delete Operation**
- Confirmation dialog
- Success message on delete
- Error handling
- Remove from grid on success

#### **Authorization**
- Page requires DASHBOARD_ADMIN privilege
- Unauthorized users see NotAuthorized page
- User info displays in header/nav

### Technical Requirements

#### **Service Layer**
Implement `IDashboardUserService` with:
- `Task<List<DashboardUserFlat>> FetchAndStoreDataAsync()`
- `Task<DashboardUserFlat> CreateAsync(DashboardUserDto dto)`
- `Task<DashboardUserFlat> UpdateAsync(DashboardUserDto dto)`
- `Task DeleteAsync(string id)`
- `DashboardUserDto ConvertToDto(DashboardUserFlat flat)`
- `DashboardUserFlat ConvertToFlat(DashboardUserDto dto)`

#### **Components**
- `DashboardUserPage.razor` - Main page with grid
- `DashboardUserForm.razor` - Create/Edit form
- `TronTextInput.razor` - Reusable text input with validation
- `TronCheckbox.razor` - Reusable checkbox
- `PageLayout.razor` - Base page layout

#### **Testing**
- xUnit: 10+ tests for service layer
  - Fetch data
  - Create user
  - Update user
  - Delete user
  - DTO conversions
  - Error handling
- bUnit: 5+ tests for components
  - Grid renders
  - Form validation
  - Form submission
  - Error display
- Playwright: 1 E2E test
  - Full CRUD workflow (create → edit → delete)

## POC Scope (Out of Scope)

The following are **explicitly excluded** to keep POC focused:

❌ **Other Pages**
- All pages except DashboardUser
- Navigation to other pages (stubs only)

❌ **Advanced Grid Features**
- Export to CSV/Excel
- Column reordering
- Column hiding
- Bulk operations
- Row grouping
- Virtual scrolling (unless needed for performance)

❌ **Advanced Form Features**
- Multi-step forms
- File uploads
- Rich text editing
- Date pickers

❌ **Styling Polish**
- Pixel-perfect UI matching React app
- Responsive design optimization
- Animations/transitions
- Focus on **functionality** over **aesthetics**

❌ **Other Features**
- Document Space features
- Charts/metrics
- Audit logging
- Real-time updates
- Offline support

❌ **Production Concerns**
- Performance optimization (unless blocking)
- Accessibility audit
- Cross-browser testing
- Security hardening
- Production deployment

## POC Implementation Checklist

### Week 1: Foundation (Days 1-5)

#### Day 1: Project Setup
- [ ] Create Blazor Server solution
- [ ] Set up project structure (Web, Core, Tests)
- [ ] Add NuGet packages (MudBlazor, Radzen, NSwag, bUnit, Moq)
- [ ] Configure launchSettings.json
- [ ] Verify build and run

#### Day 2: API Client
- [ ] Copy `tron-common-api.json` to Core project
- [ ] Configure NSwag.MSBuild for C# generation
- [ ] Generate DashboardUserControllerApi client
- [ ] Generate DashboardUserDto model
- [ ] Test API client in isolation

#### Day 3: Authentication
- [ ] Implement `TronAuthenticationStateProvider`
- [ ] Call `/api/v1/dashboard-users/self` endpoint
- [ ] Map to ClaimsPrincipal with privileges
- [ ] Configure HttpClient with auth
- [ ] Test auth flow

#### Day 4: Base Components
- [ ] Create `PageLayout.razor` with nav/header
- [ ] Create `TronTextInput.razor` with validation support
- [ ] Create `TronCheckbox.razor`
- [ ] Create `ConfirmDialog.razor`
- [ ] Test components in isolation

#### Day 5: Service Layer
- [ ] Create `IDashboardUserService` interface
- [ ] Implement `DashboardUserService` class
- [ ] Implement DTO conversion methods
- [ ] Wire up dependency injection
- [ ] Write xUnit tests for service

### Week 2: UI Implementation (Days 6-10)

#### Day 6-7: Grid Page
- [ ] Create `DashboardUserPage.razor`
- [ ] Integrate Radzen DataGrid
- [ ] Add columns (UUID, Email, DashboardAdmin)
- [ ] Add filtering by email
- [ ] Add sorting
- [ ] Wire up data loading from service
- [ ] Add "Add New" button

#### Day 8-9: Form Implementation
- [ ] Create `DashboardUserForm.razor`
- [ ] Add email field with validation
- [ ] Add checkbox for Dashboard Admin
- [ ] Implement create mode
- [ ] Implement edit mode (with UUID display)
- [ ] Track form modification state
- [ ] Add success/error messages
- [ ] Wire up to service

#### Day 10: Delete & Integration
- [ ] Add delete button to grid
- [ ] Implement confirmation dialog
- [ ] Wire up delete to service
- [ ] Test full CRUD flow
- [ ] Fix bugs

### Week 3: Testing & Review (Days 11-15)

#### Day 11-12: Unit Tests
- [ ] Complete xUnit service tests (10+ tests)
- [ ] Write bUnit component tests (5+ tests)
- [ ] Achieve 80%+ service coverage
- [ ] Fix failing tests

#### Day 13: E2E Test
- [ ] Set up Playwright test project
- [ ] Write CRUD workflow test
- [ ] Configure test authentication
- [ ] Run test against localhost

#### Day 14: Documentation & Cleanup
- [ ] Document patterns discovered
- [ ] Document deviations from plan
- [ ] Document challenges encountered
- [ ] Clean up code, add comments
- [ ] Update migration docs

#### Day 15: POC Review
- [ ] Prepare demo
- [ ] Present to stakeholders
- [ ] Gather feedback
- [ ] Document lessons learned
- [ ] **Make Go/No-Go decision**

## Success Metrics (POC Review Gate)

The POC is considered **successful** if:

### Functional Metrics
- ✅ All CRUD operations work correctly
- ✅ Validation matches React behavior
- ✅ Grid has acceptable feature parity
- ✅ Authorization works correctly
- ✅ No critical bugs

### Quality Metrics
- ✅ Service layer: 80%+ test coverage
- ✅ Components: 50%+ test coverage
- ✅ E2E test passes consistently
- ✅ All tests pass

### Performance Metrics
- ✅ Page load < 2 seconds
- ✅ Grid renders 1000 rows < 1 second
- ✅ Form submission < 1 second

### Developer Experience
- ✅ Hot reload works most of the time
- ✅ Build time < 30 seconds (incremental)
- ✅ Debugging is acceptable
- ✅ Team consensus: approach is viable

## Go/No-Go Decision Criteria

### Go (Proceed with Full Migration) If:
- All success metrics met
- No major blockers discovered
- Team confident in approach
- Stakeholders approve

### No-Go (Halt Migration) If:
- Critical feature gaps (grid, forms, auth)
- Performance unacceptable
- Developer experience too poor
- Team lacks confidence
- Technical debt too high

### Pivot (Adjust Approach) If:
- Some metrics missed but recoverable
- Alternative patterns needed
- Technology choices need revision
- Timeline needs adjustment

## POC Deliverables

At the end of Week 3, deliver:

1. **Working Code**
   - Blazor solution with POC implemented
   - All tests passing
   - Runnable locally

2. **Documentation**
   - Lessons learned document
   - Pattern library (reusable patterns discovered)
   - Deviations from plan
   - Recommended adjustments

3. **Demo**
   - Live demonstration of CRUD operations
   - Show test execution
   - Discuss challenges

4. **Decision**
   - Clear Go/No-Go recommendation
   - Risk assessment
   - Adjusted timeline (if needed)

---

**Next Step**: Review this document, then proceed to 02-SERVICE-MAPPING.md
