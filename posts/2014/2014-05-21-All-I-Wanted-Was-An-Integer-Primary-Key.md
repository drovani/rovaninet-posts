---
title: All I Wanted Was an Integer Primary Key
category: Vigil Journey
tags:
- vigil
date: 2014-05-21
---

> The [ASP.NET Identity](http://www.asp.net/identity) system is designed to replace the previous ASP.NET Membership and Simple Membership systems. It includes profile support, OAuth integration, works with OWIN, and is included with the ASP.NET templates shipped with Visual Studio 2013.

In and of itself, it is a very nice and pleasant library that really does integrate nicely into a WebAPI or MVC project.  The template project that comes with VS2013 spins up very quickly and provides the illusion of something that the developer can just forget about.  Were that the case, I would not have this post.

All I wanted was to change the default primary key from a `GUID` to an int.  My initial thought was to change the default `ApplicationUser` from inheriting from the non-generic IdentityUser to the generic `IdentityUser<int, VigilUserLogin, VigilUserRole, VigilUserClaim>`.  Compiling worked just fine, running caused all kinds of type casting problems.  Why?  Because all of the other classes surrounding the `IdentityUser` were all coded to the `string` type (which internally is a `GUID`).


Thus began the big hunt to find out all of the places where I needed to override, inherit, or otherwise manipulate the library to get the primary key to be an integer. This meant replacing the five main classes that create the authentication and authorization schema:

- IdentityUser
- IdentityRole
- IdentityUserRole
- IdentityUserClaim
- IdentityUserLogin

Then I had to create a new `UserStore` and `UserManager` to utilize these new classes, and create a new `ClaimsPrincipal` to store and retrieve the new security identifier (Sid) for the user. Finally, a new `BaseController` class was created that all other controllers must inherit from, in order to utilize all of this new code.

```csharp
using System;
using System.Security.Claims;
using System.Web.Mvc;
using Vigil.Data;

namespace Vigil.Web.Controllers
{
    public abstract class BaseController : Controller, IDisposable
    {
        public VigilClaimsPrincipal CurrentUser
        {
            get { return new VigilClaimsPrincipal((ClaimsPrincipal)this.User); }
        }
        protected readonly IVigilContext VigilContext = new VigilContext();

        public BaseController() { }

        protected override void Dispose(bool disposing)
        {
            if (disposing && VigilContext != null)
                VigilContext.Dispose();
            base.Dispose(disposing);
        }
    }
}
```

```csharp
using System.Data.Entity;
using Microsoft.AspNet.Identity;
using Microsoft.AspNet.Identity.EntityFramework;
using Vigil.Data;
using Vigil.Data.System;

namespace Vigil.Web
{
    public interface IVigilUserStore : IUserStore<VigilUser, int> { }

    public class VigilUserStore : UserStore<VigilUser, VigilRole, int, VigilUserLogin, VigilUserRole, VigilUserClaim>, IVigilUserStore
    {
        public VigilUserStore() : base(new VigilContext()) { }
        public VigilUserStore(DbContext context) : base(context) { }
    }

    public class VigilUserManager : UserManager<VigilUser, int>
    {
        public VigilUserManager() : this(new VigilUserStore(new VigilContext())) { }
        public VigilUserManager(IVigilUserStore store) : base(store) { }
    }
}
```

```csharp
using System;
using System.Security.Claims;
using Vigil.Data;
using Vigil.Data.System;

namespace Vigil.Web
{
    public class VigilClaimsPrincipal : ClaimsPrincipal
    {
        public VigilClaimsPrincipal(ClaimsPrincipal principal) : base(principal) { }

        public int UserId
        {
            get { return Int32.Parse(this.FindFirst(ClaimTypes.Sid).Value); }
        }

        public VigilUser VigilUser
        {
            get { return (new VigilContext()).Set<VigilUser>().Find(UserId); }
        }
    }
}
```

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.Infrastructure;
using System.Data.Entity.ModelConfiguration.Conventions;
using System.Data.Entity.Validation;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNet.Identity.EntityFramework;
using Vigil.Data.System;

namespace Vigil.Data
{
    public interface IVigilContext : IDisposable
    {
        DbChangeTracker ChangeTracker { get; }
        DbContextConfiguration Configuration { get; }
        Database Database { get; }
        DbEntityEntry Entry(object entity);
        DbEntityEntry<TEntity> Entry<TEntity>(TEntity entity) where TEntity : class;
        IEnumerable<DbEntityValidationResult> GetValidationErrors();
        int SaveChanges();
        Task<int> SaveChangesAsync();
        Task<int> SaveChangesAsync(CancellationToken cancellationToken);
        DbSet<TEntity> Set<TEntity>() where TEntity : class;
        DbSet Set(Type entityType);
    }
    public class VigilContext : IdentityDbContext<VigilUser, VigilRole, int, VigilUserLogin, VigilUserRole, VigilUserClaim>, IVigilContext
    {
        public VigilContext()
            : base("VigilDb")
        {
            Database.SetInitializer<VigilContext>(new VigilDbInitializer());
        }

        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.HasDefaultSchema("vigil");
            modelBuilder.Conventions.Remove<OneToManyCascadeDeleteConvention>();
        }
    }
}
```

```csharp
using Microsoft.AspNet.Identity;
using Microsoft.AspNet.Identity.EntityFramework;

namespace Vigil.Data.System
{
    public class VigilUser : IdentityUser<int, VigilUserLogin, VigilUserRole, VigilUserClaim>, IUser<int>
    {
    }
    public class VigilRole : IdentityRole<int, VigilUserRole>, IRole<int>
    {
    }
    public class VigilUserRole : IdentityUserRole<int>
    {
    }
    public class VigilUserLogin : IdentityUserLogin<int>
    {
    }
    public class VigilUserClaim : IdentityUserClaim<int>
    {
    }
}
```

At this point, I am wrestling with the fact that now my domain model has a hard requirement on the `Microsoft.AspNet.Identity` library. I had initially hoped to have my `User` and `Role` domain models be completely separated from the actual authentication/authorization system. This creates a hard link that is going to be the death of me unless I can find a way to isolate the framework and run unit tests without requiring the database. For now, though, I am going to continue in this direction, and turn a blind eye to the technical debt accruing from this decision. Maybe before I ship this, I will find a better way to do it all, but for now at least I have my integers.