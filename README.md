# DBLogger

v.1.0.0

Logging provider for .net that allows you to log messages to Microsoft SQL and Azure SQL databases
<br>
<br>

### Installation

DBLogger is hosted at [nuget.org](https://www.nuget.org/packages/anjosoft.dblogger/). To Install, run:
``dotnet add package anjosoft.dblogger``

---

### Usage

import the ``anjosoft.dblogger`` namespace

Configure your log levels, set your SQL connection string, and specify your log table. For example:

```{
  "Logging": {
    "Database": {
      "Options": {
          "ConnectionString": "Server=MyServer;Database=MyDatabase;User Id=user;Password=UserPassword;TrustServerCertificate=True;",
          "LogTable": "dbo.ApplicationLog"
      },
      "LogLevel": {
          "Default": "Information",
          "Microsoft": "Error"
      }
    }
  },
  "AllowedHosts": "*"
}

```

Add DBLogger and configure your logger options (Program.cs):

```
builder.Logging.AddDbLogger(options =>
{
    builder.Configuration.GetSection("Logging").GetSection("Database").GetSection("Options").Bind(options);
});
```

## Create Log Table

Your application users should only have Read, Write, and Execute privileges on SQL, therefore, DBLogger should not be allowed to create the log table for you.
Create a table with the following columns:

EventId - Integer (Application error id)  
LogLevel - String (Indicates the severity of the log)  
EventName - String (Name of the event that triggered the error)  
Message - String (Main description of the event being logged)  
ExceptionMessage - String (The description of the exception itself, generated by the .NET runtime or the code that threw the exception)  
ExceptionStackTrace - String (Stack Trace associated with the exception)  
ExceptionSource - String (Identifies the assembly or code module where the exception originated)  
DateTime - Nullable DateTime (Date and Time the message was generated)  


Example:

```
create table dbo.ApplicationLog
(
    [Id] int not null identity(1, 1),
    [EventId] int null,
    [LogLevel] varchar(16),
    [EventName] varchar(128),
    [Message] nvarchar(1024),
    [ExceptionMessage] nvarchar(1024),
    [ExceptionStackTrace] nvarchar(max),
    [ExceptionSource] nvarchar(64),
    [DateTime] DateTime2,
    index IX_AppLog nonclustered ([LogLevel], [EventName], DateTime)
)
```

### Example

```
...
using anjosoft.dblogger;

namespace myapp_api.Controllers
{
    [Route("items")]
    public class ItemsController(IItemService itemService, ILogger<DBLogger> logger) : ControllerBase
    {
        private readonly IItemService itemService = itemService;
        private readonly ILogger<DBLogger> logger = logger;

        [HttpGet("{id}")]
        public async Task<IActionResult> Get(int id)
        {
            var item = await itemService.GetItemAsync(id);

            //logger.LogInformation("test");  // example no. 1            
            //throw new Exception("test"); // example no. 2
            ...
        }
        ...
```
