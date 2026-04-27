# API Client Setup: NSwag Configuration

## Overview

Generate C# API client from the existing OpenAPI specification using NSwag.

## Source OpenAPI Specification

**File**: `resources/tron-common-api.json`

This JSON file contains the complete API specification for the TRON Common API backend.

### How to Update Spec (from README.md)

```bash
# 1. Run Common API project locally (port 8088 dev, 8080 staging/production)
# 2. Navigate to: http://localhost:8088/api/api-docs/dashboard-api-v2
# 3. Copy the returned JSON
# 4. Replace contents of resources/tron-common-api.json
```

## NSwag Setup

### Installation

```bash
# Add NSwag to Core project
cd TronDashboard.Core
dotnet add package NSwag.MSBuild
```

### NSwag Configuration File

Create `TronDashboard.Core/nswag.json`:

```json
{
  "runtime": "Net80",
  "defaultVariables": null,
  "documentGenerator": {
    "fromDocument": {
      "url": "../resources/tron-common-api.json",
      "output": null,
      "newLineBehavior": "Auto"
    }
  },
  "codeGenerators": {
    "openApiToCSharpClient": {
      "clientBaseClass": null,
      "configurationClass": null,
      "generateClientClasses": true,
      "generateClientInterfaces": true,
      "clientBaseInterface": null,
      "injectHttpClient": true,
      "disposeHttpClient": false,
      "protectedMethods": [],
      "generateExceptionClasses": true,
      "exceptionClass": "ApiException",
      "wrapDtoExceptions": true,
      "useHttpClientCreationMethod": false,
      "httpClientType": "System.Net.Http.HttpClient",
      "useHttpRequestMessageCreationMethod": false,
      "useBaseUrl": true,
      "generateBaseUrlProperty": true,
      "generateSyncMethods": false,
      "generatePrepareRequestAndProcessResponseAsAsyncMethods": false,
      "exposeJsonSerializerSettings": false,
      "clientClassAccessModifier": "public",
      "typeAccessModifier": "public",
      "generateContractsOutput": false,
      "contractsNamespace": null,
      "contractsOutputFilePath": null,
      "parameterDateTimeFormat": "s",
      "parameterDateFormat": "yyyy-MM-dd",
      "generateUpdateJsonSerializerSettingsMethod": true,
      "useRequestAndResponseSerializationSettings": false,
      "serializeTypeInformation": false,
      "queryNullValue": "",
      "className": "{controller}Api",
      "operationGenerationMode": "MultipleClientsFromOperationId",
      "additionalNamespaceUsages": [],
      "additionalContractNamespaceUsages": [],
      "generateOptionalParameters": false,
      "generateJsonMethods": false,
      "enforceFlagEnums": false,
      "parameterArrayType": "System.Collections.Generic.IEnumerable",
      "parameterDictionaryType": "System.Collections.Generic.IDictionary",
      "responseArrayType": "System.Collections.Generic.List",
      "responseDictionaryType": "System.Collections.Generic.Dictionary",
      "wrapResponses": false,
      "wrapResponseMethods": [],
      "generateResponseClasses": true,
      "responseClass": "SwaggerResponse",
      "namespace": "TronDashboard.Core.ApiClient",
      "requiredPropertiesMustBeDefined": true,
      "dateType": "System.DateTimeOffset",
      "jsonConverters": null,
      "anyType": "object",
      "dateTimeType": "System.DateTimeOffset",
      "timeType": "System.TimeSpan",
      "timeSpanType": "System.TimeSpan",
      "arrayType": "System.Collections.Generic.List",
      "arrayInstanceType": "System.Collections.Generic.List",
      "dictionaryType": "System.Collections.Generic.Dictionary",
      "dictionaryInstanceType": "System.Collections.Generic.Dictionary",
      "arrayBaseType": "System.Collections.Generic.List",
      "dictionaryBaseType": "System.Collections.Generic.Dictionary",
      "classStyle": "Poco",
      "jsonLibrary": "SystemTextJson",
      "generateDefaultValues": true,
      "generateDataAnnotations": true,
      "excludedTypeNames": [],
      "excludedParameterNames": [],
      "handleReferences": false,
      "generateImmutableArrayProperties": false,
      "generateImmutableDictionaryProperties": false,
      "jsonSerializerSettingsTransformationMethod": null,
      "inlineNamedArrays": false,
      "inlineNamedDictionaries": false,
      "inlineNamedTuples": true,
      "inlineNamedAny": false,
      "generateDtoTypes": true,
      "generateOptionalPropertiesAsNullable": false,
      "generateNullableReferenceTypes": true,
      "templateDirectory": null,
      "typeNameGeneratorType": null,
      "propertyNameGeneratorType": null,
      "enumNameGeneratorType": null,
      "serviceHost": null,
      "serviceSchemes": null,
      "output": "ApiClient/GeneratedApiClient.cs",
      "newLineBehavior": "Auto"
    }
  }
}
```

### Key Configuration Highlights

| Setting | Value | Reason |
|---------|-------|--------|
| `generateClientInterfaces` | `true` | Enables dependency injection and mocking |
| `injectHttpClient` | `true` | Uses DI for HttpClient |
| `generateSyncMethods` | `false` | Only async methods (best practice) |
| `jsonLibrary` | `SystemTextJson` | Modern JSON serializer |
| `generateNullableReferenceTypes` | `true` | C# 9+ nullable reference types |
| `generateDataAnnotations` | `true` | Validation attributes on DTOs |
| `namespace` | `TronDashboard.Core.ApiClient` | Organized namespace |

### Project File Configuration

Add to `TronDashboard.Core/TronDashboard.Core.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="NSwag.MSBuild" Version="14.0.7">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
    <PackageReference Include="System.ComponentModel.Annotations" Version="5.0.0" />
  </ItemGroup>

  <!-- NSwag generation target -->
  <Target Name="NSwag" BeforeTargets="BeforeBuild" Condition="'$(Configuration)' == 'Debug'">
    <Exec Command="$(NSwagExe) run nswag.json" />
  </Target>
</Project>
```

### Generate Client

```bash
# Manual generation
cd TronDashboard.Core
dotnet build

# This will generate: ApiClient/GeneratedApiClient.cs
```

## Generated Client Structure

### Expected Output

```
TronDashboard.Core/
├── ApiClient/
│   └── GeneratedApiClient.cs  (~50,000+ lines)
│       ├── IDashboardUserControllerApi
│       ├── DashboardUserControllerApi
│       ├── IAppClientControllerApi
│       ├── AppClientControllerApi
│       ├── ... (20+ API interfaces/classes)
│       ├── DashboardUserDto
│       ├── AppClientDto
│       ├── ... (100+ DTO models)
│       └── ApiException
```

### Sample Generated Interface

```csharp
namespace TronDashboard.Core.ApiClient
{
    public partial interface IDashboardUserControllerApi
    {
        /// <summary>
        /// Get all dashboard users
        /// </summary>
        /// <returns>List of dashboard users</returns>
        /// <exception cref="ApiException">A server side error occurred.</exception>
        System.Threading.Tasks.Task<WrappedResponse<System.Collections.Generic.List<DashboardUserDto>>> GetAllDashboardUsersWrappedAsync(System.Threading.CancellationToken cancellationToken = default);
        
        /// <summary>
        /// Get self dashboard user
        /// </summary>
        /// <returns>Current user</returns>
        /// <exception cref="ApiException">A server side error occurred.</exception>
        System.Threading.Tasks.Task<DashboardUserDto> GetSelfDashboardUserAsync(System.Threading.CancellationToken cancellationToken = default);
        
        /// <summary>
        /// Add dashboard user
        /// </summary>
        /// <param name="body">Dashboard user to create</param>
        /// <returns>Created user</returns>
        /// <exception cref="ApiException">A server side error occurred.</exception>
        System.Threading.Tasks.Task<DashboardUserDto> AddDashboardUserAsync(DashboardUserDto body, System.Threading.CancellationToken cancellationToken = default);
        
        /// <summary>
        /// Update dashboard user
        /// </summary>
        /// <param name="id">User ID</param>
        /// <param name="body">Updated user data</param>
        /// <returns>Updated user</returns>
        /// <exception cref="ApiException">A server side error occurred.</exception>
        System.Threading.Tasks.Task<DashboardUserDto> UpdateDashboardUserAsync(string id, DashboardUserDto body, System.Threading.CancellationToken cancellationToken = default);
        
        /// <summary>
        /// Delete dashboard user
        /// </summary>
        /// <param name="id">User ID</param>
        /// <returns>Success</returns>
        /// <exception cref="ApiException">A server side error occurred.</exception>
        System.Threading.Tasks.Task DeleteDashboardUserAsync(string id, System.Threading.CancellationToken cancellationToken = default);
    }
}
```

## HttpClient Configuration

### Program.cs Setup

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Configure API base URL from appsettings
var apiBaseUrl = builder.Configuration["ApiSettings:BaseUrl"] ?? "/";
var apiPathPrefix = builder.Configuration["ApiSettings:PathPrefix"] ?? "api";
var fullBaseUrl = $"{apiBaseUrl}{apiPathPrefix}";

// Register HttpClient with base address
builder.Services.AddHttpClient("TronApi", client =>
{
    client.BaseAddress = new Uri(fullBaseUrl);
    client.Timeout = TimeSpan.FromSeconds(30);
    // Add default headers if needed
    client.DefaultRequestHeaders.Add("Accept", "application/json");
})
.ConfigurePrimaryHttpMessageHandler(() => new HttpClientHandler
{
    // Configure SSL/TLS if needed
    ServerCertificateCustomValidationCallback = (sender, cert, chain, sslPolicyErrors) => true // DEV ONLY
});

// Register API clients
builder.Services.AddScoped<IDashboardUserControllerApi>(sp =>
{
    var httpClientFactory = sp.GetRequiredService<IHttpClientFactory>();
    var httpClient = httpClientFactory.CreateClient("TronApi");
    return new DashboardUserControllerApi(httpClient);
});

builder.Services.AddScoped<IAppClientControllerApi>(sp =>
{
    var httpClientFactory = sp.GetRequiredService<IHttpClientFactory>();
    var httpClient = httpClientFactory.CreateClient("TronApi");
    return new AppClientControllerApi(httpClient);
});

// ... register other API clients
```

### appsettings.json

```json
{
  "ApiSettings": {
    "BaseUrl": "/",
    "PathPrefix": "api",
    "VersionPrefix": "v1",
    "Version2Prefix": "v2"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

### appsettings.Development.json

```json
{
  "ApiSettings": {
    "BaseUrl": "http://localhost:8088/",
    "PathPrefix": "api",
    "VersionPrefix": "v1",
    "Version2Prefix": "v2"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft.AspNetCore": "Information"
    }
  }
}
```

## Authentication Integration

### HTTP Message Handler for Auth

```csharp
// TronDashboard.Core/Http/AuthenticatedHttpMessageHandler.cs
public class AuthenticatedHttpMessageHandler : DelegatingHandler
{
    private readonly IHttpContextAccessor _httpContextAccessor;
    
    public AuthenticatedHttpMessageHandler(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }
    
    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, 
        CancellationToken cancellationToken)
    {
        // Forward authentication from current HTTP context to API calls
        var httpContext = _httpContextAccessor.HttpContext;
        if (httpContext?.Request.Headers.TryGetValue("Authorization", out var authHeader) == true)
        {
            request.Headers.Add("Authorization", authHeader.ToString());
        }
        
        // Forward x-forwarded-client-cert if present (SSO)
        if (httpContext?.Request.Headers.TryGetValue("x-forwarded-client-cert", out var xfccHeader) == true)
        {
            request.Headers.Add("x-forwarded-client-cert", xfccHeader.ToString());
        }
        
        return await base.SendAsync(request, cancellationToken);
    }
}

// Register in Program.cs
builder.Services.AddTransient<AuthenticatedHttpMessageHandler>();
builder.Services.AddHttpClient("TronApi")
    .AddHttpMessageHandler<AuthenticatedHttpMessageHandler>();
```

## Error Handling

### Custom Exception Wrapper

```csharp
// TronDashboard.Core/Exceptions/DataCrudException.cs
public class DataCrudException : Exception
{
    public int? StatusCode { get; }
    public Dictionary<string, string>? ValidationErrors { get; }
    
    public DataCrudException(string message, int? statusCode = null, Dictionary<string, string>? validationErrors = null)
        : base(message)
    {
        StatusCode = statusCode;
        ValidationErrors = validationErrors;
    }
    
    public static DataCrudException FromApiException(ApiException apiException)
    {
        var validationErrors = ParseValidationErrors(apiException.Response);
        
        return new DataCrudException(
            apiException.Message,
            apiException.StatusCode,
            validationErrors
        );
    }
    
    private static Dictionary<string, string>? ParseValidationErrors(string? response)
    {
        if (string.IsNullOrEmpty(response))
            return null;
        
        try
        {
            var exceptionResponse = JsonSerializer.Deserialize<ExceptionResponse>(response);
            if (exceptionResponse?.Errors == null)
                return null;
            
            return exceptionResponse.Errors.ToDictionary(
                e => e.FieldName ?? "general",
                e => e.DefaultMessage ?? "Validation error"
            );
        }
        catch
        {
            return null;
        }
    }
}

// Usage in service
public async Task<DashboardUserFlat> CreateAsync(DashboardUserDto dto, CancellationToken ct = default)
{
    try
    {
        var response = await _api.AddDashboardUserAsync(dto, ct);
        return ConvertToFlat(response);
    }
    catch (ApiException ex)
    {
        throw DataCrudException.FromApiException(ex);
    }
}
```

## Testing the API Client

### Unit Test with Mocked API

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
    public async Task FetchAndStoreDataAsync_ReturnsUsers()
    {
        // Arrange
        var users = new List<DashboardUserDto>
        {
            new() { Id = "1", Email = "test@example.com", Privileges = new List<PrivilegeDto>() }
        };
        var response = new WrappedResponse<List<DashboardUserDto>> { Data = users };
        
        _mockApi
            .Setup(x => x.GetAllDashboardUsersWrappedAsync(It.IsAny<CancellationToken>()))
            .ReturnsAsync(response);
        
        // Act
        var result = await _service.FetchAndStoreDataAsync();
        
        // Assert
        Assert.Single(result);
        Assert.Equal("test@example.com", result[0].Email);
    }
}
```

## Migration Checklist

### POC Phase
- [ ] Install NSwag.MSBuild
- [ ] Create nswag.json configuration
- [ ] Generate C# client from tron-common-api.json
- [ ] Verify DashboardUserControllerApi generated
- [ ] Configure HttpClient in Program.cs
- [ ] Register IDashboardUserControllerApi in DI
- [ ] Test API call from service
- [ ] Implement authentication forwarding
- [ ] Test error handling

### Phase 1
- [ ] Register all API clients in DI
- [ ] Implement retry policies (Polly)
- [ ] Add request/response logging
- [ ] Performance test API calls

### Phase 2-4
- [ ] Implement caching strategies
- [ ] Add circuit breaker pattern
- [ ] Monitor API client performance

---

**Next Step**: Proceed to 08-TEST-STRATEGY.md
