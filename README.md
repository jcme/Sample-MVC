# A sample MVC application architecture
A compilation of settings, technologies, and techniques which I've found to be useful together.

This project is structured in such a way as to promote testability and maintainability when your application reaches a decent size.  Additionally, I've included some of the common features that I always like to introduce into my web applications.

Architecture
----
I like to think that what I have is onion architecture, but sometimes I muddle concerns from other patterns like "ports and adapters" or "hexagonal" architecture. The general idea is to layer your application in such a way that interfaces are relied upon and implementation is injected at runtime.  In my version of onion architecture, we have the following layers:
* Domain: This is the core of your application.  The Sample.Domain.* projects represent this concern, and are broken up into Model (business classes and objects), Interfaces (contracts for how to access and interact with those objects), and Logic (implementation of those interfaces).  The Sample.Domain.* projects should only ever reference Sample.Infrastructure.Interfaces and Sample.Helpers.
* Sample.Helpers: This is where I put all my extension methods and utility classes that don't necessarily pertain to my domain model.
* Sample.Infrastructure: This is where external dependencies live, such as database access, external api consumption, configuration, etc.  Interfaces are defined in Sample.Infrastructure.Interfaces, implementation is defined in the appropriate project, and implementation is injected via unity.
* Sample.DependencyResolver: This contains my unity configuration and knows where to find all implementations of all interfaces.  This is registered in the Web layer's Global.asax.
* Sample.Web: This is a standard ASP.NET MVC layer and is the entrypoint into the application.
* Tests/Sample.*.Tests: All domain services, relevant models, and infrastructure implementations should have corresponding unit test projects.  Because we use dependency injection, we can mock dependencies to make testing a breeze.  Without dependency injection, you're really just writing integration tests.

Unit 3.x
----
Can be found in Sample.DependencyResolver.  Unity is used for dependency injection, and is the glue which bootstraps the entire application.  A lazy unity container is implemented to help with the efficiency of loading up dependencies.  The Entity Framework context is also configured to use a PerLifetimeResolver, which helps when executing threaded code in your web application.

Entity Framework
----
Can be found in Sample.Infrastructure.Repository.  Entity Framework is used for data access.  I'm always torn when implementing a DAL on which technology to use.  I feel that Entity Framework can get you up and running quickly, but for larger projects I prefer Dapper.  In this case, I've added a custom configuration to Entity Framework which removes .'s in constraint names.  I also have it currently configured to drop and create that database model on changes, although you may wish to implement a different strategy (such as migrations).  Occasionally, I will introduce a Visual Studio Database project and set my entity intialization context to null so that I can manually manage my database (useful for when you have tables/procedures outside of the scope of your web application or Entity Framework).  There is also an abstract repository which can format DbEntityValidationException's and perform bulk inserts of raw data.

ASP.NET MVC 5.x with WebAPI
----
Can be found in Sample.Web.  This is a standard ASP.NET MVC Website, using Razer for it's view technology.  There is an Audit Message Handler included which will audit all incoming API requests to the database.  This is extremely useful if a 3rd party is consuming your API, as it allows you to know the frequency of your API usage.

Log4Net
----
Log4Net is utilized across the application to log application errors.  Log4Net is accessed via including ```private ILog log = LogManager.GetLogger(System.Reflection.MethodBase.GetCurrentMethod().DeclaringType);``` in any class you wish.  The logging configuration can be found in logging.config, and it is configured to send log messages to the file system, syslog (to be configured), and the local database.  You can view the latest log messages by going to /logs, and errors can be generated by going to /error.  Logging could be re-implemented as an infrastructure item.

NUnit + Moq
----
NUnit is utilized for unit tests (could be swapped for MSTest) and Moq is utilized for mocking (could be swapped for NSubstitute).  The technology behind unit tests is less important than the fact that you have unit tests.  Always be sure you write tests for your appliation!
