# GitHub Copilot Instructions for TickerQ

## Project Overview

TickerQ is a fast, reflection-free background task scheduler for .NET built with source generators, EF Core integration, cron + time-based execution, and a real-time dashboard. The project emphasizes performance, reliability, and developer experience.

## Architecture & Key Components

### Core Libraries
- **TickerQ.Utilities**: Core models, enums, and base functionality
- **TickerQ**: Main library with dependency injection and hosting
- **TickerQ.EntityFrameworkCore**: EF Core persistence layer
- **TickerQ.Dashboard**: Real-time dashboard UI
- **TickerQ.SourceGenerator**: Compile-time source generation for performance

### Key Concepts
- **Source Generation**: Uses compile-time source generators instead of reflection
- **Ticker Functions**: Methods marked with `[TickerFunction]` attribute for job definitions
- **Time Tickers**: One-time scheduled jobs
- **Cron Tickers**: Recurring jobs based on cron expressions
- **Stateless Design**: Core components are stateless for scalability

## Coding Guidelines

### C# Style & Patterns
- Use modern C# features (pattern matching, using declarations, etc.)
- Prefer explicit typing for complex types, `var` for obvious types
- Use primary constructors for dependency injection where appropriate
- Follow async/await patterns consistently
- Use `CancellationToken` for all async operations

### Source Generator Specific
- When working with source generators, be mindful of compile-time vs runtime
- Use `IIncrementalGenerator` for new generators (preferred over `ISourceGenerator`)
- Validate syntax nodes before processing
- Handle edge cases gracefully in generators
- Generate clean, readable code with proper indentation

### Attribute-Based Design
```csharp
[TickerFunction(functionName: "MyJob", cronExpression: "0 0 * * *")]
public async Task MyJob(TickerFunctionContext<MyRequest> context, CancellationToken cancellationToken)
{
    // Implementation
}
```

### Dependency Injection
- Use constructor injection consistently
- Register services in `AddTickerQ()` extension method
- Support both scoped and singleton lifetimes appropriately
- Use `IServiceProvider` for dynamic service resolution in generated code

### Entity Framework
- Use configuration classes for entity mapping
- Support both automatic and manual configuration
- Handle migrations cleanly
- Separate infrastructure concerns from domain logic

## Testing Guidelines
- Write unit tests for core logic
- Use integration tests for EF Core scenarios
- Test source generators with syntax trees
- Mock dependencies appropriately
- Test both success and failure scenarios

## Performance Considerations
- Avoid reflection where possible (use source generation)
- Use efficient data structures for job scheduling
- Implement proper cancellation support
- Consider memory allocation patterns
- Use async/await efficiently

## Dashboard & UI
- Real-time updates using SignalR
- Clean, responsive UI design
- Support for job scheduling, monitoring, and management
- Basic authentication support
- RESTful API design

## Common Patterns

### Job Definition
```csharp
public class MyJobService(IMyDependency dependency)
{
    [TickerFunction("ProcessData", "0 */5 * * *")]
    public async Task ProcessData(TickerFunctionContext<MyRequest> context, CancellationToken cancellationToken)
    {
        var request = context.Request;
        await dependency.ProcessAsync(request, cancellationToken);
    }
}
```

### Service Registration
```csharp
services.AddTickerQ(options =>
{
    options.SetMaxConcurrency(10);
    options.AddOperationalStore<MyDbContext>(efOpt => 
    {
        efOpt.SetExceptionHandler<MyExceptionHandler>();
        efOpt.UseModelCustomizerForMigrations();
    });
    options.AddDashboard(uiOpt =>
    {
        uiOpt.BasePath = "/tickerq-dashboard";
        uiOpt.AddDashboardBasicAuth();
    });
});
```

### Scheduling Jobs
```csharp
// Time-based
await timeTickerManager.AddAsync(new TimeTicker
{
    Function = "ProcessData",
    ExecutionTime = DateTime.UtcNow.AddMinutes(5),
    Request = TickerHelper.CreateTickerRequest<MyRequest>(data),
    Retries = 3,
    RetryIntervals = new[] { 30, 60, 120 }
});

// Cron-based
await cronTickerManager.AddAsync(new CronTicker
{
    Function = "ProcessData",
    Expression = "0 */15 * * *",
    Request = TickerHelper.CreateTickerRequest<MyRequest>(data)
});
```

## Error Handling
- Use structured exception handling
- Support custom exception handlers
- Implement retry logic with configurable intervals
- Log errors appropriately
- Graceful degradation for non-critical failures

## Multi-targeting
- Support .NET Standard 2.1 for libraries
- Target appropriate .NET versions for applications
- Handle version-specific features appropriately

## Documentation
- Use XML documentation for public APIs
- Include usage examples in documentation
- Keep README and docs up to date
- Document breaking changes clearly

## Security
- Basic authentication for dashboard
- Secure service communication
- Input validation for user data
- Safe handling of sensitive information

When contributing to TickerQ, focus on maintaining high performance, clean architecture, and excellent developer experience. Always consider the impact on compile-time performance when working with source generators.