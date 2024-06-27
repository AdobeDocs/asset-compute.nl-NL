---
title: Problemen oplossen [!DNL Asset Compute Service]
description: Aangepaste toepassingen oplossen en fouten hierin opsporen met [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '273'
ht-degree: 0%

---

# Problemen oplossen {#troubleshoot}

Enkele algemene tips voor het oplossen van problemen die u kunnen helpen problemen op te lossen met de service Asset compute zijn:

* Zorg ervoor dat de JavaScript-toepassing niet vastloopt bij het opstarten. Dergelijke neerstortingen zijn gewoonlijk verwant aan een ontbrekende bibliotheek of een gebiedsdeel.
* Zorg ervoor dat in de toepassing wordt verwezen naar alle afhankelijkheden die moeten worden geïnstalleerd `package.json` bestand.
* Verzeker om het even welke fouten die uit schoonmaakbeurt bij mislukking kunnen voortkomen niet hun eigen fouten produceren die het originele probleem verbergen.

* Wanneer u het ontwikkelaarsgereedschap voor het eerst start met een nieuwe [!DNL Asset Compute Service] het eerste verwerkingsverzoek mislukt als het Asset compute Events Journal niet volledig is ingesteld. Wacht enige tijd op het dagboek aan opstelling alvorens een ander verzoek te verzenden.
* Alle vereiste API&#39;s-Asset compute, Adobe [!DNL I/O Events], Gebeurtenisbeheer en Runtime zijn opgenomen in uw Adobe [!DNL `I/O Project`] en Workspace `/register` of `/process` aanvraagfouten.

## Problemen aanmelden via Adobe [!DNL aio-cli] {#login-via-aio-cli}

Als u problemen hebt met het aanmelden bij de [!DNL Adobe Developer Console] [via de Adobe [!DNL aio-cli]](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)Voeg vervolgens handmatig de vereiste gegevens toe voor het ontwikkelen, testen en implementeren van uw aangepaste toepassing:

1. Ga naar uw Adobe Developer App Builder-project en -werkruimte op het tabblad [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) en drukken **[!UICONTROL Downloaden]** in de rechterbovenhoek. Open dit bestand en sla deze JSON op een veilige plaats op uw computer op.

1. Navigeer naar het ENV-bestand in uw Adobe Developer App Builder-toepassing.

1. De Adobe toevoegen [!DNL I/O Runtime] referenties. De Adobe ophalen [!DNL I/O Runtime] referenties van de gedownloade JSON. De aanmeldingsgegevens zijn onder `project.workspace.services.runtime`. Voeg de [!DNL Adobe I/O] Runtime referenties in `AIO_runtime_XXX` variabelen:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Voeg het absolute pad toe aan de gedownloade JSON in Stap 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. De rest van de [vereiste referenties](develop-custom-application.md) nodig is voor het ontwikkelaarsgereedschap.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
