---
title: Testen en fouten opsporen [!DNL Asset Compute Service] aangepaste toepassing
description: Testen en fouten opsporen [!DNL Asset Compute Service] aangepaste toepassing.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '775'
ht-degree: 0%

---

# Een aangepaste toepassing testen en fouten hierin opsporen {#test-debug-custom-worker}

## Eenheidstests voor een aangepaste toepassing uitvoeren {#test-custom-worker}

Installeren [Docker Desktop](https://www.docker.com/get-started) op uw computer. Als u een aangepaste worker wilt testen, voert u de volgende opdracht uit in de hoofdmap van de toepassing:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Met deze opdracht wordt een testframework voor aangepaste eenheden voor Asset compute-toepassingsacties in het project uitgevoerd, zoals hieronder wordt beschreven. Het wordt omhoog verbonden door een configuratie in `package.json` bestand. Het is ook mogelijk om JavaScript eenheidstests zoals Jest te hebben. De `aio app test` voert beide uit.

De [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) insteekmodule is ingesloten als een ontwikkelafhankelijkheid in de aangepaste toepassing, zodat deze niet hoeft te worden geïnstalleerd op build-/testsystemen.

### Testframework voor toepassingseenheid {#unit-test-framework}

Met het testframework voor de Asset compute-toepassingseenheid kunt u toepassingen testen zonder code te schrijven. Het is afhankelijk van het principe van de bron naar het weergavebestand van toepassingen. Er moet een bepaalde bestands- en mapstructuur worden ingesteld om testcase te definiëren met bronbestanden voor tests, optionele parameters, verwachte uitvoeringen en aangepaste validatiescripts. Standaard worden de uitvoeringen vergeleken voor de gelijkheid van bytes. Bovendien kunnen externe HTTP-services eenvoudig worden gecontroleerd met behulp van eenvoudige JSON-bestanden.

### Tests toevoegen {#add-tests}

Tests worden verwacht binnen de `test` map op het hoofdniveau van het project. De testgevallen voor elke toepassing moeten zich in het pad bevinden `test/asset-compute/<worker-name>`, met één map voor elk testgeval:

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

Kijk eens naar [voorbeeld aangepaste toepassingen](https://github.com/adobe/asset-compute-example-workers/) voor enkele voorbeelden . Hieronder vindt u een gedetailleerde referentie.

### Uitvoer testen {#test-output}

De `build` in de hoofdmap van de Adobe Developer App Builder-app staan de gedetailleerde testresultaten en logboekbestanden van de aangepaste toepassing. Deze details worden ook weergegeven in de uitvoer van het dialoogvenster `aio app test` gebruiken.

### Externe diensten koppelen {#mock-external-services}

U kunt externe de dienstvraag binnen uw acties simuleren door te creëren `mock-<HOST_NAME>.json` bestanden voor uw testscenario&#39;s, waarbij HOST_NAME de specifieke host is die u wilt imiteren. Een geval van het voorbeeldgebruik is een toepassing die een afzonderlijke vraag aan S3 maakt. De nieuwe teststructuur ziet er als volgt uit:

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

Het mock-bestand is een http-reactie met JSON-indeling. Zie voor meer informatie [deze documentatie](https://www.mock-server.com/mock_server/creating_expectations.html). Als er meerdere hostnamen zijn om te controleren, definieert u meerdere `mock-<mocked-host>.json` bestanden. Hieronder ziet u een voorbeeldmodelbestand voor `google.com` benoemd `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

Het voorbeeld `worker-animal-pictures` bevat een [mock-bestand](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) voor de Wikimedia-service waarmee het communiceert.

#### Bestanden delen over testgevallen {#share-files-across-test-cases}

Adobe raadt u aan relatieve symbolen te gebruiken als u deze deelt `file.*`, `params.json` of `validate` manuscripten over veelvoudige tests. Ze worden ondersteund met Git. Geef uw gedeelde bestanden een unieke naam, omdat u mogelijk andere bestanden hebt. In het onderstaande voorbeeld mengen en vergelijken de tests een paar gedeelde bestanden en hun eigen bestanden:

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### Verwachte fouten testen {#test-unexpected-errors}

Fouttestgevallen mogen geen verwachte `rendition.*` en moet de verwachte `errorReason` in de `params.json` bestand.

>[!NOTE]
>
>Indien een testcase geen verwacht geval bevat `rendition.*` en definieert het verwachte `errorReason` in de `params.json` bestand, wordt aangenomen dat het een fout betreft met `errorReason`.

Fout bij testen hoofdletterstructuur:

```json
<error_test_case>/
    file.jpg
    params.json
```

Parameterbestand met reden van fout:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Zie een volledige lijst en beschrijving van [Asset compute-foutredenen](https://github.com/adobe/asset-compute-commons#error-reasons).

## Fouten opsporen in een aangepaste toepassing {#debug-custom-worker}

De volgende stappen tonen hoe u uw douanetoepassing kunt zuiveren gebruikend de Code van Visual Studio. Het staat voor het zien van levende logboeken, raakbreekpunten en stap door code evenals het levende opnieuw laden van lokale codeveranderingen na elke activering toe.

De `aio` veel van deze stappen worden automatisch uit de verpakking gezet. Ga naar de sectie Fouten opsporen in de toepassing in het dialoogvenster [Adobe Developer App Builder-documentatie](https://developer.adobe.com/app-builder/docs/getting_started/first_app). Tot nu toe bevatten de onderstaande stappen een tijdelijke oplossing.

1. De nieuwste [wskdebug](https://github.com/apache/openwhisk-wskdebug) van GitHub en het facultatieve [ngrot](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Voeg in het JSON-bestand aanvullende instellingen toe aan uw gebruikersinstellingen. Het houdt het gebruiken van oude debugger van de Code van Visual Studio. De nieuwe heeft [enkele kwesties](https://github.com/apache/openwhisk-wskdebug/issues/74) met wskdebug: `"debug.javascript.usePreview": false`.
1. Sluit alle instanties van apps die zijn geopend via `aio app run`.
1. De nieuwste code implementeren met `aio app deploy`.
1. Alleen Asset compute ontwikkelen uitvoeren met `aio asset-compute devtool`. Houd het open.
1. In de Redacteur van de Code van Visual Studio, voeg volgende toe zuivert configuratie aan uw `launch.json`:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   De `ACTION NAME` uit de uitvoer van `aio app deploy`.

1. Selecteren `wskdebug worker` van de looppas/zuivert configuratie en druk het speel pictogram. Wacht tot het begint totdat het wordt weergegeven **[!UICONTROL Gereed voor activering]** in de **[!UICONTROL Foutopsporingsconsole]** venster.

1. Klikken **[!UICONTROL run]** in het gereedschap Ontwikkelen. U kunt de acties zien die in de redacteur van de Code van Visual Studio lopen en het logboekbegin die toont.

1. Stel een onderbrekingspunt in de code in. Dan loop ik opnieuw en het zou moeten slaan.

Eventuele codewijzigingen worden in real-time geladen en worden van kracht zodra de volgende activering plaatsvindt.

>[!NOTE]
>
>Er zijn twee activeringen voor elke aanvraag in aangepaste toepassingen. Het eerste verzoek is een webactie die zichzelf asynchroon aanroept in de SDK-code. De tweede activering is de activering die raakt aan uw code.
