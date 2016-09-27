# Clean Architecture

This demo is to show students the importance of good architecture in C# to allow for better unit testing and overall maintainability of a code base.

## High Level
Each high level step also includes which of the SOLID principles are achieved when completing the step.

* [ ] Clean dependencies
* [ ] Add Classroom.Core Project
* [ ] Add Classroom.Data Project
* [ ] Add DatabaseFactory to Classroom.Data (S,L,D)
* [ ] Add IRepository to Classroom.Core (L)
* [ ] Add Repository to Classroom.Data (S)
* [ ] Add Entity Repository Interfaces (O,I)
* [ ] Add Unit of Work (S,L,D)
* [ ] Tie the projects together
* [ ] Refactor Controllers (D)
* [ ] Test existing application

## Steps

* [ ] Create a new branch (in case things go wrong)

```
cd <your_project>
git add .
git commit -m "This is where I'd like to get back to if things go wrong"
git checkout -b CleanArchitecture
```

**Clean Dependencies**
* [ ] Remove Entity Framework from `Classroom.API`

**Add Classroom.Core Project**
* [ ] Add a new `Classroom.Core` project to the `Classroom` solution
	* [ ] Right click `Classroom` solution
	* [ ] Choose `New > New Project`
	* [ ] Select `Visual C# > Windows > Class Library`
	* [ ] Name the new project `Classroom.Core`
	* [ ] Click OK
	* [ ] Remove `Class1.cs`
	* [ ] Add `Domain` folder to `Classroom.Core`
		* [ ] Move models in `Classroom.API.Models` to `Classroom.API.Domain`
		* [ ] Rename namespace for all models to `Classroom.API.Domain`
		* [ ] Delete models folder in `Classroom.API`

**Add Classroom.Data Project**
* [ ] Add a new `Classroom.Data` project to the `Classroom` solution
	* [ ] Right click `Classroom` solution
	* [ ] Choose `New > New Project`
	* [ ] Select `Visual C# > Windows > Class Library`
	* [ ] Name the new project `Classroom.Data`
	* [ ] Click OK
	* [ ] Remove `Class1.cs`
	* [ ] Install `EntityFramework` via Nuget into `Classroom.Data`
	* [ ] Add `Infrastructure` folder to `Classroom.Data`
		* [ ] Move `ClassroomDataContext.cs` from the API Project into `Classroom.Data.Infrastructure`.
		* [ ] Rename namespace in `ClassroomDataContext.cs` to `Classroom.Data.Infrastructure`
		* [ ] Delete `Classroom.API.Infrastructure.ClassroomDataContext.cs`
		* [ ] Delete `Classroom.API.Migrations`
	* [ ] Add project reference to `Classroom.Core` in `Classroom.Data`
		* [ ] Right click References in `Classroom.Data`
		* [ ] Click `Add Reference`, check `Classroom.Core`, click OK
	* [ ] Fix using statements in `ClassroomDataContext.cs`

**Add DatabaseFactory to `Classroom.Data`**
* [ ] Add `IDatabaseFactory.cs` to `Infrastructure`
```csharp
namespace Classroom.Data.Infrastructure
{
    public interface IDatabaseFactory
    {
        ClassroomDataContext GetDataContext();
    }
}
```

* [ ] Add `DatabaseFactory.cs` to `Infrastructure`
```csharp
using System;

namespace Classroom.Data.Infrastructure
{
    public class DatabaseFactory : IDisposable, IDatabaseFactory
    {
        private readonly ClassroomDataContext _dataContext;

        public ClassroomDataContext GetDataContext()
        {
            return _dataContext ?? new ClassroomDataContext();
        }

        public DatabaseFactory()
        {
            _dataContext = new ClassroomDataContext();
        }

        public void Dispose()
        {
            _dataContext.Dispose();
        }
    }
}
```

**Repository Interface in `Classroom.Core`**
* [ ] Add `Infrastructure` folder to `Classroom.Core`
	* [ ] Add `IRepository.cs`
	* [ ] Build `IRepository.cs`
```csharp
using System;
using System.Linq;
using System.Linq.Expressions;

namespace Classroom.Core.Infrastructure
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

**Repository Implementation in `Classroom.Data`**
* [ ] Add `Repository.cs` to `Infrastructure` folder.
* [ ] Build `Repository.cs`
```csharp
using System;
using System.Data.Entity;
using System.Linq;
using System.Linq.Expressions;
using Classroom.Core.Infrastructure;

namespace Classroom.Data.Infrastructure
{
    public class Repository<EntityType> : IRepository<EntityType> where EntityType : class
    {
        protected IDatabaseFactory DatabaseFactory;

        private ClassroomDataContext _dataContext;
        protected ClassroomDataContext DataContext => _dataContext ?? (_dataContext = DatabaseFactory.GetDataContext());

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

**Entity Repository Interfaces (Interface Segregation Principle)**
* [ ] Add `Repository` folder to `Classroom.Core`
* [ ] Add an interface repository for the `Project`, `Student` and `Assignment` classes.
* [ ] For example, add `IProjectRepository.cs` to `Repository` folder
```csharp
using Classroom.Core.Infrastructure;

namespace Classroom.Core.Repository
{
    public interface IProjectRepository : IRepository<Core.Domain.Project>
    {
    }
}
```

**Entity Repository Implementation (Open Closed Principle)**
* [ ] Add `Repository` folder to `Classroom.Data`
* [ ] Add a repository implementation for the `Project`, `Student` and `Assignment` classes.
* [ ] For example, add `ProjectRepository.cs` to `Repository` folder
```csharp
using Classroom.Core.Repository;
using Classroom.Data.Infrastructure;

namespace Classroom.Data.Repository
{
    public class ProjectRepository : Repository<Core.Domain.Project>, IProjectRepository
    {
        public ProjectRepository(IDatabaseFactory databaseFactory) : base(databaseFactory)
        { }
    }
}
```

**Unit of Work**
* [ ] Add `IUnitOfWork.cs` to `Classroom.Data.Infrastructure`
```csharp
namespace Classroom.Data.Infrastructure
{
    public interface IUnitOfWork
    {
        void Commit();
    }
}
```

* [ ] Add `UnitOfWork.cs` to `Classroom.Data.Infrastructure`
```csharp
using Classroom.API.Infrastructure;

namespace Classroom.Data.Infrastructure
{
    public class UnitOfWork : IUnitOfWork
    {
        private readonly IDatabaseFactory _databaseFactory;

        private ClassroomDataContext _dataContext;
        private ClassroomDataContext DataContext
        {
            get
            {
                return _dataContext ?? (_dataContext = _databaseFactory.GetDataContext());
            }
        }

        public UnitOfWork(IDatabaseFactory databaseFactory)
        {
            _databaseFactory = databaseFactory;
        }

        public void Commit()
        {
            _dataContext.SaveChanges();
        }
    }
}
```

**Tying the projects together**
* [ ] Add reference to `Classroom.Core` and `Classroom.Data` in `Classroom.API`
* [ ] Install `SimpleInjector` and `SimpleInjector.Integration.WebApi` via Nuget into `Classroom.API`
* [ ] Build `RegisterDependencies` method in `Global.asax.cs` 
	* (See [this section of SimpleInjector documention](https://simpleinjector.readthedocs.io/en/latest/webapiintegration.html) to demo docs to students)
```csharp
using SimpleInjector;
using SimpleInjector.Integration.WebApi;
using System.Web.Http;
using Classroom.API.Infrastructure;
using Classroom.Core.Repository;
using Classroom.Data.Infrastructure;
using Classroom.Data.Repository;

namespace Classroom.API
{
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            RegisterDependencies();

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }

        private void RegisterDependencies()
        {
            // Create the container as usual.
            var container = new Container();
            container.Options.DefaultScopedLifestyle = new WebApiRequestLifestyle();

            // Register your types, for instance using the scoped lifestyle:
            container.Register<ClassroomDataContext>(Lifestyle.Scoped);
            container.Register<IUnitOfWork, UnitOfWork>();
            container.Register<IClassroomRepository, ClassroomRepository>();

            // This is an extension method from the integration package.
            container.RegisterWebApiControllers(GlobalConfiguration.Configuration);

            container.Verify();

            GlobalConfiguration.Configuration.DependencyResolver = 
                new SimpleInjectorWebApiDependencyResolver(container);
        }
    }
}
```

**Refactor Controllers**
* [ ] Refactor Controllers to use Dependency Inversion (Depend on abstractions rather than concrete implementations)

Here is an example for `ProjectsController`.
```csharp
using Classroom.Core.Domain;
using Classroom.Core.Repository;
using Classroom.Data.Infrastructure;
using System;
using System.Linq;
using System.Net;
using System.Web.Http;
using System.Web.Http.Description;

namespace Classroom.API.Controllers
{
    public class ProjectsController : ApiController
    {
        private IProjectRepository _projectRepository;
        private IUnitOfWork _unitOfWork;

        public ProjectsController(IProjectRepository projectRepository, IUnitOfWork unitOfWork)
        {
            _projectRepository = projectRepository;
            _unitOfWork = unitOfWork;
        }

        // GET: api/Projects
        public IQueryable<Project> GetProjects()
        {
            return _projectRepository.GetAll();
        }

        // GET: api/Projects/5
        [ResponseType(typeof(Project))]
        public IHttpActionResult GetProject(int id)
        {
            var project = _projectRepository.GetById(id);

            if (project == null)
            {
                return NotFound();
            }

            return Ok(project);
        }

        // PUT: api/Projects/5
        [ResponseType(typeof(void))]
        public IHttpActionResult PutProject(int id, Project project)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            if (id != project.ProjectId)
            {
                return BadRequest();
            }

            _projectRepository.Update(project);

            try
            {
                _unitOfWork.Commit();
            }
            catch (Exception)
            {
                if (!ProjectExists(id))
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

        // POST: api/Projects
        [ResponseType(typeof(Project))]
        public IHttpActionResult PostProject(Project project)
        {
            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            _projectRepository.Add(project);
            _unitOfWork.Commit();

            return CreatedAtRoute("DefaultApi", new { id = project.ProjectId }, project);
        }

        // DELETE: api/Projects/5
        [ResponseType(typeof(Project))]
        public IHttpActionResult DeleteProject(int id)
        {
            var project = _projectRepository.GetById(id);

            if (project == null)
            {
                return NotFound();
            }

            _projectRepository.Delete(project);
            _unitOfWork.Commit();

            return Ok(project);
        }

        private bool ProjectExists(int id)
        {
            return _projectRepository.Any(p => p.ProjectId == id);
        }
    }
}
```
