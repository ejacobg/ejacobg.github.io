---
title: "Pastebin Q&A"
date: 2023-10-03T14:47:17-07:00
draft: false
---

Below are several questions I've asked and answered while in the process of building the [Pastebin project](/projects/pastebin/). I originally wrote them down to help keep track of what I was currently working on, but I found myself revisiting the same questions from time to time. I figure this may not be the last time I'll need the answers to these questions, so I'm posting them here for future reference. **I cannot guarantee the correctness of any answers presented herein.**

I also maintained a single ChatGPT session for the duration of the project. You may find a copy of it here: https://chat.openai.com/share/7d3ebd85-7722-4f0f-b5dc-b44992c40cc9

## Questions

### What's the difference between the Read and Write API Servers in the System Design Primer?

Related: [Read write APIs, are they seperate services?](https://stackoverflow.com/q/60051987)

They help to implement the [CQRS](https://martinfowler.com/bliki/CQRS.html) [pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs), which separates out
*commands* and *queries* into their own separate services, in this case represented by the Read and Write APIs.

Use a load balancer/API gateway/reverse proxy (what are the differences between all 3?) to forward incoming requests to the correct service (ie. check the request method to determine what goes where). Kubernetes provides its own load balancer service, [Ocelot](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/multi-container-microservice-net-applications/implement-api-gateways-with-ocelot) can be used as an API gateway, and [YARP](https://microsoft.github.io/reverse-proxy/articles/getting-started.html) can be used as a reverse proxy.

---

### How do I create just a web API (only controllers, no views)?

` dotnet new web -n <name>` is as good as you can get. You just have to set up the `Controllers/` directory yourself.

Remember to add the new project to your solution (`dotnet sln add <name>`)

---

### Where do I place my unit tests?

~~If you're placing them inside your project directory, you can put all of your tests under a `Tests/` directory.~~ Note: this doesn't seem to work. Place your tests in a `<Project>.Test/` directory instead.

---

### How do I set up xUnit?

https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test

---

### How do I install EF Core?

Documentation: https://learn.microsoft.com/en-us/ef/core/get-started/overview/install

Confirm that major versions of the packages match the major version of the runtime you're using.

Install your database provider:

```shell
dotnet add package Microsoft.EntityFrameworkCore.SqlServer -v 5.0.17
```

Install or update the `dotnet ef` tool if you haven't already:

```shell
dotnet tool install --global dotnet-ef
```

```shell
dotnet tool update
```

Install the design package:

```shell
dotnet add package Microsoft.EntityFrameworkCore.Design --version 5.0.17
```

---

### How do I use EF Core?

Documentation: https://learn.microsoft.com/en-us/ef/core/get-started/overview/first-app

[Create your entity classes as well as your associated database context.](https://learn.microsoft.com/en-us/ef/core/get-started/overview/first-app?tabs=netcore-cli#create-the-model)

Set up your database. See [**How do I create a SQL Server database and connect it to EF Core?**](/notes/pastebin-qna/#how-do-i-create-a-sql-server-database-and-connect-it-to-ef-core)

[Run your initial migration on your new database.](https://learn.microsoft.com/en-us/ef/core/get-started/overview/first-app?tabs=netcore-cli#create-the-database)

```shell
dotnet ef migrations add InitialCreate
dotnet ef database update
```

Also see ChatGPT's answer.

---

### How do I add a UNIQUE constraint in EF Core?

https://github.com/ejacobg/HackerNewsFeed/blob/cfe21a675dec7c3cdb090d2c728bf374fc569a04/HackerNewsFeed/Data/Item.cs#L9

---

### When reading paste requests, I want the `content` field to always be present, and the `expires` field is optional. How do I implement this?

```c#
[HttpPost("/shorten")]
public async Task<ShortenResult> Shorten([FromBody] PasteInput input)
{
    if (!ModelState.IsValid)
    {
          // ...
    }
    return new ShortenResult(
        await _shortenerService.Shorten(HttpContext, input.ToPaste())
    );
}

public class PasteInput
{
    [Required] public string Content { get; set; }
    public int? Expires { get; set; }

    public Paste ToPaste()
    {
        var expires = Expires ?? 60;
        return new Paste
        {
            Content = Content,
            Expires = expires
        };
    }
}

public record ShortenResult(string Shortlink);
```

By default, missing fields will just be given their default value. Use `[Required]` to enforce that they be present.

Note: normally you have to check `ModelState.IsValid` to make sure that all validations are correct. You can use the `[ApiController]` attribute to automatically return status code 400 for bad model states.

---

### How do I set up SQL Server? What's the equivalent of pgAdmin for SQL Server?

Download SQL Server (Developer 2022) - https://www.microsoft.com/en-us/sql-server/sql-server-downloads

Using Developer since that is closest to what will be used in production, and is compatible with replication and cloud services.

Download SSMS (19.1) - https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16

---

### How do I set up my `Startup` class? How do I register my interfaces and implementations? How do I enable my controllers?

```c#
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddScoped<IGenerator, IpGenerator>();
        services.AddSingleton<IPasteStore, MemoryStore>();
        services.AddScoped<IShortenerService, TextShortener>();
        services.AddControllers();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        app.UseRouting();
        app.UseEndpoints(endpoints => endpoints.MapControllers());
    }
}
```

---

### How can I add logging to my service?

https://youtu.be/NN9Rmf0PUG4

https://www.tutorialsteacher.com/core/aspnet-core-logging

https://www.tutorialsteacher.com/core/fundamentals-of-logging-in-dotnet-core

---

### How do I create a SQL Server database and connect it to EF Core?

In SSMS, go to Object Explorer > Databases > Right-Click > New Database.

The only option you really need to do is set a name.

The connection string I'm currently using is:

```
Server=JACOB-ZBOOK;Database=Pastebin;Trusted_Connection=True;MultipleActiveResultSets=true
```

I'm not sure how exactly the connection strings work right now.

If you're running on a different computer, execute the following query in SSMS to get the server name:

```sql
select @@SERVERNAME;
```

Add this key to the top-level object of both your `appsettings.json` and `appsettings.Development.json` files:

```json
"ConnectionStrings": {
    "DefaultConnection": "Server=JACOB-ZBOOK;Database=Pastebin;Trusted_Connection=True;MultipleActiveResultSets=true"
},
```

Add this code to your `Startup` class:

```c#
public Startup(IConfiguration configuration)
{
    Configuration = configuration;
}

public IConfiguration Configuration { get; }

public void ConfigureServices(IServiceCollection services)
{
  	// Change this to the name of your context.
    services.AddDbContext<AppDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"))
    );
}
```

---

### How should I re-generate shortlinks in the event of a collision?

One method is to repeatedly attempt to save the object to the database, and re-generate the shortlink if a unique constraint violation is detected.

This might be poor practice since you [keep repeatedly hitting the database](https://stackoverflow.com/questions/24740376/best-way-to-catch-sql-unique-constraint-violations-in-c-sharp-during-inserts#:~:text=If%20you%20are%20inserting%20to%20SQL%20table%20using%20loop%20you%20are%20doing%20it%20wrong).

Another method is to perform a quick existence check for the shortlink first *before* you perform the save:

```c#
// CustomerController
public string ChangeEmail(int customerId, string newEmail)
{
   // Check if there is another customer with the new email
    Customer existingCustomer = _customerRepository.GetByEmail(newEmail);
    if (existingCustomer != null && existingCustomer.Id != customerId)
        return "Email is already taken";

    Customer customer = _customerRepository.GetById(customerId);
    customer.Email = newEmail;
    _customerRepository.Save(customer);

    return "OK";
}
```

https://enterprisecraftsmanship.com/posts/handling-unique-constraint-violations/

https://stackoverflow.com/questions/48731623/entity-framework-violation-of-unique-key-constraint

This of course makes two round trips to the database.

With PostgreSQL, you can use the [`ON CONFLICT`](https://www.postgresql.org/docs/current/sql-insert.html#SQL-ON-CONFLICT) clause to prevent this. However, EF Core doesn't support it.

If you're using SQL Server, your best bet is to try to apply some database rules or maybe even hand-write your query.

This is the solution I used:

```c#
public async Task<bool> Save(Paste paste)
{
    if (await _context.Pastes.AnyAsync(p => p.Shortlink == paste.Shortlink))
    {
        return false;
    }

    _context.Pastes.Add(paste);
    await _context.SaveChangesAsync();
    return true;
}
```

Insert the paste first, then generate the shortlink?

---

### What lifetime should I use when registering my database stores in `ConfigureServices()`?

The ASP.NET Core In Action example uses a scoped lifetime.

---

### Base62 libraries?

https://stackoverflow.com/questions/17745025/convert-a-string-into-base62

https://www.better-converter.com/Encoders-Decoders/Base62-Encode

https://www.dcode.fr/base62-encoding

- This one seems to produce different output from all the rest.

https://base62.js.org/

---

### Both the Shortener and Lengthener services make use of the same data model. Should the Lengthener service reuse the model defined in the Shortener service or define its own?

~~I'll just inherit the model from Shortener for now.~~ Update: the data model has been moved into its own library. See [**How do I create a reusable data model?**](/notes/pastebin-qna/#how-do-i-create-a-reusable-data-model)

---

### What's the command to include/reference another project file?

How do I create this line in my `.csproj` file using the command line?

```csproj
<ItemGroup>
    <ProjectReference Include="..\LoyaltyProgram\LoyaltyProgram.csproj"/>
    <ProjectReference Include="..\SpecialOffers\SpecialOffers.csproj"/>
</ItemGroup>
```

[`dotnet add reference`](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-add-reference)

---

### How do I store and record my metrics/analytics? What are the available metrics/analytics solutions?

Related: https://learn.microsoft.com/en-us/aspnet/core/log-mon/metrics/metrics?view=aspnetcore-8.0

Right now, all I really know is to use the event feed pattern to publish relevant information.

I think what I should be looking at is "logging" rather than "metrics". There are centralized logging services that I can use like Seq, ELK (Elasticsearch, Logstash, Kibana), Azure Application Insights, and Datadog.

But even then, logging isn't really what I'm looking for either. I just want a service that collects information (eg. read and write requests) from my microservices and aggregates them.

On second thought, metrics makes more sense. They are defined as ["numerical measurements reported over time"](https://learn.microsoft.com/en-us/aspnet/core/log-mon/metrics/metrics?view=aspnetcore-8.0#:~:text=numerical%20measurements%20reported%20over%20time).

---

### How do I implement a page view counter/tracker microservice?

> [Probably a better model is to cache the number of views hourly in the app somewhere, and then update them in a batch-style process.](https://stackoverflow.com/a/1269973)

# 

> [It would be really easy if I just created a TABLE Question_Views And have a row for each question.](https://stackoverflow.com/q/14034288)

# 

> [Rather than execute SQL on every page view, you should increment an in-memory int or long using Interlocked.Increment. Then on every (for example) 5,000 increments (checked using Mod) then you should hit the database and increase the pageview count by 5,000.](https://stackoverflow.com/a/44980801)

> [Also consider using Redis to store the counts, rather than whatever your existing database is. Redis is awesome for this kind of use-case.](https://stackoverflow.com/a/44980801)

---

### What's a good status code to use for expired pastes?

> [A 404 status code only indicates that the resource is missing: not whether the absence is temporary or permanent. If a resource is permanently removed, use the 410 (Gone) status instead.](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404)

`ControllerBase` does not provide methods for returning all possible status codes, only the most common ones. Use the general `StatusCode()` method if you need something else:

```c#
// If the current time is after the Created + Expires date, then return an "expired" response.
// Expired pastes will no longer be tracked by the counter.
if (paste.Created.AddMinutes(paste.Expires) > DateTime.Now)
{
    return StatusCode(StatusCodes.Status410Gone);
}
```

---

### How do I create a reusable data model?

Related: [**How do I install EF Core?**](/notes/pastebin-qna/#how-do-i-install-ef-core)

I have multiple microservices that pull from the same database. Splitting the data model across multiple services doesn't seem to be the correct solution since you would then need to run the migrations from multiple places *and* in the right order.

You can create a class library (`dotnet new classlib -n <name>`) to hold your entire database.

```shell
dotnet new classlib -n Database
```

Create a `Models/` directory (or whatever name you like), and add your data models in it.

Define a database context with your entities (eg. `Database/AppDbContext.cs`). It is possible to define multiple different database contexts with different tables inside each of them, but that is out of the scope of these instructions (and causes errors when trying to create the migrations).

When all is finished, you can create and run your migrations inside the `Database` directory like so:

```shell
dotnet ef migrations add <Migration> --startup-project <Microservice>
dotnet ef database update --startup-project <Microservice>
```

You can also use the `-s` flag instead.

If you're not inside the `Database/` directory, you can use the `-p` or `--project` flags and point to it.

Because this was made as a class library, we have to borrow another project's `Startup.cs` class to launch EF Core.

The startup project must be configured to connect to your database and use the correct provider. See [**How do I create a SQL Server database and connect it to EF Core?**](/notes/pastebin-qna/#how-do-i-create-a-sql-server-database-and-connect-it-to-ef-core)

The startup project must have the database project as a dependency for this to work. See [**What’s the command to include/reference another project file?**](/notes/pastebin-qna/#whats-the-command-to-includereference-another-project-file)

I included an `appsettings.json` file inside the `Database/` directory containing the connection string that all microservices should be using:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=JACOB-ZBOOK;Database=Pastebin;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

---

### How do I set up Serilog?

```shell
dotnet add package Serilog.AspNetCore --version 6.1.0
dotnet add package Serilog.Sinks.Console --version 4.1.0
```

---

### How do I add a custom output template to Serilog?

https://github.com/ejacobg/Pastebin/blob/main/Shortener/Program.cs

---

### How do I get rid of all the extra output?

I make 1 request and this is what shows up:

```shell
2023-09-23 22:22:07 [INF] Starting up
2023-09-23 22:22:07 [INF] Now listening on: https://localhost:5003
2023-09-23 22:22:07 [INF] Now listening on: http://localhost:5002
2023-09-23 22:22:07 [INF] Application started. Press Ctrl+C to shut down.
2023-09-23 22:22:07 [INF] Hosting environment: Development
2023-09-23 22:22:07 [INF] Content root path: D:\Programming\GitHub\ejacobg\Pastebin\Lengthener
2023-09-23 22:22:07 [INF] Request starting HTTP/2 GET https://localhost:5003/ - -
2023-09-23 22:22:08 [INF] Request finished HTTP/2 GET https://localhost:5003/ - - - 404 0 - 44.8395ms
2023-09-23 22:23:06 [INF] Request starting HTTP/1.1 GET http://localhost:5002/lengthen/7xHvT6S - -
2023-09-23 22:23:06 [INF] Executing endpoint 'Lengthener.Controllers.LengthenController.Lengthen (Lengthener)'
2023-09-23 22:23:06 [INF] Route matched with {action = "Lengthen", controller = "Lengthen"}. Executing controller action with signature System.Threading.Tasks.Task`1[Microsoft.AspNetCore.Mvc.ActionResult`1[Lengthener.Controllers.LengthenController+LengthenResult]] Lengthen(System.String) on controller Lengthe
ner.Controllers.LengthenController (Lengthener).
2023-09-23 22:23:06 [INF] Entity Framework Core 5.0.17 initialized 'PasteContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
2023-09-23 22:23:07 [INF] Executed DbCommand (49ms) [Parameters=[@__shortlink_0='?' (Size = 450)], CommandType='Text', CommandTimeout='30']
SELECT TOP(2) [p].[PasteId], [p].[Content], [p].[Created], [p].[Expires], [p].[Shortlink]
FROM [Pastes] AS [p]
WHERE [p].[Shortlink] = @__shortlink_0
2023-09-23 22:23:07 [INF] Retrieved shortlink 7xHvT6S
2023-09-23 22:23:07 [INF] Executing ObjectResult, writing value of type 'Lengthener.Controllers.LengthenController+LengthenResult'.
2023-09-23 22:23:07 [INF] Executed action Lengthener.Controllers.LengthenController.Lengthen (Lengthener) in 1271.4897ms
2023-09-23 22:23:07 [INF] Executed endpoint 'Lengthener.Controllers.LengthenController.Lengthen (Lengthener)'
2023-09-23 22:23:07 [INF] Request finished HTTP/1.1 GET http://localhost:5002/lengthen/7xHvT6S - - - 200 - application/json;+charset=utf-8 1338.8532ms
```

The only logs I care about are the "Starting up", "Now listening on:", "Application started.", and maybe the "Hosting environment:" logs. I don't care about anything else, other than logs I specifically create from my application. In this case, the *only* log I made was on line 18:

```
2023-09-23 22:23:07 [INF] Retrieved shortlink 7xHvT6S
```

I've spent a little too much time messing with this, so I'm skipping it for now.

Related: https://youtu.be/Vsu_9rDV58k

Set up your `appsettings.json` and `appsettings.Development.json`:

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning"
      }
    }
  }
}
```

Update your [logging categories](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-7.0#configure-logging) as needed.

More: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-7.0#aspnet-core-and-ef-core-categories

Update your `Program.cs` file accordingly:

```c#
public class Program
{
    public static void Main(string[] args)
    {
        Log.Logger = new LoggerConfiguration()
            .WriteTo.Console()
            .CreateBootstrapLogger();
        Log.Information("Starting up!");
        try
        {
            CreateHostBuilder(args).Build().Run();
            Log.Information("Stopped cleanly");
        }
        catch (Exception ex)
        {
            Log.Fatal(ex, "An unhandled exception occured during bootstrapping");
        }
        finally
        {
            Log.CloseAndFlush();
        }
    }
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder => { webBuilder.UseStartup<Startup>(); })
            .UseSerilog((context, services, configuration) =>
            {
                configuration
                    .ReadFrom.Configuration(context.Configuration)
                    .ReadFrom.Services(services)
                    .WriteTo.Console(outputTemplate:
                        "{Timestamp:yyyy-MM-dd HH:mm:ss} [{Level:u3}] {Message:lj}{NewLine}{Exception}");
            });
}
```

See the example project for some configuration options: https://github.com/serilog/serilog-aspnetcore/tree/dev/samples/Sample

---

### Kubernetes?

Related: https://www.jeremyjordan.me/kubernetes/

There are two types of nodes in a Kubernetes cluster: the **master** nodes, which manage the "control plane", and the **worker** nodes which execute tasks assigned by the masters.

Work is assigned to a worker in the form of a **pod**, which represents one or more container images to execute on that worker.

Operators interact with the Kubernetes cluster through a CRUD-like interface. The cluster provides several "resources" that you can operate on.

The **deployment** resource is the recommended method for deploying a Kubernetes cluster. The deployment resource specifies a **template** for instantiating a pod and the number of replicas, and Kubernetes tries to synchronize the cluster to satisfy the requests laid out in the template.

Deployments typically consist of one or more pods. We can also define a replica count for each pod in the deployment. Templates are typically defined as YAML files:

![deployment_spec](https://www.jeremyjordan.me/content/images/2019/11/deployment_spec.png)

The **service** resource allows a pod to communicate with other resources in the cluster. There are two types of service resources:

A Kubernetes Service provides you with a stable endpoint which can be used to direct traffic to the desired Pods even as the exact underlying Pods change due to updates, scaling, and failures. Services know which Pods they should send traffic to based on *labels* (key-value pairs) which we define in the Pod metadata.

![service_spec](https://www.jeremyjordan.me/content/images/2019/11/service_spec.png)

The **ingress** service exposes HTTP and HTTPS endpoints to handle traffic originating outside the cluster.

Ingress objects allow outside traffic to reach our cluster objects (typically Services).

![ingress_spec](https://www.jeremyjordan.me/content/images/2019/11/ingress_spec.png)

### `spec.template.spec.containers.ports.containerPort`?

https://github.com/ejacobg/microservices-in-dotnet/blob/main/ShoppingCart/ShoppingCart/shopping-cart.yaml#L22

This isn't set by default by Rider, but may be needed. [Documentation](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#ports) says it is required.

---

### How do I deploy SQL Server to Kubernetes?

Related: https://medium.com/codex/database-migration-when-your-service-is-running-in-kubernetes-abbe9697421d

What's the best way to apply my migrations to my SQL Server instance?

> [The recommended way to deploy migrations to a production database is by generating SQL scripts.](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli)

You can also create and run a [Bundle](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli#bundles) so that you don't have to mess around with the command-line.

**There is no explicit solution for applying migrations to SQL Server.** There are a couple options for running migrations, none of which are perfect:

- **Runtime Migrations** (`context.Database.Migrate()`) - this is the simplest solution, but fails when scaling your application since every replica tries to apply your migrations on the database.
source:: https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli#apply-migrations-at-runtime

- [**Init Containers**](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) - Init Containers are run before a pod's containers are started. They are meant to be used to apply some startup actions not present in the application. Using Init Containers has the same issue as the Runtime Migrations if we're running multiple replicas of the same pod. However, this is technically a viable option for us since it is recommended to run SQL Server in a single pod.

- **Kubernetes Jobs** - A Kubernetes Job can be used to perform the same actions as an Init Container, however Jobs do not block your pods from starting up like Init Containers do. The benefit of using a Job is that you can ensure that only 1 job is applying the migrations, so you don't run into the issues outlined above. The ideal solution would combine a Kubernetes Job that applies the migrations with an Init Container around each pod that simply waits for the Job to complete before starting the pod:
    - ![](https://miro.medium.com/v2/resize:fit:4800/format:webp/1*7egKoSZFtJbOV5oOOy39Ng.png)
    - This job can simply be a [simple Console Application](https://www.gokhan-gokalp.com/en/performing-sql-migration-operations-by-using-kubernetes-job/) that you've dockerized that just calls `context.Database.Migrate()`. A similar project is described [here](https://codebuckets.com/2020/08/14/applying-entity-framework-migrations-to-a-docker-container/).
    - You can use something like [`kubectl wait`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#wait), [`wait-for`](https://github.com/groundnuty/k8s-wait-for), or Helm hooks to block while a Job is running.
    - This approach is the recommended one described [here](https://andrewlock.net/deploying-asp-net-core-applications-to-kubernetes-part-7-running-database-migrations/#:~:text=As%20I%20say%2C%20this%20approach%20is%20one%20that%20I%27ve%20been%20using%20successfully%20for%20several%20years%20now%2C%20so%20I%20think%20it%20does%20the%20job.%20In%20the%20next%20post%20I%27ll%20go%20into%20the%20details%20of%20actually%20implementing%20this%20approach.).
    - More: https://blog.jonas.tonndorf-martini.de/database-migration-with-a-kubernetes-job-and-initcontainers-d196dec8ac96, 

- [Using `kubectl exec` to manually run the setup script.](https://saturncloud.io/blog/how-to-execute-a-sql-script-file-in-a-kubernetes-pod-a-guide/#step-3-executing-the-sql-script)
    - This example uses MySQL; don't know if this is applicable to SQL Server.
    - [Using `kubectl exec` is not a good way to program a Kubernetes cluster.](https://stackoverflow.com/questions/73086332/how-to-run-kubectl-exec-on-scripts-that-never-end#:~:text=Using%20kubectl%20exec%20is%20not%20a%20good%20way%20to%20program%20a%20Kubernetes%20cluster.)

- Using [Container Lifecycle Hooks](https://stackoverflow.com/q/59521684).

There may be some solutions using technologies like Helm, but that's kinda out of the scope of this project.

Update: it seems that the official SQL Server Docker image supports initialization through SQL files. I've changed the scripts to use the `MSSQL_SA_PASSWORD` instead of `SA_PASSWORD`. More: https://stackoverflow.com/q/76233039, https://hub.docker.com/_/microsoft-mssql-server?tab=description, https://github.com/microsoft/mssql-docker/tree/master/linux/preview/examples/mssql-customize

---

### What connection string do I use when deploying to Kubernetes?

I'm currently using this connection string:

```json
"ConnectionStrings": {
    "Kubernetes": "Server=sql-server,1433;Database=Pastebin;User Id=SA;Password=StrongPassw0rd"
},
```

Find more connection string parameters here: https://learn.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlconnection.connectionstring?view=dotnet-plat-ext-7.0

---

### Using the Ingress service on Docker Desktop Kubernetes?

Seems that this manifest needs to be applied first: https://stackoverflow.com/questions/65193758/enable-ingress-controller-on-docker-desktop-with-wls2

---

### My Ingress service doesn't have a local IP address assigned to it (Docker Desktop Kubernetes). How to fix?

Make sure your services have the correct ports exposed:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lengthener
spec:
  selector:
    app: lengthener
  ports:
    - protocol: TCP
      port: 5002
      targetPort: 80
  type: NodePort
```

I had originally had the `port` and `targetPort` fields swapped.

[This answer](https://stackoverflow.com/a/54784023) helped me check my ports.

See [shopping-cart.yaml](https://github.com/ejacobg/microservices-in-dotnet/blob/main/ShoppingCart/ShoppingCart/shopping-cart.yaml) for a correct, annotated manifest example.

Make sure your Ingress points to the `targetPort` of your services:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pastebin
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /shorten
            pathType: Prefix
            backend:
              service:
                name: shortener
                port:
                  number: 80
```

When using Kubernetes on Windows Docker Desktop, the Ingress service is run inside the WSL2 virtual machine by default, which isn't accessible from Windows. This is why your `kubectl` commands cannot report the the IP address of the Ingress, just showing `localhost`. To find the IP address, you have to go inside the WSL VM. See https://stackoverflow.com/a/69113528 and https://stackoverflow.com/a/70025145.

```shell
$ wsl
$ ip a | grep eth0
6: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    inet 172.19.xxx.yyy/20 brd 172.19.xxx.yyy scope global eth0
```
Your virtual machine would be accessible on `172.19.xxx.yyy`

To get the ingress port use `kubectl`:

```shell
$ kubectl get svc -n ingress-nginx
NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.103.245.12   localhost     80:30770/TCP,443:30260/TCP   59m
ingress-nginx-controller-admission   ClusterIP      10.102.42.87    <none>        443/TCP                      59m
```

In my case, I used port 80 (using 30770 returns a connection refused error).

---

### Does a container's exposed ports apply to all containers build on top of it?

Rider's default file defines the base container as this:

```Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
```

The final container looks like this:

```Dockerfile
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Lengthener.dll"]
```

Will the final container also have the base container's exposed ports?

No. Set your ports on the final container.

### Docker `COPY`

[`COPY` obeys the following rules:](https://docs.docker.com/engine/reference/builder/#:~:text=be%20used%20instead.-,COPY,obeys%20the%20following%20rules%3A,-The)

The `<src>` path must be inside the *context* of the build; you cannot `COPY ../something /something`, because the first step of a `docker build` is to send the context directory (and subdirectories) to the docker daemon.

---

## Unanswered Questions

### How do I inject my database connection string into my database context?

### How do I Dockerize and push to my local registry?

### What's Docker Compose?

### Secrets management?

The database needs a password. How do I supply this without hard-coding it? I also need to supply it in the SQL Server manifest.
