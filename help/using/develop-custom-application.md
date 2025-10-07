---
title: Ontwikkelen voor  [!DNL Asset Compute Service]
description: Creeer douanetoepassingen gebruikend  [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: 94fd8c0888185f64825046b7999655e9501a71fe
workflow-type: tm+mt
source-wordcount: '1489'
ht-degree: 0%

---

# Een aangepaste toepassing ontwikkelen {#develop}

Voordat u begint met het ontwikkelen van een aangepaste toepassing:

* Zorg ervoor dat aan alle [ eerste vereisten ](/help/using/understand-extensibility.md#prerequisites-and-provisioning) wordt voldaan.
* Installeer de [ vereiste softwarehulpmiddelen ](/help/using/setup-environment.md#create-dev-environment).
* Zie [ opstelling uw milieu ](setup-environment.md) om ervoor te zorgen u bereid bent om een douanetoepassing tot stand te brengen.

## Een aangepaste toepassing maken {#create-custom-application}

Zorg ervoor om [ Adobe lucht-cli ](https://github.com/adobe/aio-cli) te hebben plaatselijk geïnstalleerd.

1. Om een douanetoepassing tot stand te brengen, [ creeer een project van App Builder ](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). Voer hiervoor `aio app init <app-name>` in de terminal uit.

   Als u niet reeds hebt het programma geopend, zet dit bevel browser ertoe aan vragend u om in [ Adobe Developer Console ](https://developer.adobe.com/console/user/servicesandapis) met uw Adobe ID te ondertekenen. Zie [ hier ](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) voor meer informatie bij het ondertekenen binnen van de klem.

   Adobe raadt u aan zich eerst aan te melden. Als u kwesties hebt, dan volg de instructies [ om een app tot stand te brengen zonder het programma te openen ](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Nadat u zich hebt aangemeld, volgt u de aanwijzingen in de CLI en selecteert u de `Organization` , `Project` en `Workspace` die u voor de toepassing wilt gebruiken. Kies het project en de werkruimte die u creeerde toen u [ opstelling uw milieu ](setup-environment.md). Selecteer `Which extension point(s) do you wish to implement ?` bij de aanwijzing `DX Asset Compute Worker` :

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. Selecteer `Which Adobe I/O App features do you want to enable for this project?` wanneer dit wordt gevraagd bij `Actions` . Schakel de optie `Web Assets` uit als webelementen verschillende verificatie- en autorisatiecontroles gebruiken.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Selecteer `Which type of sample actions do you want to create?` bij de aanwijzing `Adobe Asset Compute Worker` :

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Volg de rest herinneringen en open de nieuwe toepassing in de Code van Visual Studio (of uw favoriete coderedacteur). Het bevat de basiscode en voorbeeldcode voor een aangepaste toepassing.

   Lees hier over de [ belangrijkste componenten van een App Builder app ](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   De hefboomwerkingen van de malplaatjetoepassing Adobe [ Asset Compute SDK ](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) voor het uploaden, het downloaden, en het orchestreren van toepassingsuitvoeringen zodat moeten de ontwikkelaars slechts de logica van de douanetoepassing uitvoeren. In de map `actions/<worker-name>` voegt u in het bestand `index.js` de aangepaste toepassingscode toe.

Zie [ de toepassingen van de voorbeelddouane ](#try-sample) voor voorbeelden en ideeën voor douanetoepassingen.

### Referenties toevoegen {#add-credentials}

Wanneer u zich aanmeldt bij het maken van de toepassing, worden de meeste App Builder-gegevens verzameld in uw ENV-bestand. Voor het gebruik van het ontwikkelaarsgereedschap zijn echter aanvullende gegevens vereist.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Inloggegevens voor opslag van het gereedschap Ontwikkelaar {#developer-tool-credentials}

Het hulpmiddel voor ontwikkelaars om aangepaste apps met behulp van [!DNL Asset Compute service] te evalueren, vereist het gebruik van een container voor cloudopslag. Deze container is essentieel voor het opslaan van testbestanden en voor het ontvangen en presenteren van uitvoeringen die door de apps worden gemaakt.

>[!NOTE]
>
>Deze container staat los van de cloudopslag van [!DNL Adobe Experience Manager] als een [!DNL Cloud Service] . Dit geldt alleen voor het ontwikkelen en testen met het Asset Compute-hulpprogramma voor ontwikkelaars.

Zorg ervoor om toegang tot a [ gesteunde container van de wolkenopslag ](https://github.com/adobe/asset-compute-devtool#prerequisites) te hebben. Deze container wordt gebruikt collectief door diverse ontwikkelaars voor verschillende projecten wanneer noodzakelijk.

#### Referenties toevoegen aan ENV-bestand {#add-credentials-env-file}

Voeg de volgende gegevens voor het ontwikkelprogramma in het `.env` -bestand in. Het bestand bevindt zich in de hoofdmap van uw App Builder-project:
<!--
1. Add the absolute path to the private key file created while adding services to your App Builder Project:

    ```conf
    ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
    ```

   >[!NOTE]
   >
   >JWT is deprecated and Private Key is not available for download. While we are working on updating the testing tools, note that custom workers created using OAuth can be deployed but devtools would not work.
-->
1. Download het bestand van de Adobe Developer Console. Ga naar de hoofdmap van het project en klik op Alles downloaden rechtsboven in het scherm. Het bestand wordt gedownload met `<namespace>-<workspace>.json` als bestandsnaam. Voer een van de volgende handelingen uit:

   * Wijzig de naam van het bestand in `console.json` en verplaats het in de hoofdmap van het project.
   * U kunt ook het absolute pad naar het JSON-bestand voor Adobe Developer Console-integratie toevoegen. Dit dossier is het zelfde [`console.json` ](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) dossier dat in uw projectwerkruimte wordt gedownload.

     ```conf
     ASSET_COMPUTE_INTEGRATION_FILE_PATH=
     ```

1. Voeg S3 of Azure opslaggeloofsbrieven toe. U hebt slechts toegang tot één cloudopslagoplossing nodig.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>Het bestand `config.json` bevat referenties. Voeg vanuit uw project het JSON-bestand toe aan uw `.gitignore` -bestand om te voorkomen dat het wordt gedeeld. Hetzelfde geldt voor `.env` - en `.aio` -bestanden.

## De toepassing uitvoeren {#run-custom-application}

Alvorens de toepassing met het de ontwikkelaarshulpmiddel van Asset Compute uit te voeren, vorm behoorlijk de [ geloofsbrieven ](#developer-tool-credentials).

Gebruik de opdracht `aio app run` om de toepassing uit te voeren in het gereedschap Ontwikkelaar. De toepassing implementeert de handeling naar Adobe [!DNL I/O Runtime] en start het hulpprogramma voor ontwikkeling op uw lokale computer. Dit hulpmiddel wordt gebruikt om toepassingsverzoeken tijdens ontwikkeling te testen. Hier volgt een voorbeeld van een verzoek om uitvoering:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>Gebruik de markering `--local` niet met de opdracht `run` . De functie werkt niet met aangepaste toepassingen van [!DNL Asset Compute] en het Asset Compute-ontwikkelaarsgereedschap. Aangepaste toepassingen worden aangeroepen door de service [!DNL Asset Compute] die geen toegang heeft tot handelingen die op de lokale computers van de ontwikkelaar worden uitgevoerd.

Zie [ hier ](test-custom-application.md) om uw toepassing te testen en te zuiveren. Wanneer u wordt gebeëindigd ontwikkelend uw douanetoepassing, [ stelt uw douanetoepassing ](deploy-custom-application.md) op.

## Probeer de voorbeeldtoepassing van Adobe {#try-sample}

Hieronder vindt u voorbeelden van aangepaste toepassingen:

* [ worker-basic ](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [ arbeider-dier-beelden ](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Aangepaste toepassing sjabloon {#template-custom-application}

[ worker-basic ](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) is een malplaatjetoepassing. Het produceert een vertoning door het brondossier eenvoudig te kopiëren. De inhoud van deze toepassing is de sjabloon die wordt ontvangen wanneer u `Adobe Asset Compute` kiest bij het maken van de audio-app.

Het toepassingsdossier, [`worker-basic.js` ](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) gebruikt [`asset-compute-sdk` ](https://github.com/adobe/asset-compute-sdk#overview) om het brondossier te downloaden, elke vertoningsverwerking te ordenen, en de resulterende vertoningen terug naar wolkenopslag te uploaden.

De [`renditionCallback` ](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) die binnen de toepassingscode wordt bepaald is waar te om alle logica van de toepassingsverwerking uit te voeren. De rendercallback in `worker-basic` kopieert eenvoudig de inhoud van het bronbestand naar het weergavebestand.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Een externe API aanroepen {#call-external-api}

In de toepassingscode kunt u externe API-aanroepen maken die u helpen bij de verwerking van toepassingen. Hieronder ziet u een voorbeeld van een toepassingsbestand dat een externe API aanroept.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Bijvoorbeeld, [`worker-animal-pictures` ](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) doet een haalverzoek aan een statische URL van Wikimedia gebruikend de [`node-httptransfer` ](https://github.com/adobe/node-httptransfer#node-httptransfer) bibliotheek.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Aangepaste parameters doorgeven {#pass-custom-parameters}

U kunt aangepaste gedefinieerde parameters doorgeven via de weergaveobjecten. Binnen de toepassing kan naar deze instructies worden verwezen in [`rendition` instructies ](https://github.com/adobe/asset-compute-sdk#rendition) . Een voorbeeld van een weergaveobject is:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Een voorbeeld van een toepassingsbestand dat toegang heeft tot een aangepaste parameter is het volgende:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

`example-worker-animal-pictures` gaat een douaneparameter [`animal` over ](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) om te bepalen welk dossier van Wikimedia moet halen.

## Ondersteuning voor verificatie en autorisatie {#authentication-authorization-support}

Standaard worden bij aangepaste Asset Compute-toepassingen verificatie- en verificatiecontroles voor het App Builder-project geleverd. Ingeschakeld door de `require-adobe-auth` -annotatie in te stellen op `true` in de `manifest.yml` .

### Andere Adobe API&#39;s openen {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Voeg de API-services toe aan de [!DNL Asset Compute] Console-werkruimte die in Setup is gemaakt. Deze services maken deel uit van het JWT-toegangstoken dat door [!DNL Asset Compute Service] wordt gegenereerd. Het token en andere gegevens zijn toegankelijk binnen het object application `params` .

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Referenties doorgeven voor systemen van derden {#pass-credentials-for-tp}

Om geloofsbrieven voor andere externe diensten te behandelen, ga hen als standaardparameters over de acties. Ze worden automatisch tijdens de doortocht versleuteld. Voor meer informatie, zie [ creërend acties in de de ontwikkelaarsgids van Adobe I/O Runtime ](https://developer.adobe.com/runtime/docs/guides/using/creating_actions/). Stel ze vervolgens in met behulp van omgevingsvariabelen tijdens de implementatie. Deze parameters zijn toegankelijk in het `params` -object binnen de handeling.

Stel de standaardparameters in in de `inputs` in de `manifest.yml` :

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

De expressie `$VAR` leest de waarde van een omgevingsvariabele met de naam `VAR` .

Tijdens het ontwikkelen kunt u de waarde in het lokale `.env` -bestand toewijzen. De reden hiervoor is dat `aio` omgevingsvariabelen automatisch importeert uit `.env` -bestanden, samen met de variabelen die zijn ingesteld door de initiërende shell. In dit voorbeeld ziet het `.env` -bestand er als volgt uit:

```CONF
#...
SECRET_KEY=secret-value
```

Voor productieplaatsing zou men de milieuvariabelen in het systeem van CI kunnen plaatsen, bijvoorbeeld gebruikend geheimen in Acties GitHub. Tot slot hebt u als volgt toegang tot de standaardparameters binnen de toepassing:

```javascript
const key = params.secretKey;
```

## Toepassingen vergroten/verkleinen {#sizing-workers}

Een toepassing loopt in een container in Adobe [!DNL I/O Runtime] met [ grenzen ](https://developer.adobe.com/runtime/docs/guides/using/system_settings/) die door `manifest.yml` kunnen worden gevormd:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

Vanwege de uitgebreide verwerking door Asset Compute-toepassingen moet u deze limieten aanpassen voor optimale prestaties (groot genoeg voor de verwerking van binaire elementen) en efficiëntie (zonder bronnen te verspillen vanwege ongebruikt containergeheugen).

De standaardtime-out voor acties in Runtime is een minuut, maar deze kan worden verhoogd door de limiet `timeout` in milliseconden in te stellen. Verhoog deze tijd als u grotere bestanden wilt verwerken. Houd rekening met de totale tijd die nodig is om de bron te downloaden, het bestand te verwerken en de vertoning te uploaden. Als een handeling uitvalt, dat wil zeggen dat de activering niet wordt geretourneerd vóór de opgegeven time-outlimiet, verwijdert Runtime de container en gebruikt deze niet opnieuw.

Asset Compute-toepassingen zijn van nature gebonden aan netwerk- en schijfinvoer of uitvoer. Het bronbestand moet eerst worden gedownload. De verwerking is vaak bronintensief en de resulterende uitvoeringen worden dan opnieuw geüpload.

U kunt het geheugen dat aan een actiecontainer wordt toegewezen, in megabytes opgeven met de parameter `memorySize` . Momenteel bepaalt deze parameter ook hoeveel CPU toegang heeft tot de container, en het belangrijkste is dat het een belangrijk element is van de gebruikskosten van Runtime (grotere containers kosten meer). Gebruik hier een grotere waarde wanneer uw verwerking meer geheugen of CPU vereist maar zorg ervoor dat u geen bronnen verspilt, aangezien hoe groter de containers zijn, hoe lager de totale doorvoer is.

Bovendien is het mogelijk om de gelijktijdig uitgevoerde handelingen in een container te beheren met de instelling `concurrency` . Deze instelling is het aantal gelijktijdige activeringen dat een enkele container (van dezelfde handeling) krijgt. In dit model, is de actiecontainer als server Node.js die veelvoudige gezamenlijke verzoeken, tot die grens ontvangt. De standaardwaarde `memorySize` in de runtime is ingesteld op 200 MB, ideaal voor kleinere App Builder-handelingen. Voor Asset Compute-toepassingen kan deze standaardinstelling overdreven zijn vanwege de zwaardere lokale verwerking en het hogere schijfgebruik. Sommige toepassingen, afhankelijk van hun implementatie, werken mogelijk ook niet goed met gelijktijdige activiteit. De Asset Compute SDK zorgt ervoor dat activeringen worden gescheiden door bestanden naar verschillende unieke mappen te schrijven.

Test toepassingen om de optimale getallen voor `concurrency` en `memorySize` te vinden. Grotere containers = de hogere geheugengrens kon voor meer gelijktijdige uitvoering toestaan maar kon ook verkwistend voor lager verkeer zijn.
