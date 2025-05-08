---
title: De werking van een aangepaste toepassing begrijpen
description: Het interne werk van  [!DNL Asset Compute Service]  douanetoepassing helpen begrijpen hoe het werkt.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '691'
ht-degree: 0%

---

# Interfaces van een aangepaste toepassing {#how-custom-application-works}

Gebruik de volgende illustratie om inzicht te krijgen in de end-to-end workflow wanneer een digitaal element wordt verwerkt met een aangepaste toepassing van een client.

![ de toepassingswerkschema van de Douane ](assets/customworker.svg)

*Cijfer: Stappen betrokken wanneer het verwerken van activa gebruikend Adobe [!DNL Asset Compute Service].*

## Registratie {#registration}

De client moet [`/register`](api.md#register) één keer vóór de eerste aanvraag naar [`/process`](api.md#process-request) aanroepen, zodat deze de dagboek-URL voor het ontvangen van Adobe [!DNL I/O Events] -gebeurtenissen voor Adobe Asset Compute kan instellen en ophalen.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

De [`@adobe/asset-compute-client` ](https://github.com/adobe/asset-compute-client#usage) JavaScript bibliotheek kan in toepassingen worden gebruikt NodeJS om alle noodzakelijke stappen van registratie, verwerking aan asynchrone gebeurtenisbehandeling te behandelen. Voor meer informatie over de vereiste kopballen, zie [ Authentificatie en Vergunning ](api.md).

## Verwerking {#processing}

De cliënt verzendt a [ verwerkings ](api.md#process-request) verzoek.

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

De client is verantwoordelijk voor de juiste opmaak van de uitvoeringen met vooraf ondertekende URL&#39;s. De [`@adobe/node-cloud-blobstore-wrapper` ](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript-bibliotheek kan in NodeJS-toepassingen worden gebruikt om URL&#39;s vooraf te ondertekenen. Momenteel ondersteunt de bibliotheek alleen Azure Blob Storage en AWS S3 Containers.

De verwerkingsaanvraag retourneert een `requestId` die kan worden gebruikt voor opiniepeilingen van [!DNL Adobe I/O] -gebeurtenissen.

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

De [!DNL Asset Compute Service] verzendt de aanvragen voor de uitvoering van de aangepaste toepassing naar de aangepaste toepassing. Er wordt een HTTP POST gebruikt naar de opgegeven toepassings-URL, de beveiligde webactie-URL van App Builder. Voor alle aanvragen wordt het HTTPS-protocol gebruikt om de gegevensbeveiliging te maximaliseren.

[ SDK van Asset Compute ](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) die door een douanetoepassing wordt gebruikt behandelt het POST van HTTP- verzoek. Ook wordt het downloaden van de bron, het uploaden van uitvoeringen, het verzenden van Adobe [!DNL I/O Events] en het afhandelen van fouten afgehandeld.

<!-- TBD: Add the application diagram. -->

### Toepassingscode {#application-code}

De code van de douane moet slechts callback verstrekken die het plaatselijk beschikbare brondossier (`source.path`) neemt. De `rendition.path` is de locatie waar het uiteindelijke resultaat van een aanvraag voor de verwerking van middelen wordt geplaatst. De douanetoepassing gebruikt callback om de plaatselijk beschikbare brondossiers in een vertoningsdossier te veranderen gebruikend de binnen overgegaan naam (`rendition.path`). Een aangepaste toepassing moet naar `rendition.path` schrijven om een uitvoering te maken:

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

Een aangepaste toepassing heeft alleen betrekking op lokale bestanden. De [ SDK van Asset Compute ](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) behandelt het downloaden van het brondossier.

### Vertoning maken {#rendition-creation}

SDK roept een asynchrone [ functie van de vertoningscallback ](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) voor elke vertoning.

De callback functie heeft toegang tot de [ bron ](https://github.com/adobe/asset-compute-sdk#source) en [ vertoning ](https://github.com/adobe/asset-compute-sdk#rendition) voorwerpen. Het `source.path` bestaat al en is het pad naar de lokale kopie van het bronbestand. `rendition.path` is het pad waarin de verwerkte vertoning moet worden opgeslagen. Tenzij de [ disableSourceDownload vlag ](https://github.com/adobe/asset-compute-sdk#worker-options-optional) wordt geplaatst, moet de toepassing precies `rendition.path` gebruiken. Anders kan de SDK het weergavebestand niet vinden of identificeren en mislukt.

De overdreven vereenvoudiging van het voorbeeld wordt gedaan om de anatomie van een douanetoepassing te illustreren en te concentreren. De toepassing kopieert het bronbestand alleen naar de weergavebestemming.

Voor meer informatie over de parameters van de vertoningscallback, zie [ Asset Compute SDK API ](https://github.com/adobe/asset-compute-sdk#api-details).

### Uitvoeringen uploaden {#upload-rendition}

Nadat elke vertoning wordt gecreeerd en in een dossier met de weg opgeslagen die door `rendition.path` wordt verstrekt, uploadt [ Asset Compute SDK ](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) elke vertoning aan een wolkenopslag (of AWS of Azure). Een aangepaste toepassing krijgt meerdere uitvoeringen tegelijkertijd als en alleen als de binnenkomende aanvraag meerdere uitvoeringen bevat die verwijzen naar dezelfde toepassings-URL. Het uploaden naar de cloudopslag vindt plaats na elke uitvoering en voordat de callback voor de volgende uitvoering wordt uitgevoerd.

De `batchWorker()` heeft een ander gedrag. Alle uitvoeringen worden verwerkt en pas nadat ze zijn verwerkt, worden ze geüpload.

## [!DNL Adobe I/O] Gebeurtenissen {#aio-events}

De SDK verzendt Adobe [!DNL I/O Events] voor elke uitvoering. Deze gebeurtenissen zijn van het type `rendition_created` of `rendition_failed` afhankelijk van het resultaat. Voor meer informatie, zie [ asynchrone gebeurtenissen van Asset Compute ](api.md#asynchronous-events).

## [!DNL Adobe I/O] Gebeurtenissen ontvangen {#receive-aio-events}

De client pollt het Adobe [!DNL I/O Events] Journal op basis van de logica van het verbruik. De eerste journaal-URL is de URL die wordt opgegeven in de API-reactie van `/register` . Gebeurtenissen kunnen worden geïdentificeerd met de `requestId` die aanwezig is in de gebeurtenissen en die hetzelfde is als die wordt geretourneerd in `/process` . Elke vertoning heeft een aparte gebeurtenis die wordt verzonden zodra de vertoning is geüpload (of mislukt). Wanneer de client een overeenkomende gebeurtenis ontvangt, kan de client de resulterende uitvoeringen weergeven of op een andere manier afhandelen.

De bibliotheek van JavaScript [`asset-compute-client` ](https://github.com/adobe/asset-compute-client#usage) maakt de dagboekopiniepeiling eenvoudig gebruikend de `waitActivation()` methode om alle gebeurtenissen te krijgen.

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

Voor details op hoe te om dagboekgebeurtenissen te krijgen, zie Adobe [[!DNL I/O Events]  API ](https://developer.adobe.com/events/docs/guides/api/journaling_api/).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
