---
title: Stel  [!DNL Asset Compute Service]  douanetoepassing op
description: Stel  [!DNL Asset Compute Service]  douanetoepassing op.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '170'
ht-degree: 0%

---

# Een aangepaste toepassing implementeren {#deploy-custom-application}

Om uw toepassing op te stellen, gebruik [ lucht app ](https://github.com/adobe/aio-cli#aio-appdeploy) bevel opstelt. In de terminal geeft de opdracht een URL weer om toegang te krijgen tot de aangepaste toepassing. De URL heeft de notatie `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]` .

Om zelfde URL te krijgen zonder de toepassing opnieuw op te stellen, gebruik [`aio app get-url` ](https://github.com/adobe/aio-cli#aio-app-get-url-action) bevel.

Gebruik URL in a [ het Profiel van de Verwerking in  [!DNL Experience Manager]  als a  [!DNL Cloud Service] ](https://experienceleague.adobe.com/nl/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use) om uw toepassing met [!DNL Experience Manager] als a [!DNL Cloud Service] te integreren.

Zorg ervoor dat uw App Builder-project en -werkruimte overeenkomen met de [!DNL Experience Manager] -omgeving als een [!DNL Cloud Service] -omgeving waarin u de handeling wilt gebruiken. Het heeft verschillende omgevingen voor ontwikkeling, staging en productie. U kunt de omgeving controleren door `AIO_runtime_*` -referenties te controleren die in het ENV-bestand in de hoofdmap van de Adobe Developer App Builder-toepassing zijn gedefinieerd. Als u bijvoorbeeld wilt implementeren in een `Stage` -werkruimte, heeft de `AIO_runtime_namespace` de indeling `xxxxxx_xxxxxxxxx_stage` . Als u wilt integreren met [!DNL Experience Manager] als een [!DNL Cloud Service] productieomgeving, gebruikt u toepassings-URL&#39;s uit uw Adobe Developer App Builder `Production` -werkruimte.

>[!CAUTION]
>
>Gebruik geen persoonlijke werkruimte in kritieke [!DNL Experience Manager] omgevingen.

>[!MORELIKETHIS]
>
>* [ Begrijp en beheer milieu&#39;s in  [!DNL Experience Manager]  als a  [!DNL Cloud Service] ](https://experienceleague.adobe.com/nl/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments).
