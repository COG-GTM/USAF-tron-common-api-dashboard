# Component Mapping: React → Blazor

## Overview

This document maps all 41 React components in `src/components/` to their Blazor equivalents, plus 17 page components in `src/pages/`.

## Component Pattern Translation

### React Component Pattern
```tsx
// src/components/forms/TextInput/TextInput.tsx
import React from 'react';

interface TextInputProps {
  id: string;
  name: string;
  type?: string;
  defaultValue?: string;
  error?: boolean;
  onChange?: (event: React.ChangeEvent<HTMLInputElement>) => void;
  disabled?: boolean;
}

function TextInput(props: TextInputProps) {
  return (
    <input
      id={props.id}
      name={props.name}
      type={props.type || 'text'}
      defaultValue={props.defaultValue}
      className={`usa-input ${props.error ? 'usa-input--error' : ''}`}
      onChange={props.onChange}
      disabled={props.disabled}
    />
  );
}

export default TextInput;
```

### Blazor Component Pattern
```razor
@* TronTextInput.razor *@
<MudTextField 
    @bind-Value="@Value"
    Label="@Label"
    For="@For"
    Variant="Variant.Outlined"
    Error="@Error"
    ErrorText="@ErrorText"
    Disabled="@Disabled"
    HelperText="@HelperText"
    Required="@Required"
    Class="@AdditionalClass" />

@code {
    [Parameter] public string? Value { get; set; }
    [Parameter] public EventCallback<string> ValueChanged { get; set; }
    [Parameter] public string? Label { get; set; }
    [Parameter] public Expression<Func<string>>? For { get; set; }
    [Parameter] public bool Error { get; set; }
    [Parameter] public string? ErrorText { get; set; }
    [Parameter] public bool Disabled { get; set; }
    [Parameter] public string? HelperText { get; set; }
    [Parameter] public bool Required { get; set; }
    [Parameter] public string? AdditionalClass { get; set; }
}
```

## Component Inventory

### Form Components (High Priority - POC Needs)

| React Component | Blazor Component | Library | POC | Notes |
|----------------|------------------|---------|-----|-------|
| `TextInput` | `TronTextInput` | MudBlazor `MudTextField` | ✅ | Email, text inputs |
| `Checkbox` | `TronCheckbox` | MudBlazor `MudCheckBox` | ✅ | Dashboard Admin flag |
| `FormGroup` | `TronFormGroup` | Custom wrapper | ✅ | Label + input + validation |
| `Form` | `EditForm` | Blazor built-in | ✅ | Form container |
| `Label` | `MudText` or `<label>` | MudBlazor | ✅ | Field labels |
| `SubmitActions` | `TronSubmitActions` | Custom | ✅ | Save/Cancel buttons |
| `SuccessErrorMessage` | `TronAlert` | MudBlazor `MudAlert` | ✅ | Success/error display |
| `Select` | `TronSelect` | MudBlazor `MudSelect` | ❌ | Not in POC |
| `Radio` | `TronRadioGroup` | MudBlazor `MudRadioGroup` | ❌ | Not in POC |
| `Fieldset` | `<fieldset>` | HTML | ❌ | Not in POC |
| `TextInputWithDelete` | `TronTextInputWithDelete` | Custom | ❌ | Not in POC |

### Layout Components

| React Component | Blazor Component | Library | POC | Notes |
|----------------|------------------|---------|-----|-------|
| `PageFormat` | `PageLayout` | Custom | ✅ | Main page wrapper |
| `PageTitle` | `MudText` | MudBlazor | ✅ | Page heading |
| `Sidebar` | `MudDrawer` | MudBlazor | ⚠️ | Nav sidebar (stub for POC) |
| `SidebarContainer` | N/A | MudBlazor `MudLayout` | ⚠️ | Layout wrapper |
| `SidebarItem` | `MudNavLink` | MudBlazor | ⚠️ | Nav items |
| `NestedSidebarNav` | `MudNavMenu` | MudBlazor | ⚠️ | Nested nav |
| `HeaderUserInfo` | `TronUserInfo` | Custom | ⚠️ | User display (stub) |
| `HeaderUserEditor` | `TronUserEditor` | Custom | ❌ | User editing |
| `BreadCrumbTrail` | `MudBreadcrumbs` | MudBlazor | ❌ | Not in POC |
| `Card` | `MudCard` | MudBlazor | ❌ | Not in POC |
| `TabBar` | `MudTabs` | MudBlazor | ❌ | Not in POC |

### Grid Components

| React Component | Blazor Component | Library | POC | Notes |
|----------------|------------------|---------|-----|-------|
| `Grid` (AG Grid wrapper) | `RadzenDataGrid` | Radzen | ✅ | Main data grid |
| `CheckboxCellRenderer` | Grid template | Radzen template | ❌ | Custom cell |
| `DeleteCellRenderer` | Grid template | Radzen template | ✅ | Delete button |
| `ApiSpecCellRenderer` | Grid template | Radzen template | ❌ | Not in POC |
| `ComboBoxCellRenderer` | Grid template | Radzen template | ❌ | Not in POC |
| `LoadingCellRenderer` | Grid template | Radzen template | ❌ | Not in POC |
| `MetricCellRenderer` | Grid template | Radzen template | ❌ | Not in POC |
| `PrivilegeCellRenderer` | Grid template | Radzen template | ❌ | Not in POC |
| `UnusedEndpointCellRenderer` | Grid template | Radzen template | ❌ | Not in POC |
| `DocSpaceItemRenderer` | Grid template | Radzen template | ❌ | Document Space |
| `DocumentRowActionCellRenderer` | Grid template | Radzen template | ❌ | Document Space |

### UI Utility Components

| React Component | Blazor Component | Library | POC | Notes |
|----------------|------------------|---------|-----|-------|
| `Spinner` | `MudProgressCircular` | MudBlazor | ✅ | Loading indicator |
| `Modal` | `MudDialog` | MudBlazor | ✅ | Dialogs (delete confirm) |
| `GenericDialog` | `TronDialog` | MudBlazor `MudDialog` | ✅ | Generic dialog |
| `ConfirmDialog` | `TronConfirmDialog` | Custom | ✅ | Confirmation dialogs |
| `Toast` / `ToastContainer` | `MudSnackbar` | MudBlazor | ✅ | Notifications |
| `Button` | `MudButton` | MudBlazor | ✅ | All buttons |
| `BackdropOverlay` | `MudOverlay` | MudBlazor | ❌ | Not in POC |
| `SideDrawer` | `MudDrawer` | MudBlazor | ❌ | Not in POC |
| `ErrorBoundary` | `ErrorBoundary` | Custom | ⚠️ | Error handling |
| `ProtectedRoute` | `AuthorizeView` | Blazor built-in | ✅ | Authorization |
| `CopyToClipboard` | JS Interop | Custom | ❌ | Not in POC |
| `Accordion` | `MudExpansionPanel` | MudBlazor | ❌ | Not in POC |
| `InfoNotice` | `MudAlert` | MudBlazor | ❌ | Not in POC |
| `ItemChooser` | `TronItemChooser` | Custom | ❌ | Not in POC |
| `AppInfoTag` | `MudChip` | MudBlazor | ❌ | Not in POC |
| `DropDown` | `MudMenu` | MudBlazor | ❌ | Not in POC |

### Chart Components

| React Component | Blazor Component | Library | POC | Notes |
|----------------|------------------|---------|-----|-------|
| `Chart` / `ChartWithTitleAndTotal` | `ApexChart` | Blazor.ApexCharts | ❌ | Not in POC |
| `KpiMiniChart` | `ApexChart` | Blazor.ApexCharts | ❌ | Not in POC |
| `MiniChart` | `ApexChart` | Blazor.ApexCharts | ❌ | Not in POC |
| `StatusCard` | `MudCard` | MudBlazor | ❌ | Not in POC |

### Page-Specific Components

| React Component | Blazor Component | Library | POC | Notes |
|----------------|------------------|---------|-----|-------|
| `DataCrudFormPage` | `TronCrudPage` | Custom | ✅ | Generic CRUD page |
| `DataCrudDelete` | `TronDeleteDialog` | Custom | ✅ | Delete confirmation |
| Document Space components (20+) | Various | Custom | ❌ | Phase 3 |

## POC Component Implementation Plan

### Week 1: Foundation Components

#### 1. PageLayout.razor
Base layout component with nav, header, content area.

```razor
@inherits LayoutComponentBase

<MudThemeProvider />
<MudDialogProvider />
<MudSnackbarProvider />

<MudLayout>
    @* For POC, stub out navigation *@
    <MudAppBar Elevation="1">
        <MudText Typo="Typo.h6">TRON Common API Dashboard</MudText>
        <MudSpacer />
        <MudText>@_userEmail</MudText>
    </MudAppBar>
    
    <MudMainContent Class="pa-4">
        <MudContainer MaxWidth="MaxWidth.ExtraExtraLarge">
            <MudText Typo="Typo.h4" Class="mb-4">@PageTitle</MudText>
            @ChildContent
        </MudContainer>
    </MudMainContent>
</MudLayout>

@code {
    [Parameter] public string PageTitle { get; set; } = string.Empty;
    [Parameter] public RenderFragment? ChildContent { get; set; }
    [Inject] private IAuthorizedUserService UserService { get; set; } = default!;
    
    private string _userEmail = string.Empty;
    
    protected override async Task OnInitializedAsync()
    {
        var user = UserService.AuthorizedUser;
        _userEmail = user?.Email ?? "Unknown";
    }
}
```

#### 2. TronTextInput.razor
Text input with validation support.

```razor
<MudTextField 
    @bind-Value="@Value"
    Label="@Label"
    For="@For"
    Variant="Variant.Outlined"
    Error="@Error"
    ErrorText="@ErrorText"
    Disabled="@Disabled"
    HelperText="@HelperText"
    Required="@Required"
    InputType="@InputType"
    Class="@Class" />

@code {
    [Parameter] public string? Value { get; set; }
    [Parameter] public EventCallback<string> ValueChanged { get; set; }
    [Parameter] public string? Label { get; set; }
    [Parameter] public Expression<Func<string>>? For { get; set; }
    [Parameter] public bool Error { get; set; }
    [Parameter] public string? ErrorText { get; set; }
    [Parameter] public bool Disabled { get; set; }
    [Parameter] public string? HelperText { get; set; }
    [Parameter] public bool Required { get; set; }
    [Parameter] public InputType InputType { get; set; } = InputType.Text;
    [Parameter] public string? Class { get; set; }
}
```

#### 3. TronCheckbox.razor
Checkbox component.

```razor
<MudCheckBox 
    @bind-Checked="@Checked"
    Label="@Label"
    Disabled="@Disabled"
    Color="Color.Primary"
    Class="@Class">
    @if (ChildContent != null)
    {
        @ChildContent
    }
</MudCheckBox>

@code {
    [Parameter] public bool Checked { get; set; }
    [Parameter] public EventCallback<bool> CheckedChanged { get; set; }
    [Parameter] public string? Label { get; set; }
    [Parameter] public bool Disabled { get; set; }
    [Parameter] public string? Class { get; set; }
    [Parameter] public RenderFragment? ChildContent { get; set; }
}
```

#### 4. TronAlert.razor
Success/error message display.

```razor
@if (Visible)
{
    <MudAlert 
        Severity="@Severity"
        Variant="Variant.Filled"
        Dense="true"
        Class="my-2"
        CloseIcon="@(ShowCloseButton ? Icons.Material.Filled.Close : null)"
        CloseIconClicked="@OnClose">
        @Message
    </MudAlert>
}

@code {
    [Parameter] public bool Visible { get; set; }
    [Parameter] public string? Message { get; set; }
    [Parameter] public Severity Severity { get; set; } = Severity.Info;
    [Parameter] public bool ShowCloseButton { get; set; }
    [Parameter] public EventCallback OnClose { get; set; }
}
```

#### 5. TronSubmitActions.razor
Form action buttons (Save/Cancel).

```razor
<MudStack Row="true" Justify="Justify.FlexEnd" Class="mt-4">
    <MudButton 
        Variant="Variant.Outlined" 
        OnClick="@OnCancel"
        Disabled="@IsSubmitting">
        Cancel
    </MudButton>
    
    <MudButton 
        Variant="Variant.Filled" 
        Color="Color.Primary"
        ButtonType="ButtonType.Submit"
        Disabled="@(!IsValid || !IsModified || IsSubmitting)">
        @if (IsSubmitting)
        {
            <MudProgressCircular Size="Size.Small" Indeterminate="true" />
        }
        else
        {
            @ButtonText
        }
    </MudButton>
</MudStack>

@code {
    [Parameter] public bool IsValid { get; set; }
    [Parameter] public bool IsModified { get; set; }
    [Parameter] public bool IsSubmitting { get; set; }
    [Parameter] public string ButtonText { get; set; } = "Save";
    [Parameter] public EventCallback OnCancel { get; set; }
}
```

### Week 2: Page Components

#### DashboardUserPage.razor
Main grid page.

```razor
@page "/dashboard-users"
@attribute [Authorize(Policy = "RequireDashboardAdmin")]
@inject IDashboardUserService DashboardUserService
@inject IDialogService DialogService
@inject ISnackbar Snackbar

<PageLayout PageTitle="Dashboard Users">
    <MudStack>
        <MudStack Row="true" Justify="Justify.SpaceBetween" AlignItems="AlignItems.Center">
            <MudText Typo="Typo.h6">Manage Dashboard Users</MudText>
            <MudButton 
                Variant="Variant.Filled" 
                Color="Color.Primary"
                StartIcon="@Icons.Material.Filled.Add"
                OnClick="@OpenCreateDialog">
                Add New User
            </MudButton>
        </MudStack>
        
        @if (DashboardUserService.IsLoading)
        {
            <MudProgressLinear Indeterminate="true" />
        }
        else if (!string.IsNullOrEmpty(DashboardUserService.Error))
        {
            <MudAlert Severity="Severity.Error">@DashboardUserService.Error</MudAlert>
        }
        else
        {
            <RadzenDataGrid 
                Data="@DashboardUserService.DashboardUsers" 
                TItem="DashboardUserFlat"
                AllowFiltering="true"
                AllowSorting="true"
                AllowPaging="true"
                PageSize="20"
                RowClick="@OnRowClick">
                <Columns>
                    <RadzenDataGridColumn TItem="DashboardUserFlat" Property="Id" Title="UUID" Width="250px" />
                    <RadzenDataGridColumn TItem="DashboardUserFlat" Property="Email" Title="Email" />
                    <RadzenDataGridColumn TItem="DashboardUserFlat" Property="HasDashboardAdmin" Title="Dashboard Admin">
                        <Template Context="user">
                            <MudCheckBox Checked="@user.HasDashboardAdmin" Disabled="true" />
                        </Template>
                    </RadzenDataGridColumn>
                    <RadzenDataGridColumn TItem="DashboardUserFlat" Title="Actions" Width="100px" Sortable="false" Filterable="false">
                        <Template Context="user">
                            <MudIconButton 
                                Icon="@Icons.Material.Filled.Delete" 
                                Color="Color.Error"
                                Size="Size.Small"
                                OnClick="@(() => DeleteUser(user))" />
                        </Template>
                    </RadzenDataGridColumn>
                </Columns>
            </RadzenDataGrid>
        }
    </MudStack>
</PageLayout>

@code {
    protected override async Task OnInitializedAsync()
    {
        await DashboardUserService.FetchAndStoreDataAsync();
    }
    
    private async Task OpenCreateDialog()
    {
        var dialog = await DialogService.ShowAsync<DashboardUserForm>("Create Dashboard User");
        var result = await dialog.Result;
        
        if (!result.Canceled)
        {
            Snackbar.Add("User created successfully", Severity.Success);
            await DashboardUserService.FetchAndStoreDataAsync();
        }
    }
    
    private async Task OnRowClick(DataGridRowMouseEventArgs<DashboardUserFlat> args)
    {
        var parameters = new DialogParameters { ["User"] = args.Data };
        var dialog = await DialogService.ShowAsync<DashboardUserForm>("Edit Dashboard User", parameters);
        var result = await dialog.Result;
        
        if (!result.Canceled)
        {
            Snackbar.Add("User updated successfully", Severity.Success);
            await DashboardUserService.FetchAndStoreDataAsync();
        }
    }
    
    private async Task DeleteUser(DashboardUserFlat user)
    {
        var confirmed = await DialogService.ShowMessageBox(
            "Confirm Delete",
            $"Are you sure you want to delete {user.Email}?",
            yesText: "Delete", cancelText: "Cancel");
        
        if (confirmed == true)
        {
            await DashboardUserService.DeleteAsync(user.Id!);
            Snackbar.Add("User deleted successfully", Severity.Success);
        }
    }
}
```

#### DashboardUserForm.razor
Create/Edit form dialog.

```razor
@inject IDashboardUserService DashboardUserService

<MudDialog>
    <DialogContent>
        <EditForm Model="@_model" OnValidSubmit="@HandleSubmit">
            <DataAnnotationsValidator />
            
            @if (_isEditMode)
            {
                <MudTextField 
                    Label="UUID"
                    Value="@_model.Id"
                    Disabled="true"
                    Variant="Variant.Outlined"
                    Class="mb-4" />
            }
            
            <MudTextField 
                @bind-Value="_model.Email"
                Label="Email"
                For="@(() => _model.Email)"
                Variant="Variant.Outlined"
                Required="true"
                InputType="InputType.Email"
                Class="mb-4" />
            
            <MudCheckBox 
                @bind-Checked="_model.HasDashboardAdmin"
                Label="Dashboard Admin"
                Color="Color.Primary" />
            
            <ValidationSummary />
            
            @if (!string.IsNullOrEmpty(_errorMessage))
            {
                <MudAlert Severity="Severity.Error" Class="mt-4">@_errorMessage</MudAlert>
            }
            
            @if (_showSuccess)
            {
                <MudAlert Severity="Severity.Success" Class="mt-4">User saved successfully!</MudAlert>
            }
        </EditForm>
    </DialogContent>
    
    <DialogActions>
        <MudButton OnClick="@Cancel" Disabled="@_isSubmitting">Cancel</MudButton>
        <MudButton 
            Color="Color.Primary" 
            Variant="Variant.Filled"
            OnClick="@HandleSubmit"
            Disabled="@(!_isFormValid || !_isFormModified || _isSubmitting || _showSuccess)">
            @if (_isSubmitting)
            {
                <MudProgressCircular Size="Size.Small" Indeterminate="true" />
            }
            else
            {
                @(_isEditMode ? "Update" : "Create")
            }
        </MudButton>
    </DialogActions>
</MudDialog>

@code {
    [CascadingParameter] MudDialogInstance MudDialog { get; set; } = default!;
    [Parameter] public DashboardUserFlat? User { get; set; }
    
    private DashboardUserFormModel _model = new();
    private bool _isEditMode => !string.IsNullOrEmpty(User?.Id);
    private bool _isSubmitting;
    private bool _showSuccess;
    private string? _errorMessage;
    private bool _isFormValid = true;
    private bool _isFormModified = false;
    
    protected override void OnInitialized()
    {
        if (User != null)
        {
            _model = new DashboardUserFormModel
            {
                Id = User.Id,
                Email = User.Email,
                HasDashboardAdmin = User.HasDashboardAdmin
            };
        }
    }
    
    private async Task HandleSubmit()
    {
        _isSubmitting = true;
        _errorMessage = null;
        
        try
        {
            var dto = DashboardUserService.ConvertToDto(new DashboardUserFlat
            {
                Id = _model.Id,
                Email = _model.Email,
                HasDashboardAdmin = _model.HasDashboardAdmin
            });
            
            if (_isEditMode)
            {
                await DashboardUserService.UpdateAsync(dto);
            }
            else
            {
                await DashboardUserService.CreateAsync(dto);
            }
            
            _showSuccess = true;
            await Task.Delay(1500);
            MudDialog.Close(DialogResult.Ok(true));
        }
        catch (Exception ex)
        {
            _errorMessage = ex.Message;
        }
        finally
        {
            _isSubmitting = false;
        }
    }
    
    private void Cancel() => MudDialog.Cancel();
    
    public class DashboardUserFormModel
    {
        public string? Id { get; set; }
        
        [Required(ErrorMessage = "Email is required")]
        [EmailAddress(ErrorMessage = "Invalid email address")]
        [StringLength(255, MinimumLength = 1)]
        public string Email { get; set; } = string.Empty;
        
        public bool HasDashboardAdmin { get; set; }
    }
}
```

## Component Testing Strategy

### bUnit Component Tests

```csharp
public class TronTextInputTests : TestContext
{
    [Fact]
    public void TronTextInput_RendersWithLabel()
    {
        // Arrange
        var component = RenderComponent<TronTextInput>(parameters => parameters
            .Add(p => p.Label, "Email")
            .Add(p => p.Value, "test@example.com"));
        
        // Act
        var input = component.Find("input");
        
        // Assert
        Assert.NotNull(input);
        Assert.Equal("test@example.com", input.GetAttribute("value"));
    }
    
    [Fact]
    public void TronTextInput_DisplaysErrorState()
    {
        // Arrange
        var component = RenderComponent<TronTextInput>(parameters => parameters
            .Add(p => p.Error, true)
            .Add(p => p.ErrorText, "Invalid email"));
        
        // Act
        var errorText = component.Find(".mud-input-error");
        
        // Assert
        Assert.Contains("Invalid email", errorText.TextContent);
    }
}

public class DashboardUserPageTests : TestContext
{
    [Fact]
    public async Task DashboardUserPage_LoadsUsers()
    {
        // Arrange
        var mockService = new Mock<IDashboardUserService>();
        var users = new ObservableCollection<DashboardUserFlat>
        {
            new() { Id = "1", Email = "test@example.com", HasDashboardAdmin = true }
        };
        mockService.Setup(x => x.DashboardUsers).Returns(users);
        mockService.Setup(x => x.FetchAndStoreDataAsync(default)).ReturnsAsync(users.ToList());
        
        Services.AddSingleton(mockService.Object);
        Services.AddMudServices();
        
        // Act
        var component = RenderComponent<DashboardUserPage>();
        
        // Assert
        component.WaitForAssertion(() => 
            Assert.Contains("test@example.com", component.Markup));
    }
}
```

---

**Next Step**: Proceed to 04-ROUTE-MAPPING.md
