# Validation Rules Migration: TypeScript → C#

## Overview

Extract and port all validation logic from `src/utils/validation-utils.ts` to C# DataAnnotations and custom validators.

## Current Validation Functions

### Source File Analysis

**File**: `src/utils/validation-utils.ts` (233 lines)

Contains:
- 7 validation functions
- Hookstate validation helpers
- Form modification tracking
- Schema utilities

## Validation Function Mapping

### 1. Email Validation

**TypeScript:**
```typescript
const emailRegex = /^[a-zA-Z0-9.!#$%&'*+\/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)+$/;

export function validateEmail(email: string | null | undefined): boolean {
  return email == null || email.length === 0 || emailRegex.test(email);
}
```

**C# (DataAnnotations):**
```csharp
// Built-in attribute
[EmailAddress(ErrorMessage = "Email is not valid.")]
public string Email { get; set; } = string.Empty;
```

**C# (Custom Attribute - if need exact regex):**
```csharp
public class TronEmailAddressAttribute : ValidationAttribute
{
    private static readonly Regex EmailRegex = new(
        @"^[a-zA-Z0-9.!#$%&'*+\/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)+$",
        RegexOptions.Compiled);
    
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is null or string { Length: 0 })
            return ValidationResult.Success;
        
        if (value is string email && EmailRegex.IsMatch(email))
            return ValidationResult.Success;
        
        return new ValidationResult(ErrorMessage ?? "Email is not valid.");
    }
}

// Usage
[TronEmailAddress]
public string Email { get; set; } = string.Empty;
```

### 2. Phone Validation

**TypeScript:**
```typescript
export function validPhone(text: string | undefined | null): boolean {
    return !text || /^(?:\([2-9]\d{2}\) ?|[2-9]\d{2}(?:-?| ?))[2-9]\d{2}[- ]?\d{4}$/.test(text);
}
```

**C# (Custom Attribute):**
```csharp
public class ValidPhoneAttribute : ValidationAttribute
{
    private static readonly Regex PhoneRegex = new(
        @"^(?:\([2-9]\d{2}\) ?|[2-9]\d{2}(?:-?| ?))[2-9]\d{2}[- ]?\d{4}$",
        RegexOptions.Compiled);
    
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is null or string { Length: 0 })
            return ValidationResult.Success;
        
        if (value is string phone && PhoneRegex.IsMatch(phone))
            return ValidationResult.Success;
        
        return new ValidationResult(ErrorMessage ?? "Phone Number is not valid.");
    }
}

// Usage
[ValidPhone(ErrorMessage = "Phone Number is not valid.")]
public string? Phone { get; set; }
```

### 3. DoD ID Validation

**TypeScript:**
```typescript
export function validDoDId(text: string | undefined | null): boolean {
  return !text || /^\d{5,10}$/.test(text);
}
```

**C# (Custom Attribute):**
```csharp
public class ValidDoDIdAttribute : ValidationAttribute
{
    private static readonly Regex DoDIdRegex = new(@"^\d{5,10}$", RegexOptions.Compiled);
    
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is null or string { Length: 0 })
            return ValidationResult.Success;
        
        if (value is string dodId && DoDIdRegex.IsMatch(dodId))
            return ValidationResult.Success;
        
        return new ValidationResult(ErrorMessage ?? "DoD ID is invalid.");
    }
}

// Usage
[ValidDoDId]
public string? DoDId { get; set; }
```

### 4. Required String Validation

**TypeScript:**
```typescript
export function validateRequiredString(value: string | null | undefined): boolean {
  if (value == null) {
    return false;
  }
  
  const trimmed = value.trim();
  
  if (trimmed.length <= 0)
    return false;
  
  return true;
}
```

**C# (Built-in):**
```csharp
[Required(ErrorMessage = "Cannot be blank or empty.")]
public string Name { get; set; } = string.Empty;
```

**C# (Custom with trim logic):**
```csharp
public class RequiredTrimmedStringAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is null)
            return new ValidationResult(ErrorMessage ?? "Cannot be blank or empty.");
        
        if (value is string str && str.Trim().Length > 0)
            return ValidationResult.Success;
        
        return new ValidationResult(ErrorMessage ?? "Cannot be blank or empty.");
    }
}

// Usage
[RequiredTrimmedString]
public string Name { get; set; } = string.Empty;
```

### 5. String Length Validation

**TypeScript:**
```typescript
export function validateStringLength(value: string | null | undefined, minLength = 1, maxLength = 255) {
  if (value == null || value.length === 0)
    return true;
  
  const trimmed = value.trim();
  
  if (trimmed.length < minLength)
    return false;
  
  if (trimmed.length > maxLength)
    return false;
  
  return true;
}
```

**C# (Built-in):**
```csharp
[StringLength(255, MinimumLength = 1, ErrorMessage = "Must have 1 to 255 characters.")]
public string Email { get; set; } = string.Empty;
```

**C# (Custom with trim):**
```csharp
public class TrimmedStringLengthAttribute : StringLengthAttribute
{
    public TrimmedStringLengthAttribute(int maximumLength) : base(maximumLength) { }
    
    public override bool IsValid(object? value)
    {
        if (value is null or string { Length: 0 })
            return true;
        
        if (value is string str)
        {
            var trimmed = str.Trim();
            return trimmed.Length >= MinimumLength && trimmed.Length <= MaximumLength;
        }
        
        return base.IsValid(value);
    }
}

// Usage
[TrimmedStringLength(255, MinimumLength = 1, 
    ErrorMessage = "Must have 1 to 255 characters.")]
public string Email { get; set; } = string.Empty;
```

### 6. Document Space Name Validation

**TypeScript:**
```typescript
export function validateDocSpaceName(name: string): boolean {
  if (!name) return false;
  if (/\s+/.test(name)) return false;
  if (name.length < 3 || name.length > 63) return false;
  if (/[A-Z]+/.test(name)) return false;
  if (!/^[a-z0-9]/.test(name) || !/[a-z0-9]$/.test(name)) return false;
  if (/[0-9]{,3}\.[0-9]{,3}\.[0-9]{,3}\.[0-9]{,3}/.test(name)) return false;
  if (name.startsWith("xn--")) return false;
  if (name.endsWith("-s3alias")) return false;
  if (name.indexOf("/") !== -1) return false;
  return true;
}
```

**C# (Complex Custom Attribute):**
```csharp
public class ValidDocSpaceNameAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is not string name || string.IsNullOrEmpty(name))
            return new ValidationResult("Document space name is required.");
        
        // Whitespace check
        if (Regex.IsMatch(name, @"\s+"))
            return new ValidationResult("Document space name cannot contain whitespace.");
        
        // Length constraints
        if (name.Length < 3 || name.Length > 63)
            return new ValidationResult("Document space name must be 3-63 characters.");
        
        // No uppercase
        if (Regex.IsMatch(name, "[A-Z]+"))
            return new ValidationResult("Document space name must be lowercase.");
        
        // Must start and end with letter or number
        if (!Regex.IsMatch(name, @"^[a-z0-9]") || !Regex.IsMatch(name, @"[a-z0-9]$"))
            return new ValidationResult("Document space name must start and end with a letter or number.");
        
        // Cannot be IP address shape
        if (Regex.IsMatch(name, @"[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"))
            return new ValidationResult("Document space name cannot be an IP address.");
        
        // Cannot start with xn--
        if (name.StartsWith("xn--"))
            return new ValidationResult("Document space name cannot start with 'xn--'.");
        
        // Cannot end with -s3alias
        if (name.EndsWith("-s3alias"))
            return new ValidationResult("Document space name cannot end with '-s3alias'.");
        
        // No slashes
        if (name.Contains('/'))
            return new ValidationResult("Document space name cannot contain slashes.");
        
        return ValidationResult.Success;
    }
}

// Usage
[ValidDocSpaceName]
public string Name { get; set; } = string.Empty;
```

### 7. Folder Name Validation

**TypeScript:**
```typescript
export function validateFolderName(name: string): boolean {
  if (!name) return false;
  if (name.trim() === '') return false;
  if (name.length > 255) return false;
  return /^[A-Za-z0-9-._()\s]+$/.test(name)
    && (name.match(/(\.)/g) || []).length <= 1
    && !(/\s[.]|[.]\s/.test(name));
}
```

**C# (Custom Attribute):**
```csharp
public class ValidFolderNameAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is not string name || string.IsNullOrWhiteSpace(name))
            return new ValidationResult("Folder name is required.");
        
        if (name.Length > 255)
            return new ValidationResult("Folder name cannot exceed 255 characters.");
        
        // Must match allowed characters
        if (!Regex.IsMatch(name, @"^[A-Za-z0-9-._()\s]+$"))
            return new ValidationResult("Folder name contains invalid characters.");
        
        // No more than 1 dot
        if (name.Count(c => c == '.') > 1)
            return new ValidationResult("Folder name can have at most one dot.");
        
        // No spaces before or after dot
        if (Regex.IsMatch(name, @"\s[.]|[.]\s"))
            return new ValidationResult("Folder name cannot have spaces adjacent to dots.");
        
        return ValidationResult.Success;
    }
}

// Usage
[ValidFolderName]
public string FolderName { get; set; } = string.Empty;
```

### 8. PubSub Subscriber Address Validation

**TypeScript:**
```typescript
export function validateSubscriberAddress(url: string | undefined): boolean {
  return url != null && (url.length === 0 || /^http:\/\/(?!tron-common-api).+?\.(?!tron-common-api).+?\.svc.cluster.local\//.test(url));
}
```

**C# (Custom Attribute):**
```csharp
public class ValidSubscriberAddressAttribute : ValidationAttribute
{
    private static readonly Regex UrlRegex = new(
        @"^http:\/\/(?!tron-common-api).+?\.(?!tron-common-api).+?\.svc.cluster.local\/",
        RegexOptions.Compiled);
    
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is null or string { Length: 0 })
            return ValidationResult.Success;
        
        if (value is string url && UrlRegex.IsMatch(url))
            return ValidationResult.Success;
        
        return new ValidationResult(ErrorMessage ?? "Invalid subscriber address.");
    }
}

// Usage
[ValidSubscriberAddress]
public string? SubscriberAddress { get; set; }
```

### 9. Checkbox Privileges Validation

**TypeScript:**
```typescript
export function validateCheckboxPrivileges(values: boolean[]): boolean {
  return values.some(value => value === true);
}
```

**C# (Custom Attribute for Lists):**
```csharp
public class AtLeastOneSelectedAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is IEnumerable<bool> booleans && booleans.Any(b => b))
            return ValidationResult.Success;
        
        return new ValidationResult(ErrorMessage ?? "At least one option must be selected.");
    }
}

// Usage
[AtLeastOneSelected]
public List<bool> Privileges { get; set; } = new();
```

## Validation Attribute Library

Create a centralized library of custom validators:

```csharp
// TronDashboard.Core/Validation/ValidationAttributes.cs
namespace TronDashboard.Core.Validation;

/// <summary>
/// Email validation matching TRON Common API rules
/// </summary>
public class TronEmailAddressAttribute : ValidationAttribute
{
    // Implementation above
}

/// <summary>
/// US Phone number validation
/// </summary>
public class ValidPhoneAttribute : ValidationAttribute
{
    // Implementation above
}

/// <summary>
/// DoD ID validation (5-10 digits)
/// </summary>
public class ValidDoDIdAttribute : ValidationAttribute
{
    // Implementation above
}

/// <summary>
/// Required string with trimming
/// </summary>
public class RequiredTrimmedStringAttribute : ValidationAttribute
{
    // Implementation above
}

/// <summary>
/// String length validation with trimming
/// </summary>
public class TrimmedStringLengthAttribute : StringLengthAttribute
{
    // Implementation above
}

/// <summary>
/// Document space name validation (AWS S3 bucket rules)
/// </summary>
public class ValidDocSpaceNameAttribute : ValidationAttribute
{
    // Implementation above
}

/// <summary>
/// Folder/file name validation
/// </summary>
public class ValidFolderNameAttribute : ValidationAttribute
{
    // Implementation above
}

/// <summary>
/// PubSub subscriber address validation
/// </summary>
public class ValidSubscriberAddressAttribute : ValidationAttribute
{
    // Implementation above
}

/// <summary>
/// At least one checkbox must be selected
/// </summary>
public class AtLeastOneSelectedAttribute : ValidationAttribute
{
    // Implementation above
}
```

## Form Model Example

### DashboardUser Form Model

```csharp
public class DashboardUserFormModel
{
    public string? Id { get; set; }
    
    [Required(ErrorMessage = "Email is required")]
    [TronEmailAddress(ErrorMessage = "Email is not valid.")]
    [TrimmedStringLength(255, MinimumLength = 1, ErrorMessage = "Email must be 1-255 characters.")]
    public string Email { get; set; } = string.Empty;
    
    public bool HasDashboardAdmin { get; set; }
}
```

### Person Form Model

```csharp
public class PersonFormModel
{
    [Required]
    [TrimmedStringLength(255, MinimumLength = 1)]
    public string FirstName { get; set; } = string.Empty;
    
    [Required]
    [TrimmedStringLength(255, MinimumLength = 1)]
    public string LastName { get; set; } = string.Empty;
    
    [TronEmailAddress]
    [TrimmedStringLength(255)]
    public string? Email { get; set; }
    
    [ValidPhone]
    public string? Phone { get; set; }
    
    [ValidDoDId]
    public string? DoDId { get; set; }
}
```

## Client-Side vs Server-Side Validation

### Both Sides (Recommended)

```razor
<EditForm Model="@_model" OnValidSubmit="@HandleSubmit">
    @* Client-side validation *@
    <DataAnnotationsValidator />
    
    @* Server-side validation happens in service layer *@
    <MudTextField 
        @bind-Value="_model.Email"
        Label="Email"
        For="@(() => _model.Email)"
        Immediate="true" @* Validate on every keystroke *@
        />
    
    <ValidationSummary />
</EditForm>
```

### Server-Side Validation in Service

```csharp
public async Task<DashboardUserFlat> CreateAsync(DashboardUserDto dto, CancellationToken ct = default)
{
    // Manual validation if needed
    if (string.IsNullOrWhiteSpace(dto.Email))
        throw new ValidationException("Email is required");
    
    if (!IsValidEmail(dto.Email))
        throw new ValidationException("Email is not valid");
    
    try
    {
        var response = await _api.AddDashboardUserAsync(dto, ct);
        return ConvertToFlat(response);
    }
    catch (ApiException ex) when (ex.StatusCode == 400)
    {
        // Parse backend validation errors
        throw new ValidationException("Validation failed", ParseErrors(ex));
    }
}
```

## Testing Validation

### xUnit Tests for Validators

```csharp
public class TronEmailAddressAttributeTests
{
    private readonly TronEmailAddressAttribute _validator = new();
    
    [Theory]
    [InlineData("test@example.com", true)]
    [InlineData("user.name+tag@domain.co.uk", true)]
    [InlineData("invalid", false)]
    [InlineData("@example.com", false)]
    [InlineData("test@", false)]
    [InlineData(null, true)] // Null is valid (use [Required] to enforce)
    [InlineData("", true)]   // Empty is valid
    public void TronEmailAddress_ValidatesCorrectly(string? email, bool expectedValid)
    {
        // Arrange
        var context = new ValidationContext(new { Email = email });
        
        // Act
        var result = _validator.GetValidationResult(email, context);
        
        // Assert
        if (expectedValid)
            Assert.Equal(ValidationResult.Success, result);
        else
            Assert.NotEqual(ValidationResult.Success, result);
    }
}
```

## Migration Checklist

### POC Phase
- [x] Create TronEmailAddressAttribute
- [x] Create RequiredTrimmedStringAttribute
- [x] Create TrimmedStringLengthAttribute
- [x] Test validation in DashboardUser form
- [ ] Verify error messages display correctly
- [ ] Test client-side validation performance

### Phase 1
- [ ] Port all 9 validation functions to C# attributes
- [ ] Create validation attribute library
- [ ] Write xUnit tests for all validators
- [ ] Test in all forms

### Phase 2-4
- [ ] Add backend validation error parsing
- [ ] Implement custom validation for complex scenarios
- [ ] Performance test validation in large forms

---

**Next Step**: Proceed to 07-API-CLIENT-SETUP.md
