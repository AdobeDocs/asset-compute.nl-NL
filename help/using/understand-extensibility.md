---
title: Begrijp over uitbreiden  [!DNL Asset Compute Service]
description: Wanneer en hoe te om de  [!DNL Asset Compute Service]  functionaliteit uit te breiden om de verwerking van douaneactiva te doen.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 0%

---

# Inleiding tot uitbreidbaarheid {#introduction-to-extensibilty}

Vele vertoningsvereisten zoals het omzetten in formaten en het resizing van beelden worden gericht door [&#x200B; Profielen van de Verwerking in  [!DNL Experience Manager]  als a  [!DNL Cloud Service] &#x200B;](https://experienceleague.adobe.com/nl/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview). De complexere bedrijfsvereisten kunnen een douane-gecreeerde oplossing vereisen die de behoeften van een organisatie aanpast. [!DNL Asset Compute Service] kan worden uitgebreid door aangepaste toepassingen te maken die worden aangeroepen vanuit Procesprofielen in [!DNL Experience Manager] . Deze douanetoepassingen behandelen na de [&#x200B; gesteunde gebruiksgevallen &#x200B;](https://experienceleague.adobe.com/nl/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] is alleen beschikbaar voor gebruik met [!DNL Experience Manager] as a [!DNL Cloud Service] .

De douanetoepassingen zijn headless [&#x200B; Adobe Developer App Builder &#x200B;](https://github.com/AdobeDocs/app-builder) apps. Het uitbreiden [!DNL Asset Compute Service] met douanetoepassingen wordt eenvoudig gemaakt door [&#x200B; Asset Compute SDK &#x200B;](https://github.com/adobe/asset-compute-sdk) en het ontwikkelaarswerktuig van Adobe Developer App Builder. Deze hulpmiddelen laten ontwikkelaars zich op bedrijfslogica concentreren. Het maken van aangepaste toepassingen is net zo eenvoudig als het maken van een Adobe-handeling zonder normale serverless [!DNL I/O Runtime] . Het is één Node.js JavaScript functie. Het [&#x200B; basis voorbeeld van de douanetoepassing &#x200B;](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) illustreert het.

## Vereisten voor vereisten en voorzieningen {#prerequisites-and-provisioning}

Zorg ervoor dat u aan de volgende voorwaarden voldoet:

* Adobe Developer App Builder-gereedschappen zijn op uw computer geïnstalleerd.
* Een [!DNL Experience Cloud] -organisatie. Voor meer informatie, ga naar [&#x200B; Begin uw Reis van App Builder &#x200B;](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* De Experience Organization moet [!DNL Experience Manager] hebben als een [!DNL Cloud Service] ingeschakeld.
* [!DNL Adobe Experience Cloud] -organisatie maakt deel uit van het [!DNL Adobe Developer App Builder] developer nieak peek-programma. Ga naar [&#x200B; hoe te om voor toegang &#x200B;](https://developer.adobe.com/app-builder/docs/overview/getting_access) toe te passen.
* Verzeker een ontwikkelaarrol of beheerdertoestemmingen in de organisatie voor de ontwikkelaar.
* Controleer of Adobe [[!DNL aio-cli] &#x200B;](https://github.com/adobe/aio-cli) lokaal is geïnstalleerd.

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
