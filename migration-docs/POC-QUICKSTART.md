# POC Quick Start Guide

Get the DashboardUser POC running in 3 weeks.

## Week 1: Foundation (Days 1-5)

### Day 1: Project Setup

```bash
# Navigate to project root
cd /Users/jakecosme/Documents/GitHub/USAF-tron-common-api-dashboard

# Create Blazor solution structure
mkdir -p TronDashboard.Blazor
cd TronDashboard.Blazor

# Create projects
dotnet new blazorserver -n TronDashboard.Web -f net9.0
dotnet new classlib -n TronDashboard.Core -f net9.0
dotnet new xunit -n TronDashboard.Tests -f net9.0
dotnet new nunit -n TronDashboard.E2E -f net9.0

# Create solution
dotnet new sln -n TronDashboard

# Add projects to solution
dotnet sln add TronDashboard.Web/TronDashboard.Web.csproj
dotnet sln add TronDashboard.Core/TronDashboard.Core.csproj
dotnet sln add TronDashboard.Tests/TronDashboard.Tests.csproj
dotnet sln add TronDashboard.E2E/TronDashboard.E2E.csproj

# Add project references
dotnet add TronDashboard.Web reference TronDashboard.Core
dotnet add TronDashboard.Tests reference TronDashboard.Core
dotnet add TronDashboard.Tests reference TronDashboard.Web

# Add NuGet packages
cd TronDashboard.Web
dotnet add package MudBlazor --version 7.1.0
dotnet add package Radzen.Blazor --version 5.0.0

cd ../TronDashboard.Core
dotnet add package NSwag.MSBuild --version 14.0.7
dotnet add package System.ComponentModel.Annotations

cd ../TronDashboard.Tests
dotnet add package bUnit --version 1.26.64
dotnet add package bUnit.web --version 1.26.64
dotnet add package Moq --version 4.20.70
dotnet add package FluentAssertions --version 6.12.0

cd ../TronDashboard.E2E
dotnet add package Microsoft.Playwright --version 1.41.0
dotnet add package Microsoft.Playwright.NUnit --version 1.41.0

# Build to verify
cd ..
dotnet build
```

**Verify:** `dotnet run --project TronDashboard.Web` should start a blank Blazor app on https://localhost:5001

### Day 2: API Client Generation

```bash
cd TronDashboard.Core

# Copy OpenAPI spec from React project
cp ../../resources/tron-common-api.json ./tron-common-api.json

# Create nswag.json (copy content from 07-API-CLIENT-SETUP.md)
# Then generate client
dotnet build

# Verify GeneratedApiClient.cs was created
ls ApiClient/GeneratedApiClient.cs
```

**Verify:** Check that `IDashboardUserControllerApi` interface exists in generated file.

### Day 3: Authentication Setup

Create `TronDashboard.Core/Authentication/TronAuthenticationStateProvider.cs`:

```csharp
using Microsoft.AspNetCore.Components.Authorization;
using System.Security.Claims;
using TronDashboard.Core.ApiClient;
using TronDashboard.Core.Services;

namespace TronDashboard.Core.Authentication;

public class TronAuthenticationStateProvider : AuthenticationStateProvider
{
    private readonly IAuthorizedUserService _authorizedUserService;
    
    public TronAuthenticationStateProvider(IAuthorizedUserService authorizedUserService)
    {
        _authorizedUserService = authorizedUserService;
    }
    
    public override async Task<AuthenticationState> GetAuthenticationStateAsync()
    {
        try
        {
            var user = await _authorizedUserService.FetchAndStoreAuthorizedUserAsync();
            var identity = CreateIdentityFromUser(user);
            return new AuthenticationState(new ClaimsPrincipal(identity));
        }
        catch
        {
            // Not authenticated
            return new AuthenticationState(new ClaimsPrincipal(new ClaimsIdentity()));
        }
    }
    
    private ClaimsIdentity CreateIdentityFromUser(DashboardUserDto user)
    {
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.Email, user.Email ?? string.Empty),
            new Claim(ClaimTypes.NameIdentifier, user.Id ?? string.Empty)
        };
        
        if (user.Privileges != null)
        {
            foreach (var privilege in user.Privileges)
            {
                claims.Add(new Claim("Privilege", privilege.Name ?? string.Empty));
            }
        }
        
        return new ClaimsIdentity(claims, "TronAuth");
    }
}
```

Update `TronDashboard.Web/Program.cs`:

```csharp
using Microsoft.AspNetCore.Components.Authorization;
using MudBlazor.Services;
using TronDashboard.Core.Authentication;
using TronDashboard.Core.Services;
using TronDashboard.Core.ApiClient;

var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();
builder.Services.AddMudServices();

// Configure API HttpClient
var apiBaseUrl = builder.Configuration["ApiSettings:BaseUrl"] ?? "http://localhost:8088/";
var apiPathPrefix = builder.Configuration["ApiSettings:PathPrefix"] ?? "api";
var fullBaseUrl = $"{apiBaseUrl}{apiPathPrefix}";

builder.Services.AddHttpClient("TronApi", client =>
{
    client.BaseAddress = new Uri(fullBaseUrl);
    client.Timeout = TimeSpan.FromSeconds(30);
});

// Register API clients
builder.Services.AddScoped<IDashboardUserControllerApi>(sp =>
{
    var httpClientFactory = sp.GetRequiredService<IHttpClientFactory>();
    var httpClient = httpClientFactory.CreateClient("TronApi");
    return new DashboardUserControllerApi(httpClient);
});

// Register services
builder.Services.AddScoped<IAuthorizedUserService, AuthorizedUserService>();
builder.Services.AddScoped<IDashboardUserService, DashboardUserService>();

// Register authentication
builder.Services.AddScoped<AuthenticationStateProvider, TronAuthenticationStateProvider>();
builder.Services.AddAuthorizationCore(options =>
{
    options.AddPolicy("RequireDashboardAdmin", policy =>
        policy.RequireClaim("Privilege", "DASHBOARD_ADMIN"));
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.MapBlazorHub();
app.MapFallbackToPage("/_Host");

app.Run();
```

Add to `appsettings.Development.json`:

```json
{
  "ApiSettings": {
    "BaseUrl": "http://localhost:8088/",
    "PathPrefix": "api"
  }
}
```

### Day 4: Base Components

Create `TronDashboard.Web/Shared/PageLayout.razor` (copy from 03-COMPONENT-MAPPING.md)

Create `TronDashboard.Web/Components/`:
- `TronTextInput.razor`
- `TronCheckbox.razor`
- `TronAlert.razor`
- `TronSubmitActions.razor`

### Day 5: Service Layer

Create `TronDashboard.Core/Services/IDashboardUserService.cs` and implementation (copy from 02-SERVICE-MAPPING.md)

Create `TronDashboard.Core/Models/DashboardUserFlat.cs`:

```csharp
namespace TronDashboard.Core.Models;

public class DashboardUserFlat
{
    public string? Id { get; set; }
    public string Email { get; set; } = string.Empty;
    public bool HasDashboardAdmin { get; set; }
}
```

Write xUnit tests in `TronDashboard.Tests/Services/DashboardUserServiceTests.cs`

**End of Week 1 Deliverable:** Foundation infrastructure complete, all tests passing.

---

## Week 2: UI Implementation (Days 6-10)

### Day 6-7: Grid Page

Create `TronDashboard.Web/Pages/DashboardUser/DashboardUserPage.razor` (copy full implementation from 03-COMPONENT-MAPPING.md)

**Test:** Can navigate to `/dashboard-users`, see grid (mocked data for now)

### Day 8-9: Form Implementation

Create `TronDashboard.Web/Pages/DashboardUser/DashboardUserForm.razor`

**Test:** Can open create dialog, see form fields, validation works

### Day 10: Integration

Wire up CRUD operations to service.

**Test:** Full workflow - create → edit → delete works end-to-end

**End of Week 2 Deliverable:** Working CRUD page with all operations.

---

## Week 3: Testing & Review (Days 11-15)

### Day 11-12: Unit Tests

Complete service tests (10+ tests).
Write component tests (5+ tests with bUnit).

### Day 13: E2E Test

Set up Playwright, write full CRUD workflow test.

### Day 14: Documentation

Document patterns, challenges, deviations from plan.

### Day 15: POC Review

Prepare demo, present to stakeholders, make Go/No-Go decision.

**End of Week 3 Deliverable:** Complete POC with tests, documentation, and decision.

---

## Common Commands

```bash
# Build solution
dotnet build

# Run web app
dotnet run --project TronDashboard.Web

# Run tests
dotnet test TronDashboard.Tests

# Run E2E tests
dotnet test TronDashboard.E2E

# Generate code coverage
dotnet test TronDashboard.Tests --collect:"XPlat Code Coverage"

# Regenerate API client
cd TronDashboard.Core && dotnet build
```

---

## Troubleshooting

### API Client Not Generating
- Check `nswag.json` is in `TronDashboard.Core/`
- Check `tron-common-api.json` exists
- Run `dotnet build` in Core project
- Check build output for NSwag errors

### Authentication Failing
- Verify backend is running on port 8088
- Check `appsettings.Development.json` has correct API URL
- Add logging to `TronAuthenticationStateProvider`

### Components Not Rendering
- Verify MudBlazor services registered in `Program.cs`
- Check `_Host.cshtml` includes MudBlazor CSS/JS
- Clear browser cache

### Tests Failing
- Check mock setup matches actual interface
- Verify async test patterns (use `await`)
- Check test data is valid

---

## Success Checklist

At end of 3 weeks, verify:

- [ ] Blazor app runs on https://localhost:5001
- [ ] Can navigate to `/dashboard-users`
- [ ] Grid displays users (from API or mocked)
- [ ] Can create new user
- [ ] Can edit existing user
- [ ] Can delete user
- [ ] Form validation works
- [ ] Success/error messages display
- [ ] Authorization enforces DASHBOARD_ADMIN
- [ ] 10+ service tests pass
- [ ] 5+ component tests pass
- [ ] 1 E2E test passes
- [ ] Code builds without warnings
- [ ] Documentation updated

**If all checked: POC SUCCESS → Proceed with full migration**

**If blockers: Document issues → Adjust approach**
