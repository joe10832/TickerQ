# TickerQ Source Generator Development Instructions

## Overview
When working with TickerQ's source generators, you're dealing with compile-time code generation that creates the glue code between user-defined ticker functions and the TickerQ runtime.

## Key Files
- `TickerQIncrementalSourceGenerator.cs` - Modern incremental source generator (preferred)
- `TickerQClassicSourceGenerator.cs` - Legacy source generator for older .NET versions

## Source Generator Principles

### Incremental Generation (Preferred)
- Use `IIncrementalGenerator` for better performance
- Implement proper caching and change detection
- Use `RegisterSourceOutput` for compilation outputs
- Filter syntax nodes efficiently at the earliest stage

### Syntax Processing
```csharp
// Efficient syntax filtering
var tickerMethods = context.SyntaxProvider
    .CreateSyntaxProvider(
        predicate: (node, _) => node is MethodDeclarationSyntax m && m.AttributeLists.Count > 0,
        transform: GetTickerMethodIfAny
    )
    .Where(m => m.HasValue);
```

### Code Generation Patterns

#### Function Registration
Generate dictionary entries for each ticker function:
```csharp
tickerFunctionDelegateDict.Add("FunctionName", ("cronExpression", TickerTaskPriority.Normal, 
    async (cancellationToken, serviceProvider, context) => {
        // Generated wrapper code
    }));
```

#### Service Resolution
```csharp
// For non-static methods
var service = serviceProvider.GetService<MyService>();
await service.MyMethod(parameters);

// For static methods
await MyClass.MyMethod(parameters);
```

#### Context Handling
```csharp
// Generic context conversion
var genericContext = await ToGenericContextWithRequest<RequestType>(
    context, serviceProvider, tickerId, tickerType);
```

## Validation Requirements

### Class Validation
- Must be public or internal
- Must be concrete (not abstract)
- Must have accessible constructor

### Method Validation
- Must be public
- Must be async (return Task)
- Must have correct parameter signature
- Must have TickerFunction attribute

### Parameter Validation
```csharp
// Valid signatures:
Task Method(CancellationToken cancellationToken)
Task Method(TickerFunctionContext context, CancellationToken cancellationToken)  
Task Method(TickerFunctionContext<T> context, CancellationToken cancellationToken)
```

## Error Handling
- Report diagnostics using `context.ReportDiagnostic()`
- Use descriptive error messages
- Include location information in diagnostics
- Handle edge cases gracefully

## Performance Considerations
- Cache semantic models appropriately
- Minimize string allocations
- Use StringBuilder for code generation
- Filter syntax nodes early in the pipeline
- Avoid processing the same symbols multiple times

## Generated Code Structure
```csharp
// Generated factory class
public static class TickerQInstanceFactory
{
    [ModuleInitializer] // For .NET 5+
    public static void Initialize()
    {
        var tickerFunctionDelegateDict = new Dictionary<string, (string, TickerTaskPriority, TickerFunctionDelegate)>();
        
        // Generated function registrations
        RegisterFunctions();
        RegisterRequestTypes();
    }
    
    // Generated service factory methods
    private static MyService CreateMyService(IServiceProvider serviceProvider) { ... }
}
```

## Debugging Tips
- Use `#if DEBUG` blocks for debugging output
- Generate readable code for easier debugging
- Include version comments in generated files
- Test with various method signatures
- Validate generated code compiles correctly

## Multi-targeting Considerations
- Handle .NET Framework vs .NET Core differences
- Use appropriate preprocessor directives
- Support both incremental and classic generators
- Test across target frameworks

## Testing Source Generators
- Use `GeneratorDriver` for unit testing
- Test with various syntax patterns
- Verify generated code correctness
- Test error conditions and edge cases
- Use compilation tests to verify generated code compiles

When modifying source generators, always consider backward compatibility and performance impact on build times.