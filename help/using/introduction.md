---
title: Inleiding tot de [!DNL Asset Compute Service]
description: "[!DNL Asset Compute Service] is een service voor de verwerking van eigen middelen in de cloud die de complexiteit vermindert en de schaalbaarheid verbetert."
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '278'
ht-degree: 0%

---

# Overzicht van [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] is de scalable en verlengbare dienst van [!DNL Adobe Experience Cloud] om digitale elementen te verwerken. Het kan beeld, video, document, en andere dossierformaten in verschillende vertoningen met inbegrip van duimnagels, gehaalde tekst en meta-gegevens, en archieven omzetten.

Ontwikkelaars hebben de mogelijkheid om aangepaste elementtoepassingen (ook wel aangepaste workers genoemd) in te sluiten voor het afhandelen van aangepaste gebruiksgevallen. De dienst werkt aan de Adobe [!DNL I/O Runtime]. Het kan worden uitgebreid door [!DNL Adobe Developer App Builder] toepassingen zonder kop geschreven in Node.js. Ze kunnen aangepaste bewerkingen uitvoeren, zoals externe API&#39;s aanroepen om afbeeldingsbewerkingen uit te voeren of gebruikmaken van [!DNL Adobe Sensei] ondersteuning.

[!DNL Adobe Developer App Builder] is een raamwerk voor het ontwikkelen en implementeren van aangepaste webtoepassingen op Adobe [!DNL I/O Runtime] Adobe Experience Cloud-oplossingen uitbreiden. Om aangepaste toepassingen te maken, kunnen ontwikkelaars [!DNL React Spectrum] (UI toolkit van Adobe), creeer microservices, creeer douanegebeurtenissen, en orkestreer APIs. Zie [documentatie van Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>Op dit moment wordt [!DNL Asset Compute Service] kan alleen via [!DNL Experience Manager] als [!DNL Cloud Service]. Beheerders maken verwerkingsprofielen die de [!DNL Asset Compute Service] om elementen door te geven voor verwerking. Zie [gebruik van microservices en verwerkingsprofielen voor bedrijfsmiddelen](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

## Ondersteund gebruik van [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] ondersteunt een aantal gangbare gevallen voor bedrijfsgebruik, zoals eenvoudige beeldverwerking, specifieke conversies voor Adobe van toepassingen en het maken van aangepaste toepassingen die complexe zakelijke vereisten ordenen.

U kunt de [!DNL Asset Compute] webservice voor het genereren van miniaturen voor verschillende bestandstypen, hoogwaardige afbeeldingsrenderingen voor de [ondersteunde bestandsindelingen](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support). Zie [gebruik gevallen die via aangepaste configuratie worden ondersteund](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>De service biedt geen opslag van bedrijfsmiddelen. Gebruikers geven deze informatie en geven verwijzingen naar de locatie van bron- en uitvoerbestanden in de cloudopslag.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Overzicht van de verwerking van bedrijfsmiddelen met assetmicroservices in [!DNL Adobe Experience Manager] als [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview).
>* [Documentatie van Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).
>* [Ondersteunde bestandsindelingen voor verwerking](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
