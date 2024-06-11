
--------------------------------------------------------------------------------------------------

terminal 

Microsoft.EntityFrameworkCore.Design
Microsoft.EntityFrameworkCore.SqlServer 
dotnet tool install --global dotnet-ef
dotnet ef migrations add Init 
dotnet ef database update

---------------------------------------------------------------------------------------------------

Program.cs

using Microsoft.EntityFrameworkCore;
using WebApplication1.Data;
using WebApplication1.Services;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.AddDbContext<ApplicationContext>(
    options => options.UseSqlServer("Name=ConnectionStrings:Default"));
builder.Services.AddControllers();
builder.Services.AddScoped<AccountService>();
builder.Services.AddScoped<ProductService>();


var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.MapControllers();

app.Run();


-------------------------------------------------------------------------------------------------------------------

connection string: Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=apbd;Integrated Security=True;Connect Timeout=30;Encrypt=False;Trust Server Certificate=False;Application Intent=ReadWrite;Multi Subnet Failover=False

{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "Default": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=apbd;Integrated Security=True;Connect Timeout=30;Encrypt=False;Trust Server Certificate=False;Application Intent=ReadWrite;Multi Subnet Failover=False"
  }
}

---------------------------------------------------------------------------------------------------------------------------

Services

private readonly ApplicationContext _context;

    public AccountService(ApplicationContext context)
    {
        _context = context;
    }

-----------------------------------------------------------------------------------------------------------------------

Application context

public class ApplicationContext : DbContext
{
    protected ApplicationContext()
    {
    }

    public ApplicationContext(DbContextOptions options) : base(options)
    {
    }
	public DbSet<Accounts> AccountsEnumerable { get; set; }
    public DbSet<Categories> CategoriesEnumerable  { get; set; }
    public DbSet<Products> ProductsEnumerable { get; set; }
	...
	protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Seeding Roles
        modelBuilder.Entity<Roles>().HasData(
            new Roles { PK_role = 1, Name = "Admin" },
            new Roles { PK_role = 2, Name = "User" }
        );

        // Seeding Accounts
        modelBuilder.Entity<Accounts>().HasData(
            new Accounts { PK_account = 1, FK_Role = 1, First_Name = "John", Last_Name = "Doe", Email = "john.doe@example.com", Phone = "123456789" },
            new Accounts { PK_account = 2, FK_Role = 2, First_Name = "Jane", Last_Name = "Smith", Email = "jane.smith@example.com", Phone = "987654321" }
        );
		...
		
---------------------------------------------------------------------------------------------------------------------

Controller

public class AccountController : ControllerBase
{
    private readonly AccountService _accountService;

    public AccountController(AccountService accountService)
    {
        _accountService = accountService;
    }

------------------------------------------------------------------------------------------------------------

Table model

	public int FK_Product { get; set; }
    
    [ForeignKey(nameof(FK_Product))]
    public Products Products { get; set; }= null!; -> klasa z ktora powiazane
	
	[PrimaryKey(nameof(FK_Product),nameof(FK_Category))] -> jesli wiecej niz 1 pk
