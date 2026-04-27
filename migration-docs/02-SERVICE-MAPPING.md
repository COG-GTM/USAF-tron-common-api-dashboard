# Service Layer Mapping: TypeScript → C#

## Overview

This document maps all 33 TypeScript service files to their C# equivalents. The current architecture uses service classes that wrap Hookstate atoms and OpenAPI clients - this pattern translates well to C# with dependency injection.

## Service Pattern Translation

### TypeScript Pattern (Current)
```typescript
// State atom
const dashboardUserState = createState<DashboardUserFlat[]>([]);

// Service class
export default class DashboardUserService implements DataService<DashboardUserFlat, DashboardUserDto> {
  constructor(
    public state: State<DashboardUserFlat[]>,
    private dashboardUserApi: DashboardUserControllerApiInterface
  ) {}
  
  fetchAndStoreData(): CancellableDataRequest<DashboardUserFlat[]> {
    const response = this.dashboardUserApi.getAllDashboardUsersWrapped();
    const data = response.then(r => this.convertDashboardUsersToFlat(r.data.data));
    this.state.set(data);
    return { promise: data, cancelTokenSource };
  }
}

// Hook for consumption
export const useDashboardUserState = (): DashboardUserService => 
  wrapDashboardUserState(useState(dashboardUserState), dashboardUserApi);
```

### C# Pattern (Target)
```csharp
// Service interface
public interface IDashboardUserService : IDataService<DashboardUserFlat, DashboardUserDto>
{
    ObservableCollection<DashboardUserFlat> DashboardUsers { get; }
    Task<List<DashboardUserFlat>> FetchAndStoreDataAsync(CancellationToken ct = default);
    Task<DashboardUserFlat> CreateAsync(DashboardUserDto dto, CancellationToken ct = default);
    Task<DashboardUserFlat> UpdateAsync(DashboardUserDto dto, CancellationToken ct = default);
    Task DeleteAsync(string id, CancellationToken ct = default);
}

// Service implementation
public class DashboardUserService : IDashboardUserService
{
    private readonly IDashboardUserControllerApi _api;
    private readonly ObservableCollection<DashboardUserFlat> _dashboardUsers = new();
    
    public DashboardUserService(IDashboardUserControllerApi api)
    {
        _api = api;
    }
    
    public ObservableCollection<DashboardUserFlat> DashboardUsers => _dashboardUsers;
    
    public async Task<List<DashboardUserFlat>> FetchAndStoreDataAsync(CancellationToken ct = default)
    {
        var response = await _api.GetAllDashboardUsersWrappedAsync(ct);
        var flats = ConvertDashboardUsersToFlat(response.Data);
        
        _dashboardUsers.Clear();
        foreach (var flat in flats)
            _dashboardUsers.Add(flat);
            
        return flats;
    }
}

// DI registration (Program.cs)
builder.Services.AddScoped<IDashboardUserService, DashboardUserService>();

// Component consumption (Blazor)
@inject IDashboardUserService DashboardUserService

@code {
    protected override async Task OnInitializedAsync()
    {
        await DashboardUserService.FetchAndStoreDataAsync();
    }
}
```

## Complete Service Inventory

### Core Services (22 State Modules)

| # | TypeScript Service | C# Service | Priority | Complexity | Notes |
|---|-------------------|------------|----------|------------|-------|
| 1 | `dashboard-user-service.ts` | `DashboardUserService` | **POC** | Low | CRUD, simple DTO conversion |
| 2 | `authorized-user-service.ts` | `AuthorizedUserService` | **POC** | Low | Single user, auth state |
| 3 | `app-clients-service.ts` | `AppClientService` | High | Medium | Relationships with developers |
| 4 | `app-source-service.ts` | `AppSourceService` | High | Medium | Relationships with endpoints |
| 5 | `person-service.ts` | `PersonService` | High | Medium | CRUD with organization link |
| 6 | `organization-service.ts` | `OrganizationService` | High | Medium | CRUD with people link |
| 7 | `document-space-service.ts` | `DocumentSpaceService` | Low | **High** | Complex file operations |
| 8 | `audit-log-service.ts` | `AuditLogService` | Medium | Low | Read-only, filtering |
| 9 | `health-service.ts` | `HealthService` | High | Low | Simple health check |
| 10 | `kpi-service.ts` | `KpiService` | Medium | Medium | Data aggregation |
| 11 | `logfile-service.ts` | `LogfileService` | Medium | Medium | Two services (current, past) |
| 12 | `my-digitize-apps-service.ts` | `MyDigitizeAppsService` | Medium | Low | Read-only listing |
| 13 | `pub-sub-service.ts` | `PubSubService` | Medium | Medium | CRUD subscriptions |
| 14 | `scratch-storage-service.ts` | `ScratchStorageService` | Medium | Medium | Key-value CRUD |
| 15 | `app-info-service.ts` | `AppInfoService` | High | Low | Read-only app metadata |
| 16 | `privilege-service.ts` | `PrivilegeService` | **POC** | Low | Read-only privileges list |
| 17 | `event-request-log-service.ts` | `EventRequestLogService` | Low | Medium | Read-only logs |
| 18 | `metric-service.ts` | `MetricService` | Low | High | Time-series data |
| 19 | `app-client-user-metric-service.ts` | `AppClientUserMetricService` | Low | High | Aggregated metrics |
| 20 | `app-endpoint-metric-service.ts` | `AppEndpointMetricService` | Low | High | Endpoint metrics |
| 21 | `app-source-metric-service.ts` | `AppSourceMetricService` | Low | High | Source metrics |
| 22 | `document-space-*-service.ts` (6 files) | `DocumentSpace*Service` | Low | **High** | Document space ecosystem |

### Data Service Base Classes

| TypeScript | C# | Purpose |
|-----------|-----|---------|
| `abstract-data-service.ts` | `IDataService<TFlat, TDto>` | Base interface for CRUD services |
| `data-service.ts` | `DataService<TFlat, TDto>` | Base implementation |
| `data-service-utils.ts` | `DataServiceExtensions` | Static utility methods |
| `abstract-global-state-service.ts` | `IGlobalStateService` | Base for global state services |
| `global-state-service.ts` | `GlobalStateService` | Base implementation |

## Detailed Service Mappings

### 1. DashboardUserService (POC Priority)

**Source**: `src/state/dashboard-user/dashboard-user-service.ts`

**Key Methods**:
```typescript
// TypeScript
fetchAndStoreData(): CancellableDataRequest<DashboardUserFlat[]>
convertDashboardUsersToFlat(users: DashboardUserDto[]): DashboardUserFlat[]
convertToFlat(user: DashboardUserDto): DashboardUserFlat
convertToDto(flat: DashboardUserFlat): DashboardUserDto
sendCreate(toCreate: DashboardUserDto): Promise<DashboardUserFlat>
sendUpdate(toUpdate: DashboardUserDto): Promise<DashboardUserFlat>
sendDelete(toDelete: DashboardUserFlat): Promise<void>
convertRowDataToEditableData(rowData: DashboardUserFlat): Promise<DashboardUserDto>
```

**C# Translation**:
```csharp
public interface IDashboardUserService
{
    ObservableCollection<DashboardUserFlat> DashboardUsers { get; }
    bool IsLoading { get; }
    string? Error { get; }
    
    Task<List<DashboardUserFlat>> FetchAndStoreDataAsync(CancellationToken ct = default);
    List<DashboardUserFlat> ConvertDashboardUsersToFlat(List<DashboardUserDto> users);
    DashboardUserFlat ConvertToFlat(DashboardUserDto user);
    DashboardUserDto ConvertToDto(DashboardUserFlat flat);
    Task<DashboardUserFlat> CreateAsync(DashboardUserDto dto, CancellationToken ct = default);
    Task<DashboardUserFlat> UpdateAsync(DashboardUserDto dto, CancellationToken ct = default);
    Task DeleteAsync(string id, CancellationToken ct = default);
    Task<DashboardUserDto> ConvertRowDataToEditableDataAsync(DashboardUserFlat rowData);
}
```

**Key Differences**:
- `CancellableDataRequest` → `CancellationToken` parameter
- Synchronous conversions stay synchronous
- `Promise<T>` → `Task<T>`
- Error handling via exceptions instead of error state (or both)
- `State<T>` → `ObservableCollection<T>` for reactive collections

### 2. AuthorizedUserService (POC Priority)

**Source**: `src/state/authorized-user/authorized-user-service.ts`

**Key Methods**:
```typescript
fetchAndStoreAuthorizedUser(): Promise<DashboardUserDto>
authorizedUserHasPrivilege(privilegeType: PrivilegeType): boolean
authorizedUserHasAnyPrivilege(privilegeTypes: PrivilegeType[]): boolean
setDocumentSpaceDefaultId(spaceId: string): void
```

**C# Translation**:
```csharp
public interface IAuthorizedUserService
{
    DashboardUserDto? AuthorizedUser { get; }
    bool IsLoading { get; }
    string? Error { get; }
    
    Task<DashboardUserDto> FetchAndStoreAuthorizedUserAsync(CancellationToken ct = default);
    bool AuthorizedUserHasPrivilege(PrivilegeType privilegeType);
    bool AuthorizedUserHasAnyPrivilege(params PrivilegeType[] privilegeTypes);
    void SetDocumentSpaceDefaultId(string spaceId);
}
```

**Integration with Authentication**:
```csharp
// This service backs the AuthenticationStateProvider
public class TronAuthenticationStateProvider : AuthenticationStateProvider
{
    private readonly IAuthorizedUserService _userService;
    
    public override async Task<AuthenticationState> GetAuthenticationStateAsync()
    {
        var user = await _userService.FetchAndStoreAuthorizedUserAsync();
        var identity = CreateIdentityFromUser(user);
        return new AuthenticationState(new ClaimsPrincipal(identity));
    }
    
    private ClaimsIdentity CreateIdentityFromUser(DashboardUserDto user)
    {
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(ClaimTypes.NameIdentifier, user.Id)
        };
        
        foreach (var privilege in user.Privileges)
        {
            claims.Add(new Claim("Privilege", privilege.Name));
        }
        
        return new ClaimsIdentity(claims, "TronAuth");
    }
}
```

### 3. AppClientService

**Source**: `src/state/app-clients/app-clients-service.ts`

**Complexity**: Medium (has developers relationship)

**Key Methods**:
```typescript
fetchAndStoreData()
sendCreate(toCreate: AppClientDto)
sendUpdate(toUpdate: AppClientDto)
sendDelete(toDelete: AppClientFlat)
addDeveloperToAppClient(appClient: AppClientDto, email: string)
removeDeveloperFromAppClient(appClient: AppClientDto, email: string)
```

**C# Translation**:
```csharp
public interface IAppClientService : IDataService<AppClientFlat, AppClientDto>
{
    ObservableCollection<AppClientFlat> AppClients { get; }
    Task<List<AppClientFlat>> FetchAndStoreDataAsync(CancellationToken ct = default);
    Task<AppClientFlat> CreateAsync(AppClientDto dto, CancellationToken ct = default);
    Task<AppClientFlat> UpdateAsync(AppClientDto dto, CancellationToken ct = default);
    Task DeleteAsync(string id, CancellationToken ct = default);
    Task<AppClientDto> AddDeveloperToAppClientAsync(AppClientDto appClient, string email, CancellationToken ct = default);
    Task<AppClientDto> RemoveDeveloperFromAppClientAsync(AppClientDto appClient, string email, CancellationToken ct = default);
}
```

### 4. DocumentSpaceService

**Source**: `src/state/document-space/document-space-service.ts` (+ 5 related services)

**Complexity**: **HIGH** - Most complex service in the application

**Related Services**:
- `document-space-service.ts` - Main space CRUD
- `document-space-global-service.ts` - Global state coordination
- `document-space-privilege-service.ts` - Permission management
- `document-space-download-url-service.ts` - Download URL generation
- `document-space-membership-service.ts` - User memberships
- `spaces-page-service.ts`, `recents-page-service.ts`, `memberships-page-service.ts` - Page-level state

**Key Challenges**:
- File upload/download operations
- Folder navigation
- Archive/unarchive operations
- Favorites management
- Complex grid with nested items
- Drag & drop support

**Defer to Phase 3** (Weeks 13-16)

## Service Migration Priority

### POC (Week 1-3)
1. **DashboardUserService** - Full implementation
2. **AuthorizedUserService** - Full implementation  
3. **PrivilegeService** - Read-only, simple

### Phase 1 (Weeks 4-8)
4. **AppInfoService** - Read-only, simple
5. **HealthService** - Read-only, simple
6. **PersonService** - CRUD
7. **OrganizationService** - CRUD
8. **AppClientService** - Complex CRUD
9. **AppSourceService** - Complex CRUD

### Phase 2 (Weeks 9-12)
10. **PubSubService** - CRUD
11. **ScratchStorageService** - Key-value CRUD
12. **AuditLogService** - Read-only with filtering
13. **LogfileService** - Log viewing
14. **MyDigitizeAppsService** - Read-only
15. **KpiService** - Data aggregation

### Phase 3 (Weeks 13-16)
16. **DocumentSpaceService** + 5 related - Complex file operations

### Phase 4 (Weeks 17-18)
17. **MetricService** - Time-series
18. **AppClientUserMetricService** - Aggregations
19. **AppEndpointMetricService** - Aggregations
20. **AppSourceMetricService** - Aggregations
21. **EventRequestLogService** - Read-only logs

## Common Patterns

### Error Handling

**TypeScript**:
```typescript
catch (error) {
  return Promise.reject(prepareDataCrudErrorResponse(error));
}
```

**C#**:
```csharp
catch (ApiException ex)
{
    throw new DataCrudException(ex.Message, ex.StatusCode, ParseValidationErrors(ex));
}
```

### DTO Conversion

**TypeScript**:
```typescript
convertToFlat(user: DashboardUserDto): DashboardUserFlat {
  return {
    id: user.id,
    email: user.email || '',
    hasDashboardAdmin: user.privileges?.some(p => p.name === PrivilegeType.DASHBOARD_ADMIN) || false
  };
}
```

**C#**:
```csharp
public DashboardUserFlat ConvertToFlat(DashboardUserDto user)
{
    return new DashboardUserFlat
    {
        Id = user.Id,
        Email = user.Email ?? string.Empty,
        HasDashboardAdmin = user.Privileges?.Any(p => p.Name == PrivilegeType.DASHBOARD_ADMIN) ?? false
    };
}
```

### State Updates

**TypeScript**:
```typescript
// Add to state
this.state[this.state.length].set(dashboardUserFlat);

// Update in state
const index = this.state.get().findIndex(item => item.id === flat.id);
this.state[index].set(flat);

// Remove from state
this.state.find(item => item.id.get() === toDelete.id)?.set(none);
```

**C#**:
```csharp
// Add to state
_dashboardUsers.Add(dashboardUserFlat);
// ObservableCollection automatically notifies UI

// Update in state
var existing = _dashboardUsers.FirstOrDefault(x => x.Id == flat.Id);
if (existing != null)
{
    var index = _dashboardUsers.IndexOf(existing);
    _dashboardUsers[index] = flat;
}

// Remove from state
var toRemove = _dashboardUsers.FirstOrDefault(x => x.Id == id);
if (toRemove != null)
    _dashboardUsers.Remove(toRemove);
```

## Testing Pattern

### xUnit Service Tests

```csharp
public class DashboardUserServiceTests
{
    private readonly Mock<IDashboardUserControllerApi> _mockApi;
    private readonly DashboardUserService _service;
    
    public DashboardUserServiceTests()
    {
        _mockApi = new Mock<IDashboardUserControllerApi>();
        _service = new DashboardUserService(_mockApi.Object);
    }
    
    [Fact]
    public async Task FetchAndStoreDataAsync_PopulatesCollection()
    {
        // Arrange
        var dtos = new List<DashboardUserDto>
        {
            new() { Id = "1", Email = "test@example.com" }
        };
        _mockApi.Setup(x => x.GetAllDashboardUsersWrappedAsync(It.IsAny<CancellationToken>()))
            .ReturnsAsync(new WrappedResponse<List<DashboardUserDto>> { Data = dtos });
        
        // Act
        await _service.FetchAndStoreDataAsync();
        
        // Assert
        Assert.Single(_service.DashboardUsers);
        Assert.Equal("test@example.com", _service.DashboardUsers[0].Email);
    }
    
    [Fact]
    public async Task CreateAsync_AddsToCollection()
    {
        // Arrange
        var dto = new DashboardUserDto { Email = "new@example.com" };
        var created = new DashboardUserDto { Id = "123", Email = "new@example.com" };
        _mockApi.Setup(x => x.AddDashboardUserAsync(dto, It.IsAny<CancellationToken>()))
            .ReturnsAsync(created);
        
        // Act
        var result = await _service.CreateAsync(dto);
        
        // Assert
        Assert.NotNull(result);
        Assert.Single(_service.DashboardUsers);
        Assert.Equal("123", result.Id);
    }
}
```

---

**Next Step**: Proceed to 03-COMPONENT-MAPPING.md
