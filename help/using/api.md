---
title: '[!DNL Asset Compute Service] HTTP-API'
description: '[!DNL Asset Compute Service] HTTP API om aangepaste toepassingen te maken.'
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '2862'
ht-degree: 0%

---

# [!DNL Asset Compute Service] HTTP-API {#asset-compute-http-api}

Het gebruik van de API is beperkt tot ontwikkelingsdoeleinden. De API wordt als context verstrekt wanneer het ontwikkelen van douanetoepassingen. [!DNL Adobe Experience Manager] als [!DNL Cloud Service] gebruikt de API om de verwerkingsgegevens door te geven aan een aangepaste toepassing. Voor meer informatie, zie [ de activamicrodiensten van het Gebruik en Profielen van de Verwerking ](https://experienceleague.adobe.com/nl/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] is alleen beschikbaar voor gebruik met [!DNL Experience Manager] as a [!DNL Cloud Service] .

Elke client van de [!DNL Asset Compute Service] HTTP API moet deze flow op hoog niveau volgen:

1. Een client wordt ingericht als een [!DNL Adobe Developer Console] -project in een IMS-organisatie. Elke afzonderlijke client (systeem of omgeving) vereist een eigen afzonderlijk project om de gegevensstroom van de gebeurtenis te scheiden.

1. Een cliënt produceert een toegangstoken voor de technische rekening gebruikend [ JWT (de Rekening van de Dienst) Authentificatie ](https://developer.adobe.com/developer-console/docs/guides/).

1. Een client roept [`/register`](#register) slechts eenmaal aan om de journaal-URL op te halen.

1. Een client roept [`/process`](#process-request) aan voor elk element waarvoor de client uitvoeringen wil genereren. De vraag is asynchroon.

1. Een cliënt pollt regelmatig het dagboek om [ gebeurtenissen ](#asynchronous-events) te ontvangen. Het ontvangt gebeurtenissen voor elke gevraagde vertoning wanneer de vertoning met succes wordt verwerkt (`rendition_created` gebeurtenistype) of als er een fout (`rendition_failed` gebeurtenistype) is.

De [ adobe-asset-compute-cliënt ](https://github.com/adobe/asset-compute-client) module maakt het gemakkelijk om API in code Node.js te gebruiken.

## Verificatie en autorisatie {#authentication-and-authorization}

Voor alle API&#39;s is verificatie van toegangstoken vereist. De verzoeken moeten de volgende kopballen plaatsen:

1. `Authorization` kopbal met dragerteken, dat het technische rekeningsteken is, dat via [ JWT uitwisseling ](https://developer.adobe.com/developer-console/docs/guides/) van het project van Adobe Developer Console wordt ontvangen. Het [ werkingsgebied ](#scopes) wordt hieronder gedocumenteerd.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` met de IMS-organisatie-id.

1. `x-api-key` met de client-id van het [!DNL Adobe Developers Console] -project.

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

Voor deze bereiken moet het [!DNL Adobe Developer Console] -project zijn geabonneerd op `Asset Compute` , `I/O Events` en `I/O Management API` -services. De uitsplitsing van individuele scènes is:

* Basis
   * bereik: `openid,AdobeID`

* Asset Compute
   * metarek: `asset_compute_meta`
   * bereik: `asset_compute,read_organizations`

* Adobe [!DNL `I/O Events`]
   * metarek: `event_receiver_api`
   * bereik: `event_receiver,event_receiver_api`

* Adobe [!DNL `I/O Management API`]
   * metarek: `ent_adobeio_sdk`
   * bereik: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registratie {#register}

Elke cliënt van [!DNL Asset Compute service] - een uniek [!DNL Adobe Developer Console] project dat aan de dienst wordt ingetekend - moet [ registreren ](#register-request) alvorens verwerkingsverzoeken te maken. De registratiestap retourneert het unieke gebeurtenisjournaal dat vereist is om de asynchrone gebeurtenissen van de uitvoering op te halen.

Aan het eind van zijn levenscyclus, kan een cliënt [&#128279;](#unregister-request) unregister.

### Aanvraag registreren {#register-request}

Met deze API-aanroep wordt een [!DNL Asset Compute] -client ingesteld en wordt de URL van het gebeurtenisdagboek weergegeven. Dit proces is een epidemische bewerking en hoeft slechts één keer voor elke client te worden aangeroepen. Het kan opnieuw worden geroepen om het dagboek URL terug te winnen.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/register` |
| Koptekst `Authorization` | Alle [ toestemmings verwante kopballen ](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel, door clients ingesteld voor een unieke end-to-end-id van de verwerkingsverzoeken in alle systemen. |
| Aanvragingsinstantie | Moet leeg zijn. |

### Reactie registreren {#register-response}

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Hetzelfde als de aanvraagheader van `X-Request-Id` of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen, of steunverzoeken, of allebei. |
| Responsinstantie | Een JSON-object met velden `journal` , `ok` of `requestId` . |

De HTTP-statuscodes zijn:

* **200 Succes**: Wanneer het verzoek succesvol is. De URL van `journal` ontvangt meldingen over de resultaten van de asynchrone verwerking die via `/process` wordt gestart. Er wordt een melding weergegeven van `rendition_created` -gebeurtenissen wanneer het proces is voltooid, of `rendition_failed` -gebeurtenissen als het proces is mislukt.

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 Onbevoegd**: komt voor wanneer het verzoek geen geldige [ authentificatie ](#authentication-and-authorization) heeft. Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.

* **403 Verboden**: komt voor wanneer het verzoek geen geldige [ vergunning ](#authentication-and-authorization) heeft. Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het Adobe Developer Console project (technische rekening) wordt niet ingetekend aan alle vereiste diensten.

* **429 Te veel verzoeken**: komt voor wanneer deze cliënt of anders het systeem overlaadt. De cliënten zouden met een [ exponentiële backoff ](https://en.wikipedia.org/wiki/Exponential_backoff) opnieuw moeten proberen. Het lichaam is leeg.
* **4xx fout**: Wanneer er een andere cliëntfout en registratie ontbrak. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx fout**: komt voor wanneer er een andere fout van de serverzijde en de registratie ontbrak. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### Aanvraag ongedaan maken {#unregister-request}

Met deze API-aanroep wordt de registratie van een [!DNL Asset Compute] -client ongedaan gemaakt. Nadat de registratie ongedaan is gemaakt, kan `/process` niet meer worden aangeroepen. Als u de API-aanroep voor een niet-geregistreerde client of een nog te registreren client gebruikt, wordt een `404` -fout geretourneerd.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/unregister` |
| Koptekst `Authorization` | Alle [ toestemmings verwante kopballen ](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel. Clients kunnen deze instellen voor een unieke end-to-end-id van de verwerkingsverzoeken in verschillende systemen. |
| Aanvragingsinstantie | Leeg. |

### Reactie ongedaan maken {#unregister-response}

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Hetzelfde als de aanvraagheader van `X-Request-Id` of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen of steunverzoeken. |
| Responsinstantie | Een JSON-object met `ok` - en `requestId` -velden. |

De statuscodes zijn:

* **200 Succes**: komt voor wanneer de registratie en het dagboek worden gevonden en verwijderd.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 Onbevoegd**: komt voor wanneer het verzoek geen geldige [ authentificatie ](#authentication-and-authorization) heeft. Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.

* **403 Verboden**: komt voor wanneer het verzoek geen geldige [ vergunning ](#authentication-and-authorization) heeft. Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het Adobe Developer Console project (technische rekening) wordt niet ingetekend aan alle vereiste diensten.

* **404 niet gevonden**: Deze status verschijnt wanneer de verstrekte geloofsbrieven niet geregistreerd of ongeldig zijn.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 Te veel verzoeken**: komt voor wanneer het systeem wordt overbelast. De cliënten zouden met een [ exponentiële backoff ](https://en.wikipedia.org/wiki/Exponential_backoff) opnieuw moeten proberen. Het lichaam is leeg.

* **4xx fout**: komt voor wanneer er een andere cliëntfout was en unregister ontbrak. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx fout**: komt voor wanneer er een andere fout van de serverzijde en de registratie ontbrak. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## Procesaanvraag {#process-request}

De bewerking `process` verzendt een taak die een bronelement transformeert in meerdere uitvoeringen, op basis van de instructies in de aanvraag. Meldingen over een geslaagde voltooiing (gebeurtenistype `rendition_created` ) of over eventuele fouten (gebeurtenistype `rendition_failed` ) worden verzonden naar een gebeurtenisvenster dat één keer met [`/register`](#register) moet worden opgehaald voordat een aantal `/process` -aanvragen kan worden ingediend. Onjuist gevormde verzoeken ontbreken onmiddellijk met een 400 foutencode.

Er wordt verwezen naar binaire getallen via URL&#39;s, zoals vooraf ondertekende URL&#39;s voor Amazon AWS S3 of Azure Blob Storage SAS-URL&#39;s. Wordt gebruikt voor het lezen van het element `source` (`GET` URLs) en het schrijven van de vertoningen (`PUT` URLs). De klant is verantwoordelijk voor het genereren van deze vooraf ondertekende URL&#39;s.

| Parameter | Waarde |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pad | `/process` |
| MIME-type | `application/json` |
| Koptekst `Authorization` | Alle [ toestemmings verwante kopballen ](#authentication-and-authorization). |
| Koptekst `x-request-id` | Optioneel. Clients kunnen een unieke end-to-end-id instellen om verwerkingsverzoeken in verschillende systemen bij te houden. |
| Aanvragingsinstantie | Deze moet de JSON-indeling van het procesverzoek hebben, zoals hieronder wordt beschreven. Er worden instructies gegeven over welk element moet worden verwerkt en welke uitvoeringen moeten worden gegenereerd. |

### Verzoek om verwerking JSON {#process-request-json}

De aanvraaghoofdtekst van `/process` is een JSON-object met dit schema op hoog niveau:

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
| `source` | `object` | Beschrijf het bronelement dat wordt verwerkt. Zie beschrijving van [ de objecten van Source gebieden ](#source-object-fields) hieronder. Optioneel op basis van de gevraagde renditie-indeling (bijvoorbeeld `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Uitvoeringen die moeten worden gegenereerd op basis van het bronbestand. Elk vertoningsvoorwerp steunt a [ vertoningsinstructie ](#rendition-instructions). Vereist. | `[{ "target": "https://....", "fmt": "png" }]` |

De `source` kan een `<string>` zijn die als een URL wordt gezien of een `<object>` met een extra veld. De volgende varianten zijn vergelijkbaar:

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
| `name` | `string` | Naam Source-elementbestand. Een bestandsextensie in de naam kan worden gebruikt als er geen MIME-type wordt gedetecteerd. De bestandsnaam die in het URL-pad is opgegeven, krijgt prioriteit. En, heeft het voorrang over de dossiernaam in de `content-disposition` kopbal van de binaire bron. De standaardwaarde is &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Source-bestandsgrootte in bytes. Heeft voorrang op `content-length` header van de binaire resource. | `10234` |
| `mimetype` | `string` | MIME-type voor Source-elementbestand. Heeft voorrang op de `content-type` -header van de binaire bron. | `"image/jpeg"` |

### Een volledig aanvraagvoorbeeld `process` {#complete-process-request-example}

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

De `/process` -aanvraag wordt onmiddellijk geretourneerd met succes of als de aanvraag is mislukt op basis van de validatie van de basisaanvraag. De werkelijke verwerking van elementen gebeurt asynchroon.

| Parameter | Waarde |
|-----------------------|------------------------------------------------------|
| MIME-type | `application/json` |
| Koptekst `X-Request-Id` | Hetzelfde als de aanvraagheader van `X-Request-Id` of een uniek gegenereerde header. Gebruik voor het identificeren van verzoeken over systemen of steunverzoeken. |
| Responsinstantie | Een JSON-object met `ok` - en `requestId` -velden. |

Statuscodes:

* **200 Succes**: Als het verzoek met succes werd voorgelegd. Response JSON bevat `"ok": true` :

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Ongeldig verzoek**: Als het verzoek onjuist gestructureerd is, bijvoorbeeld, als het vereiste gebieden in de nuttige lading JSON mist. Response JSON bevat `"ok": false` :

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 onbevoegd**: Wanneer het verzoek geen geldige [ authentificatie ](#authentication-and-authorization) heeft. Een voorbeeld kan een ongeldige toegangstoken of een ongeldige API-sleutel zijn.
* **403 Verboden**: Wanneer het verzoek geen geldige [ vergunning ](#authentication-and-authorization) heeft. Een voorbeeld zou een geldig toegangstoken kunnen zijn, maar het Adobe Developer Console project (technische rekening) wordt niet ingetekend aan alle vereiste diensten.
* **429 Te veel verzoeken**: komt voor wanneer het systeem, of toe te schrijven aan deze bepaalde cliënt of van algemene vraag wordt overweldigd. De cliënten kunnen met een [ exponentiële backoff ](https://en.wikipedia.org/wiki/Exponential_backoff) opnieuw proberen. Het lichaam is leeg.
* **4xx fout**: Wanneer er een andere cliëntfout was. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx fout**: Wanneer er een andere fout van de serverzijde was. Gewoonlijk wordt een JSON-reactie als deze geretourneerd, hoewel dat niet voor alle fouten wordt gegarandeerd:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

De meeste cliënten zijn waarschijnlijk geneigd om het zelfde verzoek met [ exponentiële backoff ](https://en.wikipedia.org/wiki/Exponential_backoff) op om het even welke fout *behalve* configuratiekwesties zoals 401 of 403, of ongeldige verzoeken zoals 400 opnieuw te proberen. Afgezien van de reguliere tariefbeperking door middel van 429 reacties, kan een uitval of beperking van een tijdelijke dienst leiden tot 5xx fouten. Het zou dan raadzaam zijn om na een bepaalde periode opnieuw te proberen.

Alle JSON-reacties (indien aanwezig) bevatten `requestId` . Dit is dezelfde waarde als de `X-Request-Id` -header. Adobe raadt aan de koptekst te lezen omdat deze altijd aanwezig is. `requestId` wordt ook geretourneerd in alle gebeurtenissen die betrekking hebben op het verwerken van aanvragen als `requestId` . Clients mogen geen aanname maken over de opmaak van deze tekenreeks. Dit is een ondoorzichtige tekenreeks-id.

## Aanmelden bij nabewerking {#opt-in-to-post-processing}

De [ SDK van Asset Compute ](https://github.com/adobe/asset-compute-sdk) steunt een reeks basisbeeld naverwerkingsopties. Aangepaste workers kunnen zich expliciet aanmelden bij nabewerking door het veld `postProcess` voor het weergaveobject in te stellen op `true` .

De ondersteunde gebruiksgevallen zijn:

* Uitsnijden is een uitvoering van een rechthoek waarvan de grenzen door middel van uitsnijden.w, uitsnijden.h, uitsnijden.x en uitsnijden.y zijn gedefinieerd. De uitsnijddetails worden opgegeven in het veld `instructions.crop` van het weergaveobject.
* Wijzig de grootte van afbeeldingen met breedte, hoogte of beide. De tekens `instructions.width` en `instructions.height` definiëren deze in het weergaveobject. Als u alleen de breedte of hoogte wilt wijzigen, stelt u slechts één waarde in. Met Compute Service behoudt u de hoogte-breedteverhouding.
* Stel de kwaliteit in voor een JPEG-afbeelding. `instructions.quality` definieert deze in het weergaveobject. Een kwaliteitsniveau van 100 staat voor de hoogste kwaliteit, terwijl een lagere waarde een lagere kwaliteit betekent.
* Maak geïnterlinieerde afbeeldingen. `instructions.interlace` definieert deze in het weergaveobject.
* Stel DPI in om de gerenderde grootte aan te passen voor publicatie op het bureaublad door de schaal aan te passen die op de pixels wordt toegepast. De `instructions.dpi` definieert deze in het weergaveobject om de dpi-resolutie te wijzigen. Als u echter het formaat van de afbeelding wilt wijzigen, zodat deze dezelfde grootte heeft bij een andere resolutie, gebruikt u de instructies van `convertToDpi` .
* Wijzig de grootte van de afbeelding zodanig dat de weergegeven breedte of hoogte gelijk blijft aan het origineel bij de opgegeven doelresolutie (DPI). `instructions.convertToDpi` definieert deze in het weergaveobject.

## Watermerkelementen {#add-watermark}

De [ SDK van Asset Compute ](https://github.com/adobe/asset-compute-sdk) steunt het toevoegen van een watermerk aan PNG, JPEG, TIFF, en de beelddossiers van GIF. Het watermerk wordt toegevoegd na de vertoningsinstructies in het `watermark` -object op de vertoning.

Watermerken worden uitgevoerd tijdens de nabewerking van de vertoning. Om activa van het watermerk te voorzien, opteert de douanearbeider [ in post-verwerking ](#opt-in-to-post-processing) door het gebied `postProcess` op het vertoningsvoorwerp aan `true` te plaatsen. Als de worker niet meedoet, wordt er geen watermerken toegepast, zelfs niet als het watermerkobject is ingesteld op het weergaveobject in de aanvraag.

## Vertoningsinstructies {#rendition-instructions}

Hieronder vindt u de beschikbare opties voor de array `renditions` in [`/process`](#process-request) .

### Algemene velden {#common-fields}

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | De doelindeling van de uitvoeringen kan ook `text` zijn voor tekstextractie en `xmp` voor het extraheren van XMP-metagegevens als XML. Zie [ gesteunde formaten ](https://experienceleague.adobe.com/nl/docs/experience-manager-cloud-service/content/assets/file-format-support) | `png` |
| `worker` | `string` | URL van a [ douanetoepassing ](develop-custom-application.md). Moet een `https://` URL zijn. Als dit veld aanwezig is, maakt een aangepaste toepassing de vertoning. Een ander setveld voor uitvoering wordt vervolgens gebruikt in de aangepaste toepassing. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | De URL waarnaar de gegenereerde uitvoering moet worden geüpload met HTTP PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Vooraf ondertekende URL met meerdere delen uploadgegevens voor de gegenereerde uitvoering. Deze informatie is voor [ AEM/Oak Directe Binaire Upload ](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) met dit [ multipart uploadgedrag ](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br> Gebieden:<ul><li>`urls`: array van tekenreeksen, één voor elke vooraf ondertekende deel-URL</li><li>`minPartSize`: de minimale grootte die voor één onderdeel moet worden gebruikt = url</li><li>`maxPartSize`: de maximale grootte die voor één onderdeel kan worden gebruikt = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Optioneel. De client bestuurt de gereserveerde ruimte en geeft deze door zoals bij weergavegebeurtenissen. Hiermee kan een client aangepaste informatie toevoegen om uitvoeringsgebeurtenissen te identificeren. Het mag niet worden gewijzigd of vertrouwd op in douanetoepassingen, aangezien de cliënten vrij zijn om het op elk ogenblik te veranderen. | `{ ... }` |

### Vertoningsspecifieke velden {#rendition-specific-fields}

Voor een lijst van momenteel gesteunde dossierformaten, zie [ gesteunde dossierformaten ](https://experienceleague.adobe.com/nl/docs/experience-manager-cloud-service/content/assets/file-format-support).

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `*` | `*` | Geavanceerd, kunnen de douanevelden worden toegevoegd dat a [ douanetoepassing ](develop-custom-application.md) begrijpt. | |
| `embedBinaryLimit` | `number` in bytes | Wanneer de bestandsgrootte van de vertoning kleiner is dan de opgegeven waarde, wordt deze opgenomen in de gebeurtenis die wordt verzonden nadat het maken is voltooid. De maximaal toegestane grootte voor insluiten is 32 kB (32 x 1024 bytes). Als een vertoning groter is dan de limiet van `embedBinaryLimit` , wordt deze geplaatst op een locatie in de cloudopslag en wordt deze niet ingesloten in de gebeurtenis. | `3276` |
| `width` | `number` | Breedte in pixels. alleen voor afbeeldingsuitvoeringen. | `200` |
| `height` | `number` | Hoogte in pixels. alleen voor afbeeldingsuitvoeringen. | `200` |
|                   |          | De hoogte-breedteverhouding blijft altijd behouden als: <ul> <li> Zowel `width` als `height` worden opgegeven en de afbeelding past op de grootte terwijl de hoogte-breedteverhouding behouden blijft </li><li> Als er maar `width` of `height` wordt opgegeven, gebruikt de resulterende afbeelding de corresponderende dimensie terwijl de hoogte-breedteverhouding behouden blijft</li><li> Als `width` of `height` niet is opgegeven, wordt de oorspronkelijke pixelgrootte van de afbeelding gebruikt. Het hangt van het brontype af. Voor sommige indelingen, zoals PDF-bestanden, wordt een standaardgrootte gebruikt. Er kan een maximumgrootte zijn.</li></ul> | |
| `quality` | `number` | Geef JPEG-kwaliteit op in het bereik van `1` tot `100` . Alleen van toepassing op afbeeldingsuitvoeringen. | `90` |
| `xmp` | `string` | Wordt alleen gebruikt door terugschrijven van XMP-metagegevens. Het is XMP met base64-codering om terug te schrijven naar de opgegeven uitvoering. | |
| `interlace` | `bool` | Maak geïnterlinieerde PNG- of GIF- of progressieve JPEG door deze in te stellen op `true` . Dit heeft geen invloed op andere bestandsindelingen. | |
| `jpegSize` | `number` | Grootte JPEG-bestand, ongeveer in bytes. Alle `quality` -instellingen worden genegeerd. Dit heeft geen invloed op andere indelingen. | |
| `dpi` | `number` of `object` | Stel de DPI voor x en y in. Voor de eenvoud kan de waarde ook op één getal worden ingesteld, dat voor zowel x als y wordt gebruikt. Dit heeft geen invloed op de afbeelding zelf. | `96` of `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` of `object` | x- en y-DPI-waarden opnieuw samplen terwijl de fysieke grootte behouden blijft. Voor de eenvoud kan de waarde ook op één getal worden ingesteld, dat voor zowel x als y wordt gebruikt. | `96` of `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lijst met bestanden die moeten worden opgenomen in het ZIP-archief (`fmt=zip`). Elk item kan een URL-tekenreeks zijn of een object met de velden:<ul><li>`url`: URL om bestand te downloaden</li><li>`path`: Sla het bestand onder dit pad op in het ZIP-bestand</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Dubbele verwerking voor ZIP-archieven (`fmt=zip`). Standaard wordt een fout gegenereerd bij meerdere bestanden die in het ZIP-bestand onder hetzelfde pad zijn opgeslagen. Als u `duplicate` instelt op `ignore` , wordt alleen het eerste element opgeslagen en de rest genegeerd. | `ignore` |
| `watermark` | `object` | Bevat instructies over het [ watermerk ](#watermark-specific-fields). |  |

### Watermerkspecifieke velden {#watermark-specific-fields}

De PNG-indeling wordt gebruikt als een watermerk.

| Naam | Type | Beschrijving | Voorbeeld |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Schaal van het watermerk, tussen `0.0` en `1.0` . `1.0` betekent dat het watermerk de oorspronkelijke schaal (1:1) heeft en de lagere waarden de grootte van het watermerk verminderen. | De waarde `0.5` betekent de helft van de oorspronkelijke grootte. |
| `image` | `url` | URL naar het PNG-bestand dat moet worden gebruikt voor het watermerk. | |

## Asynchrone gebeurtenissen {#asynchronous-events}

Wanneer de verwerking van een vertoning is voltooid of wanneer een fout optreedt, wordt een gebeurtenis verzonden naar een Adobe [!DNL `I/O Events Journal`] . Clients moeten luisteren naar de journaal-URL die via [`/register`](#register) wordt geboden. De journaalreactie bevat een array `event` die bestaat uit één object voor elke gebeurtenis, waarvan het veld `event` de werkelijke gebeurtenislading bevat.

Het Adobe [!DNL `I/O Events`] -type voor alle gebeurtenissen van de lus [!DNL Asset Compute Service] is `asset_compute` . Het tijdschrift wordt alleen automatisch op dit gebeurtenistype geabonneerd en er is geen verdere vereiste om te filteren op basis van het gebeurtenistype [!DNL Adobe Developer] . De service-specifieke gebeurtenistypen zijn beschikbaar in de eigenschap `type` van de gebeurtenis.

### Gebeurtenistypen {#event-types}

| Gebeurtenis | Beschrijving |
|---------------------|-------------|
| `rendition_created` | Verzonden voor elke correct verwerkte en geüploade vertoning. |
| `rendition_failed` | Verzonden voor elke uitvoering die niet kon worden verwerkt of geüpload. |

### Gebeurteniskenmerken {#event-attributes}

| Kenmerk | Type | Gebeurtenis | Beschrijving |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Tijdstempel toen de gebeurtenis in vereenvoudigd uitgebreid [ wordt verzonden ISO-8601 ](https://en.wikipedia.org/wiki/ISO_8601) formaat, zoals die door JavaScript [ wordt bepaald Date.toISOString () ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | De aanvraag-id van de oorspronkelijke aanvraag naar `/process` , gelijk aan `X-Request-Id` header. |
| `source` | `object` | `*` | De `source` van de `/process` request. |
| `userData` | `object` | `*` | De `userData` van de vertoning van de `/process` request indien ingesteld. |
| `rendition` | `object` | `rendition_*` | Het overeenkomende weergaveobject dat wordt doorgegeven in `/process` . |
| `metadata` | `object` | `rendition_created` | De [ meta-gegevens ](#metadata) eigenschappen van de vertoning. |
| `errorReason` | `string` | `rendition_failed` | De mislukking van de vertoning [ reden ](#error-reasons) als om het even welk. |
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
| `RenditionTooLarge` | De vertoning kan niet worden geüpload met de vooraf ondertekende URL&#39;s in `target` . De werkelijke weergavegrootte is beschikbaar als metagegevens in `repo:size` en wordt door de client gebruikt om deze vertoning opnieuw te verwerken met het juiste aantal vooraf ondertekende URL&#39;s. |
| `GenericError` | Een andere onverwachte fout. |
