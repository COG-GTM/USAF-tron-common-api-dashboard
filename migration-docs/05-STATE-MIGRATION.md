# State Management Migration: Hookstate → Blazor

## Overview

This document explains how to migrate from Hookstate reactive state management to Blazor's state management patterns.

## Current Pattern: Hookstate

### Hookstate Architecture

**Global State Atoms:**
```typescript
// src/state/dashboard-user/dashboard-user-state.ts
const dashboardUserState = createState<DashboardUserFlat[]>([]);
const dashboardUserDtoCache = createState<Record<string, DashboardUserDto>>({});
```

**Service Wrapper:**
```typescript
export const useDashboardUserState = () => wrapDashboardUserState(
    useState(dashboardUserState),
    dashboardUserApi,
    useState(dashboardUserDtoCache)
);
```

**Component Consumption:**
```typescript
const dashboardUserService = useDashboardUserState();

useEffect(() => {
    dashboardUserService.fetchAndStoreData();
}, []);

// State automatically reactive
return (
    <div>
        {dashboardUserService.dashboardUsers.map(user => ...)}
    </div>
);
```

### Hookstate Features Used

1. **Reactive State** - Changes propagate automatically to UI
2. **Promised State** - `state.promised` tracks async operations
3. **Error State** - `state.error` captures errors
4. **Nested State** - `state[index].set(value)` for arrays
5. **Plugins** - `Validation`, `Initial`, `Touched` for forms

## Blazor State Management Patterns

### Pattern 1: Scoped Services (Recommended)

Best match for current Hookstate architecture. Service lifetime matches component tree lifetime.

**Service Definition:**
```csharp
public interface IDashboardUserService
{
    ObservableCollection<DashboardUserFlat> DashboardUsers { get; }
    bool IsLoading { get; }
    string? Error { get; }
    event EventHandler? StateChanged;
    
    Task<List<DashboardUserFlat>> FetchAndStoreDataAsync(CancellationToken ct = default);
    // ... other methods
}

public class DashboardUserService : IDashboardUserService, INotifyPropertyChanged
{
    private readonly ObservableCollection<DashboardUserFlat> _dashboardUsers = new();
    private bool _isLoading;
    private string? _error;
    
    public event PropertyChangedEventHandler? PropertyChanged;
    public event EventHandler? StateChanged;
    
    public ObservableCollection<DashboardUserFlat> DashboardUsers => _dashboardUsers;
    
    public bool IsLoading
    {
        get => _isLoading;
        private set
        {
            if (_isLoading != value)
            {
                _isLoading = value;
                OnPropertyChanged();
                OnStateChanged();
            }
        }
    }
    
    public string? Error
    {
        get => _error;
        private set
        {
            if (_error != value)
            {
                _error = value;
                OnPropertyChanged();
                OnStateChanged();
            }
        }
    }
    
    public async Task<List<DashboardUserFlat>> FetchAndStoreDataAsync(CancellationToken ct = default)
    {
        IsLoading = true;
        Error = null;
        
        try
        {
            var response = await _api.GetAllDashboardUsersWrappedAsync(ct);
            var flats = ConvertDashboardUsersToFlat(response.Data);
            
            _dashboardUsers.Clear();
            foreach (var flat in flats)
                _dashboardUsers.Add(flat);
            
            return flats;
        }
        catch (Exception ex)
        {
            Error = ex.Message;
            return new List<DashboardUserFlat>();
        }
        finally
        {
            IsLoading = false;
        }
    }
    
    protected void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
    
    protected void OnStateChanged()
    {
        StateChanged?.Invoke(this, EventArgs.Empty);
    }
}
```

**DI Registration (Program.cs):**
```csharp
// Scoped = per-user session in Blazor Server
builder.Services.AddScoped<IDashboardUserService, DashboardUserService>();
```

**Component Consumption:**
```razor
@inject IDashboardUserService DashboardUserService
@implements IDisposable

@if (DashboardUserService.IsLoading)
{
    <MudProgressCircular Indeterminate="true" />
}
else if (!string.IsNullOrEmpty(DashboardUserService.Error))
{
    <MudAlert Severity="Severity.Error">@DashboardUserService.Error</MudAlert>
}
else
{
    <RadzenDataGrid Data="@DashboardUserService.DashboardUsers" TItem="DashboardUserFlat">
        @* Grid columns *@
    </RadzenDataGrid>
}

@code {
    protected override async Task OnInitializedAsync()
    {
        DashboardUserService.StateChanged += OnServiceStateChanged;
        await DashboardUserService.FetchAndStoreDataAsync();
    }
    
    private void OnServiceStateChanged(object? sender, EventArgs e)
    {
        // Force UI re-render when service state changes
        InvokeAsync(StateHasChanged);
    }
    
    public void Dispose()
    {
        DashboardUserService.StateChanged -= OnServiceStateChanged;
    }
}
```

### Pattern 2: CascadingValue (For Cross-Component State)

Use when multiple components need same state without injection.

**State Container:**
```csharp
public class AppStateContainer
{
    public DashboardUserDto? CurrentUser { get; private set; }
    public event EventHandler? StateChanged;
    
    public void SetCurrentUser(DashboardUserDto? user)
    {
        CurrentUser = user;
        StateChanged?.Invoke(this, EventArgs.Empty);
    }
}
```

**App.razor:**
```razor
<CascadingValue Value="@_appState">
    <Router AppAssembly="@typeof(App).Assembly">
        @* Router content *@
    </Router>
</CascadingValue>

@code {
    private AppStateContainer _appState = new();
}
```

**Component Consumption:**
```razor
[CascadingParameter] public AppStateContainer AppState { get; set; } = default!;

@code {
    protected override void OnInitialized()
    {
        AppState.StateChanged += OnAppStateChanged;
    }
    
    private void OnAppStateChanged(object? sender, EventArgs e)
    {
        InvokeAsync(StateHasChanged);
    }
}
```

### Pattern 3: Local Component State

For state that doesn't need sharing, use simple fields.

```razor
@code {
    private List<DashboardUserFlat> _users = new();
    private bool _isLoading;
    private string? _errorMessage;
    
    protected override async Task OnInitializedAsync()
    {
        _isLoading = true;
        StateHasChanged(); // Force UI update
        
        try
        {
            _users = await DashboardUserService.FetchAndStoreDataAsync();
        }
        catch (Exception ex)
        {
            _errorMessage = ex.Message;
        }
        finally
        {
            _isLoading = false;
            StateHasChanged();
        }
    }
}
```

## Form State Migration

### Hookstate Form Pattern

```typescript
// Form state with plugins
const formState = useState<DashboardUserFlat>({
    id: props.data?.id || "",
    email: props.data?.email || "",
    hasDashboardAdmin: false,
});

formState.attach(Validation);
formState.attach(Initial);
formState.attach(Touched);

// Validation
Validation(formState.email).validate(validateEmail, validationErrors.invalidEmail, 'error');
Validation(formState.email).validate(validateRequiredString, validationErrors.requiredText, 'error');

// Check if modified
function isFormModified() {
    return Initial(formState.email).modified() || Initial(formState.hasDashboardAdmin).modified();
}

// Check validation
const isValid = Validation(formState).valid();
```

### Blazor Form Pattern with EditContext

```razor
<EditForm Model="@_model" OnValidSubmit="@HandleSubmit" OnInvalidSubmit="@HandleInvalidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />
    
    <MudTextField 
        @bind-Value="_model.Email"
        Label="Email"
        For="@(() => _model.Email)"
        Required="true"
        InputType="InputType.Email" />
    
    <MudCheckBox 
        @bind-Checked="_model.HasDashboardAdmin"
        Label="Dashboard Admin" />
    
    <TronSubmitActions 
        IsValid="@_editContext.Validate()"
        IsModified="@_editContext.IsModified()"
        IsSubmitting="@_isSubmitting"
        OnCancel="@Cancel" />
</EditForm>

@code {
    private DashboardUserFormModel _model = new();
    private EditContext _editContext = null!;
    private bool _isSubmitting;
    
    protected override void OnInitialized()
    {
        _model = new DashboardUserFormModel
        {
            Email = User?.Email ?? string.Empty,
            HasDashboardAdmin = User?.HasDashboardAdmin ?? false
        };
        
        _editContext = new EditContext(_model);
        _editContext.OnFieldChanged += HandleFieldChanged;
    }
    
    private void HandleFieldChanged(object? sender, FieldChangedEventArgs e)
    {
        // Trigger validation and UI update
        StateHasChanged();
    }
    
    private async Task HandleSubmit()
    {
        if (!_editContext.Validate())
            return;
        
        _isSubmitting = true;
        
        try
        {
            // Submit logic
        }
        finally
        {
            _isSubmitting = false;
        }
    }
    
    public class DashboardUserFormModel
    {
        [Required(ErrorMessage = "Email is required")]
        [EmailAddress(ErrorMessage = "Invalid email address")]
        [StringLength(255, MinimumLength = 1, ErrorMessage = "Email must be 1-255 characters")]
        public string Email { get; set; } = string.Empty;
        
        public bool HasDashboardAdmin { get; set; }
    }
}
```

## State Update Patterns

### Array Updates

**Hookstate:**
```typescript
// Add
this.state[this.state.length].set(newItem);

// Update
const index = this.state.get().findIndex(item => item.id === updatedItem.id);
this.state[index].set(updatedItem);

// Remove
this.state.find(item => item.id.get() === toDelete.id)?.set(none);
```

**Blazor ObservableCollection:**
```csharp
// Add
_dashboardUsers.Add(newItem);

// Update
var existing = _dashboardUsers.FirstOrDefault(x => x.Id == updatedItem.Id);
if (existing != null)
{
    var index = _dashboardUsers.IndexOf(existing);
    _dashboardUsers[index] = updatedItem;
}

// Remove
var toRemove = _dashboardUsers.FirstOrDefault(x => x.Id == id);
if (toRemove != null)
    _dashboardUsers.Remove(toRemove);
```

### Async State Updates

**Hookstate Promised State:**
```typescript
const data = Promise.resolve(apiCall());
this.state.set(data); // Sets state to promised

// Later: check if still loading
if (this.state.promised) {
    // Loading...
}
```

**Blazor Pattern:**
```csharp
public async Task<List<T>> FetchDataAsync()
{
    IsLoading = true;
    
    try
    {
        var data = await _api.GetDataAsync();
        _items.Clear();
        foreach (var item in data)
            _items.Add(item);
        return data;
    }
    catch (Exception ex)
    {
        Error = ex.Message;
        return new List<T>();
    }
    finally
    {
        IsLoading = false;
    }
}
```

## State Persistence (Optional)

### Blazor ProtectedSessionStorage / ProtectedLocalStorage

```csharp
@inject ProtectedSessionStorage SessionStorage

private async Task SaveStateAsync()
{
    await SessionStorage.SetAsync("dashboardUsers", _dashboardUsers.ToList());
}

private async Task LoadStateAsync()
{
    var result = await SessionStorage.GetAsync<List<DashboardUserFlat>>("dashboardUsers");
    if (result.Success && result.Value != null)
    {
        _dashboardUsers.Clear();
        foreach (var item in result.Value)
            _dashboardUsers.Add(item);
    }
}
```

## Reactive UI Updates

### Automatic Updates with ObservableCollection

```csharp
// ObservableCollection automatically notifies UI of changes
_dashboardUsers.Add(newUser);
// UI automatically updates - no StateHasChanged() needed (in most cases)
```

### Manual Updates with StateHasChanged()

```csharp
private async Task LoadDataAsync()
{
    _isLoading = true;
    StateHasChanged(); // Force UI update to show loading
    
    _users = await _service.FetchDataAsync();
    
    _isLoading = false;
    StateHasChanged(); // Force UI update to show data
}
```

### InvokeAsync for Thread-Safe Updates

```csharp
private void OnExternalEvent(object? sender, EventArgs e)
{
    // May be called from non-UI thread
    InvokeAsync(() =>
    {
        _someValue = newValue;
        StateHasChanged();
    });
}
```

## State Management Best Practices

### 1. Service Lifetime Selection

| Pattern | Lifetime | Use Case |
|---------|----------|----------|
| Singleton | App lifetime | Truly global state (config, static data) |
| Scoped | Per-user session | User-specific state (most services) |
| Transient | Per-request | Stateless operations |

**For this app: Use Scoped for all data services.**

### 2. ObservableCollection vs List

| Type | When to Use |
|------|------------|
| `ObservableCollection<T>` | Data binding to UI components (grids, lists) |
| `List<T>` | Internal data manipulation, DTOs |

### 3. Property Changed Notifications

```csharp
// Implement INotifyPropertyChanged for reactive properties
private string? _email;
public string? Email
{
    get => _email;
    set
    {
        if (_email != value)
        {
            _email = value;
            OnPropertyChanged();
            OnStateChanged(); // Custom event for manual subscribers
        }
    }
}
```

### 4. Dispose Pattern for Event Subscriptions

```razor
@implements IDisposable

@code {
    protected override void OnInitialized()
    {
        Service.StateChanged += OnServiceStateChanged;
    }
    
    public void Dispose()
    {
        Service.StateChanged -= OnServiceStateChanged;
    }
}
```

### 5. Loading States

Always track loading state to prevent UI flickering:

```csharp
public async Task LoadDataAsync()
{
    IsLoading = true;
    Error = null;
    
    try
    {
        var data = await _api.GetDataAsync();
        UpdateCollection(data);
    }
    catch (Exception ex)
    {
        Error = ex.Message;
    }
    finally
    {
        IsLoading = false; // Always reset loading state
    }
}
```

## Migration Checklist

### POC Phase
- [x] Define IDashboardUserService with ObservableCollection
- [x] Implement loading/error state tracking
- [x] Add StateChanged event for manual subscriptions
- [x] Test service in component with dependency injection
- [x] Verify ObservableCollection updates trigger UI updates
- [ ] Test form state with EditContext
- [ ] Verify validation works equivalently

### Phase 1
- [ ] Convert all 22 state modules to Blazor services
- [ ] Standardize loading/error patterns
- [ ] Test all service state updates
- [ ] Verify no memory leaks from event subscriptions

### Phase 2-4
- [ ] Implement state persistence where needed
- [ ] Add caching strategies
- [ ] Optimize state update performance

---

**Next Step**: Proceed to 06-VALIDATION-RULES.md
