This is an exampkle on how to host angular app using angular-cli inside a DotNet Core Web.api project

# 1. Configure ASP.NET Core
dotnet new webapi

# 2. Modify .csproj
    Since we will eventually have TypeScript files in our project, we should also disable any TypeScript compilation errors.
    <TypeScriptCompileBlocked>true</TypeScriptCompileBlocked>

# 3. Modify Staryp.cs to use default route
    app.UseDefaultFiles();
    app.UseStaticFiles();
    
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");

        routes.MapRoute(
            name: "spa-fallback",
            template: "{*url}",
            defaults: new { controller = "Home", action = "Index" });
    });

# 4.  add angular app to your application
    ng new core-angular-example-client 

# 5. Enable  Http & Forms module
    Open your src/app/app.module.ts file and import the FormsModule and HttpModule

# 6. add a proxy.conf.json to the root 
    {
        "/api": {
            "target": "http://localhost:7834/",
            "secure": false
        }
    }

# 7. modify package.json of angular app so that npm to run build scripts 
    "scripts": {
        "ng": "ng",
        "start": "ng serve --proxy-config proxy.conf.json",
        "build": "ng build && xcopy dist\\*.* ..\\wwwroot\\ /s /e /Y",
        "build-prd": "ng build --prod --env=prod && xcopy dist\\*.* ..\\wwwroot\\ /s /e /Y",

# 8. create a package.json that redirect npm commands to the angular app folder
    {
    "name": "core-angular-example",
    "version": "0.0.0",
    "license": "MIT",
    "scripts": {
        "ng": "ng",
        "start": "cd core-angular-example-client && npm start",
        "build": "cd core-angular-example-client && npm build",
        "build-prd": "cd core-angular-example-client && npm build-prd"
    }
    }














