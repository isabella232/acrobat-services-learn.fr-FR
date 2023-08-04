---
title: Utilisation de l’API PDF Services pour exporter du PDF vers Word, PowerPoint, etc.
description: Découvrez comment exécuter l'opération d'exportation de l'API PDF Services à l'aide des fichiers d'exemple pour les langages Node.js, Java et .Net
feature: PDF Services API
role: Developer
level: Intermediate
type: Tutorial
jira: KT-6674
thumbnail: KT-6674.jpg
exl-id: 55f5b04e-0249-47d9-9131-2f9ec01db7e8
source-git-commit: 5222e1626f4e79c02298e81d621216469753ca72
workflow-type: tm+mt
source-wordcount: '500'
ht-degree: 5%

---

# Utilisation de l’API PDF Services pour exporter du PDF vers Word, PowerPoint, etc.

![Créer une image de héros PDF](assets/ExportPDF_hero.jpg)

L’API Adobe PDF Services convertit les fichiers de PDF en MS Office, texte et images à l’aide d’API. Il existe de nombreux cas d’utilisation courants pour déverrouiller des PDF existants pour l’édition et l’analyse de contenu et, avec les services de PDF, les développeurs d’API peuvent facilement intégrer cette fonctionnalité aux systèmes et applications existants. Convertissez des fichiers de PDF au format MS Word pour modifier le contenu, valider des contrats et les envoyer ultérieurement pour signature, afin de personnaliser les workflows contractuels. Vous pouvez également exporter du contenu PDF au format MS Excel pour les calculs de factures et de comptes ou l’analyse de données.

L’opération d’exportation prend en charge les conversions de fichiers PDF suivantes :

* PDF à Microsoft Word (DOC, DOCX)
* PDF à Microsoft PowerPoint (PPTX)
* PDF à Microsoft Excel (XLSX)
* PDF au texte (RTF)
* PDF à l’image (JPEG, PNG)

Dans ce tutoriel, découvrez les principes de base de l’exécution de votre première opération d’exportation d’API PDF Services à l’aide de fichiers d’exemple pour les langages Node.js, Java et .Net.

## Étape 1 : Créez vos informations d’identification et configurez votre environnement :

Utilisez les tutoriels de prise en main ci-dessous pour créer vos identifiants d’API, télécharger des fichiers d’exemple et configurer votre environnement.

[Prise en main de l’API PDF Services et de Java](gettingstartedjava.md)

[Prise en main de l&#39;API PDF Services et de .Net](gettingstartednet.md)

[Prise en main de l&#39;API PDF Services et de Node.js](createpdffromhtml.md)

## Étape 2 : Exécutez l’opération d’exportation PDF à l’aide des fichiers d’exemple

**Java**

1. Ouvrez une invite de commande.

1. Transformez les répertoires en exemple de répertoire de code.

   Par exemple, C:\Temp\PDFToolsAPI\adobe-dc-pdf-tools-sdk-java-samples

1. Exécutez la commande suivante:

   `mvn -f pom.xml exec:java -Dexec.mainClass=com.adobe.platform.operation.samples.exportpdf.ExportPDFToDOCX`

Votre PDF est créé dans le répertoire src/main/resources.

**.Net**

1. Ouvrez une invite de commande.

1. Transformez les répertoires en exemple de répertoire de code.

   Par exemple, C:\Temp\PDFToolsAPI\adobe-dc-pdf-tools-sdk-NetSamples

1. Remplacez les répertoires par le répertoire ExportPDFtoDocx.

1. Exécutez la commande suivante:

   `dotnet run ExportPDFToDocx.csproj`

Votre PDF est installé dans le même répertoire.

**Node.js**

1. Ouvrez une invite de commande.

1. Transformez les répertoires en exemple de répertoire de code.

   Par exemple, C:\Temp\PDFToolsAPI\adobe-dc-pdf-tools-sdk-node-samples

1. Exécutez la commande suivante:

   `node src/ocr/ocr-pdf.js`

Votre PDF est créé à l’emplacement désigné dans la sortie, qui est par défaut le répertoire pdfServicesSdkResult.

## Réflexions finales

Vous devriez maintenant avoir un exemple concret qui peut être importé dans vos applications existantes pour commencer une validation de principe. Dans chacun des exemples de répertoires, vous pouvez voir un autre exemple pour exporter des fichiers PDF au format image. Les mêmes étapes ci-dessus vous permettent également d’exécuter cet exemple. Pour passer à un autre format, vous pouvez mettre à jour le code vers le nouveau format que vous souhaitez :

SupportedTargetFormats.PPTX

Et le résultat de destination :

output/exportPdfOutput.PPTX

Dans un autre format.

## Ressources et étapes suivantes

* Pour une aide et une assistance supplémentaires, consultez la page [[!DNL Adobe Acrobat Services] API](https://community.adobe.com/t5/document-cloud-sdk/bd-p/Document-Cloud-SDK?page=1&amp;sort=latest_replies&amp;filter=all) forum de communauté

* API PDF Services [Documentation](https://www.adobe.com/go/pdftoolsapi_doc)

* [FAQ](https://community.adobe.com/t5/document-cloud-sdk/faq-for-document-services-pdf-tools-api/m-p/10726197) pour les questions d’API PDF Services

* [Nous contacter](https://www.adobe.com/go/pdftoolsapi_requestform) pour toute question sur les licences et les tarifs
