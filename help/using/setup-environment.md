---
title: Stel de ontwikkelomgeving in die vereist is voor  [!DNL Asset Compute Service]
description: De milieu opstelling van de ontwikkelaar voor  [!DNL Asset Compute Service]  beginnen en douanecode te creëren te testen.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: db38b9dc27505aa7e04cf58a646005fc2e0e8782
workflow-type: tm+mt
source-wordcount: '353'
ht-degree: 0%

---

# Een ontwikkelomgeving instellen {#create-dev-environment}

Volg deze vereisten en instructies om een installatie te maken waarmee u [!DNL Asset Compute Service] kunt ontwikkelen.

1. [ verkrijgt toegang en geloofsbrieven ](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) voor [!DNL Adobe Developer App Builder].

1. [ opstelling het lokale milieu ](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) en de vereiste hulpmiddelen.

1. Hier volgen nog enkele gereedschappen die u helpen probleemloos aan de slag te gaan met het ontwikkelen van:

   * [ Git ](https://git-scm.com/)
   * [ de Desktop van de Docker ](https://www.docker.com/get-started)
   * [ NodeJS ](https://nodejs.org) (v14 LTS, worden de oneven versies niet geadviseerd) en [ NPM ](https://www.npmjs.com). Gebruiker van OS X HomeBrew kan `brew install node` doen om beide te installeren. Anders, download het van de [ downloaden NodeJS pagina ](https://nodejs.org/en/)
   * winde die voor NodeJS goed is, beveelt Adobe [ Code van Visual Studio (de Code van VS) ](https://code.visualstudio.com) aan aangezien het gesteunde winde voor debugger is. U kunt om het even welke andere winde als coderedacteur gebruiken, maar geavanceerd gebruik (bijvoorbeeld, debugger) wordt nog niet gesteund
   * De nieuwste Adobe installeren [[!DNL aio-cli] ](https://github.com/adobe/aio-cli) (`aio`)
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Zorg ervoor om aan de [ eerste vereisten ](/help/using/understand-extensibility.md#prerequisites-and-provisioning) te voldoen

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Een App Builder-project instellen {#create-App-Builder-project}

1. Zorg ervoor dat er een systeembeheerder- of ontwikkelaarrol is in de [!DNL Experience Cloud] -organisatie. Een systeemadmin, in [ Admin Console ](https://adminconsole.adobe.com/overview), plaatst omhoog deze rol.

1. Logboek op [ Adobe Developer Console ](https://developer.adobe.com/console/user/servicesandapis). Zorg ervoor dat u deel uitmaakt van dezelfde [!DNL Experience Cloud] -organisatie als de [!DNL Experience Manager] as a [!DNL Cloud Service] -integratie. Voor meer informatie over Adobe Developer Console, ga naar [ documentatie van de Console ](https://developer.adobe.com/developer-console/docs/guides/).

1. [ creeer een project van App Builder ](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). Klik op **[!UICONTROL Create new project]** > **[!UICONTROL Project from template]** . Selecteer App Builder. Er wordt een nieuw App Builder-project gemaakt met twee werkruimten: `Production` en `Stage` . Voeg desgewenst extra werkruimten toe, bijvoorbeeld `Development` .

1. Selecteer in het App Builder-project een werkruimte en abonneer de services die nodig zijn voor Asset Compute. Klik **toevoegen aan Project** > **API** en voeg `Asset Compute` toe, `IO Events`, en `IO Events Management` diensten. Wanneer de eerste API wordt toegevoegd, wordt gevraagd om een persoonlijke sleutel te maken. Sla deze gegevens op uw computer op omdat u deze sleutel nodig hebt om uw aangepaste toepassing met het ontwikkelaarsgereedschap te testen.

   >[!NOTE]
   >
   >JWT is afgekeurd en persoonlijke sleutel kan niet worden gedownload. Terwijl wij aan het bijwerken van de testhulpmiddelen werken, merk op dat de douanearbeiders die met OAuth worden gecreeerd kunnen worden opgesteld maar de apparaten niet zouden werken.

## Volgende stap {#next-step}

Met uw milieu opstelling, bent u klaar om [ een douanetoepassing ](develop-custom-application.md) tot stand te brengen.

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
