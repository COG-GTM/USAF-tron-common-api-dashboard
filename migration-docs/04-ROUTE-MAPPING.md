# Route Mapping: React Router → Blazor Routing

## Overview

Maps all routes from `src/routes.ts` to Blazor `@page` directives and navigation configuration.

## Current Route Structure (React Router v5)

### Route Configuration (`src/routes.ts`)

```typescript
export enum RoutePath {
    HOME = '/',
    HEALTH = '/health',
    PERSON = '/person',
    APP_CLIENT = '/app-clients',
    ORGANIZATION = '/organization',
    LOGFILE = '/logfile',
    DASHBOARD_USER = '/dashboard-user',
    SCRATCH_STORAGE = '/scratch-storage',
    APP_SOURCE = '/app-source',
    MY_DIGITIZE_APPS = '/digitize-apps',
    PUB_SUB = '/pubsub',
    AUDIT_LOG = '/audit-log',
    KPI = '/kpi',
    DOCUMENT_SPACE_SPACES = '/document-space/spaces',
    DOCUMENT_SPACE_RECENTS = '/document-space/recents',
    DOCUMENT_SPACE_FAVORITES = '/document-space/favorites',
    DOCUMENT_SPACE_ARCHIVED = '/document-space/archived',
    APP_SOURCE_METRIC = '/app-source/:id/metrics/:type/:name/:method?',
    API_TEST = '/app-api/:apiId',
    NOT_FOUND = '/not-found',
    NOT_AUTHORIZED = '/not-authorized',
}
```

## Blazor Route Mapping

### Top-Level Routes

| React Route | React Component | Blazor Route | Blazor Page | Auth Policy | Priority |
|-------------|----------------|--------------|-------------|-------------|----------|
| `/` | `HomePage` | `/` | `Pages/Home/HomePage.razor` | Any authenticated | High |
| `/health` | `HealthPage` | `/health` | `Pages/Health/HealthPage.razor` | `DashboardUser` | High |
| `/dashboard-user` | `DashboardUserPage` | `/dashboard-users` | `Pages/DashboardUser/DashboardUserPage.razor` | `DashboardAdmin` | **POC** |
| `/person` | `PersonPage` | `/people` | `Pages/Person/PersonPage.razor` | `DashboardAdmin` | High |
| `/organization` | `OrganizationPage` | `/organizations` | `Pages/Organization/OrganizationPage.razor` | `DashboardAdmin` | High |
| `/app-clients` | `AppClientPage` | `/app-clients` | `Pages/AppClient/AppClientPage.razor` | `DashboardAdmin,AppClientDeveloper` | High |
| `/app-source` | `AppSourcePage` | `/app-sources` | `Pages/AppSource/AppSourcePage.razor` | `DashboardAdmin,AppSourceAdmin` | High |
| `/logfile` | `LogfilePage` | `/logs` | `Pages/Logfile/LogfilePage.razor` | `DashboardAdmin` | Medium |
| `/audit-log` | `AuditLogPage` | `/audit-logs` | `Pages/AuditLog/AuditLogPage.razor` | `DashboardAdmin` | Medium |
| `/pubsub` | `PubSubPage` | `/subscriptions` | `Pages/PubSub/PubSubPage.razor` | `DashboardAdmin,AppClientDeveloper` | Medium |
| `/scratch-storage` | `ScratchStoragePage` | `/scratch-storage` | `Pages/ScratchStorage/ScratchStoragePage.razor` | `DashboardAdmin,ScratchAdmin` | Medium |
| `/digitize-apps` | `MyDigitizeAppsPage` | `/my-apps` | `Pages/MyDigitizeApps/MyDigitizeAppsPage.razor` | `DashboardUser` | Medium |
| `/kpi` | `KpiPage` | `/kpi` | `Pages/Kpi/KpiPage.razor` | `DashboardAdmin` | Medium |
| `/not-authorized` | `NotAuthorizedPage` | `/not-authorized` | `Pages/NotAuthorized/NotAuthorizedPage.razor` | None | High |
| `/not-found` | `NotFoundPage` | `/not-found` | `Pages/NotFound/NotFoundPage.razor` | None | High |

### Document Space Routes

| React Route | Blazor Route | Blazor Page | Auth Policy | Priority |
|-------------|--------------|-------------|-------------|----------|
| `/document-space/spaces` | `/document-spaces` | `Pages/DocumentSpace/DocumentSpacePage.razor` | `DocumentSpaceUser,DashboardAdmin` | Low |
| `/document-space/recents` | `/document-spaces/recents` | `Pages/DocumentSpace/Recents/DocumentSpaceRecentsPage.razor` | `DocumentSpaceUser,DashboardAdmin` | Low |
| `/document-space/favorites` | `/document-spaces/favorites` | `Pages/DocumentSpace/Favorites/DocumentSpaceFavoritesPage.razor` | `DocumentSpaceUser,DashboardAdmin` | Low |
| `/document-space/archived` | `/document-spaces/archived` | `Pages/DocumentSpace/Archived/DocumentSpaceArchivedPage.razor` | `DocumentSpaceUser,DashboardAdmin` | Low |

### Dynamic Routes (with parameters)

| React Route | Blazor Route | Blazor Page | Parameters | Priority |
|-------------|--------------|-------------|------------|----------|
| `/app-source/:id/metrics/:type/:name/:method?` | `/app-sources/{id}/metrics/{type}/{name}/{method?}` | `Pages/AppSource/Metrics/MetricPage.razor` | `id`, `type`, `name`, `method?` | Low |
| `/app-api/:apiId` | `/api-test/{apiId}` | `Pages/ApiTest/ApiTestPage.razor` | `apiId` | Low |

## Blazor Routing Implementation

### Page Definition with @page Directive

```razor
@* DashboardUserPage.razor *@
@page "/dashboard-users"
@attribute [Authorize(Policy = "RequireDashboardAdmin")]

<PageLayout PageTitle="Dashboard Users">
    @* Page content *@
</PageLayout>
```

### Route Parameters

```razor
@* MetricPage.razor *@
@page "/app-sources/{id}/metrics/{type}/{name}/{method?}"
@attribute [Authorize(Policy = "RequireAppSourceAdmin")]

@code {
    [Parameter] public string Id { get; set; } = string.Empty;
    [Parameter] public string Type { get; set; } = string.Empty;
    [Parameter] public string Name { get; set; } = string.Empty;
    [Parameter] public string? Method { get; set; }
    
    protected override async Task OnParametersSetAsync()
    {
        // Load metrics based on parameters
        await LoadMetricsAsync(Id, Type, Name, Method);
    }
}
```

### Authorization Policies (Program.cs)

```csharp
builder.Services.AddAuthorizationCore(options =>
{
    // Single privilege policies
    options.AddPolicy("RequireDashboardAdmin", policy =>
        policy.RequireClaim("Privilege", "DASHBOARD_ADMIN"));
    
    options.AddPolicy("RequireDashboardUser", policy =>
        policy.RequireClaim("Privilege", "DASHBOARD_USER", "DASHBOARD_ADMIN", 
            "APP_SOURCE_ADMIN", "SCRATCH_READ", "SCRATCH_WRITE", "SCRATCH_ADMIN", 
            "APP_CLIENT_DEVELOPER", "DOCUMENT_SPACE_USER"));
    
    options.AddPolicy("RequireAppClientDeveloper", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c => c.Type == "Privilege" && 
                (c.Value == "DASHBOARD_ADMIN" || c.Value == "APP_CLIENT_DEVELOPER"))));
    
    options.AddPolicy("RequireAppSourceAdmin", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c => c.Type == "Privilege" && 
                (c.Value == "DASHBOARD_ADMIN" || c.Value == "APP_SOURCE_ADMIN"))));
    
    options.AddPolicy("RequireDocumentSpaceUser", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c => c.Type == "Privilege" && 
                (c.Value == "DOCUMENT_SPACE_USER" || c.Value == "DASHBOARD_ADMIN"))));
    
    options.AddPolicy("RequireScratchAdmin", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c => c.Type == "Privilege" && 
                (c.Value == "DASHBOARD_ADMIN" || c.Value == "SCRATCH_ADMIN"))));
});
```

## Navigation Configuration

### Navigation Menu Structure

```csharp
// Models/NavigationItem.cs
public class NavigationItem
{
    public string Name { get; set; } = string.Empty;
    public string Path { get; set; } = string.Empty;
    public string Icon { get; set; } = string.Empty;
    public string[] RequiredPrivileges { get; set; } = Array.Empty<string>();
    public List<NavigationItem>? Children { get; set; }
    public Func<string, bool>? HideCondition { get; set; }
}

// Services/NavigationService.cs
public class NavigationService
{
    private readonly List<NavigationItem> _menuItems = new()
    {
        new NavigationItem
        {
            Name = "Home",
            Path = "/",
            Icon = Icons.Material.Filled.Home,
            RequiredPrivileges = new[] { "DASHBOARD_USER", "DASHBOARD_ADMIN", "APP_SOURCE_ADMIN", 
                "SCRATCH_READ", "SCRATCH_WRITE", "SCRATCH_ADMIN", "APP_CLIENT_DEVELOPER" }
        },
        new NavigationItem
        {
            Name = "Health",
            Path = "/health",
            Icon = Icons.Material.Filled.HealthAndSafety,
            RequiredPrivileges = new[] { "DASHBOARD_USER", "APP_CLIENT_DEVELOPER" }
        },
        new NavigationItem
        {
            Name = "Records",
            Path = "#",
            Icon = Icons.Material.Filled.Article,
            RequiredPrivileges = new[] { "DASHBOARD_ADMIN", "APP_CLIENT_DEVELOPER" },
            Children = new List<NavigationItem>
            {
                new() { Name = "People", Path = "/people", RequiredPrivileges = new[] { "DASHBOARD_ADMIN" } },
                new() { Name = "Organizations", Path = "/organizations", RequiredPrivileges = new[] { "DASHBOARD_ADMIN" } },
                new() { Name = "Subscriptions", Path = "/subscriptions", RequiredPrivileges = new[] { "DASHBOARD_ADMIN", "APP_CLIENT_DEVELOPER" } }
            }
        },
        new NavigationItem
        {
            Name = "Apps",
            Path = "#",
            Icon = Icons.Material.Filled.Apps,
            RequiredPrivileges = new[] { "APP_CLIENT_DEVELOPER", "DASHBOARD_ADMIN", "APP_SOURCE_ADMIN" },
            Children = new List<NavigationItem>
            {
                new() { Name = "App Clients", Path = "/app-clients", RequiredPrivileges = new[] { "DASHBOARD_ADMIN", "APP_CLIENT_DEVELOPER" } },
                new() { Name = "App Sources", Path = "/app-sources", RequiredPrivileges = new[] { "DASHBOARD_ADMIN", "APP_SOURCE_ADMIN" } }
            }
        },
        new NavigationItem
        {
            Name = "Digitize",
            Path = "#",
            Icon = Icons.Material.Filled.Cloud,
            RequiredPrivileges = new[] { "DASHBOARD_USER" },
            Children = new List<NavigationItem>
            {
                new() { Name = "My Digitize Apps", Path = "/my-apps", RequiredPrivileges = new[] { "DASHBOARD_USER" } },
                new() { Name = "Scratch Storage Apps", Path = "/scratch-storage", RequiredPrivileges = new[] { "DASHBOARD_ADMIN", "SCRATCH_ADMIN" } }
            }
        },
        new NavigationItem
        {
            Name = "Document Space",
            Path = "#",
            Icon = Icons.Material.Filled.Folder,
            RequiredPrivileges = new[] { "DOCUMENT_SPACE_USER", "DASHBOARD_ADMIN" },
            HideCondition = enclave => enclave == "IL2",
            Children = new List<NavigationItem>
            {
                new() { Name = "Spaces", Path = "/document-spaces", RequiredPrivileges = new[] { "DOCUMENT_SPACE_USER", "DASHBOARD_ADMIN" } },
                new() { Name = "My Recent Uploads", Path = "/document-spaces/recents", RequiredPrivileges = new[] { "DOCUMENT_SPACE_USER", "DASHBOARD_ADMIN" } },
                new() { Name = "Favorites", Path = "/document-spaces/favorites", RequiredPrivileges = new[] { "DOCUMENT_SPACE_USER", "DASHBOARD_ADMIN" } },
                new() { Name = "Archived Files", Path = "/document-spaces/archived", RequiredPrivileges = new[] { "DOCUMENT_SPACE_USER", "DASHBOARD_ADMIN" } }
            }
        },
        new NavigationItem
        {
            Name = "System",
            Path = "#",
            Icon = Icons.Material.Filled.Settings,
            RequiredPrivileges = new[] { "DASHBOARD_ADMIN" },
            Children = new List<NavigationItem>
            {
                new() { Name = "Dashboard Users", Path = "/dashboard-users", RequiredPrivileges = new[] { "DASHBOARD_ADMIN" } },
                new() { Name = "Logfile", Path = "/logs", RequiredPrivileges = new[] { "DASHBOARD_ADMIN" } },
                new() { Name = "Audit Logs", Path = "/audit-logs", RequiredPrivileges = new[] { "DASHBOARD_ADMIN" } },
                new() { Name = "KPI", Path = "/kpi", RequiredPrivileges = new[] { "DASHBOARD_ADMIN" } }
            }
        }
    };
    
    public IEnumerable<NavigationItem> GetMenuItems(ClaimsPrincipal user, string? enclave = null)
    {
        return _menuItems
            .Where(item => UserHasPrivilege(user, item.RequiredPrivileges))
            .Where(item => item.HideCondition?.Invoke(enclave ?? string.Empty) != true)
            .Select(item => new NavigationItem
            {
                Name = item.Name,
                Path = item.Path,
                Icon = item.Icon,
                RequiredPrivileges = item.RequiredPrivileges,
                Children = item.Children?
                    .Where(child => UserHasPrivilege(user, child.RequiredPrivileges))
                    .ToList()
            })
            .Where(item => item.Children?.Any() != false || item.Path != "#");
    }
    
    private bool UserHasPrivilege(ClaimsPrincipal user, string[] requiredPrivileges)
    {
        if (requiredPrivileges.Length == 0) return true;
        
        return requiredPrivileges.Any(privilege =>
            user.HasClaim("Privilege", privilege));
    }
}
```

### Navigation Menu Component

```razor
@* Shared/NavMenu.razor *@
@inject NavigationService NavService
@inject AuthenticationStateProvider AuthStateProvider

<MudNavMenu>
    @foreach (var item in _menuItems)
    {
        @if (item.Children?.Any() == true)
        {
            <MudNavGroup Title="@item.Name" Icon="@item.Icon" Expanded="false">
                @foreach (var child in item.Children)
                {
                    <MudNavLink Href="@child.Path" Icon="@child.Icon">
                        @child.Name
                    </MudNavLink>
                }
            </MudNavGroup>
        }
        else
        {
            <MudNavLink Href="@item.Path" Icon="@item.Icon">
                @item.Name
            </MudNavLink>
        }
    }
</MudNavMenu>

@code {
    private List<NavigationItem> _menuItems = new();
    
    protected override async Task OnInitializedAsync()
    {
        var authState = await AuthStateProvider.GetAuthenticationStateAsync();
        _menuItems = NavService.GetMenuItems(authState.User).ToList();
    }
}
```

## Programmatic Navigation

### NavigationManager Usage

```csharp
// In component
@inject NavigationManager Navigation

@code {
    private void NavigateToHome()
    {
        Navigation.NavigateTo("/");
    }
    
    private void NavigateToUser(string userId)
    {
        Navigation.NavigateTo($"/dashboard-users/{userId}");
    }
    
    private void NavigateWithQueryString()
    {
        Navigation.NavigateTo("/app-clients?filter=active");
    }
    
    private void ForceReload()
    {
        Navigation.NavigateTo("/current-page", forceLoad: true);
    }
}
```

## Route Fallback (404 Handling)

### App.razor Configuration

```razor
<Router AppAssembly="@typeof(App).Assembly">
    <Found Context="routeData">
        <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)">
            <NotAuthorized>
                <NotAuthorizedPage />
            </NotAuthorized>
            <Authorizing>
                <MudProgressCircular Indeterminate="true" />
            </Authorizing>
        </AuthorizeRouteView>
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <NotFoundPage />
        </LayoutView>
    </NotFound>
</Router>
```

## POC Route Implementation

For POC, implement only:

✅ **Required Routes:**
- `/` - HomePage (stub)
- `/dashboard-users` - DashboardUserPage (full implementation)
- `/not-authorized` - NotAuthorizedPage
- `/not-found` - NotFoundPage

⚠️ **Stub Routes (no implementation, just navigation):**
- `/health` - HealthPage (stub with message "Coming soon")
- All other routes - Redirect to HomePage or show stub

### POC App.razor

```razor
<Router AppAssembly="@typeof(App).Assembly">
    <Found Context="routeData">
        <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)">
            <NotAuthorized>
                @if (context.User.Identity?.IsAuthenticated == true)
                {
                    <NotAuthorizedPage />
                }
                else
                {
                    <RedirectToLogin />
                }
            </NotAuthorized>
            <Authorizing>
                <PageLayout PageTitle="Loading...">
                    <MudStack AlignItems="AlignItems.Center" Justify="Justify.Center" Style="height: 400px;">
                        <MudProgressCircular Indeterminate="true" Size="Size.Large" />
                        <MudText>Authenticating...</MudText>
                    </MudStack>
                </PageLayout>
            </Authorizing>
        </AuthorizeRouteView>
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <NotFoundPage />
        </LayoutView>
    </NotFound>
</Router>
```

## Migration Checklist

### POC Phase
- [x] Define `/dashboard-users` route
- [x] Create authorization policies
- [x] Implement NotAuthorizedPage
- [x] Implement NotFoundPage
- [x] Stub HomePage
- [x] Test navigation
- [ ] Test authorization enforcement

### Phase 1
- [ ] Migrate all core page routes
- [ ] Implement full navigation menu
- [ ] Test all navigation paths
- [ ] Ensure authorization works for all routes

### Phase 2-4
- [ ] Migrate remaining routes
- [ ] Implement dynamic routes with parameters
- [ ] Test query string handling
- [ ] Verify deep linking works

---

**Next Step**: Proceed to 05-STATE-MIGRATION.md
