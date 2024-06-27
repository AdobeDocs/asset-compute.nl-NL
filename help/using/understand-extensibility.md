---
title: Begrijp over uitbreiden [!DNL Asset Compute Service]
description: Wanneer en hoe breidt u de [!DNL Asset Compute Service] functionaliteit voor aangepaste verwerking van elementen.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 0%

---

# Inleiding tot uitbreidbaarheid {#introduction-to-extensibilty}

Aan veel vereisten voor vertoningen, zoals het omzetten in indelingen en het vergroten of verkleinen van afbeeldingen, wordt voldaan door [Profielen verwerken in [!DNL Experience Manager] als [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview). De complexere bedrijfsvereisten kunnen een douane-gecreeerde oplossing vereisen die de behoeften van een organisatie aanpast. [!DNL Asset Compute Service] kan worden uitgebreid door aangepaste toepassingen te maken die worden aangeroepen vanuit Process Profiles in [!DNL Experience Manager]. Deze aangepaste toepassingen kunnen [ondersteunde gebruiksgevallen](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] is alleen beschikbaar voor gebruik met [!DNL Experience Manager] als [!DNL Cloud Service].

De aangepaste toepassingen zijn zonder kop [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) apps. Uitbreiden [!DNL Asset Compute Service] met aangepaste toepassingen eenvoudig wordt gemaakt via de [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk) en Adobe Developer App Builder Developer Tool. Deze hulpmiddelen laten ontwikkelaars zich op bedrijfslogica concentreren. Aangepaste toepassingen maken is net zo eenvoudig als het maken van een Adobe zonder normale serverless [!DNL I/O Runtime] handeling. Het is één JavaScript-functie Node.js. De [basis, aangepast toepassingsvoorbeeld](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) illustreert dit.

## Vereisten voor vereisten en voorzieningen {#prerequisites-and-provisioning}

Zorg ervoor dat u aan de volgende voorwaarden voldoet:

* Adobe Developer App Builder-gereedschappen zijn op uw computer geïnstalleerd.
* An [!DNL Experience Cloud] organisatie. Ga voor meer informatie naar [App Builder-reis starten](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* De Ervaringsorganisatie moet beschikken over [!DNL Experience Manager] als [!DNL Cloud Service] ingeschakeld.
* [!DNL Adobe Experience Cloud] organisatie maakt deel uit van de [!DNL Adobe Developer App Builder] ontwikkelaar neigt peek programma . Ga naar [toegangsaanvraag indienen](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Verzeker een ontwikkelaarrol of beheerdertoestemmingen in de organisatie voor de ontwikkelaar.
* Ervoor zorgen dat Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli) is lokaal geïnstalleerd.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
