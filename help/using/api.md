---
title: "[!DNL Asset Compute Service] HTTP-API"
description: "[!DNL Asset Compute Service] HTTP-API om aangepaste toepassingen te maken."
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '2862'
ht-degree: 0%

---

# [!DNL Asset Compute Service] HTTP-API {#asset-compute-http-api}

Het gebruik van de API is beperkt tot ontwikkelingsdoeleinden. De API wordt als context verstrekt wanneer het ontwikkelen van douanetoepassingen. [!DNL Adobe Experience Manager] als [!DNL Cloud Service] gebruikt de API om de verwerkingsgegevens door te geven aan een aangepaste toepassing. Zie voor meer informatie [Middelenmicroservices en verwerkingsprofielen gebruiken](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] is alleen beschikbaar voor gebruik met [!DNL Experience Manager] als [!DNL Cloud Service].

Elke client van de [!DNL Asset Compute Service] HTTP API moet deze stroom op hoog niveau volgen:

1. Een client is ingericht als een [!DNL Adobe Developer Console] in een IMS-organisatie. Elke afzonderlijke client (systeem of omgeving) vereist een eigen afzonderlijk project om de gegevensstroom van de gebeurtenis te scheiden.

1. Een cliënt produceert een toegangstoken voor de technische rekening gebruikend [JWT-verificatie (serviceaccount)](https://developer.adobe.com/developer-console/docs/guides/).

1. Een clientaanroep [`/register`](#register) slechts eenmaal om de journaal-URL op te halen.

1. Een clientaanroep [`/process`](#process-request) voor elk element waarvoor het uitvoeringen wil genereren. De vraag is asynchroon.

1. Een klant pollt regelmatig het dagboek naar [ontvangen, gebeurtenissen](#asynchronous-events). Er worden gebeurtenissen voor elke aangevraagde uitvoering ontvangen wanneer de vertoning met succes is verwerkt (`rendition_created` gebeurtenistype) of als er een fout is (`rendition_failed` gebeurtenistype).

De [adobe-asset-compute-client](https://github.com/adobe/asset-compute-client) maakt het gemakkelijk om API in code Node.js te gebruiken.

## Verificatie en autorisatie {#authentication-and-authorization}

Voor alle API&#39;s is verificatie van toegangstoken vereist. De verzoeken moeten de volgende kopballen plaatsen:

1. `Authorization` header met token aan toonder, de token van een technische account, ontvangen via [JWT-uitwisseling](https://developer.adobe.com/developer-console/docs/guides/) uit Adobe Developer Console-project. De [bereik](#scopes) worden hieronder beschreven.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` met de IMS-organisatie-id.

1. `x-api-key` met de client-id van de [!DNL Adobe Developers Console] project.

### Segmenten {#scopes}

Verzeker het volgende werkingsgebied voor het toegangstoken:

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Deze gebieden vereisen [!DNL Adobe Developer Console] project waarop moet worden geabonneerd `Asset Compute`, `I/O Events`, en `I/O Management API` diensten. De uitsplitsing van individuele scènes is:

* Basis
   * bereik: `openid,AdobeID`

* Asset compute
   * metareaal: `asset_compute_meta`
   * bereik: `asset_compute,read_organizations`

* Adobe [!DNL `I/O Events`]
   * metareaal: `event_receiver_api`
   * bereik: `event_receiver,event_receiver_api`

* Adobe [!DNL `I/O Management API`]
   * metareaal: `ent_adobeio_sdk`
   * bereik: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registratie {#register}

Elke client van de [!DNL Asset Compute service] - een unieke [!DNL Adobe Developer Console] project geabonneerd op de dienst - most [registreren](#register-request) voordat u verzoeken om verwerking indient. De registratiestap retourneert het unieke gebeurtenisjournaal dat vereist is om de asynchrone gebeurtenissen van de uitvoering op te halen.

Aan het einde van de levenscyclus kan een client [unregister](#unregister-request).

### Aanvraag registreren {#register-request}

Deze API-aanroep stelt een [!DNL Asset Compute] en geeft de URL van het gebeurtenisjournaal weer. Dit proces is een epidemische bewerking en hoeft slechts één keer voor elke client te worden aangeroepen. Het kan opnieuw worden geroepen om het dagboek URL terug te winnen.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/register` |
| Koptekst `Authorization` | Alles [vergunninggerelateerde koppen](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel, door clients ingesteld voor een unieke end-to-end-id van de verwerkingsverzoeken in alle systemen. |
| Aanvragingsinstantie | Moet leeg zijn. |

### Reactie registreren {#register-response}

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Dezelfde waarden als bij `X-Request-Id` aanvraagkoptekst of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen, of steunverzoeken, of allebei. |
| Responsinstantie | Een JSON-object met `journal`, `ok`, of `requestId` velden. |

De HTTP-statuscodes zijn:

* **200 succesvol**: Wanneer de aanvraag is geslaagd. De `journal` URL ontvangt meldingen over de resultaten van de asynchrone verwerking die via `/process`. Waarschuwing van `rendition_created` gebeurtenissen na succesvolle voltooiing, of `rendition_failed` gebeurtenissen als het proces mislukt.

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 Niet-toegelaten**: treedt op wanneer de aanvraag niet geldig is [verificatie](#authentication-and-authorization). Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.

* **403 Niet toegestaan**: treedt op wanneer de aanvraag niet geldig is [autorisatie](#authentication-and-authorization). Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het Adobe Developer Console project (technische rekening) wordt niet ingetekend aan alle vereiste diensten.

* **429 Te veel verzoeken**: vindt plaats wanneer deze client of anders het systeem overlaadt. Clients moeten het opnieuw proberen met een [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff). Het lichaam is leeg.
* **4xx-fout**: Als er een andere clientfout is opgetreden en registratie is mislukt. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx-fout**: komt voor wanneer er een andere serverfout was en de registratie ontbrak. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### Aanvraag ongedaan maken {#unregister-request}

Met deze API-aanroep wordt de registratie van [!DNL Asset Compute] client. Nadat de registratie ongedaan is gemaakt, is het niet meer mogelijk om `/process`. Wanneer de API-aanroep voor een niet-geregistreerde client wordt gebruikt of een nog te registreren client een `404` fout.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/unregister` |
| Koptekst `Authorization` | Alles [vergunninggerelateerde koppen](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel. Clients kunnen deze instellen voor een unieke end-to-end-id van de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Leeg. |

### Reactie ongedaan maken {#unregister-response}

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Dezelfde waarden als bij `X-Request-Id` aanvraagkoptekst of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen of steunverzoeken. |
| Responsinstantie | Een JSON-object met `ok` en `requestId` velden. |

De statuscodes zijn:

* **200 succesvol**: vindt plaats wanneer de registratie en het journaal worden gevonden en verwijderd.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 Niet-toegelaten**: treedt op wanneer de aanvraag niet geldig is [verificatie](#authentication-and-authorization). Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.

* **403 Niet toegestaan**: treedt op wanneer de aanvraag niet geldig is [autorisatie](#authentication-and-authorization). Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het Adobe Developer Console project (technische rekening) wordt niet ingetekend aan alle vereiste diensten.

* **404 Niet gevonden**: Deze status wordt weergegeven wanneer de opgegeven gegevens niet zijn geregistreerd of ongeldig zijn.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 Te veel verzoeken**: vindt plaats wanneer het systeem wordt overbelast. Clients moeten het opnieuw proberen met een [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff). Het lichaam is leeg.

* **4xx-fout**: komt voor wanneer er een andere cliëntfout was en unregister ontbrak. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx-fout**: komt voor wanneer er een andere serverfout was en de registratie ontbrak. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## Procesaanvraag {#process-request}

De `process` bewerking verzendt een taak die een bronelement in meerdere uitvoeringen omzet, op basis van de instructies in de aanvraag. Meldingen over geslaagde voltooiing (gebeurtenistype) `rendition_created`) of eventuele fouten (gebeurtenistype) `rendition_failed`) worden verzonden naar een gebeurtenisjournaal dat moet worden opgehaald met [`/register`](#register) om het even welk aantal `/process` verzoeken. Onjuist gevormde verzoeken ontbreken onmiddellijk met een 400 foutencode.

Er wordt verwezen naar binaire getallen via URL&#39;s, zoals vooraf ondertekende URL&#39;s voor Amazon AWS S3 of Azure Blob Storage SAS-URL&#39;s. Wordt gebruikt voor het lezen van de `source` activa (`GET` URL&#39;s) en de vertoningen schrijven (`PUT` URL&#39;s). De klant is verantwoordelijk voor het genereren van deze vooraf ondertekende URL&#39;s.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/process` |
| MIME-type | `application/json` |
| Koptekst `Authorization` | Alles [vergunninggerelateerde koppen](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel. Clients kunnen een unieke end-to-end-id instellen om verwerkingsverzoeken in verschillende systemen bij te houden. |
| Aanvragingsinstantie | Deze moet de JSON-indeling van het procesverzoek hebben, zoals hieronder wordt beschreven. Er worden instructies gegeven over welk element moet worden verwerkt en welke uitvoeringen moeten worden gegenereerd. |

### Verzoek om verwerking JSON {#process-request-json}

De verzoekende instantie `/process` is een JSON-object met dit schema op hoog niveau:

```json
{
    "source": "",
    "renditions" : []
}
```

De beschikbare velden zijn:

| Naam | Type | Beschrijving | Voorbeeld |
|--------------|----------|-------------|---------|
| `source` | `string` | URL van het bronelement dat wordt verwerkt. Optioneel, op basis van de gevraagde renditie-indeling (bijvoorbeeld `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Beschrijf het bronelement dat wordt verwerkt. Zie beschrijving van [Source-objectvelden](#source-object-fields) hieronder. Optioneel op basis van de gevraagde renditie-indeling (bijvoorbeeld `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Uitvoeringen die moeten worden gegenereerd op basis van het bronbestand. Elk weergaveobject ondersteunt een [renderinstructie](#rendition-instructions). Vereist. | `[{ "target": "https://....", "fmt": "png" }]` |

De `source` kan `<string>` die als een URL wordt beschouwd of een `<object>` met een extra veld. De volgende varianten zijn vergelijkbaar:

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Source-objectvelden {#source-object-fields}

| Naam | Type | Beschrijving | Voorbeeld |
|-----------|----------|-------------|---------|
| `url` | `string` | URL van het bronelement dat moet worden verwerkt. Vereist. | `"http://example.com/image.jpg"` |
| `name` | `string` | Naam Source-elementbestand. Een bestandsextensie in de naam kan worden gebruikt als er geen MIME-type wordt gedetecteerd. De bestandsnaam die in het URL-pad is opgegeven, krijgt prioriteit. En de bestandsnaam in het dialoogvenster `content-disposition` header van de binaire bron. De standaardwaarde is &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Source-bestandsgrootte in bytes. Heeft voorrang op `content-length` header van de binaire bron. | `10234` |
| `mimetype` | `string` | MIME-type voor Source-elementbestand. Heeft voorrang op de `content-type` header van de binaire bron. | `"image/jpeg"` |

### Een complete `process` aanvraagvoorbeeld {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## Procesreactie {#process-response}

De `/process` het verzoek keert onmiddellijk met een succes of een mislukking terug die op de basisverzoekbevestiging wordt gebaseerd. De werkelijke verwerking van elementen gebeurt asynchroon.

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Dezelfde waarden als bij `X-Request-Id` aanvraagkoptekst of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen of steunverzoeken. |
| Responsinstantie | Een JSON-object met `ok` en `requestId` velden. |

Statuscodes:

* **200 succesvol**: Als het verzoek is verzonden. Response JSON omvat `"ok": true`:

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Ongeldig verzoek**: Als het verzoek bijvoorbeeld niet correct is gestructureerd, als het vereiste velden in de JSON-payload heeft. Response JSON omvat `"ok": false`:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 Niet-toegelaten**: Wanneer het verzoek niet geldig is [verificatie](#authentication-and-authorization). Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.
* **403 Niet toegestaan**: Wanneer het verzoek niet geldig is [autorisatie](#authentication-and-authorization). Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het Adobe Developer Console project (technische rekening) wordt niet ingetekend aan alle vereiste diensten.
* **429 Te veel verzoeken**: Komt voor wanneer het systeem overweldigd is, of toe te schrijven aan deze bepaalde cliënt of aan algemene vraag. De clients kunnen het opnieuw proberen met een [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff). Het lichaam is leeg.
* **4xx-fout**: Als er een andere clientfout is opgetreden. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx-fout**: Als er een andere serverfout is opgetreden. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

De meeste klanten zijn waarschijnlijk geneigd hetzelfde verzoek opnieuw uit te voeren met [exponentiële backoff](https://en.wikipedia.org/wiki/Exponential_backoff) bij elke fout *behalve* configuratieproblemen zoals 401 of 403 of ongeldige verzoeken zoals 400. Afgezien van de reguliere tariefbeperking door middel van 429 reacties, kan een uitval of beperking van een tijdelijke dienst leiden tot 5xx fouten. Het zou dan raadzaam zijn om na een bepaalde periode opnieuw te proberen.

Alle JSON-reacties (indien aanwezig) bevatten de `requestId`, dat dezelfde waarde is als de `X-Request-Id` header. Adobe raadt aan de koptekst te lezen, omdat deze altijd aanwezig is. De `requestId` wordt ook geretourneerd in alle gebeurtenissen met betrekking tot verwerkingsverzoeken als `requestId`. Clients mogen geen aanname maken over de opmaak van deze tekenreeks. Dit is een ondoorzichtige tekenreeks-id.

## Aanmelden bij nabewerking {#opt-in-to-post-processing}

De [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk) ondersteunt een aantal basisopties voor nabewerking van afbeeldingen. Aangepaste workers kunnen zich expliciet aanmelden bij naverwerking door het veld in te stellen `postProcess` op het weergaveobject naar `true`.

De ondersteunde gebruiksgevallen zijn:

* Uitsnijden is een uitvoering van een rechthoek waarvan de grenzen door middel van uitsnijden.w, uitsnijden.h, uitsnijden.x en uitsnijden.y zijn gedefinieerd. De uitsnijddetails worden opgegeven in de `instructions.crop` veld.
* Wijzig de grootte van afbeeldingen met breedte, hoogte of beide. De `instructions.width` en `instructions.height` definieert deze in het weergaveobject. Als u alleen de breedte of hoogte wilt wijzigen, stelt u slechts één waarde in. Met Compute Service behoudt u de hoogte-breedteverhouding.
* Stel de kwaliteit in voor een JPEG-afbeelding. De `instructions.quality` definieert deze in het weergaveobject. Een kwaliteitsniveau van 100 staat voor de hoogste kwaliteit, terwijl een lagere waarde een lagere kwaliteit betekent.
* Maak geïnterlinieerde afbeeldingen. De `instructions.interlace` definieert deze in het weergaveobject.
* Stel DPI in om de gerenderde grootte aan te passen voor publicatie op het bureaublad door de schaal aan te passen die op de pixels wordt toegepast. De `instructions.dpi` definieert deze in het weergaveobject om de dpi-resolutie te wijzigen. Als u de afbeelding echter zo wilt vergroten of verkleinen dat deze dezelfde grootte heeft bij een andere resolutie, gebruikt u de opdracht `convertToDpi` instructies.
* Wijzig de grootte van de afbeelding zodanig dat de weergegeven breedte of hoogte gelijk blijft aan het origineel bij de opgegeven doelresolutie (DPI). De `instructions.convertToDpi` definieert deze in het weergaveobject.

## Watermerkelementen {#add-watermark}

De [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk) ondersteunt het toevoegen van een watermerk aan PNG-, JPEG-, TIFF- en GIF-afbeeldingsbestanden. Het watermerk wordt toegevoegd na de vertoningsinstructies in het dialoogvenster `watermark` object op de vertoning.

Watermerken worden uitgevoerd tijdens de nabewerking van de vertoning. De aangepaste worker voor watermerkelementen [opteert in naverwerking](#opt-in-to-post-processing) door het veld in te stellen `postProcess` op het weergaveobject naar `true`. Als de worker niet meedoet, wordt er geen watermerken toegepast, zelfs niet als het watermerkobject is ingesteld op het weergaveobject in de aanvraag.

## Vertoningsinstructies {#rendition-instructions}

Hieronder vindt u de beschikbare opties voor de `renditions` array in [`/process`](#process-request).

### Algemene velden {#common-fields}

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | De doelindeling van vertoningen kan ook `text` voor tekstextractie en `xmp` voor het extraheren van XMP metagegevens als xml. Zie [ondersteunde indelingen](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support) | `png` |
| `worker` | `string` | URL van een [aangepaste toepassing](develop-custom-application.md). Moet een `https://` URL. Als dit veld aanwezig is, maakt een aangepaste toepassing de vertoning. Een ander setveld voor uitvoering wordt vervolgens gebruikt in de aangepaste toepassing. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | De URL waarnaar de gegenereerde uitvoering moet worden geüpload met HTTP-PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Vooraf ondertekende URL met meerdere delen uploadgegevens voor de gegenereerde uitvoering. Deze informatie is bedoeld voor [AEM/Oak Direct - Binair uploaden](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) met [uploadgedrag voor meerdere delen](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Velden:<ul><li>`urls`: array van tekenreeksen, één voor elke vooraf ondertekende deel-URL</li><li>`minPartSize`: de minimumgrootte voor één onderdeel = url</li><li>`maxPartSize`: de maximumgrootte voor één onderdeel = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Optioneel. De client bestuurt de gereserveerde ruimte en geeft deze door zoals bij weergavegebeurtenissen. Hiermee kan een client aangepaste informatie toevoegen om uitvoeringsgebeurtenissen te identificeren. Het mag niet worden gewijzigd of vertrouwd op in douanetoepassingen, aangezien de cliënten vrij zijn om het op elk ogenblik te veranderen. | `{ ... }` |

### Vertoningsspecifieke velden {#rendition-specific-fields}

Voor een lijst met momenteel ondersteunde bestandsindelingen raadpleegt u [ondersteunde bestandsindelingen](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `*` | `*` | Geavanceerde aangepaste velden kunnen worden toegevoegd aan een [aangepaste toepassing](develop-custom-application.md) begrijpt. | |
| `embedBinaryLimit` | `number` in bytes | Wanneer de bestandsgrootte van de vertoning kleiner is dan de opgegeven waarde, wordt deze opgenomen in de gebeurtenis die wordt verzonden nadat het maken is voltooid. De maximaal toegestane grootte voor insluiten is 32 kB (32 x 1024 bytes). Als een vertoning groter is dan de `embedBinaryLimit` beperkt, wordt deze op een locatie in de cloudopslag geplaatst en wordt niet ingesloten in de gebeurtenis. | `3276` |
| `width` | `number` | Breedte in pixels. alleen voor afbeeldingsuitvoeringen. | `200` |
| `height` | `number` | Hoogte in pixels. alleen voor afbeeldingsuitvoeringen. | `200` |
|                   |          | De hoogte-breedteverhouding blijft altijd behouden als: <ul> <li> Beide `width` en `height` worden opgegeven, past de afbeelding in de grootte aan terwijl de verhouding behouden blijft </li><li> Als alleen `width` of `height` is opgegeven, gebruikt de resulterende afbeelding de corresponderende dimensie terwijl de hoogte-breedteverhouding behouden blijft</li><li> Indien `width` of `height` niet is opgegeven, wordt de oorspronkelijke pixelgrootte van de afbeelding gebruikt. Het hangt van het brontype af. Voor sommige indelingen, zoals PDF-bestanden, wordt een standaardgrootte gebruikt. Er kan een maximumgrootte zijn.</li></ul> | |
| `quality` | `number` | JPEG-kwaliteit opgeven in het bereik van `1` tot `100`. Alleen van toepassing op afbeeldingsuitvoeringen. | `90` |
| `xmp` | `string` | Wordt alleen gebruikt door XMP terugverwijzing van metagegevens. Het is XMP base64 gecodeerd om terug te schrijven naar de opgegeven uitvoering. | |
| `interlace` | `bool` | Geïnterlinieerde PNG- of GIF- of progressieve JPEG maken door deze in te stellen op `true`. Dit heeft geen invloed op andere bestandsindelingen. | |
| `jpegSize` | `number` | De grootte van het JPEG-bestand in bytes wordt benaderd. Alle `quality` instellen. Dit heeft geen invloed op andere indelingen. | |
| `dpi` | `number` of `object` | Stel de DPI voor x en y in. Voor de eenvoud kan de waarde ook op één getal worden ingesteld, dat voor zowel x als y wordt gebruikt. Dit heeft geen invloed op de afbeelding zelf. | `96` of `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` of `object` | x- en y-DPI-waarden opnieuw samplen terwijl de fysieke grootte behouden blijft. Voor de eenvoud kan de waarde ook op één getal worden ingesteld, dat voor zowel x als y wordt gebruikt. | `96` of `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lijst met bestanden die moeten worden opgenomen in het ZIP-archief (`fmt=zip`). Elk item kan een URL-tekenreeks zijn of een object met de velden:<ul><li>`url`: URL om bestand te downloaden</li><li>`path`: Sla het bestand onder dit pad op in ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Dubbele verwerking voor ZIP-archieven (`fmt=zip`). Standaard wordt een fout gegenereerd bij meerdere bestanden die in het ZIP-bestand onder hetzelfde pad zijn opgeslagen. Instelling `duplicate` tot `ignore` resulteert alleen in het eerste element dat moet worden opgeslagen en de rest die moet worden genegeerd. | `ignore` |
| `watermark` | `object` | Bevat instructies over [watermerk](#watermark-specific-fields). |  |

### Watermerkspecifieke velden {#watermark-specific-fields}

De PNG-indeling wordt gebruikt als een watermerk.

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Schaal van het watermerk tussen `0.0` en `1.0`. `1.0` betekent dat het watermerk de oorspronkelijke schaal (1:1) heeft en de laagste waarden de grootte van het watermerk verminderen. | Een waarde van `0.5` de helft van de oorspronkelijke grootte. |
| `image` | `url` | URL naar het PNG-bestand dat moet worden gebruikt voor het watermerk. | |

## Asynchrone gebeurtenissen {#asynchronous-events}

Wanneer de verwerking van een vertoning is voltooid of wanneer een fout optreedt, wordt een gebeurtenis naar een Adobe verzonden [!DNL `I/O Events Journal`]. Clients moeten luisteren naar de journaal-URL die via [`/register`](#register). De dagboekreactie omvat een `event` array die bestaat uit één object voor elke gebeurtenis, waarvan de `event` bevat het daadwerkelijke gebeurtenislaadveld.

De Adobe [!DNL `I/O Events`] type voor alle gebeurtenissen van [!DNL Asset Compute Service] is `asset_compute`. Het dagboek wordt automatisch ingetekend op dit gebeurtenistype slechts en er is geen verdere vereiste om te filtreren die op [!DNL Adobe Developer] Type gebeurtenis. De de dienstspecifieke gebeurtenistypes zijn beschikbaar in `type` eigenschap van de gebeurtenis.

### Gebeurtenistypen {#event-types}

| Gebeurtenis | Beschrijving |
|---------------------|-------------|
| `rendition_created` | Verzonden voor elke correct verwerkte en geüploade vertoning. |
| `rendition_failed` | Verzonden voor elke uitvoering die niet kon worden verwerkt of geüpload. |

### Gebeurteniskenmerken {#event-attributes}

| Kenmerk | Type | Gebeurtenis | Beschrijving |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Tijdstempel wanneer de gebeurtenis in het vereenvoudigde verlengde is verzonden [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) indeling, zoals gedefinieerd door JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | De aanvraag-id van het oorspronkelijke verzoek aan `/process`, gelijk aan `X-Request-Id` header. |
| `source` | `object` | `*` | De `source` van de `/process` verzoek. |
| `userData` | `object` | `*` | De `userData` van de vertoning van de `/process` request if set. |
| `rendition` | `object` | `rendition_*` | Het overeenkomende weergaveobject dat wordt doorgegeven `/process`. |
| `metadata` | `object` | `rendition_created` | De [metagegevens](#metadata) eigenschappen van de vertoning. |
| `errorReason` | `string` | `rendition_failed` | Vertoning mislukt [reden](#error-reasons) indien van toepassing. |
| `errorMessage` | `string` | `rendition_failed` | De tekst die meer details geeft over de eventuele uitvoerfout. |

### Metagegevens {#metadata}

| Eigenschap | Beschrijving |
|--------|-------------|
| `repo:size` | De grootte van de vertoning in bytes. |
| `repo:sha1` | De sha1-samenvatting van de uitvoering. |
| `dc:format` | Het MIME-type van de vertoning. |
| `repo:encoding` | De charsetcodering van de vertoning voor het geval dat deze een op tekst gebaseerde indeling is. |
| `tiff:ImageWidth` | De breedte van de vertoning in pixels. Alleen aanwezig voor afbeeldingsuitvoeringen. |
| `tiff:ImageLength` | De lengte van de vertoning in pixels. Alleen aanwezig voor afbeeldingsuitvoeringen. |

### Foutredenen {#error-reasons}

| Reden | Beschrijving |
|---------|-------------|
| `RenditionFormatUnsupported` | De aangevraagde renditie-indeling wordt niet ondersteund voor de opgegeven bron. |
| `SourceUnsupported` | De specifieke bron wordt niet ondersteund, ook al wordt het type ondersteund. |
| `SourceCorrupt` | De brongegevens zijn beschadigd. Dit omvat lege bestanden. |
| `RenditionTooLarge` | De vertoning kan niet worden geüpload met de vooraf ondertekende URL&#39;s in `target`. De werkelijke rendergrootte is beschikbaar als metagegevens in `repo:size` en wordt door de client gebruikt om deze vertoning opnieuw te verwerken met het juiste aantal vooraf ondertekende URL&#39;s. |
| `GenericError` | Een andere onverwachte fout. |
