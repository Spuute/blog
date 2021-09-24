## Inledning

lorem

## Beskriv kort applikationen, vad gör den?

Då vi föregående lektion byggde en databas så valde jag då att spara och hämta recept (maträtter) från databasen, så denna lektion har jag byggt en razorpage applikation 
för detta och snurrat upp den i Azure App Service.

## Beskriv koden

Det är en helt vanlig Razor Page Web Applikation som jag har skapat upp två sidor till, en addDish för att lägga till recept till databasen och en dishes för att 
hämta alla sparade recept från databasen och presentera detta i varsitt kort. 

Jag skapade som vanligt upp projektet via kommandotolken med dotnet new webapp -name app.

Eftersom jag använder VS Code så har man inte samma "smidighet" som med Visual Studio där man bara kan högerklicka och skapa en ny razorpage. Om man skulle skapa en 
ny fil *.cshtml.cs och *.cshtml så kommer dom sakna koppling till applikationen och man kommer inte kunna nå dom när man kör applikationen. 
Det jag har gjort är att skapa dessa sidor addDish och dishes via kommandotolken med dotnet new page -n addDish -na MITT_NAMESPACE -o Pages

-n för att ge ett namn till sidan
-na för att tilldela vilket namespace den ska använda
-o för att säga vilken mapp den ska ligga i

Razorpages fungerar som så att när man går in på en sida, som exempelvis .../dishes så kommer metoden OnGet() att köras, så det jag har gjort då är att lägga in kod för
att göra ett anrop mot den funktion jag skapade förra lektionen. Jag har installerat Nugetpaketet RestSharp för jag tycker att det var väldigt smidigt att jobba med
under tidigare kurser,

```csharp
 public void OnGet()
        {
            var client = new RestClient("https://restapi-cosmosdb.azurewebsites.net/api");
            var request = new RestRequest($"/recipe?", DataFormat.Json);

            var response = client.Execute<List<Recipe>>(request);
            Recipes = response.Data;
        }
```

Det man gör här är att man först sätter upp sin RestClient med url till mitt api, därefter vid request så säger man vilken endpoint det gäller och att man vill få tillbaka
Json. Response tilldelas det man får när man kör sin request. Därefter säger jag att listan med recept (Recipes) ska innehålla response.Data vilket är de man får tillbaka
vid sitt anrop. 

För att presentera detta har jag i dishes.cshtml följande kod

```csharp
@page
@model dishesModel
@{
    ViewData["Title"] = "Dishes";
}
<h3>Sparade recept</h3>


@foreach (var recipe in Model.Recipes) 
{
    <div class="card-deck">
        <div class="card text-center border-primary mb-3" style="width: 18rem;">
            <img class="card-img-top" src="..." alt="Bild på maträtt">
            <div class="card-body">
               <h5 class="card-title">@Html.DisplayFor(m => aa.Name)</h5>
               <p class="card-text">Här kommer beskrivning synas när applikationen utökas</p>
               <a href="#" class="btn btn-primary">Läs mer</a>
            </div>
        </div>
    </div>
}
```

Jag har en vanlig h3 titel som säger Sparade recept, men sen komm vi in på det intressanta. Eftersom detta är razorpages så kan man mixa c# kod med html kod, och det jag
gör är att jag med en foreach loop, loopar igenom alla resultat man har fått från sitt anrop, och för varje resultat (recept) så skapar jag ett kort som jag lägger in namnet 
från maträtten till, under <h5> så har jag lagt till @Html.DisplayFor(m => recipe.Name). Jag använder mig av bootstrap för detta kort, därav klassnamnen. Det går väldigt
snabbt att använda sig av bootstrap då det finns med från start med razorpages, så det är bara att gå in på dokumentationen för bootstrap och kolla vad man gillar och 
sen köra på. 

## Hur har du fått den att köra i Azure App Serive?

Jag började med att via portal.azure.com skapa mig en ny App Service, inga större konstigheter där då det bara är att hitta den sen trycka sig vidare så är det att som att
skapa en function, resource group eller databas. Man fyller i det som ska fyllas i och kör en create. 

För att köra den i App Service började jag med att skapa en Dockerfile till projektet, och det gjorde jag likt föregående lektion så den kommer jag inte gå in någonting 
på i detta inlägg, men något som jag däremot känner för att påpeka är ett problem som jag stötte på med min Dockerfile som höll på att göra mig riktigt gråhårig. När jag
skapade min webapp döpte jag den till app, och jag återanvända min dockerfile från föregående lektion och där har jag användt mig av /app som WORKDIR. 
Min Dockerfile gick att bygga och köra, men jag kom inte in på localhost, och satt väldigt länge och provade fram och tillbaka med att konfigurera om min Dockerfile, 
innan det slutligen slog mig att det kanske blir någon form av konflikt när man använder samma namn på WORKDIR som man har på sitt projekt, så jag bytte i all
trötthet och hast namn på WORKDIR till ee och då fungerade det magiskt. Detta är någonting som jag lärt mig en läxa av och kommer vara väldigt noga med att 
kontrollera i framtiden. 

Därefter följade jag en guide för att publicera detta från VS Code: LÄNKLÄNKLÄNKLÄNKLÄNK 

## Vad skulle det kosta att driva detta, stort samt litet

När jag skapade upp detta så var jag lite snabb så jag valde Standard TIER Istället för Basic, så jag kommer utgå denna kontroll efter det jag skapade upp. 

### Liten skala

Om jag väljer Den lägsta instancen 1,75gb ram 50gb storage samt 1 core så hamnar månsdakostnaden på $69.35

### Stor skala

Om man istället väljer den största instansen på 7gb ram, 50gb storage samt 4 cores så hamnar månadskostnaden på $277.40

Om jag ska dra någon slutsats av denna priskontroll vi gjort föregående lektion samt denna så känns det onekligen som att man behöver vara ganska insatt i hur mycket 
och kraftfull hårdvara som kommer att behövas när man ska snurra upp någonting, och det känns väl just nu lite som en halv vetenskap bara det. 
