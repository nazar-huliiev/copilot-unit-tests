---
description: This file provides guidelines for writing clean, maintainable, and idiomatic C# code with a focus on functional patterns and proper abstraction.
applyTo: '**/*.cs'
---

**Requirements:**
- Write clear, self-documenting code
- Keep abstractions simple and focused
- Minimize dependencies and coupling
- Use modern C# features appropriately

## General Instructions
### Formatting
- Use 4 spaces for indentation; avoid tabs.
- Prefer file-scoped namespace declarations and single-line using directives.
- Insert a newline before the opening curly brace of any code block (e.g., after `if`, `for`, `while`, `foreach`, `using`, `try`, etc.).
- Ensure that the final return statement of a method is on its own line.
- Use pattern matching and switch expressions wherever possible.
- Use `nameof` instead of string literals when referring to member names.
- To ensure a clear separation between different parts of a class or object, use exactly one empty line to separate methods, properties, constructors, or other class-level members.

### Code Coverage Analysis
- Always use coverlet.collector when analyzing code coverage in unit tests.
- Run coverage with:
```console
dotnet test --collect:"XPlat Code Coverage"
```
- Parse the generated coverage.cobertura.xml or coverage.json file.
- Identify methods with less than 70% coverage (use method coverage as the primary metric).
- Provide a simple report that lists:
  - Class name
  - Method name
  - Coverage percentage
- Suggest writing additional unit tests to improve coverage for methods below the threshold.
- Do not convert test results to other formats (e.g., don’t force LCOV, HTML).
- When suggesting unit tests, focus on untested or partially tested methods and edge cases.

## Naming
### Avoid using not descriptive and misleading names and constants
It is important to ensure that the once-written source code can be read like a book. It should be understandable and natural. Descriptive names make the code readable and searchable. A good name reveals the intentions of the developer who wrote the code.

Bad:
```csharp
int wd;
```
Good:
```csharp
int firstWorkingDayOfAMonth;
```

Bad:
```csharp
var fetchedData = db.Retriever().ToList();
```
Good:
```csharp
var longRunningSqlQueries = _sqlMonitor.GetLongRunningQueries().ToList();
```

Bad:
```csharp
long memory = GC.GetTotalMemory(true);

if (memory > 1048576)
{
    Console.WriteLine("Alert");
}
```
Good:
```csharp
const long MaxMemoryUsageInBytes = 1024 * 1024;

long currentHeapSize = GC.GetTotalMemory(true);

if (currentHeapSize > MaxMemoryUsageInBytes)
{
    Console.WriteLine("Alert");
}
```
### Use meaningful and pronounceable variable names

Unpronounceable names make team discussions about the code, as well as performance reviews, difficult. Make sure your code is prepared for that.

Bad:
```csharp
public class Invoice
{
    public Datetime IssDa { get; set; }
    public Datetime LModUId { get; set; }  
}
```
Good:
```csharp
public class Invoice
{
    public Datetime IssueDate { get; set; }
    public Datetime LastModificationUserId { get; set; }
}
```

### Use explanatory variables
When performing advanced operations it’s a good idea to break the code into multiple steps. The result of each step may be held in the variable with a meaningful name. That action will provide clarity for your code.

Bad:
```csharp
const string Address = "301 Pine St, Philadelphia, PA 19106";

string addressRegex = @"^(?<BuildingNumber>\d+)\s(?<StreetName>[\w\s]+),\s(?<City>[\w\s]+),\s(?<State>[A-Z]{2})\s(?<ZipCode>\d{5}(?:-\d{4})?)$";

var match = Regex.Match(Address, addressRegex);

if (match.Success)
{
    Console.WriteLine($"{match.Groups[3].Value}, ({match.Groups[5].Value}, {match.Groups[4].Value}), {match.Groups[2].Value}, {match.Groups[1].Value}");
}
```
Good:

Decrease dependence on regex by naming sub-patterns.
```csharp
const string Address = "301 Pine St, Philadelphia, PA 19106";

string addressRegex = @"^(?<BuildingNumber>\d+)\s(?<StreetName>[\w\s]+),\s(?<City>[\w\s]+),\s(?<State>[A-Z]{2})\s(?<ZipCode>\d{5}(?:-\d{4})?)$";

var match = Regex.Match(Address, addressRegex);

if (match.Success)
{
    var place = match.Groups["City"]?.Value;
    var zipCode = match.Groups["ZipCode"]?.Value;
    var street = match.Groups["StreetName"]?.Value;
    var state = match.Groups["State"]?.Value;
    var buildingNumber = match.Groups["BuildingNumber"]?.Value;

    Console.WriteLine($"{place}, ({zipCode}, {state}), {street}, {buildingNumber}");
}
```
### Naming constant values

Use Pascal casing for naming constants when creating new solutions. However, make sure you use the same capitalization rules across your codebase. Just be consistent in case the team has chosen a different approach.

Bad:
```csharp
const int TIMEOUT_IN_SECONDS = 7;
```
Good:
```csharp
const int TimeoutInSeconds = 7;
```

### Avoid Hungarian Notation

Despite Hungarian Notation being widely adopted in the past, modern development tools offer no additional benefits from it. Therefore, it should be avoided in variable and function parameter naming as well.

Bad:
```csharp
int iNumberOfPages;
string strMessage;
DateTime dLastModificationDate;
```
Good:
```csharp
int numberOfPages;
string message;
DateTime lastModificationDate;
```

Please note that in some cases Hungarian notation is advisable. That includes naming interfaces (such as IQueryable, IEnumerable, etc.), which helps distinguish interfaces from abstract classes.

### Use camelCase notation for variables
Using camelCase notation for variables and method parameters is advised.
Bad:
```csharp
string invoicenumber;
string invoice_city;
```
Good:
```csharp
string invoiceNumber;
string invoiceCity;
```

Bad:
```csharp
public double IssueInvoice(int issuerid)
{
}
```
Good:
```csharp
public double IssueInvoice(int issuerId)
{
}
```
### Naming class members

Don’t repeat the name of the class by prefixing or postfixing its members. It makes code harder to read.

Bad:
```csharp
public class Invoice
{
    public DateTime InvoiceIssueDate { get; set; }
    public void InvoiceDelete();
}
```
Good:
```csharp
public class Invoice
{
    public DateTime IssueDate { get; set; }
    public void Delete();
}
```
### Avoid not descriptive aliasing

Consider avoiding using single-letter aliases as they can increase the effort needed to understand the code.

Bad:

```csharp
Person p = _dbContext.People.FirstOrDefault();
p.Obfuscate();
_accountManager.Deactivate(p.PersonId);
```
Good:
```csharp
Person personToDelete = _dbContext.People.FirstOrDefault();
personToDelete.Obfuscate();
_accountManager.Deactivate(personToDelete.PersonId);
```
## General coding patterns
### Avoid magic values

Avoid putting strings or numbers directly into your code if they impact application behavior. Such values usually need to be replicated throughout which can lead to bugs. Storing them in a centralized location like config, enum or constant will improve code maintainability.

Bad:
```csharp
if (applicationRole == "admin")
{
   …
}
```
Good:
```csharp
public enum ApplicationRole
{

    Admin,
    ReadOnly,
    Write

}

if (role == ApplicationRole.Admin)
{
    ...
}
```
Bad:
```csharp
if (errorCode == 56)
{
    ...
}
```
Good:
```csharp
const int ErrorCodeNoAccess = 56;

if (errorCode == ErrorCodeNoAccess)
{
    ...
}
```
### Avoid negative conditionals
Bad:
```csharp
if (!(quantityRemaining <= 0 || quantityOrdered <= 0))
{
    ...
}
```

Good:
```csharp
if (quantityRemaining > 0 && quantityOrdered > 0)
{
    ...
}
```
### Avoid conditionals
Although If-else statements and switch cases can be helpful in many situations, they may lead to long-term issues such as increased code complexity and reduced readability. What is more code with complicated switch statements is hard to maintain and extend without violating the Open\Closed SOLID principle. Additionally, numerous if statements may indicate a violation of the Single Responsibility rule. Conditionals can be avoided by using polymorphism or the strategy pattern.
Bad:
```csharp
enum ConfigType { File, Database, KeyVault }
```

```csharp
public string GetApplicationConfig()
{
    switch (_configType)
    {
        case ConfigType.File:
            return GetConfigFromFile();
        case ConfigType.Database:
            return GetConfigFromDb();
        case ConfigType.KeyVault:
            return GetConfigFromKeyVault();
        default:
            return string.Empty;
    }
}
```

Good:
```csharp
interface IConfig
{
    string Get();
}

class FileConfig: IConfig
{
    public string Get()
    {
        //get config from file
    }
}

class DatabaseConfig : IConfig
{
    public string Get()
    {
        //get config from DB
    }
}

class KeyVaultConfig : IConfig
{
    public string Get()
    {
        //get config from KeyVault
    }
}

public string GetApplicationConfig()
{
    return _config.Get(); //where _config is of IConfig type
}
```
### Avoid type-checking
Bad:
```csharp
public void DeleteUserData(object userDataStorage)
{
    if (userDataStorage.GetType() == typeof(FileSystem))
    {
        (userDataStorage as FileSystem).RemoveFile();
    }
    else if (userDataStorage.GetType() == typeof(SqlDatabase))
    {
        (userDataStorage as SqlDatabase).DeleteRows();
    }
}
```

Good:
```csharp
public void DeleteUserData(UserStorage userDataStorage)
{
userDataStorage.Delete();
}
```
### Remove unused code
Unused, old code is cluttering the project repository. Don’t hesitate to remove it. Since you have source control in place, you can recover it anytime.
Bad:
```csharp
public void ProcessDataDeprecated()
{
}

public void ProcessDataV1()
{
}

public void ProcessDataV2()
{
}

public void ProcessDataNew()
{
}
```
Good:
```csharp
public void ProcessData()
{
}
```
### Use method chaining
This pattern is widely adopted in libraries. It makes the code concise.
Good:
```csharp
public class Logger
{
    private List<string> _logs = new List<string>();

    public Logger Add(string message)
    {
        _logs.Add(message);
        return this;
    }

    public Logger Clear()
    {
        _logs.Clear();
        return this;
    }

    public Logger Save()
    {
        //saves to persistent storage
        return this;
    }

    public Logger AddTag(string tag)
    {
        for (int i = 0; i < _logs.Count; i++)
        {
            _logs[i] = $"{_logs[i]}#{tag}";
        }

        return this;
    }
}




Logger l = new Logger();

l.Add("Hello")
    .Add("Greetings from the project")
    .AddTag("debug")
    .AddTag($"correlationId:{Guid.NewGuid()}")
    .Save()
    .Clear();
```
### Don’t repeat yourself (DRY)
Preserving the DRY principle reduces code duplication by ensuring that logic is implemented once and reused consistently. This usually comes down to introducing abstraction layers that enhance maintainability, readability, and testability across the codebase, while facilitating easier refactoring.
Bad:
```csharp
public class SqlReport
{
    public string LinkToTheReport { get; }
    public string Author { get; }

    public void Generate()
    {
        //generate SQL commands which produce the report
    }

    public void Get()
    {
        DownloadReport();
        SetWaterMark();
        SendToScreen();
    }
}

public class GraphicalReport
{
    public string LinkToTheReport { get; }
    public string Author { get; }

    public void Generate()
    {
        //generate a visual report
    }

    public void Get()
    {
        DownloadReport();
        SetWaterMark();
        SendToScreen();
    }
}
```

Good:
```csharp
public abstract class Report
{
    public string LinkToTheReport { get; }
    public string Author { get; }

    public abstract void Generate();

    public virtual void Get()
    {
        DownloadReport();
        SetWaterMark();
        SendToScreen();
    }
}

public class SqlReport: Report
{
    public override void Generate()
    {
        //generate SQL commands which produce the report
    }
}

public class GraphicalReport: Report
{
    public override void Generate()
    {
        //generate visual report
    }
}
```

## Comments
### Avoid comments

Comment on the code only if that’s absolutely necessary. Comments should not be an excuse for poor-quality code. Aim to write self-explaining code and comment only in exceptional cases when your intentions cannot be conveyed in any other way.

Bad:
```csharp
public class Shop
{
    private List<Item> commodity = new List<Item>();

    public bool OrderCommodity(Item itemToOrder, int quantity)
    {
        //check if we have enough items in stock
        if (itemToOrder.QuntityInStock - itemToOrder.QuntityReserved - itemToOrder.QuntityDeffective >= quantity)
        {
            //don't buy, make a reservation first
            itemToOrder.QuntityReserved += quantity;
        }
        else
        {
            return false;
        }

        ProcessPayment();

        //buy
        itemToOrder.QuntityInStock -= quantity;
        //descrease reservation quantity
        itemToOrder.QuntityReserved -= quantity;

        return true;
    }

    private void ProcessPayment()
    {
        //
    }
}
```

Good:
```csharp
public class Shop
{
    private List<Item> commodity = new List<Item>();

    public bool OrderCommodity(Item itemToOrder, int quantity)
    {
        if (GetAvailableQuantity(itemToOrder) >= quantity)
        {
            Reserve(itemToOrder, quantity);
        }
        else
        {
            return false;
        }

        ProcessPayment();

        Buy(itemToOrder, quantity);
        CancelReservation(itemToOrder, quantity);

        return true;
    }

    private int GetAvailableQuantity(Item itemToCheck)
    {
        return itemToCheck.QuntityInStock - itemToCheck.QuntityReserved - itemToCheck.QuntityDeffective;
    }

    private void Reserve(Item itemToReserve, int quantity)
    {
        itemToReserve.QuntityReserved += quantity;
    }

    private void CancelReservation(Item itemReserved, int quantity)
    {
        itemReserved.QuntityReserved -= quantity;
    }

    private void Buy(Item itemToBuy, int quantity)
    {
        itemToBuy.QuntityInStock -= quantity;
    }

    private void ProcessPayment()
    {
        //
    }
}
```

### Don’t keep changelog comments
Since version control systems are widely adopted nowadays, there’s no need to create journal comments
Bad:
```csharp
/**
* 2024-06-15: Refactored database connection logic
* 2023-11-04: Optimized rendering performance
* 2022-08-27: Updated authentication module
* 2021-04-20 Deprecated old API endpoints
* 2019-01-30: Implemented real-time notifications
*/

static void Main(string[] args)
{
}
```
Good:
```csharp
static void Main(string[] args)
{
}
```
### Delete commented out code
Instead of commenting out unnecessary code, delete it. You can always use source control to revert it.
Bad:
```csharp
Process();
// CheckStock();
// Log();
// ReleaseResources();
```
Good:
```csharp
Process();
```
### Avoid positional markers
A positional marker is a text block used to organize source code visually. However, adhering to clean code principles leads to self-descriptive code, eliminating the need for positional markers usage.
Bad:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    ////////////////////////////////////////////////////////////////////////////////
    // Registering dependencies
    services.AddScoped<IMyService, MyService>();
    services.AddSingleton<IOtherService, OtherService>();
    ////////////////////////////////////////////////////////////////////////////////
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ////////////////////////////////////////////////////////////////////////////////
    // Middlewares
    app.UseRouting();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
    ////////////////////////////////////////////////////////////////////////////////
}
```

Good:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddScoped<IMyService, MyService>();
    services.AddSingleton<IOtherService, OtherService>();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseRouting();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```
## Functions
### Side effects
A function is considered to have a side-effect if it modifies the state out of its local scope, such as changing the value of non-local variables, writing to the database, amending files, etc. Avoid distributing side effects among multiple functions. Ideally, centralize the modification of external objects in one place. Centralizing side effects helps with application maintenance by reducing bugs, and increasing the testability and predictability of your code.

Bad:
```csharp
public class ShoppingCart
{
    private const string Currency = "USD";
    private string _item;

    public string GetItemWithCurrency()
    {
        _item = $"{Currency} {_item}";  //side effect
        return _item;
    }
}
```
Good:
```csharp
public class ShoppingCart
{
    private const string Currency = "USD";
    private string _item;

    public string GetItemWithCurrency()
    {
        string itemWithCurrency = $"{Currency} {_item}";
        return itemWithCurrency;
    }
}
```
### Avoid flags parameters
Using a flag in the function argument list may indicate a code smell suggesting a violation of the Single Responsibility rule. Consider separating the code paths for each scenario to eliminate unnecessary conditionals. Use interfaces where appropriate.
Bad:
```csharp
class SeatReservation
{
    public string SeatNumber { get; set; }
    public DateTime Start { get; set; }
    public DateTime End { get; set; }
}

const int StandardReservationDurationInMins= 30;
const int VIPReservationDurationInMins = 60;
List<SeatReservation> _reservedSeats = new List<SeatReservation>();

public void ReserveSeat(string seatNumber, bool isVIP)
{
    int reservationDuration = isVIP ? StandardReservationDurationInMins: VIPReservationDurationInMins;

    _reservedSeats.Add(new SeatReservation {
        SeatNumber = seatNumber,
        Start = DateTime.UtcNow,
        End = DateTime.UtcNow.AddMinutes(reservationDuration) });
}
```

Good:
```csharp
class SeatReservation
{
    public string SeatNumber { get; set; }
    public DateTime Start { get; set; }
    public DateTime End { get; set; }
}

const int StandardReservationDurationInMins = 30;
const int VIPReservationDurationInMins = 60;
static List<SeatReservation> _reservedSeats = new List<SeatReservation>();

interface IReservationService
{
    void Reserve(string seatNumber);
}

class ReservationService : IReservationService
{
    public void Reserve(string seatNumber)
    {
        _reservedSeats.Add(new SeatReservation
        {
            SeatNumber = seatNumber,
            Start = DateTime.UtcNow,
            End = DateTime.UtcNow.AddMinutes(StandardReservationDurationInMins)
        });
    }
}

class VIPReservationService : IReservationService
{
    public void Reserve(string seatNumber)
    {
        _reservedSeats.Add(new SeatReservation
        {
            SeatNumber = seatNumber,
            Start = DateTime.UtcNow,
            End = DateTime.UtcNow.AddMinutes(VIPReservationDurationInMins)
        });
    }
}
```
### Function arguments

Keeping the number of function parameters low is beneficial for readability and maintainability. It simplifies the testing process by reducing the complexity. Less complex code also means fewer chances of introducing bugs. Consider encapsulating arguments in a higher-level structure if they are logically related.

Bad:
```csharp
public void SetupDbConnection(string serverAddress, string defaultDbName, int port, int timeoutInSeconds)
{
    //...
}
```
Good:
```csharp
class ConnectionConfig
{
    public string ServerAddress { get; set; }
    public string DefaultDbName { get; set; }
    public int Port { get; set; }
    public int TimeoutInSeconds { get; set; }
}

public void SetupDbConnection(ConnectionConfig conConfig)
{
    //...
}
```
### Functions should have one purpose
Functions that focus on a single task are easier to understand, making the code more intuitive. Such functions are simpler to maintain and more reusable.

Bad:
```csharp
public void CreateOrder(Item commodityName, int quantity)
{
    if (item.QuantityAvailable - item.QuantityDeffective >= quantity)
    {
        Buy(commodityName, quantity);
    }
}
```

Good:
```csharp
public void CreateOrder(Item orderItem, int quantity)
{
    if (IsInStock(orderItem, quantity))
    {
        Buy(commodityName, quantity);
    }
}

public bool IsInStock(Item stockItem, int quantity)
{
    if (stockItem.QuantityAvailable - stockItem.QuantityDeffective >= quantity)
    {
        return true;
    }
}
```

### Function names should clearly explain their purpose
Bad:
```csharp
public class Order
{
    //...

    public void Process()
    {
        GenerateDbOrder();
        MakeReservation();

        if (PaymentSuccessful(_orderNumber))
        {
            GenerateInvoice();
        }
    }
}

Order newOrder = new Order("Green watercan", 3);
newOrder.Process();
```

Good:
```csharp
public class Order
{
    //...
    public void Buy()
    {
        GenerateDbOrder();
        MakeReservation();

        if (PaymentSuccessful(_orderNumber))
        {
            GenerateInvoice();
        }
    }
}

Order newOrder = new Order("Green watercan", 3);
newOrder.Buy();
```

### Function callers and callees proximity
The function invocation (caller) should be located close to the code that defines the function (callee) if they are both defined in the same source file.

Visually placing function callers before callees makes the code easier to read and understand, as related pieces of code are near each other, reducing the need to jump around the file to see how different parts interact.

Ideally, the code should be read from top to bottom, like a book.
Bad:
```csharp
public class Order
{
    private void LockCommodity()
    {
        //...
    }

    private void MakeReservation();
    {
        LockCommodity();
        _orderDb.DecreaseStockLevel();
    }

    private void GenerateInvoice()
    {
        //...
    }

    public void Buy()
    {
        MakeReservation();
        GenerateInvoice();
    }
}
```
Good:
```csharp
public class Order
{
    public void Buy()
    {
        MakeReservation();
        GenerateInvoice();
    }

    private void MakeReservation();
    {
        LockCommodity();
        _orderDb.DecreaseStockLevel();
    }

    private void LockCommodity()
    {
        //...
    }

    private void GenerateInvoice()
    {
        //...
    }
}
```
### Extract conditionals into separate functions
Identify complex conditionals and move them into separate functions. By naming these functions clearly, you reveal intentions, which reduces the time needed to understand your code.

Bad:
```csharp
class Shop
{
    public void PlaceOrder(string commodityName, int quantity)
    {
            if (!string.IsNullOrEmpty(commodityName)
            && !commodityName.Any(char.IsDigit)
            && !commodityName.Any(char.IsUpper)
            && _stockLevelQuantity - _deffectiveQuantity - _reservedQuantity >= quantity
            && _dbStockRetriever.FindItem(commodityName).IsActive)
        {
            Buy(commodityName, quantity);
        }
    }
}
```
Good:
```csharp
class Shop
{
    public void PlaceOrder(string commodityName, int quantity)
    {
        if (IsCommoditynameValid(commodityName)
            && IsAvailable(commodityName, quantity)
            && IsActive(commodityName))
        {
            Buy(commodityName, quantity);
        }
    }

    private bool IsCommodityNameValid(string commodityName)
    {
        return !string.IsNullOrEmpty(commodityName)
        && !commodityName.Any(char.IsDigit)
        && !commodityName.Any(char.IsUpper);
    }

    private bool IsAvailable(int quantity)
    {
        return _stockLevelQuantity - _deffectiveQuantity - _reservedQuantity >= quantity;
    }

    private bool IsActive(string commodityName)
    {
        return _dbStockRetriever.FindItem(commodityName).IsActive;
    }
}
```
### Use default arguments
Prefer default arguments over placing default values inside the function body. This will improve the readability of your code.
Bad:
```csharp
private const int DefaultTimeoutInSeconds = 30;

public void GetUsers(int? timeoutInSeconds)
{
    if (!timeoutInSeconds.HasValue)
    {
        timeoutInSeconds = DefaultTimeoutInSeconds;
    }

    //...
}
```
Good:
```csharp
private const int DefaultTimeoutInSeconds = 30;

public void GetUsers(int? timeoutInSeconds = DefaultTimeoutInSeconds)
{
    //...
}
```
### Return early
Too many nested conditional statements increase cyclomatic complexity. Returning early from the function helps with quicker understanding of the code.
Bad:
```csharp
public bool IsAdmin(string userName)
{
    var user = _users.First(u => u.UserName == userName);

    if (user.IsActive)
    {
        Console.WriteLine("User is active");

        if (!IsUserBlacklisted(user))
        {
            Console.WriteLine("User is not blacklisted");

            var permissions = user.GetPermissions();

            if (permissions.CanRead)
            {
                return IsUserAdmin(user);
            }
            else
            {
                return false;
            }
        }
        else
        {
            return false;
        }
    }
    else
    {
        return false;
    }
}
```
Good:
```csharp
public bool IsAdmin(string userName)
{
    var user = _users.First(u => u.UserName == userName);

    if (!user.IsActive)
    {
        return false;
    }

    Console.WriteLine("User is active");

    if (IsUserBlacklisted(user))
    {
        return false;
    }

    Console.WriteLine("User is not blacklisted");

    var permissions = user.GetPermissions();

    if (!permissions.CanRead)
    {
        return false;
    }

    return IsUserAdmin(user);
}
```
## Classes and code composition
### Use accessors and mutators
Use access modifiers to implement the hermitization of class members. Taking advantage of this approach helps you avoid accessing and changing them from outside of class. Proper use of access modifiers restricts the number of places where class members are used or modified, thus improving overall code reliability by preventing accidental modifications.

Bad:
```csharp
public class OrderItem
{
    public int Quantity { get; set; }
    public string Name { get; set; }
}

public class Order
{
    public List<OrderItem> Items;
}

Order newOrder = new Order();

newOrder.Items.Add(new OrderItem { Name = "Red tulip", Quantity = -10 });
newOrder.Items.Add(new OrderItem { Name = "Red tulip", Quantity = 5 });
newOrder.Items.Clear();
```

Good:
```csharp
public class OrderItem
{
    public int Quantity { get; set; }
    public string Name { get; set; }
}
public class Order
{
    private List<OrderItem> _items;

    public void AddItem(OrderItem orderItem)
    {
        if (_items.Exists(i => i.Name == orderItem.Name))
        {
            throw new Exception("Item already exists");
        }

        if (orderItem.Quantity <= 0)
        {
            throw new Exception("Quantity cannot be negative");
        }

        _items.Add(orderItem);
    }

Order newOrder = new Order();
newOrder.AddItem(new OrderItem { Name = "Red tulip", Quantity = 5 });
```

### Singleton (anti)pattern
The Singleton pattern may be considered an anti-pattern. Despite its tempting convenience and potential to solve specific technical problems, it breaks application modularity. This is because it is not possible to move a class that uses the Singleton pattern as a dependency to a different context. This limitation can be particularly problematic when isolating classes for unit testing. It is similar to the issues encountered when using global variables.

Bad:
```csharp
public class DatabaseConnection
{
    private static DatabaseConnection _instance;

    private DatabaseConnection()
    {
    }

    public static DatabaseConnection GetInstance()
    {
        if (_instance == null)
        {
            _instance = new DatabaseConnection();
        }

        return _instance;
    }

    // ...
}

public class OrderRetriever
{
    private DatabaseConnection _dbInstance;

    public OrderRetriever()
    {
        _dbInstance = DatabaseConnection.GetInstance();
    }
}
```
Good:
```csharp
public class DatabaseConnection
{
    // ...
}

//register class as singleton
services.AddSingleton<DatabaseConnection>();

public class OrderRetriever
{
    private DatabaseConnection _dbInstance;

    //inject single instance using Dependency Injection
    public OrderRetriever(DatabaseConnection dbConnection)
    {
        _dbInstance = dbConnection;
    }
}
```

### Prefer composition over inheritance
Both are valid programming techniques. However, inheritance is sometimes overused to make the code appear more reusable. Nevertheless, inherited code is much harder to maintain and amend as every change to a superclass impacts all subclasses. The rule of thumb is to use inheritance only when there is a clear “is-a” relationship between classes; in other cases, use composition.

Bad:
```csharp
public class Scenario
{
    public string Title { get; set; }
    public string Author { get; set; }
    public int NumberOfPages { get; set; }

    public Scenario(string title, string author, int numberOfPages)
    {
        Title = title;
        Author = author;
        NumberOfPages = numberOfPages;
    }
}

//movie that was made based on some scenario
//Movie-Scenario is a "has-a" relationship, inheritance was improperly used
public class Movie: Scenario
{
    private string _title;
    private string _director;

    public Movie(string title, string director, string scenarioTitle,
        string scenarioAuthor, int numberOfPages):

        base(scenarioTitle, scenarioAuthor, numberOfPages)
    {
        _title = title;
        _director = director;
    }
}
```
Good:
```csharp
public class Scenario
{
    public string Title { get; set; }
    public string Author { get; set; }
    public int NumberOfPages { get; set; }

    public Scenario(string title, string author, int numberOfPages)
    {
        Title = title;
        Author = author;
        NumberOfPages = numberOfPages;
    }
}

public class Movie
{
    private string _title;
    private string _director;

    //has-a relationship
    private Scenario _scenario;

    public Movie(string title, string director, string bookTitle, Scenario scenario)
    {
        _title = title;
        _director = director;
        _scenario = Scenario;
    }
}
```
## SOLID
The SOLID principles are a set of guidelines that help create easy-to-maintain and flexible code. They define how to handle dependencies between classes, reduce coupling, and promote reusable code.

SOLID stands for:
- S - Single Responsibility Principle (SRP)
- O - Open/Closed Principle (OCP)
- L - Liskov Substitution Principle (LSP)
- I - Interface Segregation Principle (ISP)
- D - Dependency Inversion Principle (DIP)

### Single Responsibility Principle (SRP)
Each class should have only one responsibility. Focusing on one thing at a time helps maintain readable and understandable code.

Bad:
```csharp
class Order
{
    //one responsibility - saving the order
    public void SaveOrder()
    {
        //..
    }

    //another responsibility - creating an invoice in PDF format
    public void CreatePDFInvoice()
    {
        //..
    }
}
```

Good:
```csharp
class Order
{
    public void SaveOrder()
    {
        //..
    }
}

class InvoiceGenerator()
{
    public void GeneratePDFInvoice(Order order)
    {
        //Generate invoice in PDF format
    }
}
```

### Open/Closed Principle (OCP)
Classes should be open for extension but closed for modification. This means they should allow their behavior to be extended without altering their source code, promoting flexibility and reducing the risk of introducing bugs when adding new features.

Bad:
```csharp
public enum ConversionType
{
    Html,
    Json,
    Xml
}

public class TextConverter
{
    public string Convert(string text, ConversionType format)
    {
        switch (format)
        {
            case ConversionType.Html:
                return ConvertToHtml(text);
            case ConversionType.Json:
                return ConvertToJson(text);
            case ConversionType.Xml:
                return ConvertToXml(text);
            default:
                return text;
        }
    }
}
```

Good:
```csharp
public class JsonConvrter : ITextConverter
    {
    public string Convert(string text)
    {
        //convert to JSON format
    }
}

public class HtmlConvrter: ITextConverter
    {
    public string Convert(string text)
    {
        //convert to HTML format
    }
}

public class XmlConvrter : ITextConverter
    {
    public string Convert(string text)
    {
        //convert to XML format
    }
}

public class TextConverter
{
    private readonly ITextConverter _converter;

    public TextConverter(ITextConverter converter)
    {
        _converter = converter;
    }

    public string Convert(string text)
    {
        return _converter.Convert(text);
    }
}
```
### Liskov Substitution Principle (LSP)
Derived classes must be substitutable for their base classes without altering the correctness of the program. This means that objects of a base should be replaceable with objects of a subclass without affecting the functionality. Adhering to this principle ensures that a class hierarchy remains reliable and consistent.

Bad:
```csharp
interface IMessageTransmitter
{
    bool Send(string payload);
}

class MessageTransmitter: IMessageTransmitter
{
    public bool Send(string payload)
    {
        //send message
        return true;
    }
}

class MessageTransmitterWithLimit: IMessageTransmitter
{
    private int _totalCharactersSent { get; set; }
    private readonly int _charactersSentLimit;

    public MessageTransmitterWithLimit(int charactersSentLimit)
    {
        _charactersSentLimit = charactersSentLimit;
    }

    public bool Send(string payload)
    {
        if (_charactersSentLimit < _totalCharactersSent + payload.Length)
        {
            throw new Exception("Max limit has been reached. Can not send more");
        }

        //send message

        return true;
    }
}
```
Good:
```csharp
interface IMessageTransmitter
{
    bool Send(string payload);
    bool CanSend(string payload);
}

class MessageTransmitter: IMessageTransmitter
{
    public bool Send(string payload)
    {
        //send message
        return true;
    }

    public bool CanSend(string payload)
    {
        return true;
    }
}

class MessageTransmitterWithLimit: IMessageTransmitter
{
    private int _totalCharactersSent { get; set; }
    private readonly int _charactersSentLimit;

    public MessageTransmitterWithLimit(int charactersSentLimit)
    {
        _charactersSentLimit = charactersSentLimit;
    }

    public bool CanSend(string payload)
    {
        return _charactersSentLimit >= _totalCharactersSent + payload.Length;
    }

    public bool Send(string payload)
    {
        if (!CanSend(payload))
        {
        	_logger.Log("Max limit has been reached. Can not send more");
        	return false;
  	  }

        //send message

        return true;
    }
}
```
### Interface Segregation Principle (ISP)
Clients should not be forced to depend on interfaces they do not use. This means that larger interfaces should be divided into smaller, more specific ones so that clients only need to know about the methods that are relevant to them. Keeping the proper level of interface granularity helps with decoupling and reduces the impact of changes.

Bad:
```csharp
public interface IReport
{
    void Generate();
    void ChangeFormat();
    void Edit();
    void Save();
    void Get();
}

public class Report : IReport
{
    public void Generate()
    {
        //...
    }

    public void ChangeFormat()
    {
        //...
    }

    public void Edit()
    {
        //...
    }

    public void Save()
    {
        //...
    }

    public void Get()
    {
        //…
    }
}

public class ReadOnlyReport: IReport
{
    public void Generate()
    {
        //...
    }

    public void ChangeFormat()
    {
        //...
    }

    public void Edit()
    {
        //report does not support modifications
        throw new NotImplementedException();
    }

    public void Save()
    {
        //report does not support modifications
        throw new NotImplementedException();
    }

    public void Get()
    {
        //...
    }
}
```
Good:
```csharp
public interface IReport
{
    void Generate();
    void ChangeFormat();
    void Get();
}

public interface IEditableReport
{
    public void Save();
    public void Edit();
}

public class Report : IReport, IEditableReport
{
    public void Generate()
    {
        //...
    }

    public void ChangeFormat()
    {
        //...
    }
    public void Edit()
    {
        //...
    }

    public void Save()
    {
        //...
    }
    public void Get()
    {
        //...
    }
}

public class ReadOnlyReport : IReport
{
    public void Generate()
    {
        //...
    }

    public void ChangeFormat()
    {
        //...
    }

    public void Get()
    {
        //...
    }
}
```
### Dependency Inversion Principle (DIP)
The rule consists of two parts:
1. High-level modules should not depend on low-level modules. Both should depend on abstractions.
2. Abstractions should not depend upon details. Details should depend on abstractions.

DIP promotes flexibility and maintainability in software design by separation of software modules. This allows for easier changes and extensions to the system without impacting other parts of the codebase, fostering a testable and resilient architecture.

Bad:
```csharp
//Low-level module
public class Payment
{
    public void Pay()
    {
        //...
    }
}

// High-level module
public class OrderService
{
    private Payment _payment;

    public OrderService(Payment payment)
    {
        _payment = payment;
    }

    public void Pay()
    {
        _payment.Pay();
    }
}
```

Good:
```csharp
public interface IPayment
{
    void Pay();
}

// Low-level module
public class Payment : IPayment
{
    public void Pay()
    {
        //...
    }
}

// High-level module
public class OrderService
{
    private IPayment _payment;

    public OrderService(IPayment payment)
    {
        _payment = payment;
    }

    public void Pay()
    {
        _payment.Pay();
    }
```
Bad:
```csharp
public class PaymentMethodProvider
{
    //...
}

//abstraction depends on details
public interface IPayment
{
    void Pay(PaymentMethodProvider provider);
}
```

Good:
```csharp
public interface IPaymentMethodProvider
{
    //...
}

public class PaymentMethodProvider: IPaymentMethodProvider
{
    //...
}

public interface IPayment
{
    void Pay(IPaymentMethodProvider provider);
}
```
## Testing
### Importance of testing
Testing ensures the software's reliability by identifying bugs early, reducing the cost and effort of fixing them compared to bugs found late, in the production environment. It’s an essential factor of high-quality software that increases development teams’ confidence about their work.

### A unit test should test only one thing
Ensure that your unit test is checking only one thing at a time.

Bad:
```csharp
[Fact]
public void Add_EmptyOrDuplicatedOrderItem_ReturnsFalse()
{
    Stock sockFigures = new Stock();

    bool canAddItemWith0Quantity = sockFigures.Add("Yellow hat", 0);
    Assert.False(canAddItemWith0Quantity);

    sockFigures.Add("Purple shirt", 1);
    bool canAddDuplicatedItem = sockFigures.Add("Purple shirt", 2);
    Assert.False(canAddDuplicatedItem);
}
```
Good:
```csharp
[Fact]
public void Add_OrderItemWith0Quantity_ReturnsFalse()
{
    Stock sockFigures = new Stock();

    bool canAddItemWith0Quantity = sockFigures.Add("Yellow hat", 0);
    Assert.False(canAddItemWith0Quantity);
}

[Fact]
public void Add_DuplicatedOrderItem_ReturnsFalse()
{
    Stock sockFigures = new Stock();

    sockFigures.Add("Purple shirt", 1);
    bool canAddDuplicatedItem = sockFigures.Add("Purple shirt", 2);

    Assert.False(canAddDuplicatedItem);
}
```
### AAA Pattern
Follow the Arrange, Act, Assert pattern to organize unit tests. This helps maintain a clear separation between test logic, configuration, and assertion parts.

Bad:
```csharp
[Fact]
public void Add_EmptyOrderItem_ReturnsFalse()
{
    //Arrange
    Stock sockFigures = new Stock();

    //Assert
    Assert.False(sockFigures.Add("Yellow hat", 0));
}
```

Good:
```csharp
[Fact]
public void Add_EmptyOrderItem_ReturnsFalse()
{
    //Arrange
    Stock sockFigures = new Stock();

    //Act
    bool wasItemAddedSuccessfuly = sockFigures.Add("Yellow hat", 0);

    //Assert
    Assert.False(wasItemAddedSuccessfuly);
}
```

## Error Handling
### Rethrowing errors
If you decide to re-throw an exception from within a catch block, avoid using ‘throw ex’ as it clears the original error‘s stack trace. Instead, use `throw’ to preserve it. Alternatively, you can wrap the previous exception in your custom exception. This will help you with investigations and bug detection.

Bad:
```csharp
try
{
    order.ClearItems();
}
catch (Exception ex)
{
    _logger.Log(ex);
    throw ex;
}
```
Good:
```csharp
try
{
    order.ClearItems();
}
catch (Exception ex)
{
    _logger.Log(ex);
    throw;
}
```
Good:
```csharp
try
{
    order.ClearItems();
}
catch (Exception ex)
{
    _logger.Log(ex);
    throw new OrderItemsModificationException(ex);
}
```
### Avoid silent exceptions
A silent exception, which is an exception that is caught, but not handled, nor logged, is considered a bad practice because it causes a loss of debugging information. Additionally, masking underlying exceptions introduces the risk of unpredictable application behavior.

Bad:
```csharp
try
{
    order.ClearItems();
}
catch (Exception ex)
{
    
}
```
Good:
```csharp
try
{
    order.ClearItems();
}
catch (Exception ex)
{
    _logger.Log(ex);
}
```
### Use multiple catch blocks instead of conditionals.
If you need to take an action based on the type of exception, it’s better use multiple catch blocks for exception handling instead of multiple if statements.

Bad:
```csharp
try
{
    var first = orders.First();
}
catch (Exception ex)
{
    if (ex is ArgumentNullException)
    {
        //..
    }
    else if (ex is InvalidOperationException)
    {
        //..
    }
}
```
Good:
```csharp
try
{
    var first = orders.First();
}
catch (ArgumentNullException ex)
{
    //..
}
catch (InvalidOperationException ex)
{
    //..
}
```
