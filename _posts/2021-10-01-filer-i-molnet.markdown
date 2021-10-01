---
layout: post
title:  "Filer i molnet"
---

## Inledning

Denna lektion valde jag och Jonas att gå på Silver steget och det vi denna lektion ska skapa är två applikationer, en konsollapplikation som ska kunna spara filer till Azure Blob Storage samt en Razor Page applikation som ska läsa in alla bilder från vår container. 

Jag påbörjade projektet genom att skapa dessa två applikationer genom konsollen som vanligt, därefter skapade jag en solution fil och kopplade samman dessa applikationer med den genom 
```
dotnet sln add 
```

## Diagram för data

lorem

Bilden här illustrerar när jag kör applikationerna "offline" via localhost. 

## Beskriv koden

### Konsollapplikationen

Jag började med att skapa en Services mapp, därefter skapade jag en BlobManager klass och placerade den där. Där i har jag en metod UploadBlob() som innehåller koden för att ladda upp en bild (blob) till en container i mitt Azure Storage. 

```csharp
public static void UploadBlob()
        {
            var filePath = @"C:\test-image.jpg";
            var connectionString = "DefaultEndpointsProtocol=https;AccountName=lesson07gold;AccouaaaaaaaaaaaMqRSLUBsibHK/eH5FAV1F24pghALDnTHQh1DOvIovgKm+mnXZm/SC43kGhgkqOp6Cqw==;EndpointSuffix=core.windows.net";

            BlobClient blobClient = new BlobClient(connectionString: connectionString, blobContainerName: "imagesba0bde9d-ae96-4310-a026-bd3d81af150d", blobName: $"image{Guid.NewGuid().ToString()}.jpg");

            blobClient.Upload(filePath);

            var blobUri = blobClient.Uri.AbsoluteUri;
            Console.WriteLine(blobUri);
        }
```
Jag följer inte direkt best practice när det kommer till att ha min connectionstring helt öppet i koden, men det är enbart i konsollapplikationen som den ligger öppet, i min webapp så ligger den i appsettings.json. 

Här har jag hårdkodat in en filepath till en testbild som jag har på min hårddisk. 

Jag börjar sedan med att skapa en ny isntans av BlobClient, och parametrarna den tar är:
* connectionString - Min Connectionstring jag hämtat från Azure
* blobContainerName - Namnet på den container jag vill ladda upp mina blobs till. 
* blobName - Här kan jag välja att döpa filerna jag laddar upp, och som ni kan se behöver det alltså inte vara samma filnamn som filen har på min dator. Här har jag i exemplet ovan valt att döpa alla bilder jag laddar upp till image följt av ett Guid vilket försäkrar mig om att det inte kommer bli några krockar där man försöker ladda upp en bild som har ett filnamn som redan existerar där. 

Därefter anropar jag min instans av blobClient med .Uri.AbsoluteUri för att tilldela variabeln blobUri med uri:n till den bild jag precis har laddat upp, och därefter printar jag den till konsollen. 

### Webapp

Jag skapade en ny Razor page genom CLI med följande kommando
```
dotnet new page -n Images -na webbapp.Pages -o Pages
```

* -n - här väljer man vilket namn sidan ska ha
* -na - här anger man vilket namespace den ska ligga i
* -o - här anger man vilken mapp den ska hamna i, och jag har valt Pages då alla andra sidor ligger där. 

När denna är skapad så börjar jag med att öppna Images.cshtml.cs och skriver följande kod

```csharp
public class ImagesModel : PageModel
    {
        public IConfiguration _config { get; set; }
        
        public ImagesModel(IConfiguration config)
        {
            _config = config;
        }
        public List<Uri> allBlobs = new List<Uri>();
        public async Task<IActionResult> OnGet()
        {
            string connectionString = _config.GetConnectionString("Default");
            var storageAccount = CloudStorageAccount.Parse(connectionString);
            var _blobClient = storageAccount.CreateCloudBlobClient();
            var _blobContainer = _blobClient.GetContainerReference("imagesba0bde9d-ae96-4310-a026-bd3d81af150d");

            BlobContinuationToken blobContinuationToken = null;

            do
            {
                var response = await _blobContainer.ListBlobsSegmentedAsync(blobContinuationToken);
                foreach (IListBlobItem blob in response.Results)
                {
                    if (blob.GetType() == typeof(CloudBlockBlob))
                        allBlobs.Add(blob.Uri);
                }
                blobContinuationToken = response.ContinuationToken;
            } while (blobContinuationToken != null);

            return Page();
        }
    }
```

Jag börjar med att injecera IConfiguration för att kunna använda mig av _config.GetConnectionString("Default") då jag valt att för denna applikation spara min connectionstring i appsettings.json. 
Därefter skapar jag en lista med Uri´s och döper den till allBlobs. 

Sen kommer min connectionstring och sen storageAcount som hämtas genom min connectionstring. 
Därefter skapar jag en ny blocClient baserat på mitt storageaccount, och efter det anger jag vilken container det är jag vill använda. 

Därefter kommer en do while loop som körs så länge blobcontinuationToken inte är null. 

Jag loopar igenom alla blob items i det response jag får genom ListBlobsSegmentAsync och därefter kommer en ifsats som kollar så blob.GetType är av typen CloudBlockBlob, om den är det så lägger jag till uri:n till min lista med Uri:s som jag skapde tidigare. 
Därefter retunerar jag sidan. 

I min Images.cshtml är koden så simpel den bara kan bli. 

```csharp
@foreach (var uri in Model.allBlobs)
{
    <img src="@uri" />
}
```
Jag loopar helt enkelt bara igenom alla uri:s från listan jag skapade och därefter lägger jag in dom som en bild, och på detta vis så kommer alla bilder i min blob att visas en efter en. Detta kan man såklart göra snyggare om man nu ska göra detta på riktigt, men av tidsskäl då dessa uppgifter ska gå fort så valde jag att bara ha det så simpelt det kan vara men ändå ge det resultat som uppgiften kräver. 

BILD

## Vad skulle det kosta att driva en applikation som sparar och läser filer i Azure?

När jag kollar på Azure Price Calculator och väljer 
Storage Account - Block blob storage, premium tier, north europe samt en kapacitet på 1000gb 
samt en App Service North europe, linux Premium V3 tier så hamnar jag på en total månadskostnad av $303.47

## Vad gör Microsoft för att säkra min blob data?

Huvudsakligen är det tre punkter 

* Data i vila
* Data vid överföring
* Azure key management 

Azure krypterar data som ligger på sina servrar vilket skyddar den data man har lagrad mot oönskat intrång. Denna typ av kryptering skyddar de innehåll man har på hårddisken om den skulle gå sönder eller bli stulen. 
Om man inte gör några ändringar så är all data som skrivs till Azure Blob Storage krypterad när den sparas till disk och dekrypterad när man använder bland annat Azure Key Vault.

Data vid överföring innebär när data laddas upp från en datorn till bland annat molnlagringen. Denna data måste måste skyddas mot diverse hot och det är något som Microsoft löser genom att alla sessioner med Azure tjänster och dess datacenter  är säkrade genom TLS protokoll och PFS. 

Man kan säga att basen för allting är Azure Key vault som sparar alla nycklar som exempelvis connectionstring etc. 