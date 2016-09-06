# Clean Architecture

This demo is to show students the importance of good architecture in C# to allow for better unit testing and overall maintainability of a code base.

## High Level
* [ ] Clean dependencies
* [ ] Add Todo.Core Project
* [ ] Add Todo.Data Project
* [ ] Add DatabaseFactory to Todo.Data
* [ ] Add IRepository to Todo.Core
* [ ] Add Repository to Todo.Data
* [ ] Add Entity Repository Interfaces (Interface Segregation Principle)
* [ ] Add Unit of Work
* [ ] Tie the projects together
* [ ] Refactor TodoController
* [ ] Test existing application

## Steps

* [ ] Open the "unclean" Todo Project

**Clean Dependencies**
* [ ] Remove Entity Framework from `Todo.API`

**Add Todo.Core Project**
* [ ] Add a new `Todo.Core` project to the `Todo` solution
	* [ ] Right click `Todo` solution
	* [ ] Choose `New > New Project`
	* [ ] Select `Visual C# > Windows > Class Library`
	* [ ] Name the new project `Todo.Core`
	* [ ] Click OK
	* [ ] Remove `Class1.cs`
	* [ ] Add `Domain` folder to `Todo.Core`
		* [ ] Move `Todo.API.Models.Todo.cs` to `Todo.Core.Domain` folder.
		* [ ] Rename namespace in `Todo.cs` to `Todo.Core.Domain`
		* [ ] Delete `Todo.API.Models.Todo.cs`.

**Add Todo.Data Project**
* [ ] Add a new `Todo.Data` project to the `Todo` solution
	* [ ] Right click `Todo` solution
	* [ ] Choose `New > New Project`
	* [ ] Select `Visual C# > Windows > Class Library`
	* [ ] Name the new project `Todo.Core`
	* [ ] Click OK
	* [ ] Remove `Class1.cs`
	* [ ] Install `EntityFramework` via Nuget into `Todo.Data`
	* [ ] Add `Infrastructure` folder to `Todo.Data`
		* [ ] Move `Todo.API.Infrastructure.TodoDataContext.cs` to `Todo.Data.Infrastructure` folder.
		* [ ] Rename namespace in `TodoDataContext.cs` to `Todo.Data.Infrastructure`
		* [ ] Delete `Todo.API.Infrastructure.TodoDataContext.cs`
		* [ ] Delete `Todo.API.Migrations` (We'll add these later in `Todo.Data`)
	* [ ] Add project reference to `Todo.Core` in `Todo.Data`
		* [ ] Right click References in `Todo.Data`
		* [ ] Click `Add Reference`, check `Todo.Core`, click OK
	* [ ] Fix using statements in `TodoDataContext.cs`

**Add DatabaseFactory to `Todo.Data`**
* [ ] Add `IDatabaseFactory.cs` to `Infrastructure` ([Click here to scroll to code sample](#idatabasefactory.cs))
* [ ] Add `DatabaseFactory.cs` to `Infrastructure` ([Click here to scroll to code sample](#idatabasefactory.cs))

**Repository Interface in `Todo.Core`**
* [ ] Add `Infrastructure` folder to `Todo.Core`
	* [ ] Add `IRepository.cs`
	* [ ] Build `IRepository.cs` ([Click here to scroll to code sample](#irepository.cs))

**Repository Implementation in `Todo.Data`**
* [ ] Add `Repository.cs` to `Infrastructure` folder.
* [ ] Build `Repository.cs` ([Click here to scroll to code sample](#repository.cs)) 

**Entity Repository Interfaces (Interface Segregation Principle)**
* [ ] Add `Repository` folder to `Todo.Core`
* [ ] Add `ITodoRepository.cs` to `Repository` folder ([Click here to scroll to code sample](#itodorepository.cs))

**Entity Repository Implementation (Open Closed Principle)**
* [ ] Add `Repository` folder to `Todo.Data`
* [ ] Add `TodoRepository.cs` to `Repository` folder ([Click here to scroll to code sample](#todorepository.cs))

**Unit of Work**
* [ ] Add `IUnitOfWork.cs` to `Todo.Data.Infrastructure` ([Click here to scroll to code sample](#iunitofwork.cs))
* [ ] Add `UnitOfWork.cs` to `Todo.Data.Infrastructure` ([Click here to scroll to code sample](#unitofwork.cs))

**Tying the projects together**
* [ ] Add reference to `Todo.Core` and `Todo.Data` in `Todo.API`
* [ ] Install `SimpleInjector` and `SimpleInjector.Integration.WebApi` via Nuget into `Todo.API`
* [ ] Build `RegisterDependencies` method in `Global.asax.cs` 
	* (See [this section of SimpleInjector documention](https://simpleinjector.readthedocs.io/en/latest/webapiintegration.html) to demo docs to students)
	* [Click here to scroll to code sample](#global.asax.cs)

**Refactor TodoController**
* [ ] Refactor TodoController to use Dependency Inversion (Depend on abstractions rather than concrete implementations)
	* ([Click here to scroll to code sample](#todocontroller.cs))

**Test application**
* [ ] Test the existing application


### Code Samples

#### IDatabaseFactory.cs
```csharp
namespace Todo.Data.Infrastructure
{
    public interface IDatabaseFactory
    {
        TodoDataContext GetDataContext();
    }
}
```

#### DatabaseFactory.cs
```csharp
using System;

namespace Todo.Data.Infrastructure
{
    public class DatabaseFactory : IDisposable, IDatabaseFactory
    {
        private readonly TodoDataContext _dataContext;

        public TodoDataContext GetDataContext()
        {
            return _dataContext ?? new TodoDataContext();
        }

        public DatabaseFactory()
        {
            _dataContext = new TodoDataContext();
        }

        public void Dispose()
        {
            _dataContext.Dispose();
        }
    }
}
```

#### IRepository.cs
```csharp
using System;
using System.Linq;
using System.Linq.Expressions;

namespace Todo.Core.Infrastructure
{
    public interface IRepository<EntityType> where EntityType : class
    {
        // CREATE
        void Add(EntityType entity);

        // READ
        EntityType GetById(object id);
        IQueryable<EntityType> GetAll();
        IQueryable<EntityType> GetWhere(Expression<Func<EntityType, bool>> lambda);
        int Count();
        int Count(Expression<Func<EntityType, bool>> lambda);
        bool Any(Expression<Func<EntityType, bool>> lambda);

        // UPDATE
        void Update(EntityType entity);

        // DELETE
        EntityType Delete(object entityId);
        EntityType Delete(EntityType entity);
    }
}

```

#### Repository.cs
Use the `Implement Interface` quick action to show how interfaces work

```csharp
using System;
using System.Data.Entity;
using System.Linq;
using System.Linq.Expressions;
using Todo.Core.Infrastructure;

namespace Todo.Data.Infrastructure
{
    public class Repository<EntityType> : IRepository<EntityType> where EntityType : class
    {
        protected IDatabaseFactory DatabaseFactory;

        private TodoDataContext _dataContext;
        protected TodoDataContext DataContext => _dataContext ?? (_dataContext = DatabaseFactory.GetDataContext());

        protected IDbSet<EntityType> DbSet { get; set; }

        public Repository(IDatabaseFactory databaseFactory)
        {
            DatabaseFactory = databaseFactory;

            DbSet = DataContext.Set<EntityType>();
        }

        // CREATE
        public void Add(EntityType entity)
        {
            DbSet.Add(entity);
        }

        // READ
        public EntityType GetById(object id)
        {
            return DbSet.Find(id);
        }
        public IQueryable<EntityType> GetAll()
        {
            return DbSet;
        }
        public IQueryable<EntityType> GetWhere(Expression<Func<EntityType, bool>> lambda)
        {
            return DbSet.Where(lambda);
        }
        public int Count()
        {
            return DbSet.Count();
        }
        public int Count(Expression<Func<EntityType, bool>> lambda)
        {
            return DbSet.Count(lambda);
        }
        public bool Any(Expression<Func<EntityType, bool>> lambda)
        {
            return DbSet.Any(lambda);
        }

        // UPDATE
        public void Update(EntityType entity)
        {
            DbSet.Attach(entity);
            DataContext.Entry(entity).State = EntityState.Modified;
        }

        // DELETE
        public EntityType Delete(object entityId)
        {
            var entity = GetById(entityId);
            return Delete(entity);
        }
        public EntityType Delete(EntityType entity)
        {
            return DbSet.Remove(entity);
        }
    }
}
```

#### ITodoRepository.cs
```csharp
using Todo.Core.Infrastructure;

namespace Todo.Core.Repository
{
    public interface ITodoRepository : IRepository<Core.Domain.Todo>
    {
    }
}
```

#### TodoRepository.cs
```csharp
using Todo.API.Infrastructure;
using Todo.Core.Repository;
using Todo.Data.Infrastructure;

namespace Todo.Data.Repository
{
    public class TodoRepository : Repository<Core.Domain.Todo>, ITodoRepository
    {
        public TodoRepository(TodoDataContext context) : base(context)
        { }
    }
}
```

#### IUnitOfWork.cs
```csharp
namespace Todo.Data.Infrastructure
{
    public interface IUnitOfWork
    {
        void Commit();
    }
}
```

#### UnitOfWork.cs
```csharp
using Todo.API.Infrastructure;

namespace Todo.Data.Infrastructure
{
    public class UnitOfWork : IUnitOfWork
    {
        private readonly TodoDataContext _dbContext;

        public UnitOfWork(TodoDataContext dbContext)
        {
            _dbContext = dbContext;
        }

        public void Commit()
        {
            _dbContext.SaveChanges();
        }
    }
}
```

#### Global.asax.cs
```csharp
using SimpleInjector;
using SimpleInjector.Integration.WebApi;
using System.Web.Http;
using Todo.API.Infrastructure;
using Todo.Core.Repository;
using Todo.Data.Infrastructure;
using Todo.Data.Repository;

namespace Todo.API
{
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            // Create the container as usual.
            var container = new Container();
            container.Options.DefaultScopedLifestyle = new WebApiRequestLifestyle();

            // Register your types, for instance using the scoped lifestyle:
            container.Register<TodoDataContext>(Lifestyle.Scoped);
            container.Register<IUnitOfWork, UnitOfWork>();
            container.Register<ITodoRepository, TodoRepository>();

            // This is an extension method from the integration package.
            container.RegisterWebApiControllers(GlobalConfiguration.Configuration);

            container.Verify();

            GlobalConfiguration.Configuration.DependencyResolver = 
                new SimpleInjectorWebApiDependencyResolver(container);

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }
    }
}
```

#### TodosController.cs
```csharp
using System;
using System.Linq;
using System.Net;
using System.Web.Http;
using System.Web.Http.Description;
using Todo.Core.Repository;
using Todo.Data.Infrastructure;

namespace Todo.API.Controllers
{
    public class TodosController : ApiController
    {
        private readonly ITodoRepository _todoRepository;
        private readonly IUnitOfWork _unitOfWork;

        public TodosController(ITodoRepository todoRepository, IUnitOfWork unitOfWork)
        {
            _todoRepository = todoRepository;
            _unitOfWork = unitOfWork;
        }

        // GET: api/Todos
        public IQueryable<Core.Domain.Todo> GetTodoes()
        {
            return _todoRepository.GetAll();
        }

        // GET: api/Todos/5
        [ResponseType(typeof(Core.Domain.Todo))]
        public IHttpActionResult GetTodo(int id)
        {
            var todo = _todoRepository.GetById(id);
            if (todo == null)
            {
                return NotFound();
            }

            return Ok(todo);
        }

        // PUT: api/Todos/5
        [ResponseType(typeof(void))]
        public IHttpActionResult PutTodo(int id, Core.Domain.Todo todo)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            if (id != todo.TodoId)
            {
                return BadRequest();
            }

            _todoRepository.Update(todo);

            try
            {
                _unitOfWork.Commit();
            }
            catch (Exception)
            {
                if (!TodoExists(id))
                {
                    return NotFound();
                }
                else
                {
                    throw;
                }
            }

            return StatusCode(HttpStatusCode.NoContent);
        }

        // POST: api/Todos
        [ResponseType(typeof(Core.Domain.Todo))]
        public IHttpActionResult PostTodo(Core.Domain.Todo todo)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            _todoRepository.Add(todo);
            _unitOfWork.Commit();

            return CreatedAtRoute("DefaultApi", new { id = todo.TodoId }, todo);
        }

        // DELETE: api/Todos/5
        [ResponseType(typeof(Core.Domain.Todo))]
        public IHttpActionResult DeleteTodo(int id)
        {
            var todo = _todoRepository.GetById(id);
            if (todo == null)
            {
                return NotFound();
            }

            _todoRepository.Delete(todo);
            _unitOfWork.Commit();

            return Ok(todo);
        }

        private bool TodoExists(int id)
        {
            return _todoRepository.Any(e => e.TodoId == id);
        }
    }
}
```