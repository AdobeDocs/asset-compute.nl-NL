---
title: Problemen oplossen  [!DNL Asset Compute Service]
description: Los en zuiver douanetoepassingen problemen op gebruikend  [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '272'
ht-degree: 0%

---

# Problemen oplossen {#troubleshoot}

Enkele algemene tips voor het oplossen van problemen die u kunnen helpen problemen op te lossen met Asset Compute Service zijn:

* Zorg ervoor dat de JavaScript-toepassing niet vastloopt bij het opstarten. Dergelijke neerstortingen zijn gewoonlijk verwant aan een ontbrekende bibliotheek of een gebiedsdeel.
* Zorg ervoor dat in het `package.json` -bestand van de toepassing wordt verwezen naar alle afhankelijkheden die moeten worden geïnstalleerd.
* Verzeker om het even welke fouten die uit schoonmaakbeurt bij mislukking kunnen voortkomen niet hun eigen fouten produceren die het originele probleem verbergen.

* Wanneer u het ontwikkelaarsprogramma voor het eerst start met een nieuwe [!DNL Asset Compute Service] -integratie, kan dit resulteren in een mislukking van de eerste verwerkingsaanvraag als het Asset Compute Events Journal niet volledig is ingesteld. Wacht enige tijd op het dagboek aan opstelling alvorens een ander verzoek te verzenden.
* Zorg ervoor dat alle vereiste API&#39;s (Asset Compute, Adobe [!DNL I/O Events], Events Management en Runtime) zijn opgenomen in uw Adobe [!DNL `I/O Project`] en Workspace om fouten in `/register` - of `/process` -aanvragen te voorkomen.

## Problemen aanmelden via Adobe [!DNL aio-cli] {#login-via-aio-cli}

Als u kwesties het programma openen aan [!DNL Adobe Developer Console] [&#x200B; door Adobe  [!DNL aio-cli] &#x200B;](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) hebt, dan voeg manueel de geloofsbrieven toe die voor het ontwikkelen, het testen, en het opstellen van uw douanetoepassing worden vereist:

1. Navigeer aan uw project en werkruimte van Adobe Developer App Builder op [&#x200B; Adobe Developer Console &#x200B;](https://developer.adobe.com/console/user/servicesandapis) en druk **[!UICONTROL Download]** van de hoger-juiste hoek. Open dit bestand en sla deze JSON op een veilige plaats op uw computer op.

1. Navigeer naar het ENV-bestand in uw Adobe Developer App Builder-toepassing.

1. Voeg de Adobe [!DNL I/O Runtime] -referenties toe. Haal de Adobe [!DNL I/O Runtime] -gegevens op van de gedownloade JSON. De referenties zijn onder `project.workspace.services.runtime` . Voeg de [!DNL Adobe I/O] Runtime geloofsbrieven in de `AIO_runtime_XXX` variabelen toe:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Voeg het absolute pad toe aan de gedownloade JSON in Stap 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Opstelling de rest van [&#x200B; vereiste geloofsbrieven &#x200B;](develop-custom-application.md) nodig voor het ontwikkelaarshulpmiddel.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
