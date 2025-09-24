# TickerQ Dashboard Development Instructions

## Overview
The TickerQ Dashboard provides a real-time web interface for monitoring, managing, and scheduling background jobs. It's built with modern web technologies and SignalR for real-time updates.

## Key Features
- Real-time job monitoring and status updates
- Job scheduling interface (time-based and cron-based)
- Job history and analytics
- Basic authentication support
- RESTful API for job management
- Responsive design for mobile and desktop

## Architecture Components

### Dashboard Setup
```csharp
services.AddTickerQ(options =>
{
    options.AddDashboard(uiOpt =>
    {
        uiOpt.BasePath = "/tickerq-dashboard";
        uiOpt.AddDashboardBasicAuth();
        uiOpt.EnableRealTimeUpdates = true;
        uiOpt.RefreshIntervalMs = 5000;
    });
});
```

### Authentication Configuration
```csharp
// Basic authentication
uiOpt.AddDashboardBasicAuth(authOpt =>
{
    authOpt.Username = "admin";
    authOpt.Password = "secure_password";
    authOpt.RequireHttps = true; // For production
});

// Custom authentication
uiOpt.AddCustomAuthentication<MyAuthHandler>();
```

## API Endpoints

### Job Management Endpoints
```
GET    /api/tickers/time                    # Get time-based jobs
POST   /api/tickers/time                    # Create time-based job
PUT    /api/tickers/time/{id}               # Update time-based job
DELETE /api/tickers/time/{id}               # Delete time-based job

GET    /api/tickers/cron                    # Get cron-based jobs  
POST   /api/tickers/cron                    # Create cron-based job
PUT    /api/tickers/cron/{id}               # Update cron-based job
DELETE /api/tickers/cron/{id}               # Delete cron-based job

GET    /api/tickers/functions               # Get available functions
POST   /api/tickers/{id}/trigger            # Trigger job manually
POST   /api/tickers/{id}/cancel             # Cancel running job
```

### Real-time Updates
```csharp
// SignalR hub for real-time updates
public class TickerQHub : Hub
{
    public async Task JoinGroup(string groupName)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
    }
    
    // Broadcast job status changes
    public async Task NotifyJobStatusChange(string jobId, string status)
    {
        await Clients.All.SendAsync("JobStatusChanged", jobId, status);
    }
}
```

## Frontend Development

### UI Components
- Job list view with filtering and sorting
- Job detail modal with execution history
- Job creation/editing forms
- Real-time status indicators
- Cron expression builder/validator
- Request/response payload viewer

### JavaScript Integration
```javascript
// SignalR connection
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/hubs/tickerq")
    .build();

connection.start().then(function () {
    // Join updates group
    connection.invoke("JoinGroup", "job-updates");
    
    // Listen for job status changes
    connection.on("JobStatusChanged", function (jobId, status) {
        updateJobStatus(jobId, status);
    });
});

// API integration
async function createTimeJob(jobData) {
    const response = await fetch('/api/tickers/time', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(jobData)
    });
    
    if (response.ok) {
        const job = await response.json();
        notifyJobCreated(job);
    }
}
```

### Styling Guidelines
- Use modern CSS Grid/Flexbox for layouts
- Implement responsive design principles
- Follow accessibility guidelines (ARIA labels, keyboard navigation)
- Use consistent color scheme and typography
- Implement loading states and error handling

## Data Transfer Objects

### Request DTOs
```csharp
public class AddTimeTickerRequest
{
    public string Function { get; set; }
    public DateTime ExecutionTime { get; set; }
    public string? Description { get; set; }
    public string? Request { get; set; } // JSON payload
    public int Retries { get; set; } = 0;
    public int[] RetryIntervals { get; set; }
    public Guid? BatchParent { get; set; }
    public BatchRunCondition? BatchRunCondition { get; set; }
}

public class AddCronTickerRequest  
{
    public string Function { get; set; }
    public string Expression { get; set; }
    public string? Description { get; set; }
    public string? Request { get; set; }
    public bool IsActive { get; set; } = true;
    public int Retries { get; set; } = 0;
    public int[] RetryIntervals { get; set; }
}
```

### Response DTOs
```csharp
public class TickerResponse
{
    public Guid Id { get; set; }
    public string Function { get; set; }
    public string Status { get; set; }
    public DateTime? LastExecution { get; set; }
    public DateTime? NextExecution { get; set; }
    public string? ErrorMessage { get; set; }
    public int ExecutionCount { get; set; }
}
```

## Security Considerations

### Input Validation
```csharp
[ApiController]
public class TickerController : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> CreateTimeTicker([FromBody] AddTimeTickerRequest request)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);
            
        // Validate cron expression
        if (!CronExpression.IsValidExpression(request.Expression))
            return BadRequest("Invalid cron expression");
            
        // Sanitize request payload
        var sanitizedRequest = SanitizeJsonPayload(request.Request);
        
        // Create job
        var result = await tickerManager.AddAsync(new TimeTicker { ... });
        return Ok(result);
    }
}
```

### Authorization
```csharp
// Role-based authorization
[Authorize(Roles = "TickerQAdmin")]
public class AdminTickerController : ControllerBase
{
    // Admin-only operations
}

// Custom authorization
[TickerQAuthorize(Permission = "ManageJobs")]
public async Task<IActionResult> DeleteJob(Guid id)
{
    // Implementation
}
```

## Performance Optimization

### Caching Strategies
```csharp
// Cache function metadata
services.AddMemoryCache();
services.AddSingleton<IFunctionMetadataCache, FunctionMetadataCache>();

// Response caching
[ResponseCache(Duration = 60, VaryByQueryKeys = new[] { "page", "filter" })]
public async Task<IActionResult> GetJobs([FromQuery] JobListRequest request)
{
    // Implementation
}
```

### Pagination & Filtering
```csharp
public class JobListRequest
{
    public int Page { get; set; } = 1;
    public int PageSize { get; set; } = 25;
    public string? Filter { get; set; }
    public string? Status { get; set; }
    public DateTime? From { get; set; }
    public DateTime? To { get; set; }
    public string? SortBy { get; set; }
    public string? SortDirection { get; set; } = "desc";
}
```

## Testing Dashboard Components

### API Testing
```csharp
[Test]
public async Task CreateTimeTicker_ValidRequest_ReturnsCreated()
{
    // Arrange
    var request = new AddTimeTickerRequest
    {
        Function = "TestJob",
        ExecutionTime = DateTime.UtcNow.AddMinutes(5)
    };
    
    // Act
    var response = await client.PostAsJsonAsync("/api/tickers/time", request);
    
    // Assert
    Assert.AreEqual(HttpStatusCode.Created, response.StatusCode);
}
```

### Frontend Testing
```javascript
// Jest/Testing Library tests
test('creates new time job when form is submitted', async () => {
    render(<CreateJobForm />);
    
    fireEvent.change(screen.getByLabelText('Function'), { 
        target: { value: 'TestJob' } 
    });
    fireEvent.click(screen.getByText('Create Job'));
    
    await waitFor(() => {
        expect(mockCreateJob).toHaveBeenCalledWith({
            function: 'TestJob',
            // ...
        });
    });
});
```

## Monitoring & Analytics

### Metrics Collection
```csharp
// Custom metrics
services.AddSingleton<IMetricsCollector, DashboardMetricsCollector>();

// Built-in metrics
public class JobMetrics
{
    public int TotalJobs { get; set; }
    public int PendingJobs { get; set; }
    public int RunningJobs { get; set; }
    public int CompletedJobs { get; set; }
    public int FailedJobs { get; set; }
    public double AverageExecutionTime { get; set; }
}
```

### Health Checks
```csharp
services.AddHealthChecks()
    .AddCheck<TickerQHealthCheck>("tickerq")
    .AddDbContextCheck<TickerDbContext>();
```

When developing dashboard features, focus on user experience, real-time responsiveness, and secure access to job management capabilities.