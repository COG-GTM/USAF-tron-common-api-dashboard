# Test Strategy: Jest/Cypress → xUnit/bUnit/Playwright

## Overview

Migrate and enhance test coverage from React testing stack to Blazor testing stack.

## Current Test Coverage

### Existing Tests

**Unit Tests (Jest + React Testing Library):**
- Location: `src/**/__tests__/` directories
- Framework: Jest with React Testing Library
- Coverage: Primarily component tests, some utility tests
- Count: ~15-20 test files

**E2E Tests (Cypress):**
- Location: `cypress/integration/`
- Count: 20+ test files
- Coverage: Full user workflows, CRUD operations
- Examples:
  - `dashboard-users_app-client-dev_spec.ts`
  - `app-client-app-source_spec.ts`
  - `document-space/` (multiple files)

### Coverage Gaps

❌ **Service layer untested** - No unit tests for business logic
❌ **State management untested** - Hookstate services not tested
❌ **Validation utilities partially tested**
❌ **API integration untested** (mocked in all tests)

## Target Test Stack

### 1. xUnit - Unit Testing Framework

**Purpose**: Unit tests for services, utilities, and business logic

**Setup:**
```bash
dotnet new xunit -n TronDashboard.Tests -f net9.0
dotnet sln add TronDashboard.Tests/TronDashboard.Tests.csproj
dotnet add TronDashboard.Tests reference TronDashboard.Core/TronDashboard.Core.csproj
```

**Packages:**
```xml
<PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.9.0" />
<PackageReference Include="xunit" Version="2.6.6" />
<PackageReference Include="xunit.runner.visualstudio" Version="2.5.6" />
<PackageReference Include="Moq" Version="4.20.70" />
<PackageReference Include="FluentAssertions" Version="6.12.0" />
<PackageReference Include="coverlet.collector" Version="6.0.0" />
```

### 2. bUnit - Blazor Component Testing

**Purpose**: Component tests (equivalent to React Testing Library)

**Setup:**
```bash
dotnet add TronDashboard.Tests package bUnit
dotnet add TronDashboard.Tests package bUnit.web
```

**Packages:**
```xml
<PackageReference Include="bUnit" Version="1.26.64" />
<PackageReference Include="bUnit.web" Version="1.26.64" />
<PackageReference Include="bUnit.generators" Version="1.26.64" />
```

### 3. Playwright - E2E Testing

**Purpose**: End-to-end tests (replacing Cypress)

**Setup:**
```bash
# Create separate E2E project
dotnet new nunit -n TronDashboard.E2E -f net9.0
dotnet add TronDashboard.E2E package Microsoft.Playwright
dotnet add TronDashboard.E2E package Microsoft.Playwright.NUnit

# Install Playwright browsers
pwsh bin/Debug/net9.0/playwright.ps1 install
```

**Packages:**
```xml
<PackageReference Include="Microsoft.Playwright" Version="1.41.0" />
<PackageReference Include="Microsoft.Playwright.NUnit" Version="1.41.0" />
```

## Test Patterns

### xUnit: Service Layer Tests

**Pattern:**
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
            new() { Id = "1", Email = "test@example.com", Privileges = new() }
        };
        var response = new WrappedResponse<List<DashboardUserDto>> { Data = dtos };
        
        _mockApi
            .Setup(x => x.GetAllDashboardUsersWrappedAsync(It.IsAny<CancellationToken>()))
            .ReturnsAsync(response);
        
        // Act
        await _service.FetchAndStoreDataAsync();
        
        // Assert
        _service.DashboardUsers.Should().HaveCount(1);
        _service.DashboardUsers[0].Email.Should().Be("test@example.com");
    }
    
    [Fact]
    public async Task CreateAsync_AddsToCollection()
    {
        // Arrange
        var dto = new DashboardUserDto { Email = "new@example.com", Privileges = new() };
        var created = new DashboardUserDto { Id = "123", Email = "new@example.com", Privileges = new() };
        
        _mockApi
            .Setup(x => x.AddDashboardUserAsync(dto, It.IsAny<CancellationToken>()))
            .ReturnsAsync(created);
        
        // Act
        var result = await _service.CreateAsync(dto);
        
        // Assert
        result.Should().NotBeNull();
        result.Id.Should().Be("123");
        _service.DashboardUsers.Should().ContainSingle()
            .Which.Email.Should().Be("new@example.com");
    }
    
    [Fact]
    public async Task CreateAsync_ThrowsOnApiError()
    {
        // Arrange
        var dto = new DashboardUserDto { Email = "test@example.com" };
        _mockApi
            .Setup(x => x.AddDashboardUserAsync(dto, It.IsAny<CancellationToken>()))
            .ThrowsAsync(new ApiException("API Error", 400, "", null, null));
        
        // Act & Assert
        await _service.Invoking(s => s.CreateAsync(dto))
            .Should().ThrowAsync<DataCrudException>()
            .WithMessage("*API Error*");
    }
    
    [Theory]
    [InlineData("test@example.com", true)]
    [InlineData("", false)]
    public void ConvertToDto_HandlesEmail(string email, bool hasValue)
    {
        // Arrange
        var flat = new DashboardUserFlat { Email = email, HasDashboardAdmin = true };
        
        // Act
        var dto = _service.ConvertToDto(flat);
        
        // Assert
        dto.Email.Should().Be(email);
        if (hasValue)
            dto.Email.Should().NotBeEmpty();
    }
}
```

### bUnit: Component Tests

**Pattern:**
```csharp
public class TronTextInputTests : TestContext
{
    [Fact]
    public void TronTextInput_RendersWithLabel()
    {
        // Arrange
        var cut = RenderComponent<TronTextInput>(parameters => parameters
            .Add(p => p.Label, "Email")
            .Add(p => p.Value, "test@example.com"));
        
        // Assert
        cut.Find("input").GetAttribute("value").Should().Be("test@example.com");
        cut.MarkupMatches(@"
            <div class=""mud-input-control"">
                <label>Email</label>
                <input value=""test@example.com"" />
            </div>
        ");
    }
    
    [Fact]
    public void TronTextInput_DisplaysError()
    {
        // Arrange
        var cut = RenderComponent<TronTextInput>(parameters => parameters
            .Add(p => p.Error, true)
            .Add(p => p.ErrorText, "Invalid email"));
        
        // Assert
        var errorElement = cut.Find(".mud-input-error");
        errorElement.TextContent.Should().Contain("Invalid email");
    }
    
    [Fact]
    public void TronTextInput_TriggersValueChanged()
    {
        // Arrange
        var valueChanged = false;
        var newValue = "";
        
        var cut = RenderComponent<TronTextInput>(parameters => parameters
            .Add(p => p.Value, "")
            .Add(p => p.ValueChanged, EventCallback.Factory.Create<string>(this, value =>
            {
                valueChanged = true;
                newValue = value;
            })));
        
        // Act
        cut.Find("input").Change("new@example.com");
        
        // Assert
        valueChanged.Should().BeTrue();
        newValue.Should().Be("new@example.com");
    }
}

public class DashboardUserPageTests : TestContext
{
    private readonly Mock<IDashboardUserService> _mockService;
    
    public DashboardUserPageTests()
    {
        _mockService = new Mock<IDashboardUserService>();
        Services.AddSingleton(_mockService.Object);
        Services.AddMudServices();
        
        // Mock auth
        var authContext = this.AddTestAuthorization();
        authContext.SetAuthorized("testuser");
        authContext.SetClaims(new Claim("Privilege", "DASHBOARD_ADMIN"));
    }
    
    [Fact]
    public void DashboardUserPage_DisplaysLoadingState()
    {
        // Arrange
        _mockService.Setup(x => x.IsLoading).Returns(true);
        _mockService.Setup(x => x.DashboardUsers).Returns(new ObservableCollection<DashboardUserFlat>());
        
        // Act
        var cut = RenderComponent<DashboardUserPage>();
        
        // Assert
        cut.Find(".mud-progress-linear").Should().NotBeNull();
    }
    
    [Fact]
    public async Task DashboardUserPage_LoadsUsers()
    {
        // Arrange
        var users = new ObservableCollection<DashboardUserFlat>
        {
            new() { Id = "1", Email = "test@example.com", HasDashboardAdmin = true }
        };
        
        _mockService.Setup(x => x.DashboardUsers).Returns(users);
        _mockService.Setup(x => x.IsLoading).Returns(false);
        _mockService
            .Setup(x => x.FetchAndStoreDataAsync(It.IsAny<CancellationToken>()))
            .ReturnsAsync(users.ToList());
        
        // Act
        var cut = RenderComponent<DashboardUserPage>();
        await Task.Delay(100); // Allow async operations to complete
        
        // Assert
        cut.WaitForAssertion(() => 
            cut.Markup.Should().Contain("test@example.com"));
    }
    
    [Fact]
    public async Task DashboardUserPage_OpensCreateDialog()
    {
        // Arrange
        _mockService.Setup(x => x.DashboardUsers).Returns(new ObservableCollection<DashboardUserFlat>());
        _mockService.Setup(x => x.IsLoading).Returns(false);
        
        var cut = RenderComponent<DashboardUserPage>();
        
        // Act
        var addButton = cut.Find("button:contains('Add New User')");
        addButton.Click();
        
        // Assert
        // Verify dialog opened (depends on MudBlazor dialog implementation)
        cut.WaitForAssertion(() =>
            cut.FindAll(".mud-dialog").Should().HaveCount(1));
    }
}
```

### Playwright: E2E Tests

**Pattern:**
```csharp
[TestFixture]
public class DashboardUserE2ETests : PageTest
{
    private const string BaseUrl = "http://localhost:5000";
    
    [SetUp]
    public async Task Setup()
    {
        // Navigate to app and authenticate
        await Page.GotoAsync(BaseUrl);
        
        // Mock authentication or perform real login
        await AuthenticateAsync();
    }
    
    private async Task AuthenticateAsync()
    {
        // Add authentication cookies/tokens
        await Context.AddCookiesAsync(new[]
        {
            new Cookie
            {
                Name = "auth_token",
                Value = "test_token",
                Domain = "localhost",
                Path = "/"
            }
        });
        
        await Page.ReloadAsync();
    }
    
    [Test]
    public async Task CanCreateDashboardUser()
    {
        // Navigate to dashboard users page
        await Page.GotoAsync($"{BaseUrl}/dashboard-users");
        
        // Click "Add New User" button
        await Page.ClickAsync("button:has-text('Add New User')");
        
        // Wait for dialog
        await Page.WaitForSelectorAsync(".mud-dialog");
        
        // Fill form
        await Page.FillAsync("input[name='email']", "newuser@example.com");
        await Page.ClickAsync("input[type='checkbox'][name='dashboardAdmin']");
        
        // Submit
        await Page.ClickAsync("button:has-text('Create')");
        
        // Wait for success message
        await Page.WaitForSelectorAsync(".mud-alert-success");
        
        // Verify user appears in grid
        await Expect(Page.Locator("text=newuser@example.com")).ToBeVisibleAsync();
    }
    
    [Test]
    public async Task CanEditDashboardUser()
    {
        // Navigate to dashboard users page
        await Page.GotoAsync($"{BaseUrl}/dashboard-users");
        
        // Click on a user row
        await Page.ClickAsync("tr:has-text('test@example.com')");
        
        // Wait for edit dialog
        await Page.WaitForSelectorAsync(".mud-dialog");
        
        // Modify email
        await Page.FillAsync("input[name='email']", "modified@example.com");
        
        // Submit
        await Page.ClickAsync("button:has-text('Update')");
        
        // Wait for success
        await Page.WaitForSelectorAsync(".mud-alert-success");
        
        // Verify updated email
        await Expect(Page.Locator("text=modified@example.com")).ToBeVisibleAsync();
    }
    
    [Test]
    public async Task CanDeleteDashboardUser()
    {
        // Navigate to dashboard users page
        await Page.GotoAsync($"{BaseUrl}/dashboard-users");
        
        // Find user row and click delete button
        var userRow = Page.Locator("tr:has-text('delete-me@example.com')");
        await userRow.Locator("button[aria-label='Delete']").ClickAsync();
        
        // Confirm deletion
        await Page.ClickAsync("button:has-text('Delete')");
        
        // Verify user removed
        await Expect(userRow).Not.ToBeVisibleAsync();
        
        // Verify success message
        await Expect(Page.Locator(".mud-alert-success")).ToBeVisibleAsync();
    }
    
    [Test]
    public async Task FormValidationWorks()
    {
        await Page.GotoAsync($"{BaseUrl}/dashboard-users");
        await Page.ClickAsync("button:has-text('Add New User')");
        
        // Submit empty form
        await Page.ClickAsync("button:has-text('Create')");
        
        // Verify validation errors
        await Expect(Page.Locator("text=Email is required")).ToBeVisibleAsync();
        
        // Enter invalid email
        await Page.FillAsync("input[name='email']", "invalid-email");
        
        // Verify email validation
        await Expect(Page.Locator("text=Email is not valid")).ToBeVisibleAsync();
        
        // Submit button should be disabled
        await Expect(Page.Locator("button:has-text('Create')")).ToBeDisabledAsync();
    }
}
```

## Test Coverage Goals

### POC Phase

| Category | Target | Tests |
|----------|--------|-------|
| Service Layer | 80% | DashboardUserService (10+ tests) |
| Components | 50% | TronTextInput, TronCheckbox, DashboardUserPage (5+ tests) |
| E2E | 1 workflow | Create → Edit → Delete (1 test) |

### Phase 1

| Category | Target | Tests |
|----------|--------|-------|
| Service Layer | 80% | All 7 core services |
| Components | 60% | All form components + core pages |
| E2E | 5 workflows | Core CRUD operations |

### Phases 2-4

| Category | Target | Tests |
|----------|--------|-------|
| Service Layer | 85% | All 22 services |
| Components | 65% | All components |
| E2E | 20+ workflows | All critical paths |

## Test Organization

```
TronDashboard.Tests/
├── Services/
│   ├── DashboardUserServiceTests.cs
│   ├── AppClientServiceTests.cs
│   └── ...
├── Components/
│   ├── TronTextInputTests.cs
│   ├── TronCheckboxTests.cs
│   └── ...
├── Pages/
│   ├── DashboardUserPageTests.cs
│   └── ...
├── Validation/
│   ├── TronEmailAddressAttributeTests.cs
│   └── ...
└── Utilities/
    └── ...

TronDashboard.E2E/
├── Tests/
│   ├── DashboardUserE2ETests.cs
│   ├── AppClientE2ETests.cs
│   └── ...
├── PageObjects/
│   ├── DashboardUserPage.cs
│   └── ...
└── Support/
    ├── AuthHelpers.cs
    └── TestDataHelpers.cs
```

## Continuous Integration

### GitHub Actions Workflow

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '9.0.x'
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build
      run: dotnet build --no-restore
    
    - name: Unit Tests
      run: dotnet test TronDashboard.Tests/TronDashboard.Tests.csproj --no-build --verbosity normal --collect:"XPlat Code Coverage"
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
    
    - name: Install Playwright
      run: pwsh TronDashboard.E2E/bin/Debug/net9.0/playwright.ps1 install --with-deps
    
    - name: E2E Tests
      run: dotnet test TronDashboard.E2E/TronDashboard.E2E.csproj --no-build
    
    - name: Upload Playwright artifacts
      if: always()
      uses: actions/upload-artifact@v3
      with:
        name: playwright-results
        path: TronDashboard.E2E/test-results/
```

## Test Data Management

### Test Fixtures

```csharp
public class TestDataFixtures
{
    public static DashboardUserDto CreateTestUser(string? email = null, bool isAdmin = false)
    {
        return new DashboardUserDto
        {
            Id = Guid.NewGuid().ToString(),
            Email = email ?? $"test-{Guid.NewGuid()}@example.com",
            Privileges = isAdmin 
                ? new List<PrivilegeDto> { new() { Id = "1", Name = "DASHBOARD_ADMIN" } }
                : new List<PrivilegeDto>()
        };
    }
    
    public static List<DashboardUserDto> CreateTestUsers(int count)
    {
        return Enumerable.Range(0, count)
            .Select(_ => CreateTestUser())
            .ToList();
    }
}
```

## Migration Checklist

### POC Phase
- [ ] Set up xUnit test project
- [ ] Install bUnit and MudBlazor test dependencies
- [ ] Write 10+ service layer tests
- [ ] Write 5+ component tests
- [ ] Set up Playwright test project
- [ ] Write 1 E2E workflow test
- [ ] Configure code coverage reporting
- [ ] Verify all tests pass in CI

### Phase 1
- [ ] Port Jest tests to xUnit
- [ ] Port React Testing Library tests to bUnit
- [ ] Port 5 Cypress tests to Playwright
- [ ] Achieve 80% service layer coverage
- [ ] Achieve 60% component coverage

### Phases 2-4
- [ ] Port all remaining Cypress tests
- [ ] Add integration tests
- [ ] Performance testing
- [ ] Accessibility testing
- [ ] Security testing

---

**End of Migration Documentation**

All migration mapping documents complete. Proceed with POC implementation.
