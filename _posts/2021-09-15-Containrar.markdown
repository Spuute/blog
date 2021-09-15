
Dagens inlägg kommer att handla om containrar och orkestrering och det är en del av lektionen från Måndag 13 / 9. 
Vi kommer under denna kurs primärt att använda oss av docker samt github. 

Detta inlägg kommer precis som tidigare inlägg att ha frågorna vi ska besvara samt mitt svar under.

**OBS**
När ni kommer längre ner till delar där jag använder syntax highlighting så har jag varit tvungen att lägga till ett mellanslag mellan de dubbla måsvingarna för att få koden att synas i denna markdown fil.

### Vad har ni installerat lokalt / github?

Det jag har installera lokalt på min burk är Docker desktop samt Github desktop, jag använder dock inte github desktop så mycket längre då jag försöker förflytta mig över till commandline. 

När det kommer till uppgiften så har jag lokalt skapat en container och pushat den till github container registry, och som uppgiften lyder så har jag därefter även testat att tanka hem den och köra den vilket fungerar smärtfritt. 

### Hur har du fått applikationen att köra i en container?

Efter att ha klonat ner repot som uppgiftsbeskrivningen angav "SimpleWebHalloWorld" så skapade jag tre filer,
```
touch Dockerfile
touch .dockerignore
touch docker-compose.yml
```

Därefter så testade jag så att applikationen kunde bygga för att vara säker på det innan jag påbörjade min dockerfile, så jag körde dotnet build, och därefter testade jag köra det med dotnet run. Men där fick jag problem då det stod att jag inte hade rätt sdk. Jag kör med mac och har senaste versionen installerad men var nu tvungen att leta upp en äldre version för att installera den. När jag väl fått allting att bygga och köra så påbörjade jag min dockerfile med att leta efter rätt base image och därefter tog jag min dockerfile stegvis. 

### Beskriv din Dockerfile

```Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env
WORKDIR /app
COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o out
FROM mcr.microsoft.com/dotnet/aspnet:3.1 
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "SimpleWebHalloWorld.dll"]
```

* FROM - Här anger man vilket image man vill använda, en parentimage innehåller filer och mappar som man sedan bygger sitt eget image ovanpå i detta fall dotnet SDK som innehåller flera utvecklingsverktyg. 
* WORKDIR - Här specificerar man sin "working directory". När man har angett detta så kommer alla kommandon nedan att ske inuti denna mapp och i och med detta kan man använda sig av en relativ sökväg. 
* COPY - Används för att kopiera filer och mappar, i mitt fall innebär . . att jag kopierar allting från nuvarande folder, vilket i mitt fall är app som jag angav tidigare
* RUN - Används för att köra kommandon i operativsystem så som linux, windows etc. RUN är en build-time instruktion som enbart körs när man bygger ett image. Jag börjar med att köra en dotnet restore 
* RUN - Därefter kör jag dotnet public -c Release -o out
* FROM - Här använder jag mig av ett annat baseimage för asp.net 3.1 vilket projektet använder och här behöver jag inte längre alla utvecklingsverktyg som medföljer i SDK:n
* WORKDIR - Samma som ovan
* COPY - Här kopierar jag från min byggmiljö som jag angav i första steget
* ENTRYPOINT - Används för att specificera ett kommando som man vill köra när vi startar en container. Man kan även använda CMD, men det kommandot kan enklare skrivas över än ENTRYPOINT. 

### Beskriv din github pipeline

```
    name: Publish Docker Image To Github Registry

    on: [push]

    jobs:
    build:
        runs-on: ubuntu-latest
        steps:
        - name: Checkout repo
            uses: actions/checkout@v2

        - name: Docker meta
            id: docker_meta
            uses: crazy-max/ghaction-docker-meta@v1
            with:
            images: ghcr.io/spuute/test
            tag-sha: true
            tag-edge: false
            tag-latest: true

        - name: Setup QEMU
            uses: docker/setup-qemu-action@v1

        - name: Setup Docker buildx
            uses: docker/setup-buildx-action@v1

        - name: Login to GHCR
            uses: docker/login-action@v1
            with: 
            registry: ghcr.io
            username: ${ { secrets.USERNAME }}
            password: ${ { secrets.PASSWORD }}

        - name: Build image
            uses: docker/build-push-action@v2
            with:
            tags: ${ { steps.docker_meta.outputs.tags }}
            file: ./Dockerfile
            
        - name: Push image
            uses: docker/build-push-action@v2
            with:
            push: true
            tags: ${ { steps.docker_meta.outputs.tags }}
            file: ./Dockerfile
```

Min pipeline ser ut som så att jag börjar med att tilldela den ett namn, samt säger att den ska köras vid varje push, inga konstigheter från föregående lektions övning. 

Sen börjar jag med att checka ut koden från mitt repository med actions/checkout@v2

docker meta är en action som extraherar metadata från git referenser och events och jag har förstått att detta kan vara väldigt användbart om man använder det tillsammans med just docker build push action.

```
Dokumentation: https://github.com/docker/metadata-action
```

Detta action är någonting som jag inte satt mig in i speciellt mycket, men efter att ha kollat dokumentationen så känns det som ett action jag kommer att kolla lite närmare på vid kommande projekt och jag valde att ta med det nu för att testa och se lite hur det fungerar. 

De kommande två stegen docker/setup-qwmu-action@v1 samt docker/setup-buildx-action@v1 sätter upp min docker byggmiljö. 

SETUP QEMU - Är ingenting som behövs men av samma anledning som docker meta tänkte jag att jag skulle testa använda denna då den i framtiden kan vara användbar om man vill lägga till emulation för att ha möjligheten att bygga mot fler plattformar

SETUP BUILDX - Är inte heller nödvändig. Det den gör är att den skapar och startar docker-container builder drivern.

Därefter är det dags att fixa inloggning till githubs container registry. här är jag tvungen att specificera githubs registry med registry: ghcr.io, om jag hoppar över den så kommer den automatiskt att försöka logga in mig till dockers container registry och detta eftersom det är docker som har utvecklat detta action. Här använder jag mig av två secrets, men jag skriver mer om dessa under nästa punkt. 

Därefter bygger jag min docker image med docker/build-push-action@v2 och taggarna jag använder 
```
tags: ${ { steps.docker_meta.outputs.tags }}
```

kommer från docker meta steget som jag förklarade lite längre upp. 

Detta steg pushar inte vårat image till container registryn utan det gör jag i nästa steg med i princip identisk kod, om man vill kan man här slänge in fler steg att utför efter man har byggt sitt image och det kan exempelvis vara för att kolla om det finns några sårbarheter. 

Det som jag nu lägger till i koden under with: är 
```
push: true
``` 


### Hur har du gjort med secrets?

Jag använder mig av två secrets, en för användarnamn och en för min PAT (Personal Access Token). När jag satt och skapade filen så hade jag i minnet att man kunde använda något kommando för att komma åt sitt användarnamn, men jag kom inte på vad det var så jag skapade en secret för detta. Nu när jag sitter och skriver denna blogpost kom jag på att jag skulle kunna ha använt mig utav: 

```
username: ${ { github.actor }}
```
istället för: 
```
username: ${ { secrets.USERNAME }}
```

### Referenslänkar
* Docker build push action - https://github.com/docker/build-push-action
* Docker setup qemu action - https://github.com/docker/setup-qemu-action
* Docker buildx action - https://github.com/docker/setup-buildx-action
* Docker metadata action - https://github.com/docker/metadata-action
* Youtube video som visar hur man bygger och pushar till ghcr - https://www.youtube.com/watch?v=FYIRvqdP3pQ
