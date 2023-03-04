# Heroku .NET Core Buildpack


This is the [Heroku buildpack](https://devcenter.heroku.com/articles/buildpacks) for [ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/).

[![Actions Status](https://github.com/jincod/dotnetcore-buildpack/workflows/main/badge.svg)](https://github.com/jincod/dotnetcore-buildpack/actions)

The Buildpack supports C# and F# projects. It searchs through the repository's folders to locate a `Startup.*` or `Program.*` file. If found, the `.csproj` or `.fsproj` in the containing folder will be used in the `dotnet publish <project>` command.

If repository contains **multiple** Web Applications (multiple `Startup.*` or `Program.*`), `PROJECT_FILE` and `PROJECT_NAME` environment variables allow to choose project for publishing.

## Usage

### .NET Core latest stable

```
heroku buildpacks:set jincod/dotnetcore
```

### .NET Core edge

```
heroku buildpacks:set https://github.com/jincod/dotnetcore-buildpack
```

### .NET Core Preview release

```
heroku buildpacks:set https://github.com/jincod/dotnetcore-buildpack#preview
```

### Previous releases

```
heroku buildpacks:set https://github.com/jincod/dotnetcore-buildpack#version
```

Available [releases](https://github.com/jincod/dotnetcore-buildpack/releases)

More info

- [Heroku Buildpack Registry](https://devcenter.heroku.com/articles/buildpack-registry)
- [Buildpack references](https://devcenter.heroku.com/articles/buildpacks#buildpack-references)

## Entity Framework Core Migrations

You cannot run migrations with the `dotnet ef` commands using **.NET Local Tools** once the app is built. Alternatives include:

### Enabling Automatic Migrations

- Ensure there is a .NET local tool manifest file(dotnet-tools.json) which specifies the dotnet-ef tool:

```bash
dotnet new tool-manifest
dotnet tool install dotnet-ef
```

- Ensure the `ASPNETCORE_ENVIRONMENT` environment variable is set to `Production`. ASP.NET Core scaffolding tools may create files that explicitly set it to `Development`. Heroku config will override this (`heroku config:set ASPNETCORE_ENVIRONMENT=Production`).
- Configure your app to automatically run migrations at startup by adding the following to the `.csproj` file:

```xml
<Target Name="PrePublishTarget" AfterTargets="Publish">
  <Exec Command="dotnet ef database update" />
</Target>
```
### Manually Running Migration Scripts on the Database

- Manually run SQL scripts generated by Entity Framework Core in your app's database. For example, use PG Admin to connect your Heroku Postgres service.

## Setting Connection String for heroku PostgreSQL
If you are using Heroku Postgres addon, then Heroku provides the database connection string as environment variable `DATABASE_URL`. In order to use this in an Npgsql connection string, do something like following:
```c#
using System.Text.RegularExpressions;

...

// Replace AppContext with your own DbContext Class
builder.Services.AddDbContext<AppContext>(options =>
{
  if (Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") == "Production")
  {
    var m = Regex.Match(Environment.GetEnvironmentVariable("DATABASE_URL")!, @"postgres://(.*):(.*)@(.*):(.*)/(.*)");
    options.UseNpgsql($"Server={m.Groups[3]};Port={m.Groups[4]};User Id={m.Groups[1]};Password={m.Groups[2]};Database={m.Groups[5]};sslmode=Prefer;Trust Server Certificate=true");
  } 
  else // In Development Environment
  {
    // So, use a local Connection
    options.UseNpgsql(builder.Configuration.GetValue<string>("CONNECTION_STRING"));
  }
});
```
## Node.js and NPM

```bash
heroku buildpacks:set jincod/dotnetcore
heroku buildpacks:add --index 1 heroku/nodejs
```

[Using Multiple Buildpacks for an App](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app)

## herokuish support

```bash
heroku config:set HEROKUISH=true
```

## Example

[ASP.NET Core Demo App](https://github.com/jincod/AspNet5DemoApp)

## Donation

If this project help you, you can give me a cup of coffee ☕

[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.me/jincod/5)
