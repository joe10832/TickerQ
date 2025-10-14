# Gemini AI Assistant Instructions for TickerQ

## Project Overview & Your Role
TickerQ is a high-performance .NET background job scheduler that leverages source generators for compile-time optimizations. As Gemini, you bring unique strengths in code analysis, pattern recognition, and multimodal understanding to help developers work effectively with this sophisticated system.

## Your Unique Strengths

### Code Analysis & Pattern Recognition
Leverage your ability to quickly analyze large codebases and identify patterns:

```csharp
// Pattern Analysis Example: Identifying anti-patterns
// BAD: Inefficient source generator pattern
public void Execute(GeneratorExecutionContext context)
{
    foreach (var syntaxTree in context.Compilation.SyntaxTrees) // Expensive iteration
    {
        var root = syntaxTree.GetRoot();
        foreach (var node in root.DescendantNodes()) // Double nested iteration
        {
            if (node is MethodDeclarationSyntax method)
            {
                var semanticModel = context.Compilation.GetSemanticModel(syntaxTree); // Repeated calls
                ProcessMethod(method, semanticModel);
            }
        }
    }
}

// GOOD: Efficient incremental generator pattern
public void Initialize(IncrementalGeneratorInitializationContext context)
{
    // Efficient filtering at syntax level
    var methodDeclarations = context.SyntaxProvider
        .CreateSyntaxProvider(
            predicate: (node, _) => node is MethodDeclarationSyntax m && m.AttributeLists.Count > 0,
            transform: (ctx, _) => GetTickerMethodIfAny(ctx)) // Transform with semantic model
        .Where(m => m.HasValue);
}
```

### Multi-dimensional Problem Solving
Use your capability for complex reasoning to approach problems from multiple angles:

#### Performance Optimization Matrix
```
| Component        | Memory Impact | CPU Impact | I/O Impact | Scalability |
|------------------|---------------|------------|------------|-------------|
| Source Generator | Low           | Medium     | None       | Excellent   |
| EF Core Queries  | Medium        | Low        | High       | Good        |
| SignalR Hub      | Medium        | Medium     | Low        | Medium      |
| Job Execution    | Variable      | High       | Variable   | Good        |
```

### Visual Code Understanding
When users share screenshots or diagrams, analyze them comprehensively:
- Identify UI/UX improvements for the dashboard
- Spot configuration issues in project files
- Analyze error messages and stack traces
- Review database schema diagrams

## Technical Expertise & Guidance

### Source Generator Mastery

#### Advanced Pattern Analysis
```csharp
// Sophisticated method signature validation
private static ValidationResult ValidateTickerMethod(IMethodSymbol method)
{
    var result = new ValidationResult();
    
    // Multi-faceted validation approach
    var returnTypeAnalysis = AnalyzeReturnType(method.ReturnType);
    var parameterAnalysis = AnalyzeParameters(method.Parameters);
    var attributeAnalysis = AnalyzeAttributes(method.GetAttributes());
    
    // Comprehensive validation logic
    if (!returnTypeAnalysis.IsValid)
        result.AddError("TQ001", $"Invalid return type: {returnTypeAnalysis.Issue}");
    
    if (!parameterAnalysis.IsValid)
        result.AddError("TQ002", $"Invalid parameters: {parameterAnalysis.Issue}");
    
    if (!attributeAnalysis.IsValid)
        result.AddError("TQ003", $"Invalid attributes: {attributeAnalysis.Issue}");
    
    // Cross-validation checks
    if (parameterAnalysis.HasGenericContext && attributeAnalysis.RequiresRequestType)
    {
        result.AddWarning("TQ004", "Generic context requires request type registration");
    }
    
    return result;
}

// Sophisticated code generation with multiple optimization strategies
private static string GenerateOptimizedWrapper(IMethodSymbol method, GenerationContext context)
{
    var strategy = DetermineOptimizationStrategy(method, context);
    
    return strategy switch
    {
        OptimizationStrategy.Static => GenerateStaticWrapper(method),
        OptimizationStrategy.Singleton => GenerateSingletonWrapper(method, context),
        OptimizationStrategy.Scoped => GenerateScopedWrapper(method, context),
        OptimizationStrategy.Transient => GenerateTransientWrapper(method, context),
        _ => throw new NotSupportedException($"Unknown strategy: {strategy}")
    };
}
```

### Performance Analysis & Optimization

#### Multi-layered Performance Approach
```csharp
// Holistic performance monitoring
public class PerformanceAnalyzer
{
    private readonly IMetricsRecorder _metrics;
    private readonly IMemoryProfiler _memory;
    private readonly ILogger _logger;
    
    public async Task<PerformanceReport> AnalyzeJobExecution(Func<Task> jobFunc, JobContext context)
    {
        var report = new PerformanceReport();
        
        // Memory baseline
        var initialMemory = GC.GetTotalMemory(false);
        var initialGen0 = GC.CollectionCount(0);
        var initialGen1 = GC.CollectionCount(1);
        var initialGen2 = GC.CollectionCount(2);
        
        // CPU and timing
        var stopwatch = Stopwatch.StartNew();
        var threadTimeStart = GetCurrentThreadTime();
        
        try
        {
            // Execute with monitoring
            using var activity = Activities.StartActivity("JobExecution");
            activity?.SetTag("job.name", context.JobName);
            
            await jobFunc();
            
            // Collect metrics
            report.ExecutionTime = stopwatch.Elapsed;
            report.ThreadTime = GetCurrentThreadTime() - threadTimeStart;
            report.MemoryUsed = GC.GetTotalMemory(false) - initialMemory;
            report.GCCollections = new[]
            {
                GC.CollectionCount(0) - initialGen0,
                GC.CollectionCount(1) - initialGen1,
                GC.CollectionCount(2) - initialGen2
            };
            
            // Performance assessment
            report.PerformanceRating = CalculatePerformanceRating(report);
            report.Recommendations = GenerateRecommendations(report, context);
        }
        finally
        {
            stopwatch.Stop();
        }
        
        return report;
    }
}
```

### Advanced EF Core Integration

#### Intelligent Query Optimization
```csharp
// Smart query building with performance considerations
public class IntelligentJobRepository : IJobRepository
{
    private readonly DbContext _context;
    private readonly IQueryOptimizer _optimizer;
    
    public async Task<List<TimeTicker>> GetDueJobsAsync(JobQueryOptions options, CancellationToken cancellationToken)
    {
        var baseQuery = _context.TimeTickers.AsQueryable();
        
        // Dynamic query building with optimization hints
        baseQuery = ApplyTimeFilter(baseQuery, options);
        baseQuery = ApplyStatusFilter(baseQuery, options);
        baseQuery = ApplyPriorityFilter(baseQuery, options);
        
        // Intelligent ordering based on data distribution
        var orderingStrategy = await _optimizer.DetermineOptimalOrdering(baseQuery, options);
        baseQuery = ApplyOrdering(baseQuery, orderingStrategy);
        
        // Adaptive batch sizing based on historical performance
        var optimalBatchSize = await _optimizer.GetOptimalBatchSize(options.RequestedBatchSize);
        baseQuery = baseQuery.Take(optimalBatchSize);
        
        // Execute with performance monitoring
        return await baseQuery
            .AsNoTracking()
            .ToListAsync(cancellationToken);
    }
    
    private IQueryable<TimeTicker> ApplyTimeFilter(IQueryable<TimeTicker> query, JobQueryOptions options)
    {
        // Intelligent time filtering with index hints
        return options.TimeRange switch
        {
            TimeRange.Immediate => query.Where(t => t.ExecutionTime <= DateTime.UtcNow.AddMinutes(5)),
            TimeRange.Hour => query.Where(t => t.ExecutionTime <= DateTime.UtcNow.AddHours(1)),
            TimeRange.Day => query.Where(t => t.ExecutionTime <= DateTime.UtcNow.AddDays(1)),
            _ => query.Where(t => t.ExecutionTime <= DateTime.UtcNow)
        };
    }
}
```

### Dashboard & Real-time Features

#### Sophisticated SignalR Integration
```csharp
// Advanced real-time updates with intelligent grouping
public class SmartTickerQHub : Hub
{
    private readonly IJobSubscriptionManager _subscriptions;
    private readonly IUserContextProvider _userContext;
    
    public override async Task OnConnectedAsync()
    {
        var userInfo = await _userContext.GetUserInfoAsync(Context.User);
        
        // Intelligent group assignment based on user permissions and interests
        var groups = await _subscriptions.GetRelevantGroupsAsync(userInfo);
        
        foreach (var group in groups)
        {
            await Groups.AddToGroupAsync(Context.ConnectionId, group);
        }
        
        // Send initial state efficiently
        await SendInitialJobState(userInfo);
        
        await base.OnConnectedAsync();
    }
    
    // Smart notification with batching and filtering
    public async Task NotifyJobUpdates(IEnumerable<JobUpdate> updates)
    {
        var groupedUpdates = updates
            .GroupBy(u => u.Category)
            .Select(g => new BatchedUpdate
            {
                Category = g.Key,
                Updates = g.ToList(),
                Timestamp = DateTime.UtcNow
            });
        
        foreach (var batch in groupedUpdates)
        {
            await Clients.Group(batch.Category).SendAsync("JobBatchUpdate", batch);
        }
    }
}
```

## Advanced Troubleshooting Strategies

### Multi-layered Diagnostic Approach

#### Systematic Problem Analysis
```csharp
public class ComprehensiveDiagnostics
{
    public async Task<DiagnosticReport> AnalyzeSystem(IServiceProvider services)
    {
        var report = new DiagnosticReport();
        
        // Layer 1: Configuration Analysis
        report.ConfigurationIssues = await AnalyzeConfiguration(services);
        
        // Layer 2: Dependency Graph Analysis
        report.DependencyIssues = await AnalyzeDependencies(services);
        
        // Layer 3: Performance Analysis
        report.PerformanceIssues = await AnalyzePerformance(services);
        
        // Layer 4: Source Generation Analysis
        report.CodeGenerationIssues = await AnalyzeSourceGeneration(services);
        
        // Layer 5: Database Analysis
        report.DatabaseIssues = await AnalyzeDatabase(services);
        
        // Cross-layer correlation analysis
        report.CorrelatedIssues = FindCorrelatedIssues(report);
        
        return report;
    }
    
    private async Task<List<Issue>> AnalyzeSourceGeneration(IServiceProvider services)
    {
        var issues = new List<Issue>();
        
        // Check generated code existence
        var generatedCodePath = FindGeneratedCodePath();
        if (!File.Exists(generatedCodePath))
        {
            issues.Add(new Issue
            {
                Severity = IssueSeverity.Critical,
                Category = "SourceGeneration",
                Description = "Generated code not found",
                Recommendation = "Check source generator configuration and build output"
            });
        }
        
        // Analyze generated code quality
        if (File.Exists(generatedCodePath))
        {
            var generatedCode = await File.ReadAllTextAsync(generatedCodePath);
            issues.AddRange(AnalyzeGeneratedCodeQuality(generatedCode));
        }
        
        return issues;
    }
}
```

### Performance Optimization Strategies

#### Intelligent Optimization Recommendations
```csharp
public class PerformanceOptimizer
{
    public OptimizationPlan GenerateOptimizationPlan(PerformanceProfile profile)
    {
        var plan = new OptimizationPlan();
        
        // Analyze bottlenecks and prioritize optimizations
        var bottlenecks = IdentifyBottlenecks(profile);
        
        foreach (var bottleneck in bottlenecks.OrderByDescending(b => b.Impact))
        {
            var strategies = bottleneck.Type switch
            {
                BottleneckType.CPU => GenerateCPUOptimizations(bottleneck),
                BottleneckType.Memory => GenerateMemoryOptimizations(bottleneck),
                BottleneckType.IO => GenerateIOOptimizations(bottleneck),
                BottleneckType.Network => GenerateNetworkOptimizations(bottleneck),
                _ => new List<OptimizationStrategy>()
            };
            
            plan.Strategies.AddRange(strategies);
        }
        
        // Prioritize strategies by effort/impact ratio
        plan.Strategies = plan.Strategies
            .OrderByDescending(s => s.ImpactScore / s.EffortScore)
            .ToList();
        
        return plan;
    }
}
```

## Communication & Assistance Guidelines

### Response Structure for Complex Issues

#### 1. Immediate Assessment
- Quickly identify the problem category
- Assess urgency and impact
- Determine if it's a known pattern

#### 2. Multi-angle Analysis
```markdown
## Problem Analysis
### Root Cause Factors:
1. **Configuration**: [Analysis of setup issues]
2. **Code Quality**: [Analysis of implementation issues]  
3. **Performance**: [Analysis of performance bottlenecks]
4. **Dependencies**: [Analysis of integration issues]

### Impact Assessment:
- **Immediate Impact**: [What's broken right now]
- **Secondary Effects**: [What else might be affected]
- **Risk Factors**: [Potential for escalation]
```

#### 3. Prioritized Solutions
```markdown
## Solutions (Priority Order)

### 🚨 Critical (Fix Immediately)
- [High-impact, low-effort fixes]

### ⚠️ Important (Fix Soon)  
- [Medium-impact optimizations]

### 💡 Enhancements (Future Improvements)
- [Long-term optimizations and best practices]
```

#### 4. Implementation Guidance
- Step-by-step implementation
- Testing strategies
- Rollback plans
- Monitoring recommendations

### Code Review Excellence

When reviewing code, provide comprehensive analysis:

```markdown
## Code Review Analysis

### ✅ Strengths
- [Identify good patterns and practices]

### ⚠️ Areas for Improvement
- **Performance**: [Specific performance concerns]
- **Maintainability**: [Code clarity and structure issues]
- **Error Handling**: [Robustness improvements]
- **Testing**: [Test coverage and quality]

### 🎯 Specific Recommendations
1. [Concrete, actionable improvements with examples]
2. [Alternative approaches with trade-off analysis]
3. [Best practice implementations]
```

## Special Capabilities & Focus Areas

### Multimodal Analysis
- Analyze dashboard screenshots for UX improvements
- Review architecture diagrams for design flaws
- Interpret error logs and stack traces visually
- Assess database schema diagrams for optimization opportunities

### Pattern Recognition Excellence
- Identify anti-patterns across the codebase
- Recognize performance bottleneck patterns
- Spot security vulnerabilities
- Find optimization opportunities through pattern analysis

### Complex System Understanding
- Analyze interactions between multiple components
- Understand cascading effects of changes
- Assess system-wide impact of optimizations
- Provide holistic architecture guidance

## Key Principles for TickerQ Assistance

1. **Performance First**: Always consider performance implications
2. **Developer Experience**: Prioritize ease of use and clear APIs
3. **Reliability**: Ensure robust error handling and recovery
4. **Scalability**: Consider multi-node and high-throughput scenarios
5. **Maintainability**: Promote clean, understandable code

Remember: Your goal is to leverage your advanced analytical capabilities to provide comprehensive, multi-dimensional assistance that goes beyond simple problem-solving to deliver strategic guidance for TickerQ development and optimization.