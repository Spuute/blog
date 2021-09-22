---
layout: post
title:  "Databaser i molnet"
---

## Inledning

Denna uppgift har jag valt att gå på guld steget direkt. Jag gjorde uppgiften tillsammans med Jonas. Jag har tidigare enbart jobbat med SQL databaser och aldrig ens kollat på något annat än det, men jag ville utmana mig själv lite den här gången och valde att gå vidare denna uppgift med CosmosDB. 

Det står i uppgiftsbeskrivningen att man ska lägga till automatisk deploy till azure med github actions, men jag har valt att göra detta "manuellt" via actions för att jag vill testa hur det fungerar att göra med workflow_dispatch istället, samt att jag inte vill råka deploya kod som inte går att bygga. Ska man göra detta med automatisk deploy tycker jag att man bör skapa en CI pipeline så man vet att det inte kommer att deployas någon kod som inte bygger, men jag hann inte riktigt så långt. 

## Beskriv kort applikationen, vad gör den?

Tanken bakom den är att det ska gå att spara och hämta recept till databasen. Just nu är den väldigt simpel och innehåller enbart namn på recept, men det kan byggas vidare på för att lägga till ingredienser, steg med mera.
Min applikation innehåller 3st functions:

* [FunctionName("dish")] - GET Request som hämtar ut ett recept ur databasen baserat på namn.
* [FunctionName("dishes")] - GET Request som hämtar ut alla sparade recept ur databasen.
* [FunctionName("AddDish")] - POST Request som sparar ett recept till databasen. 

Modellen för recept ligger under foldern Models och där Recipe.cs

## Beskriv koden

För att inte göra blogginlägget så långt så kommer jag bara att gå igenom en av mina functions.

```csharp
        [FunctionName("dish")] 
        public static async Task<IActionResult> GetDish(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "recipe")] HttpRequest req,
        [CosmosDB(
        databaseName: "mydb",
        collectionName: "myfirstcontainer",
        ConnectionStringSetting = "CosmosDbConnectionString")] DocumentClient client,
        ILogger log)
```
Jag börjar här med att välja att döpa denna function till dish. Detta är en GET request som ligger under route /api/recipe
Därefter kommer namnet på min databas, vilken container det är samt min connectionstring som parameter, och det är väl i princip det som är nytt från föregående lektion och det är alltså min instans till databasen. 

```csharp
var searchterm = req.Query["searchterm"];
            if (string.IsNullOrWhiteSpace(searchterm))
            {
                return (ActionResult)new NotFoundResult();
            }
```

Här spar jag söktermen från queryn till en variabel och börjar med att kolla så det inte är tomt, om det skulle vara tomt retunerar den ett NotFoundResult, annars går den vidare

```csharp
 Uri collectionUri = UriFactory.CreateDocumentCollectionUri("mydb", "myfirstcontainer");

            log.LogInformation($"Searching for: {searchterm}");

            IDocumentQuery<Recipe> query = client.CreateDocumentQuery<Recipe>(collectionUri, new FeedOptions { EnableCrossPartitionQuery = true })
                .Where(p => p.Name.Contains(searchterm))
                .AsDocumentQuery();

            while (query.HasMoreResults)
            {
                foreach (Recipe result in await query.ExecuteNextAsync())
                {
                    log.LogInformation(result.Name);
                }
            }
            return new OkObjectResult(query);
        }
```

Jag lägger med mydb och myfristcontainer som parametrar till en UriFactoory vilket skapar en collection link och tilldelas till collectionUri. Därefter loggar jag söktermen.
Nästa kodstycke med IDocumentQuery skapar en query mot cosmosDB och tar min collectionUri samt ett feedoption som parameter, därefter använder jag linq för att välja att hämta de objekt från databasen som innehåller namnet som anges i söktermen. Ex.
Söker jag på spaghetti så kommer alla recept som innehåller order spaghetti att visas. Denna körs så länge som det finns recept att visa.


## Hur har du fått den att köra i Azure functions?

Jag började med att skapa en ny resource group för denna lektion. I den skapade jag en ny function, därefter skapade jag ett nytt repo på github för denna uppgift och clonade ner repot och öppnade med VS Code. 
Jag använder mig av Azure Functions extension vilket jag använde för att skapa en ny function med http trigger. 

Därefter gjorde jag precis som i föregående uppgift, skapade en .github folder som innehåller en workflows folder som i sin tur innehåller en .yaml workflow fil som jag döpt till cdpipeline. 
Det är via den som jag deployar min function till azure, men denna gången testade jag att använda 
```
on: 
    workflow_dispatch:
```
vilket innebär att jag manuellt går in och kör min action för att deploya till azure. Jag ville testa och se hur det fungerade att göra på detta sätt istället för att ha den vid push. 
Jag har en tanke om att lägga till en CI pipeline som tidigare lektioner. 

## Hur har du tänkt kring uppdatering av databasen ifall scheman ändras? Migrations?

Eftersom jag använder mig av CosmosDB som är en NoSQL Dokument databas så är jag inte speciellt bekymrad om jag skulle behöva skala upp, då kan jag lägga till mer kapacitet genom horisontell skalning genom att öppna en till server, detta är speciellt billigt idag med molnlösningar. 
Om vi tar en relations databas som MsSQL så blir det mycket dyrare för ska man köra en sådan är det ett måste att man kör hela sin databas på en och samma server, man kan komma ifrån det genom att ha kopior av databasen på andra servrar som ständigt hämtar data från "huvuddatabasen" men det finns en risk för förlust av data om huvudbasen skulle gå ner en stund, även om det är bättre än att man står utan. Ska man skala upp en relationsdatabas blir det dyrare då man måste skala den vertikalt, alltså utöka kapaciteten på nuvarande server vilket kan bli väldigt mycket dyrare än att köpa en till liten server. 

BILD

## Vad skulle det kosta att driva detta?

Nu kan jag inte direkt påstå att jag har kunskap eller erfarenhet av hur mycket som behövs för varken stor eller liten skala, så dessa beräkningar får tas med en liten nypa salt när det kommer till kapacitet. 

### Liten skala

Om jag ska beräkna kostnad för denna CosmosDB serverless i en liten skala så väljer jag 1 miljon RUs och 10gb Transactional Storage. Då hamnar jag på en total månadskostnad på $2.78

![alt text](https://github.com/Spuute/blog/blob/main/img/liten.png "liten")

### Stor skala

Om jag istället beräknar kostnaden för denna CosmosDB serverless i en stor skala så krämar jag på med 1 miljard RUs och 500gb Transactional Storage, då hamnra jag istället på en månadskostnad på $408.00

![alt text](https://github.com/Spuute/blog/blob/main/img/stor.png?raw=true "Stor")


## Referenslänkar

* Horizontal vs Vertical scaling - https://www.cloudzero.com/blog/horizontal-vs-vertical-scaling
* NoSQL vs Relational databases - https://www.mongodb.com/scale/nosql-vs-relational-databases
* 5 reasons RDBMS aren´t working - https://www.marklogic.com/blog/relational-databases-scale/
* Azure price calculator - https://azure.microsoft.com/en-us/pricing/calculator/
