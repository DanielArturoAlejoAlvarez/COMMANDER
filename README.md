# COMMANDER (REST API)

## Description

This repository is a Software of Development with C#, .NET Core, SQL Server, Entity Framework(ORM) and Newtonsoft,etc

## Installation

Using CSharp, .NET Core 3.1 and SQL Server preferably.

## DataBase

Using SQL Server preferably.

## Apps

Using Postman, Insomnia,etc

## Usage

```html
$ git clone https://github.com/DanielArturoAlejoAlvarez/COMMANDER[NAME APP]

$ dotnet restore

$ dotnet run

```

Follow the following steps and you're good to go! Important:

![alt text](https://devblogs.microsoft.com/wp-content/uploads/sites/16/2019/02/44055766-3c8a64be-9efb-11e8-9e50-529bf563137c.gif)

## Coding

### Config

```cs
...
public void ConfigureServices(IServiceCollection services)
    {
        //Configuration our DB from settings json
        services.AddDbContext<CommanderContext>(opt=>opt.UseSqlServer(Configuration.GetConnectionString("CommanderConnection")));
        
        services.AddControllers().AddNewtonsoftJson(s=>{
            s.SerializerSettings.ContractResolver = new CamelCasePropertyNamesContractResolver();
        });
        
        services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies());
        //Adding our services and scoped to mayor control manager successfully.
        //Implementation of Singleton Pattern in C#
        //Persistence Modules Repositories
        //services.AddScoped<ICommanderRepo, MockCommanderRepo>();
        services.AddScoped<ICommanderRepo,SqlCommanderRepo>();
    }
...
```

### Controllers

```cs
...
namespace Commander.Controllers
{

    [Route("api/commands")]
    [ApiController]
    public class CommandsController : ControllerBase
    {
        private readonly ICommanderRepo _repository;
        private readonly IMapper _mapper;

        //Implementation of Dependency Injection Pattern in C#
        //Using AutoMapper of DTO Data transfer objects
        public CommandsController(ICommanderRepo repository, IMapper mapper) 
        {
            _repository = repository;
            _mapper = mapper;
        }

        //private readonly MockCommanderRepo _repository = new MockCommanderRepo();
        //GET api/commands
        [HttpGet]
        public ActionResult<IEnumerable<CommandReadDto>> GetAllCommands()
        //public ActionResult<IEnumerable<Command>> GetAllCommands()
        {
            var commandsItems = _repository.GetAllCommands();
            return Ok(_mapper.Map<IEnumerable<CommandReadDto>>(commandsItems));
            //return Ok(commandsItems);
        }

        //GET api/commands/{id}
        [HttpGet("{id}", Name="GetCommandById")]        
        public ActionResult<CommandReadDto> GetCommandById(int id)
        //public ActionResult<Command> GetCommandById(int id)
        {
            var commandItem = _repository.GetCommandById(id);
            if(commandItem != null){
                return Ok(_mapper.Map<CommandReadDto>(commandItem));
                //return Ok(commandItem);
            }
            return NotFound();            
        }

        //POST api/commands
        [HttpPost] 
        public ActionResult<CommandReadDto> CreateCommand(CommandCreateDto commandCreateDto)
        {
            var commandModel = _mapper.Map<Command>(commandCreateDto);
            _repository.CreateCommand(commandModel);
            _repository.SaveChanges();
            var commandReadDto = _mapper.Map<CommandReadDto>(commandModel);
            return CreatedAtRoute(nameof(GetCommandById), new {Id = commandReadDto.Id}, commandReadDto);
            //return Ok(commandReadDto);
        }

        [HttpPut("{id}")]
        //PUT api/commands/{id}
        public ActionResult UpdateCommand(int id, CommandUpdateDto commandUpdateDto)
        {
            var commandModelFromRepo = _repository.GetCommandById(id);
            if (commandModelFromRepo == null)
            {
                return NotFound();
            }
            _mapper.Map(commandUpdateDto, commandModelFromRepo);
            _repository.UpdateCommand(commandModelFromRepo);
            _repository.SaveChanges();
            return NoContent();
        }

        [HttpPatch("{id}")]
        //PATCH api/commands/{id}
        public ActionResult PartialCommandUpdate(int id, JsonPatchDocument<CommandUpdateDto> patchDoc)
        {
            var commandModelFromRepo = _repository.GetCommandById(id);
            if (commandModelFromRepo == null)
            {
                return NotFound();
            }
            var commandToPatch = _mapper.Map<CommandUpdateDto>(commandModelFromRepo);
            patchDoc.ApplyTo(commandToPatch,ModelState);
            if(!TryValidateModel(commandToPatch))
            {
                return ValidationProblem(ModelState);
            }
            _mapper.Map(commandToPatch, commandModelFromRepo);
            _repository.UpdateCommand(commandModelFromRepo);
            _repository.SaveChanges();
            return NoContent();
        }

        [HttpDelete("{id}")]
        //DELETE api/commands/{id}
        public ActionResult DeleteCommand(int id)
        {
            var commandModelFromRepo = _repository.GetCommandById(id);
            if (commandModelFromRepo == null)
            {
                return NotFound();
            }
            _repository.DeleteCommand(commandModelFromRepo);
            _repository.SaveChanges();
            return NoContent();
        }
    }
}
...
```

### Models

```cs
...
namespace Commander.Models
{
    public class Command
    {
        [Key]
        public int Id { get; set; }
        [Required]
        [MaxLength(250)]
        public string HowTo { get; set; }
        [Required]
        public string Line { get; set; }
        [Required]
        public string Platform { get; set; }
    }
}
...
```

### Service

```cs
...
namespace Commander.Data
{
    public class SqlCommanderRepo : ICommanderRepo
    {
        //Implementation of Dependency Injection Pattern in C#
        private readonly CommanderContext _context;

        public SqlCommanderRepo(CommanderContext context)
        {
            _context = context;
        }

        public void CreateCommand(Command cmd)
        {
            if(cmd == null) {
                throw new ArgumentNullException(nameof(cmd));
            }
            _context.Commands.Add(cmd);
        }

        public void DeleteCommand(Command cmd)
        {
            if(cmd == null) {
                throw new ArgumentNullException(nameof(cmd));
            }
            _context.Commands.Remove(cmd);
        }

        public IEnumerable<Command> GetAllCommands()
        {
            return _context.Commands.ToList();
        }
        public Command GetCommandById(int id)
        {
            return _context.Commands.FirstOrDefault(p => p.Id == id);
        }

        public bool SaveChanges()
        {
            return (_context.SaveChanges() >= 0);
        }

        public void UpdateCommand(Command cmd)
        {
            
        }
    }
}
...
```


## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/DanielArturoAlejoAlvarez/COMMANDER. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

```

```
