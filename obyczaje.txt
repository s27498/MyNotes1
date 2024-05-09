///////////////////////////////////////////////////////////////////////////////////////////////////// appsettings .json

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

////////////////////////////////////////////////////////////////////////////////////////////////////// Program.cs

using WebApplication1.Repositories;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddControllers();
builder.Services.AddScoped<NameofRepository>();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.MapControllers();

app.Run();

////////////////////////////////////////////////////////////////////////////////////////////////////////// Models

public class Procedure
{
    public string Name { get; set; }
    public string Description { get; set; }
    public DateTime Date { get; set; }
} 
---- do insert raczej nowa klasa typu NewProcedure, pola jakie wymagane w zadaniu

////////////////////////////////////////////////////////////////////////////////////////////////////////// Repositories

public class NameofRepository
{

//// configuration ////

    private readonly IConfiguration _configuration;

    public AnimalRepository(IConfiguration configuration)
    {
        _configuration = configuration;
    }

//// doesExist template ////

    public async Task<bool> doesClassnameExist(int id) // -> przyjmuje to po czym sprawdzamy, raczej id
    {
        var query = "SELECT 1 FROM Table WHERE @id = ID";
	
	//// sql connection ////
	
        await using SqlConnection connection = new SqlConnection(_configuration.GetConnectionString("Default"));
        await using SqlCommand command = new SqlCommand();
        command.Connection = connection;
        command.CommandText = query;
        command.Parameters.AddWithValue("@id", id);
        await connection.OpenAsync();		
        var reader = await command.ExecuteScalarAsync();
		
        if (reader == null)
        {
            return false;
        }

        return true;
    }
	
//// get Classname ////     --------------------------------------------------------------------------------------
	
	public async Task<Classname> getAnimal(int id)
    {
		var query = "SELECT Tablename.id, Tablename.name FROM Table WHERE @id = ID";
		
		await using SqlConnection connection = new SqlConnection(_configuration.GetConnectionString("Default"));
        await using SqlCommand command = new SqlCommand();
        command.Connection = connection;
        command.CommandText = query;
        command.Parameters.AddWithValue("@id", id);
        await connection.OpenAsync();
		
	//// read sql response to variables ////
	
		var reader = await command.ExecuteReaderAsync();
        var ClassnameIdOrdinal = reader.GetOrdinal("id"); // -> nazwy w nawiasie takie same jak nazwy tabel
        var ClassnameNameOrdinal = reader.GetOrdinal("name");
		
	//// create object containing another object field and a field of list of objects
	
		Classname classname = null;
		while (await reader.ReadAsync())
        {
            if (classname == null)
            {
                classname = new Classname()
                {
                    ID = reader.GetInt32(classnameIdOrdinal),
                    Name = reader.GetString(classnameNameOrdinal),
                    object = new Object() // -> zakladamy ze ClassName ma pole przechowujace objekty Object
                    {
                        ID = reader.GetInt32(objectIdOrdinal),
                        FirstName = reader.GetString(objectNameOrdinal),
                        LastNAme = reader.GetString(objectOrdinal)
                    },
                    objectList = new List<SomeObjects>() // -> zakladamy ze ClassName ma pole z lista objektow SomeObjects
                    {
                        new SomeObjects()
                        {
                            Name = reader.GetString(sthOrdinal),
                            Description = reader.GetString(sthOrdinal),
                            Date = reader.GetDateTime(sthOrdinal)
                        }
                    }

                };
            }
            else
            {
                classname.objectList.Add(new SomeObjects()
                {
                    Name = reader.GetString(sthOrdinal),
                    Description = reader.GetString(sthOrdinal),
                    Date = reader.GetDateTime(sthOrdinal)
                });
            }
            
        }

        return classname;
    }//----------------------------------------------------------------------------------------------------------------
	
//// add to database //// --------------------------------------------------------------------------------------------

	public async Task AddEntityWithProcedures<TEntity>(TEntity entityWithProcedures)
{
    var insert = @"INSERT INTO EntityTable VALUES(@Name, @Type, @Date, @OwnerId);
                   SELECT @@IDENTITY AS ID;"; // -> to daje auto id
    
    await using SqlConnection connection = new SqlConnection(_configuration.GetConnectionString("Default"));
    await using SqlCommand command = new SqlCommand();
    
    command.Connection = connection;
    command.CommandText = insert;
    
    command.Parameters.AddWithValue("@Name", entityWithProcedures.Name);
    command.Parameters.AddWithValue("@Type", entityWithProcedures.Type);
    command.Parameters.AddWithValue("@Date", entityWithProcedures.Date);
    command.Parameters.AddWithValue("@OwnerId", entityWithProcedures.OwnerID);
    
    await connection.OpenAsync();

    var transaction = await connection.BeginTransactionAsync();
    command.Transaction = transaction as SqlTransaction;
    
    try
    {
        var id = await command.ExecuteScalarAsync();

        foreach (var procedure in entityWithProcedures.Procedures)
        {
            command.Parameters.Clear();
            command.CommandText = "INSERT INTO Procedure_Entity VALUES(@ProcedureId, @EntityId, @Date)";
            command.Parameters.AddWithValue("@ProcedureId", procedure.ProcedureID);
            command.Parameters.AddWithValue("@EntityId", id);
            command.Parameters.AddWithValue("@Date", procedure.Date);

            await command.ExecuteNonQueryAsync();
        }

        await transaction.CommitAsync();
    }
    catch (Exception)
    {
        await transaction.RollbackAsync();
        throw;
    }
}//---------------------------------------------------------------------------------------------------------------------------


	}

}

////////////////////////////////////////////////////////////////////////////////////////////////////////// Controllers

[ApiController]
[Route("api/[controller]")]

public class ClassnameController : ControllerBase
{
    private readonly ClassNameRepository _classnameRepository;
    public ClassnameController(ClassNameRepository classnameRepository)
    {
        _classnameRepository = classnameRepository;
    }


//// HTTPGET ////

    [HttpGet]
    [Route("{id}")]
    
    public async Task<IActionResult> GetClassName(int id)
    {
        if (!await _classnameRepository.doesClassnameExist(id))
        {
            return NotFound("Classname with given id does not exist");
        }
        var classname = await _classnameRepository.getClassname(id);
        return Ok(classname);
    }
	
//// HTTPGET ////	
	
    [HttpPost]
    public async Task<IActionResult> AddAnimal(AnimalWithProcedures animalWithProcedures)
    {
        if (!await _animalRepository.doesOwnerExist(animalWithProcedures.OwnerID))
        {
            return NotFound("Owner with given id does not exist");
        }

        foreach (var VARIABLE in animalWithProcedures.Procedures)
        {
            if (!await _animalRepository.doesProcedureExist(VARIABLE.ProcedureID))
            {
                return NotFound("Procedure with given id does not exist");
            }
        }
        await _animalRepository.addAnimalWithProcedure(animalWithProcedures);
        return Created();

    }
}

