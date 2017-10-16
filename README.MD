This is an example on how to host angular app using angular-cli inside a DotNet Core Web.api project

# 1. Configure ASP.NET Core
from th root of your project:
dotnet new webapi

# 2. Modify .csproj
    Since we will eventually have TypeScript files in our project, we should also disable any TypeScript compilation errors.
      <TypeScriptCompileBlocked>true</TypeScriptCompileBlocked>
      <TypeScriptToolsVersion>Latest</TypeScriptToolsVersion>
      <IsPackable>false</IsPackable>

# 3. Modify Staryp.cs to use default route
    app.UseDefaultFiles();
    app.UseStaticFiles();

    app.Use(async (context, next) => {
        await next();
        if (context.Response.StatusCode == 404 &&
            !Path.HasExtension(context.Request.Path.Value) &&
            !context.Request.Path.Value.StartsWith("/api/")) {
                context.Request.Path = "/index.html";
                await next();
            }
    });

    // Configure application for usage as API
    // with deafult route '/api/[Controller]'
    app.UseMvcWithDefaultRoute();

    // Serve static files
    app.UseStaticFiles();

# 4.  Add angular app to your application
    from the root of your project:
    ng new angular-app
    copy all the files & folders inside core-angular-example-client to the root of your app
    rename the src folder on the root by client-src
    modify the .angular-cli.json on the root so that ng build  output the content inside the wwroot folder of the web-api project:
     

     "apps": [
    {
      "root": "src",
      "outDir": "../web-api/wwwroot",

# 5. Add wwwroot to the .gitignore file
    wwwroot/

# 6. Enable  Http & Forms module
    Open your src/app/app.module.ts file and import the FormsModule and HttpModule

# 7. Add a proxy.conf.json to the root
    {
        "/api": {
            "target": "http://localhost:7834/",
            "secure": false
        }
    }

# 8. Modify package.json of angular app so that npm run build scripts
    "scripts": {
        "ng": "ng",
        "start": "ng serve --proxy-config proxy.conf.json",
        "build": "ng build",
        "build-prd": "ng build --prod --env=prod",

# 9. Modify the csproj file so that msbuild runs the ng build
   <Target Name="EnsureNode">
    <Exec Command="node --version" ContinueOnError="true">
      <Output TaskParameter="ExitCode" PropertyName="ErrorCode" />
    </Exec>
    <Error Condition="'$(ErrorCode)' != '0'" Text="Node.js is required to build and run this project. To continue, please install Node.js from https://nodejs.org/, and then restart your command prompt or IDE." />
  </Target>
  <Target Name="DebugRunNgBuild" BeforeTargets="Build" Condition=" '$(Configuration)' == 'Debug' ">
    <CallTarget Targets="EnsureNode" />
    <Message Importance="high" Text="Performing ng build for dev build..." />
    <Exec Command="npm run-script build" />
  </Target>
  <Target Name="ReleaseRunNgBuild" BeforeTargets="Build" Condition=" '$(Configuration)' == 'Release' ">
    <CallTarget Targets="EnsureNode" />
    <Message Importance="high" Text="Install packages..." />
    <Exec Command="npm install" />
    <Message Importance="high" Text="Performing ng build for prd build..." />
    <Exec Command="npm run-script build-prd" />
  </Target>
