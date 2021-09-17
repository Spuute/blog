---
layout: post
title:  "Serverless"
---

## Vad är serverless och Function As A Service (FaaS)?

Serverless är en molnbaserad utvecklingsmodell som gör att vi utvecklare kan fokusera på att bygga och köra applikationer utan att behöva tänka på att hantera servrar.

Serverless är inte vad namnet antyder - Att det inte finns några fysiska servrar som man använder. Dessa servrar finns självklart kvar, men tanken bakom serverless är att man inte ska behöva fundera över servrarna i så hög utsträckning som man tidigare har gjort. Man ska inte behöva tänka på om operativsystemet på servern är uppdaterat, man ska inte behöver fundera på om den virtual machine man kör på är uppdaterad etc. 

När det kommer till serverless så faller det ofta in i två olika former BaaS (Backend-as-a-Service) och FaaS(Function-as-a-Service).

### FaaS

Function-as-a-Service är en event driven exekveringsmodell där utvecklarna skriver logik som distribueras i containrar som körs på begäran. När man använder FaaS för att komma åt applikationer som körs serverless genom API:er som leverantöres av FaaS tjänsten hanterar via en API Gateway.


## Beskriv din kalkylator

Det är en väldigt väldigt simpel kalkylator som används som ett api.

### Koden

```csharp
   public static class GitHubMonitorApp
    {
        [FunctionName("GitHubMonitorApp")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("Our GitHub Monitor pricessed an action.");

            string first = req.Query["first"];
            string second = req.Query["second"];

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);


            string responseMessage = string.Empty;
            try
            {
                int firstNumber = int.Parse(first);
                int secondNumber = int.Parse(second);
                int calcResult = firstNumber + secondNumber;

                responseMessage = $"{first} + {second} equals {calcResult}.";
            }
            catch
            {
                responseMessage = $"You did not enter valid integers";
            }
            return new OkObjectResult(responseMessage);
        }
    }
```

Det är i princip koden man får som bas när man skapar en ny Azure Function i VS Code, men jag tänkte att jag kunde köra med den då uppgiften var att skapa en väldigt simpel kalkylator. 
Det jag har gjort är att lägga till en try catch för att hantera ifall användaren försöker addera två bokstäver eller tecken med varandra eller en siffra vilket då ger meddelandet "You did not enter valid integers", om användaren däremot skickar in två heltal så kommer man då få tillbaka exempelvis " 5 + 4 equals 9 ". 

Eftersom man skickar strängar som parametrar så var jag tvungen att parsa dom till en int innan jag kunde addera dom med varandra. 

### Hur har du fått den att köra i Azure Functions?

Jag har deployat min function direkt från github genom ett CLI skript. Men jag testade även att skapa allting via Azure´s sida. Jag tyckte att det kändes helt okej att navigera runt på sidan, men har senaste tiden fått en liten kärlek till att försöka sköta så mycket som möjligt via CLI, och jag tycker personligen att det gick smidigare att skapa allting via ett skript i CLI:n

Man får börja med att authentisera git leveranser från sitt azure konto 
```
az functionapp deployment source update-token
--git-token [HÄR LÄGGER MAN IN SIN TOKEN MAN GENERERAT FRÅN GITHUB]
```

Det man gör sen är att skapa en resursgrupp 
```
az group create --name [Namnet man vill ha] --location [Ex. europenorth]
```

Nästa steg blir att skapa ett storage konto i den resursgrupp man precis har skapat
```
az storage account create --name [DET NAMN MAN VILL HA] --location [Ex. europenorth] --resource-group [NAMNET MAN VALDE I FÖREGÅENDE STEG] --sku Standard_LRS
```

Därefter skapar man en functions att baserat på sitt github repo
```
az functionapp create --name [VALFRITT NAMN] --storage-account [NAMNET MAN ANGAV PÅ FÖREGÅENDE STEG] --consumption-plan-region [northeurope] 
--resource-group [NAMNET MAN ANGAV I TIDIGARE STEG] --deployment-source-url [URL TILL GITHUB REPO] --deployment-source-branch [master] 
--functions-version 2
```

När detta steget är gjort är det dags att deploy:a och det gjorde jag genom
```
func azure functionapp publish NAMNET_JAG_ANGAV_TIDIGARE --force --nozip
```

### Hur har du testat applikationen?

Jag började med att testa köra applikationen lokalt genom postman 

Därefter fortsatte jag att använda postman då jag är bekväm med det programmet sedan tidigare kurser. 

### Vilka säkerhetshot finns till en applikation som denna? 

Jag tänkte ta upp lite om "Broken Authetication"

Då detta är en serverless function så körs den i stateless containrar vilket innebär att det inte är som traditionella applikationer inte är ett stort flöde som ligger på en server. Om vi har en "riktig" applikation så kommer den antagligen innehålla hundratals om inte tusentals olika funktioner som alla körs separat och där alla har sitt eget syfte och triggas från olika event. En hacker kommer försöka ta sig in genom en resurs som exempelvis ett öppet API eller en publik molnlagring.

### Har du gjort något för att säkra dig mot dessa?

I min applikation har jag faktiskt inte tagit detta i åtanke då detta enbart var en simpel kalkylator som skapats i "känna på azure" syfte

### Referenslänkar
https://www.redhat.com/en/topics/cloud-native-apps/what-is-serverless
https://docs.microsoft.com/en-us/azure/azure-functions/scripts/functions-cli-create-function-app-github-continuous
