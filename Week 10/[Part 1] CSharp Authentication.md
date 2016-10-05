# Authentication in CSharp

In this demo students will be shown how to implement Authentication in C#. 

## Sources
[Token Based Authentication - Taiseer Joudeh](http://bitoftech.net/2014/06/01/token-based-authentication-asp-net-web-api-2-owin-asp-net-identity/)<br />
[ASP.NET Identity with Entity Framework](http://odetocode.com/blogs/scott/archive/2014/01/03/asp-net-identity-with-the-entity-framework.aspx)

## Table of Contents

- [ ] Project Setup
	- [ ] Convert project to OWIN project
	- [ ] Add ASP Identity System
	- [ ] Integrate ASP Identity into existing DataContext
- [ ] Implement Registration
	- [ ] Test the Register Controller
	- [ ] Secure the Students Controller
- [ ] Implement Login
	- [ ] Test Login

## Convert project to OWIN project

- [ ] Take an existing project (like the classroom project)
- [ ] Install the OWIN NuGet packages

`Install-Package Microsoft.AspNet.WebApi.Owin`<br />
`Install-Package Microsoft.Owin.Host.SystemWeb`<br />
`Install-Package Microsoft.Owin.Cors`

- [ ] Add Owin "Startup" Class

```cs
using Microsoft.Owin;
using Owin;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Http;
 
[assembly: OwinStartup(typeof(Classroom.API.Startup))]
namespace Classroom.API
{
    public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            HttpConfiguration config = new HttpConfiguration();
            WebApiConfig.Register(config);
            app.UseCors(Microsoft.Owin.Cors.CorsOptions.AllowAll);
            app.UseWebApi(config);
        }
    }
}
```

- [ ] Delete Global.asax class

## Add ASP Identity System

- [ ] Install the Nuget packages for ASP Identity

`Install-Package Microsoft.AspNet.Identity.Owin` - Integrates AspNet.Identity with Owin<br />
`Install-Package Microsoft.AspNet.Identity.EntityFramework` - Integrates AspNet.Identity with Entity Framework so we can save users to SQL Server.

## Integrate ASP Identity into existing DataContext

- [ ] Modify the DataContext class such that the class inherits from the `IdentityDbContext` class and that a call to `base.OnModelCreating(modelBuilder);` is made at the end of the `OnModelCreating` method. This will provide all of the Entity Framework code-first mapping and DbSet properties needed to manage the identity tables in SQL Server. You can read more about this class on [Scott Allen Blog](http://odetocode.com/blogs/scott/archive/2014/01/03/asp-net-identity-with-the-entity-framework.aspx)

- [ ] Add a migration to add the new tables to SQL Server `Add-Migration AddIdentity`
- [ ] Run `Update-Database` to apply the new migration

```cs
using Classroom.Core.Domain;using System.Data.Entity;namespace Classroom.Data.Infrastructure{    public class ClassroomDataContext : IdentityDbContext<IdentityUser>    {        public ClassroomDataContext() : base(
        {        }        public IDbSet<Project> Projects { get; set; }        public IDbSet<Assignment> Assignments { get; set; }        public IDbSet<Student> Students { get; set; }        protected override void OnModelCreating(DbModelBuilder modelBuilder)        {            modelBuilder.Entity<Project>()                        .HasMany(p => p.Assignments)                        .WithRequired(a => a.Project)                        .HasForeignKey(a => a.ProjectId);            modelBuilder.Entity<Student>()                        .HasMany(s => s.Assignments)                        .WithRequired(a => a.Student)                        .HasForeignKey(a => a.StudentId);            modelBuilder.Entity<Assignment>()                        .HasKey(a => new { a.StudentId, a.ProjectId });
                        
            base.OnModelCreating(modelBuilder);        }    }}
``` 

## Implement Registration

The first thing our backend needs to facilitate is user registration.

- [ ] Create a `UserModel.cs` file that contains the fields that will eventually be provided by the registration form on the front end.

```cs
public class UserModel
{
    [Required]
    [Display(Name = "User name")]
    public string UserName { get; set; }
 
    [Required]
    [StringLength(100, ErrorMessage = "The {0} must be at least {2} characters long.", MinimumLength = 6)]
    [DataType(DataType.Password)]
    [Display(Name = "Password")]
    public string Password { get; set; }
 
    [DataType(DataType.Password)]
    [Display(Name = "Confirm password")]
    [Compare("Password", ErrorMessage = "The password and confirmation password do not match.")]
    public string ConfirmPassword { get; set; }
}
```

- [ ] Create an `AuthRepository.cs` file with the following contents

```cs
public class AuthRepository : IDisposable
{
    private ClassroomDataContext _db;
 
    private UserManager<IdentityUser> _userManager;
 
    public AuthRepository()
    {
        _ctx = new AuthContext();
        _userManager = new UserManager<IdentityUser>(new UserStore<IdentityUser>(_db));
    }
 
    public async Task<IdentityResult> RegisterUser(UserModel userModel)
    {
        IdentityUser user = new IdentityUser
        {
            UserName = userModel.UserName
        };
 
        var result = await _userManager.CreateAsync(user, userModel.Password);
 
        return result;
    }
 
    public async Task<IdentityUser> FindUser(string userName, string password)
    {
        IdentityUser user = await _userManager.FindAsync(userName, password);
 
        return user;
    }
 
    public void Dispose()
    {
        _db.Dispose();
        _userManager.Dispose();
    }
}
```

- [ ] Create a `RegisterController.cs` file with the following contents

```cs
public class RegisterController : ApiController
{
	private AuthRepository _authRepository;
	
    public RegisterController()
    {
		_authRepository = new AuthRepository();
    }

    // POST api/Account/Register
    [AllowAnonymous]
    [Route("api/register")]
    public async Task<IHttpActionResult> Register(UserModel userModel)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        var result = _authRepository.RegisterUser(userModel);

        IHttpActionResult errorResult = GetErrorResult(result);

        if (errorResult != null)
        {
            return errorResult;
        }

        return Ok();
    }

    protected override void Dispose(bool disposing)
    {
        if (disposing)
        {
            _userManager.Dispose();
        }

        base.Dispose(disposing);
    }

    private IHttpActionResult GetErrorResult(IdentityResult result)
    {
        if (result == null)
        {
            return InternalServerError();
        }

        if (!result.Succeeded)
        {
            if (result.Errors != null)
            {
                foreach (string error in result.Errors)
                {
                    ModelState.AddModelError("", error);
                }
            }

            if (ModelState.IsValid)
            {
                // No ModelState errors are available to send, so just return an empty BadRequest.
                return BadRequest();
            }

            return BadRequest(ModelState);
        }

        return null;
    }
}
```

## Test the Register Controller

Open Postman and send an HTTP Post request to the API endpoint. You should expect a `200` response.

```json
{
	"userName": "origincodeacademy",
	"password": "Password123",
	"confirmPassword": "Password123"
}
```

## Secure the Students Controller

We will now lock down the students controller so that it only returns information to authenticated users.

- [ ] Add the [Authorize] attribute to each method in StudentsController
- [ ] Try calling the Endpoint in Postman, note the `401 Unauthorized` response that is returned.

```cspublic class StudentsController : ApiController{    private ClassroomDataContext db = new ClassroomDataContext();    // GET: api/Students    [Authorize]    public IQueryable<Student> GetStudents()    {        return db.Students;    }
}
```

## Implement Authentication (Login)

So that we can successfully receive the data from the Students endpoint - let's implement login.

- [ ] Install the OAuth Nuget package

`Install-Package Microsoft.Owin.Security.OAuth`

- [ ] Add a new method named `ConfigureOAuth` to `Startup.cs`, and call it within the `Configuration` method.

```cs
public void ConfigureOAuth(IAppBuilder app)
{
    var OAuthServerOptions = new OAuthAuthorizationServerOptions()
    {
        AllowInsecureHttp = true, // change to false for production
        TokenEndpointPath = new PathString("/api/token"), // endpoint to get tokens
        AccessTokenExpireTimeSpan = TimeSpan.FromDays(1), // specify token expiration time 
        Provider = new SimpleAuthorizationServerProvider() // specify the class to be used to validate user credentials.
    };
 
    // Token Generation
    app.UseOAuthAuthorizationServer(OAuthServerOptions);
    app.UseOAuthBearerAuthentication(new OAuthBearerAuthenticationOptions());
}
```

- [ ] Implement the `AuthorizationServerProvider` class

```cs
public class AuthorizationServerProvider : OAuthAuthorizationServerProvider
{
	
	// Responsible for validating the “Client”. In our case we have only one client so we’ll always return that its validated successfully.
	public override async Task ValidateClientAuthentication(OAuthValidateClientAuthenticationContext context)
	{
		context.Validated();
	}
	
	// Responsible for validating the username and password sent to the authorization server’s token endpoint
	public override async Task GrantResourceOwnerCredentials(OAuthGrantResourceOwnerCredentialsContext context)
	{
		context.OwinContext.Response.Headers.Add("Access-Control-Allow-Origin", new[] { "*" });
		
		using(var authRepository = new AuthRepository())
		{
			var user = await authRepository.FindUser(context.UserName, context.Password);
			
			if(user == null)
			{
				context.SetError("invalid_grant", "The username or password is incorrect");
				return;
			}			
		}
		
		var identity = new ClaimsIdentity(context.Options.AuthenticationType);
		identity.AddClaim(new Claim(ClaimTypes.Name, context.UserName));
		identity.AddClaim(new Claim(ClaimTypes.Role, "user"));
		
		context.Validated(identity);
	}
}
```

## Test Login

Now, let's test the login functionality.

- [ ] Open up postman, issue an HTTP post request to `/api/token`

<img src="http://i.imgur.com/TayhDLN.png" />

You should see the following

<img src="http://i.imgur.com/BfvRaN4.png" />
