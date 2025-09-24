# TickerQ Entity Framework Core Development Instructions

## Overview
The Entity Framework Core integration provides persistent storage for TickerQ jobs with support for various database providers and clean separation of concerns.

## Key Components
- `TimeTickerConfigurations` - EF configuration for time-based jobs
- `CronTickerConfigurations` - EF configuration for cron-based jobs  
- `CronTickerOccurrenceConfigurations` - EF configuration for cron job occurrences
- `ITickerPersistenceProvider<TTimeTicker, TCronTicker>` - Abstraction for data access

## Entity Configuration Patterns

### Using Model Customizer (Recommended)
```csharp
services.AddTickerQ(options =>
{
    options.AddOperationalStore<MyDbContext>(efOpt => 
    {
        efOpt.UseModelCustomizerForMigrations(); // Recommended approach
    });
});
```

### Manual Configuration
```csharp
protected override void OnModelCreating(ModelBuilder builder)
{
    base.OnModelCreating(builder);
    
    // Apply TickerQ configurations
    builder.ApplyConfiguration(new TimeTickerConfigurations());
    builder.ApplyConfiguration(new CronTickerConfigurations());  
    builder.ApplyConfiguration(new CronTickerOccurrenceConfigurations());
}
```

## Entity Design Principles

### Table Structure
- Use appropriate primary keys (typically Guid)
- Include necessary indexes for query performance
- Support soft deletes where appropriate
- Include auditing fields (created, updated timestamps)

### Data Types
```csharp
// Proper column types
entity.Property(e => e.Id).IsRequired();
entity.Property(e => e.Function).HasMaxLength(255).IsRequired();
entity.Property(e => e.Expression).HasMaxLength(500); // For cron expressions
entity.Property(e => e.Request).HasColumnType("nvarchar(max)"); // JSON storage
```

### Relationships
```csharp
// One-to-many for cron occurrences
entity.HasMany(e => e.Occurrences)
      .WithOne(e => e.CronTicker)
      .HasForeignKey(e => e.CronTickerId)
      .OnDelete(DeleteBehavior.Cascade);
```

## Query Patterns

### Efficient Job Retrieval
```csharp
// Get due time-based jobs
var dueTickers = await context.TimeTickers
    .Where(t => t.ExecutionTime <= DateTime.UtcNow && t.Status == TickerStatus.Pending)
    .OrderBy(t => t.ExecutionTime)
    .Take(batchSize)
    .ToListAsync(cancellationToken);

// Get due cron jobs
var dueCronJobs = await context.CronTickers
    .Where(c => c.NextExecution <= DateTime.UtcNow && c.IsActive)
    .Include(c => c.Occurrences)
    .ToListAsync(cancellationToken);
```

### Batch Operations
```csharp
// Efficient batch updates
await context.Database.ExecuteSqlRawAsync(
    "UPDATE TimeTickers SET Status = {0}, CompletedAt = {1} WHERE Id IN ({2})",
    TickerStatus.Completed,
    DateTime.UtcNow,
    string.Join(",", completedIds));
```

## Migration Strategy

### Initial Migration
```bash
# Create initial migration
Add-Migration "TickerQInitialCreate" -Context MyDbContext

# Apply migration
Update-Database -Context MyDbContext
```

### Schema Updates
- Use data migrations for complex schema changes
- Maintain backward compatibility where possible
- Test migrations on production-like data
- Document breaking changes

## Performance Optimization

### Indexing Strategy
```csharp
// Key indexes for performance
entity.HasIndex(e => e.ExecutionTime).HasDatabaseName("IX_TimeTicker_ExecutionTime");
entity.HasIndex(e => e.Status).HasDatabaseName("IX_TimeTicker_Status");
entity.HasIndex(e => new { e.Function, e.Status }).HasDatabaseName("IX_TimeTicker_Function_Status");
```

### Query Optimization
- Use projections to reduce data transfer
- Implement proper pagination
- Use compiled queries for frequently executed queries
- Monitor query performance with profiling

### Connection Management
```csharp
// Proper DbContext registration
services.AddDbContext<MyDbContext>(options =>
{
    options.UseSqlServer(connectionString);
    options.EnableServiceProviderCaching();
    options.EnableSensitiveDataLogging(isDevelopment);
});
```

## Multi-tenancy Support
```csharp
// Tenant isolation
entity.HasQueryFilter(e => e.TenantId == CurrentTenant.Id);

// Tenant-specific configuration
entity.Property(e => e.TenantId).IsRequired();
entity.HasIndex(e => e.TenantId).HasDatabaseName("IX_TimeTicker_TenantId");
```

## Exception Handling
```csharp
// Custom exception handler
public class MyTickerExceptionHandler : ITickerExceptionHandler
{
    public async Task HandleAsync(Exception exception, TickerFunctionContext context)
    {
        // Log exception details
        // Update job status
        // Implement retry logic
    }
}
```

## Testing Strategies

### In-Memory Testing
```csharp
services.AddDbContext<TestDbContext>(options =>
    options.UseInMemoryDatabase(Guid.NewGuid().ToString()));
```

### Integration Testing
```csharp
// Use test containers or local database
services.AddDbContext<TestDbContext>(options =>
    options.UseSqlServer(testConnectionString));
```

## Database Provider Support
- SQL Server (primary)
- PostgreSQL
- SQLite (for testing)
- MySQL/MariaDB
- Ensure provider-specific optimizations

## Monitoring & Diagnostics
- Enable EF Core logging in development
- Monitor slow queries in production
- Track connection pool metrics
- Implement health checks for database connectivity

When working with EF Core integration, prioritize query performance, proper indexing, and clean separation between TickerQ concerns and your application's domain model.