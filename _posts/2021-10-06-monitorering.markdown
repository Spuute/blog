---
layout: post
title:  "Monitorering"
---

## Inledning

Denna lektion ska vi lägga till logging till en applikation, och detta genom Microsoft Azure Application Insights

### Implementering av Application Insights

Det första jag gjorde var att lägga till ett NuGet paket till mitt projekt, och paketet i fråga är Microsoft.ApplicationInsights.AspNetCore. Därefter lägger jag till den i min Startup.cs och under ConfigureServices

```csharp
services.AddApplicationInsightsTelemetry();
```

Här kan man välja att lägga till sin Instrumentation nyckel som en parameter till koden ovan, men om man lämnar den utan parametrar så kommer den hämta nyckeln från min appsettings fil istället

```json
 "ApplicationInsights": {
    "InstrumentationKey": "min-kod-ligger-här"
  }
```


## Beskriv kort applikationen, vad gör den?

För denna uppgift så har jag valt att ta den tidigare applikationen jag gjorde där man kan spara och läsa recept från en databas, så för mer information om den vänligen läs inlägget om den <a href="https://spuute.github.io/blog/2021/09/24/webapp-i-molnet.html">HÄR</a>.

## Diagram över hur applikationen är kopplad till andra tjänster

![diagram](https://github.com/Spuute/blog/blob/main/img/Untitled%20Diagram.drawio-3.png?raw=true)

## Beskriv koden med fokus på den del som rör din log implementation

Nu tar jag min dishes.cshtml.cs fil som används för att hämta data från databasen. 

```csharp
public class dishesModel : PageModel
    {
        private readonly ILogger<dishesModel> _logger;
        public dishesModel(ILogger<dishesModel> logger)
        {
            _logger = logger;
        }
        public List<Recipe> Test;

        public void OnGet()
        {
            try
            {
                var client = new RestClient("https://restapi-cosmosdb.azurewebsites.net/api");
                var request = new RestRequest($"/recipe?", DataFormat.Json);

                var response = client.Execute<List<Recipe>>(request);
                Test = response.Data;
                _logger.LogInformation($"Successfull read from db");
            }
            catch (System.Exception ex)
            {
                _logger.LogError(ex, ex.Message);
            }
        }
    }
```

Jag skapar en konstruktor som jag sedan använder för att injecera ILogger och tilldelar sedan min _logger den. 

Sen lägger jag till ett try-catch block där jag lägger in min kod, och väljer därefter att köra en LogInformation som loggar när man har läst in allt från databasen. 
och i min catch del så har jag en LogError som kommer logga ett exeption om det skulle bli något sådant. 

När det kommer till ILogger så finns det några olika nivåer loggar

* .LogTrace
* .LogDebug
* .LogInformation
* .LogWarning
* .LogError
* .LogCritical

Jag har valt att använda information till sånt som helt enkelt visar flödet i min applikation, som när man läst in data från databasen, och error ifall det skulle komma ett exception. 
Rätt ur lådan så att säga så fungerar ingen av ovanstående som ligger på en lägre nivå än Warnings med Application Insights, så för att fixa det har jag modifierat min appsettings.json med följande kod

```json
"LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
```

Jag har även valt att logga när min applikation startas och det gör jag genom att ha modifierat min Main metod en aning. 

```csharp
    public static void Main(string[] args)
        {
            var host = CreateHostBuilder(args).Build();
            var logger = host.Services.GetRequiredService<ILogger<Program>>();
            
            try
            {    
                logger.LogInformation("The application has started");
                host.Run();
            }
            catch (System.Exception ex)
            {
                logger.LogCritical(ex, ex.Message);
            }
            
        }
```

Normalt när man skapar en asp.net core web applikation så ser koden ut så här 

```csharp
CreateHostBuilder(args).Build().Run();
```

Jag har skapat en variabel vid namn host som jag sedan tilldelar de första stegen, men väljer att bryta ut .Run för att kunna köra den senare. 
Sen har jag skapat en variabel logger som jag sen tilldelar host med Services och GetRequiredService och därefter ILogger

Sen börjar jag med att logga att applikationen har startat och därefter kör jag host.Run. 

## Funderar på hur loggning av din applikation kan avhjälpa säkerhetsproblem i din applikation

Kolla om det går att skriva någonting om inputvalidering här!!

## Förklara dina queries, vad gör dom? Varför är denna data som tas fram intressant?

lorem

