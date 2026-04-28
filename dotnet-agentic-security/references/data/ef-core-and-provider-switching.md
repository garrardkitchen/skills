# EF Core, Code First, and Provider Switching

Use this file when the user asks about EF Core, database choice, or query performance.

## Create a new EF Core project

For a clean split, use a host project plus a data project:

```bash
dotnet new sln -n SecureApp
dotnet new webapi -n SecureApp.Api
dotnet new classlib -n SecureApp.Data

dotnet sln add SecureApp.Api/SecureApp.Api.csproj
dotnet sln add SecureApp.Data/SecureApp.Data.csproj
dotnet add SecureApp.Api/SecureApp.Api.csproj reference SecureApp.Data/SecureApp.Data.csproj
```

Install EF Core packages:

```bash
dotnet add SecureApp.Data/SecureApp.Data.csproj package Microsoft.EntityFrameworkCore
dotnet add SecureApp.Data/SecureApp.Data.csproj package Microsoft.EntityFrameworkCore.Design
dotnet add SecureApp.Data/SecureApp.Data.csproj package Microsoft.EntityFrameworkCore.Sqlite
dotnet add SecureApp.Data/SecureApp.Data.csproj package Microsoft.EntityFrameworkCore.SqlServer
dotnet add SecureApp.Data/SecureApp.Data.csproj package Microsoft.EntityFrameworkCore.Cosmos
dotnet add SecureApp.Api/SecureApp.Api.csproj package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-ef
```

If `dotnet-ef` is already installed, update it instead:

```bash
dotnet tool update --global dotnet-ef
```

## Minimal model and DbContext

```csharp
public sealed class Product
{
    public Guid Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public bool IsActive { get; set; } = true;
}

public sealed class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public DbSet<Product> Products => Set<Product>();
}
```

## Provider switching

```csharp
string provider = builder.Configuration["Database:Provider"] ?? "Sqlite";

builder.Services.AddDbContext<AppDbContext>(options =>
{
    string connectionString = builder.Configuration.GetConnectionString("AppDb")!;

    switch (provider)
    {
        case "SqlServer":
            options.UseSqlServer(connectionString);
            break;
        case "Cosmos":
            options.UseCosmos(connectionString, databaseName: "appdb");
            break;
        default:
            options.UseSqlite(connectionString);
            break;
    }
});
```

## Code first

For relational providers such as SQLite and Azure SQL, create the first migration from the startup project while storing migrations in the data project:

```bash
dotnet ef migrations add InitialCreate \
  --project SecureApp.Data/SecureApp.Data.csproj \
  --startup-project SecureApp.Api/SecureApp.Api.csproj \
  --output-dir Migrations
```

Apply the migration:

```bash
dotnet ef database update \
  --project SecureApp.Data/SecureApp.Data.csproj \
  --startup-project SecureApp.Api/SecureApp.Api.csproj
```

Generate SQL for review in controlled environments when the provider supports idempotent scripts:

```bash
dotnet ef migrations script \
  --project SecureApp.Data/SecureApp.Data.csproj \
  --startup-project SecureApp.Api/SecureApp.Api.csproj \
  --idempotent
```

- keep migrations in source control
- do not auto-apply destructive migrations blindly in production
- validate provider differences before assuming one model fits all
- install `Microsoft.EntityFrameworkCore.Design` in the startup project used by `dotnet ef`
- treat `--idempotent` as a relational workflow only; Cosmos does not use migrations, and SQLite is usually better served by `database update` or a known-from/to script

## Cosmos provider note

`Azure Cosmos DB` does not follow the relational migrations workflow above.

Use the Cosmos provider for document scenarios, and initialize containers with `EnsureCreatedAsync()` rather than `MigrateAsync()` during controlled bootstrap or deployment flows:

```csharp
if (db.Database.IsCosmos())
{
    await db.Database.EnsureCreatedAsync(cancellationToken);
}
else
{
    await db.Database.MigrateAsync(cancellationToken);
}
```

- prefer client-generated `Guid` or other explicitly assigned keys for Cosmos-backed entities
- use relational migrations for SQLite and Azure SQL
- use `EnsureCreatedAsync()` only for bootstrap with appropriate permissions, not as a routine request-path or every-instance startup step
- for RBAC/passwordless Cosmos environments, provision databases and containers ahead of time with IaC or management-plane automation
- validate partition keys, consistency, and query costs separately from relational behavior

## Seed data

Model-based seeding is useful for stable reference data:

```csharp
public sealed class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public DbSet<Product> Products => Set<Product>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>().HasData(
            new Product { Id = new Guid("11111111-1111-1111-1111-111111111111"), Name = "Starter Subscription", Price = 9.99m, IsActive = true },
            new Product { Id = new Guid("22222222-2222-2222-2222-222222222222"), Name = "Pro Subscription", Price = 29.99m, IsActive = true });
    }
}
```

If you need custom seed logic beyond migration-managed reference data, run it in a controlled initialization step rather than on every application instance during normal startup:

```csharp
public static class DbSeeder
{
    public static async Task SeedAsync(AppDbContext db, CancellationToken cancellationToken)
    {
        string[] existingNames = await db.Products
            .AsNoTracking()
            .Select(x => x.Name)
            .ToArrayAsync(cancellationToken);

        Product[] candidates =
        [
            new Product { Id = new Guid("33333333-3333-3333-3333-333333333333"), Name = "Contoso Basic", Price = 19m, IsActive = true },
            new Product { Id = new Guid("44444444-4444-4444-4444-444444444444"), Name = "Contoso Premium", Price = 49m, IsActive = true }
        ];

        Product[] missing = candidates
            .Where(candidate => !existingNames.Contains(candidate.Name, StringComparer.OrdinalIgnoreCase))
            .ToArray();

        if (missing.Length == 0)
            return;

        db.Products.AddRange(missing);
        await db.SaveChangesAsync(cancellationToken);
    }
}
```

```csharp
public sealed class DbInitializer(AppDbContext db)
{
    public async Task InitializeAsync(CancellationToken cancellationToken)
    {
        if (db.Database.IsCosmos())
            await db.Database.EnsureCreatedAsync(cancellationToken);
        else
            await db.Database.MigrateAsync(cancellationToken);

        await DbSeeder.SeedAsync(db, cancellationToken);
    }
}
```

- prefer deterministic seed data for tests and demos
- avoid mixing large business imports into app startup
- keep production bootstrap data small, explicit, and auditable
- run custom seeders in a deployment step, admin command, or one-off initialization job
- back custom seed data with a unique constraint, seed-history table, or external lock so repeated runs stay safe

## Avoid `N+1`

Prefer projection over over-eager includes when returning API data.

```csharp
var orders = await db.Orders
    .AsNoTracking()
    .Select(o => new OrderSummaryDto(
        o.Id,
        o.Customer.Name,
        o.Lines.Count,
        o.Lines.Sum(x => x.Quantity * x.UnitPrice)))
    .ToListAsync(cancellationToken);
```

## Guidance

- use `AsNoTracking()` for read-only queries
- project to DTOs for APIs
- use `Include` deliberately, not reflexively
- measure before and after changing query shape
- keep provider-specific features isolated behind clear seams
- validate SQLite, Azure SQL, and Cosmos behaviors separately before claiming portability
- enforce tenant and ownership predicates in queries that serve user-controlled resource IDs
- handle `DbUpdateConcurrencyException` for critical updates
- avoid returning EF entities directly from APIs; use DTOs to prevent over-posting and accidental data exposure
- do not run automatic destructive migrations on normal production startup

## Sources

- Microsoft Learn EF Core migrations, querying, performance, concurrency, and provider documentation
- OWASP API Security Top 10 2023 API1 and API3
