---
title: De werking van een aangepaste toepassing begrijpen
description: Interne werking van [!DNL Asset Compute Service] een aangepaste toepassing om te begrijpen hoe deze werkt.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '691'
ht-degree: 0%

---

# Interfaces van een aangepaste toepassing {#how-custom-application-works}

Gebruik de volgende illustratie om inzicht te krijgen in de end-to-end workflow wanneer een digitaal element wordt verwerkt met een aangepaste toepassing van een client.

![Aangepaste workflow voor toepassingen](assets/customworker.svg)

*Afbeelding: Stappen die nodig zijn bij het verwerken van een element met behulp van Adobe [!DNL Asset Compute Service].*

## Registratie {#registration}

De client moet [`/register`](api.md#register) één keer vóór het eerste verzoek aan [`/process`](api.md#process-request) zodat kan het opstelling en wint het dagboek URL voor het ontvangen van Adobe terug [!DNL I/O Events] Gebeurtenissen voor Asset compute van de Adobe.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

De [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript-bibliotheek kan in NodeJS-toepassingen worden gebruikt om alle noodzakelijke stappen af te handelen, van registratie, verwerking tot asynchrone gebeurtenisafhandeling. Voor meer informatie over de vereiste kopballen, zie [Verificatie en autorisatie](api.md).

## Verwerking {#processing}

De client verzendt een [verwerking](api.md#process-request) verzoek.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

De client is verantwoordelijk voor de juiste opmaak van de uitvoeringen met vooraf ondertekende URL&#39;s. De [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript-bibliotheek kan in NodeJS-toepassingen worden gebruikt om URL&#39;s vooraf te ondertekenen. Momenteel ondersteunt de bibliotheek alleen Azure Blob Storage en AWS S3 Containers.

De verwerkingsaanvraag retourneert een `requestId` die kunnen worden gebruikt voor opiniepeilingen [!DNL Adobe I/O] Gebeurtenissen.

Hieronder ziet u een voorbeeld van een aangepaste aanvraag voor de verwerking van toepassingen.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

De [!DNL Asset Compute Service] verzendt de aanvragen voor de uitvoering van de aangepaste toepassing naar de aangepaste toepassing. Er wordt een HTTP-POST gebruikt naar de opgegeven toepassings-URL, de beveiligde webactie-URL van App Builder. Voor alle aanvragen wordt het HTTPS-protocol gebruikt om de gegevensbeveiliging te maximaliseren.

De [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) die door een douanetoepassing worden gebruikt behandelt het verzoek van de POST van HTTP. Het handelt ook het downloaden van de bron, het uploaden van uitvoeringen, het verzenden van Adobe af [!DNL I/O Events] en foutafhandeling.

<!-- TBD: Add the application diagram. -->

### Toepassingscode {#application-code}

De code van de douane moet slechts callback verstrekken die het plaatselijk beschikbare brondossier (`source.path`). De `rendition.path` is de locatie waar het eindresultaat van een aanvraag voor de verwerking van bedrijfsmiddelen wordt geplaatst. De aangepaste toepassing gebruikt de callback om de lokaal beschikbare bronbestanden om te zetten in een weergavebestand met de naam die wordt doorgegeven (`rendition.path`). Een aangepaste toepassing moet schrijven naar `rendition.path` om een vertoning te maken:

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### Bronbestanden downloaden {#download-source}

Een aangepaste toepassing heeft alleen betrekking op lokale bestanden. De [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) handelt het downloaden van het bronbestand af.

### Vertoning maken {#rendition-creation}

De SDK roept een asynchroon [callback-functie voor uitvoering](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) voor elke vertoning.

De callback functie heeft toegang tot [bron](https://github.com/adobe/asset-compute-sdk#source) en [uitvoering](https://github.com/adobe/asset-compute-sdk#rendition) objecten. De `source.path` bestaat al en is het pad naar de lokale kopie van het bronbestand. De `rendition.path` Dit is het pad waar de verwerkte vertoning moet worden opgeslagen. Tenzij de [disableSourceDownload, markering](https://github.com/adobe/asset-compute-sdk#worker-options-optional) is ingesteld, moet de toepassing exact de `rendition.path`. Anders kan de SDK het weergavebestand niet vinden of identificeren en mislukt.

De overdreven vereenvoudiging van het voorbeeld wordt gedaan om de anatomie van een douanetoepassing te illustreren en te concentreren. De toepassing kopieert het bronbestand alleen naar de weergavebestemming.

Voor meer informatie over de parameters van de vertoningscallback, zie [ASSET COMPUTE SDK API](https://github.com/adobe/asset-compute-sdk#api-details).

### Uitvoeringen uploaden {#upload-rendition}

Nadat elke vertoning is gemaakt en opgeslagen in een bestand met het pad dat wordt verschaft door `rendition.path`de [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) uploadt elke vertoning naar een cloudopslag (AWS of Azure). Een aangepaste toepassing krijgt meerdere uitvoeringen tegelijkertijd als en alleen als de binnenkomende aanvraag meerdere uitvoeringen bevat die verwijzen naar dezelfde toepassings-URL. Het uploaden naar de cloudopslag vindt plaats na elke uitvoering en voordat de callback voor de volgende uitvoering wordt uitgevoerd.

De `batchWorker()` heeft een ander gedrag. Alle uitvoeringen worden verwerkt en pas nadat ze zijn verwerkt, worden ze geüpload.

## [!DNL Adobe I/O] Gebeurtenissen {#aio-events}

De SDK verzendt de Adobe [!DNL I/O Events] voor elke vertoning. Deze gebeurtenissen zijn van het type `rendition_created` of `rendition_failed` afhankelijk van het resultaat. Zie voor meer informatie [Asynchrone gebeurtenissen voor asset compute](api.md#asynchronous-events).

## Ontvangen [!DNL Adobe I/O] Gebeurtenissen {#receive-aio-events}

De client pollt de Adobe [!DNL I/O Events] dagboek volgens zijn consumptielogica . De eerste journaal-URL is de URL die wordt opgegeven in `/register` API-reactie. Gebeurtenissen kunnen worden geïdentificeerd met de `requestId` die aanwezig is in de gebeurtenissen en dezelfde is als die in `/process`. Elke vertoning heeft een aparte gebeurtenis die wordt verzonden zodra de vertoning is geüpload (of mislukt). Wanneer de client een overeenkomende gebeurtenis ontvangt, kan de client de resulterende uitvoeringen weergeven of op een andere manier afhandelen.

De JavaScript-bibliotheek [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) maakt de dagboekopiniepeiling eenvoudig gebruikend `waitActivation()` methode om alle gebeurtenissen op te halen.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

Voor details over hoe te om dagboekgebeurtenissen te krijgen, zie Adobe [[!DNL I/O Events] API](https://developer.adobe.com/events/docs/guides/api/journaling_api/).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
