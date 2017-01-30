---
title: Buenas prácticas asp.net core
updated: 2017-01-25 15:56
---

En el post de hoy me propongo recopilar algunas buenas prácticas a la hora de trabajar con <b>asp.net core</b> que he ido adquiriendo de algún proyecto, charlas
que asisto, libros o posts de la comunidad. 

### Defensive logging

Para entender el defensive logging es bueno conocer antes como funciona el logging en <b>asp.net core</b>. Este nos proporciona la interfaz <b>ILoggerFactory</b> que nos permitirá usar fácilmente un proveedor de los que propone <b>asp.net core</b> ya sea la propia consola de comandos, el debug output window, azure app service, visor de eventos… o bien un proveedor de terceros como serilog, elmah, nlog o incluso crearnos nuestro propio provider.  

<a href='https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging' title='Microsoft Docs - Logging'>Más info</a> 

Esta buena práctica está relacionada con los loglevels, estos permiten definir la cantidad de información que acabará volcándose en nuestro medio de log independientemente del que sea. Al configurar <b>logging</b> en startup posiblemente acabemos haciendo algo así. 

```
loggerFactory.AddConsole(Configuration.GetSection("Logging"));
```

Aquí le estamos diciendo que use la consola para volcar la información de <b>logging</b> y le estamos pasando la configuración de loglevels que nos viene por defecto definida en appsetttings.json. 

```
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information"
    }
  }
```

En esta <a href='https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging#log-level' title='Microsoft Docs Log Levels'>sección</a> se nos explica como usar correctamente <b>log levels</b> en nuestro código. 

```
public HomeController(IOptions<AppSettings> settings, ILogger<HomeController> logger)
{
    _settings = settings;
    _logger = logger;

    logger.LogDebug($"Home controller with settings: A: {settings.Value.ServiceAUrl}, B: {settings.Value.ServiceBUrl}");
}
```

Este código con la configuración por defecto mostrará la siguiente información al arrancar y navegar a la página de home. 

```
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET http://localhost:5000/
dbug: Restlessminds.Web.Controllers.HomeController[0]
      Home controller with settings: A: http://localhost:34927, B: http://localhost:35024
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action method Restlessminds.Web.Controllers.HomeController.Index (Restlessminds.Web) with arguments () - ModelState is Valid
```

Si modificamos en appsettings el valor de default por ‘information’ veremos que si volvemos a ejecutar la aplicación con dotnet run y examinamos el log ya nos aparece mi entrada con level debug. El propio servicio de <b>logging</b> es capaz de escribir en log o no dependiendo de si le hemos dicho que lo haga como explicaba en el párrafo anterior. El caso es que esto tiene un coste para nuestra aplicación que podríamos evitar si realizamos nosotros la comprobación de este modo que muestro. 

```
public HomeController(IOptions<AppSettings> settings, ILogger<HomeController> logger)
{
    _settings = settings;
    _logger = logger;
    
    if (logger.IsEnabled(LogLevel.Debug))
      logger.LogDebug($"Home controller with settings: A: {settings.Value.ServiceAUrl}, B: {settings.Value.ServiceBUrl}");
}
```

### Tipar settings

En la versión anterior de MVC ya se podría hacer algo para tener settings en una clase más como código, simplemente te creabas una clase estática del tipo settings y recuperas allí la configuración de appsettings o webconfig.
Aún así no podrás evitar tener "magic strings" en tu clase estática para recuperar cada setting. 

```
public static class Settings
{
   public static string ServiceAUrl = ConfigurationManager["ServiceAUrl"];
}
```

Así como en versiones anteriores de asp.net crearse una clase para concentrar allí el acceso a settings era un pattern que los desarrolladores adoptaron, en <b>asp.net core</b> es el propio framework el que te conduce a tipar los settings. 
En <b>asp.net core</b> por defecto solemos tener un fichero de settings appsettings.json en el cual podemos añadir secciones con nuestros settings o bien podemos crearnos archivos a parte y cargarlos adicionalmente en startup. 
Sea como sea cuando queremos usar un setting o sección en un servicio o controlador inyectaremos <b>IOptions&lt;NombreDeNuestraClase&gt;</b>. Previamente nos habremos creado esta clase con los settings que necesitamos como propiedades y le habremos dicho a nuestra aplicación al arrancar que los valores de los settings que necesita la clase están en la sección o archivo que corresponda. Veamos un ejemplo. 

Me crearé un fichero de settings MySettings.json con el siguiente contenido pero podría haber creado los settings en el fichero por defecto appsettings.json que suele incorporar la plantilla: 

```
"SettingA": "http://.../",
"SettingB": "http://.../"
```

Como he creado un fichero de configuración adicional tendré que decirle a la aplicación que lea sus valores al arrancar del siguiente modo:

```
 var builder = new ConfigurationBuilder()
                      .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true) 
                      .AddJsonFile("MySettings.json", optional: true, reloadOnChange: true) 
```

Para disponer de los settings tipados me crearé un objeto dummy como este:

```
public class MySettings
{
    public string SettingA {get;set;}
    public string SettingB {get;set;}
}
```

Como sabréis <b>asp.net core</b> carga todos los valores de configuración en una colección en memoria que tenemos disponible en el contexto. El siguiente paso será decirle a la aplicación que MySettings.cs es un objeto dummy que queremos usar como settings.
En ConfigureServices() de startup:

```
services.Configure<MySettings>(Configuration);
```

Por último cuando necesitemos recuperar un valor inyectaremos <b>Options&lt;MySettings&gt;</b> en nuestra clase usando la inyección de dependencias oob del framework. 

```
public MyService(IOptions<MySettings> settings)
{
  _settingA = settings.Value.SettingA;
}
```

### Bare Metal Api 
En la versión de asp.net anterior a core teníamos por un lado el framework MVC para hacer páginas web y por otro lado el framework Web api para hacer apis rest. Una de las novedades de <b>asp.net core</b> es que se unifican ambos en un sólo framework. 
Como sabréis .net core es modular, no es necesario importar la dll system.web con todos los paquetes del mundo mundial sino que podemos ir añadiendo dependencias (como paquetes nuget) a medida que necesitamos incorporar dicha funcionalidad. 
Pues bien, un error común es ponerse a crear una api que no va a hacer uso de vistas ni del motor Razor trabajando con el paquete completo MVC. 

Por tanto si vas a desarrollar una web api no importes mvc completo usa mejor <b>Microsoft.AspNetCore.Mvc.Core</b> que te proporcionará todo lo que necesitas para una api evitando de este modo mover dlls innecesarias en los procesos de restore, build y publish. Sólo tendrás que tener en cuenta lo siguiente: 

Al arrancar en vez services.addMvc le diremos que cargue el paquete alternativo y le tenemos que configurar manualmente el formatter para json. 

```
    services.AddMvcCore()
        .AddJsonFormatters();
```

Por último a nuestros controladores les tenemos que indicar que deriven de <b>ControllerBase</b> en detrimento de Controller para evitar que la clase base incorpore funcionalidad relacionada con las vistas.

