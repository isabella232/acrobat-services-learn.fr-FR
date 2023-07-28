---
title: Prise en main de l’API Adobe PDF Services et de Java
description: Les développeurs peuvent commencer en quelques minutes avec les fichiers d’exemple prêts à l’emploi fournis pour accéder à tous les services web disponibles
type: Tutorial
role: Developer
level: Beginner
feature: PDF Services API
thumbnail: KT-6676.jpg
kt: 6676
exl-id: 4a8f2119-c464-496b-bdc8-35dd387bef25
source-git-commit: b65ffa3efa3978587564eb0be0c0e7381c8c83ab
workflow-type: tm+mt
source-wordcount: '501'
ht-degree: 0%

---

# Prise en main de l’API Adobe PDF Services et de Java

![Créer une image de héros PDF](assets/GettingStartedJava_hero.jpg)

Les développeurs peuvent commencer en quelques minutes avec les fichiers d’exemple prêts à l’emploi fournis pour accéder à tous les services web disponibles. Ce didacticiel vous guide à travers toutes les étapes pour commencer à exécuter les exemples à l’aide du SDK Java PDF Services :

## Étape 1 : Obtention des informations d’identification et téléchargement des fichiers d’exemple

La première étape consiste à obtenir un identifiant (clé API) pour déverrouiller l’utilisation. [Inscrivez-vous ici pour bénéficier de la version d’essai gratuite](https://www.adobe.io/apis/documentcloud/dcsdk/gettingstarted.html) et cliquez sur &quot;Commencer&quot; pour créer vos nouvelles informations d’identification.

![Étape 1](assets/GettingStartedJava_step1.png)

Il est important de choisir un &quot;Compte personnel&quot; pour vous inscrire à l&#39;essai gratuit :

![Personnel](assets/GettingStartedJava_personal.png)

À l’étape suivante, vous allez choisir le service d’API PDF Services, puis ajouter un nom et une description pour vos informations d’identification.

Il existe une case à cocher pour &quot;Créer un exemple de code personnalisé&quot;. Sélectionnez cette option pour que vos nouvelles informations d’identification soient automatiquement ajoutées à vos fichiers d’exemple, ce qui vous évitera de les ajouter manuellement à votre projet.

Ensuite, choisissez Java comme langue pour recevoir les exemples spécifiques à Java, puis cliquez sur le bouton &quot;Create Credentials&quot;.

![Informations](assets/GettingStartedJava_credentials.png)

Vous recevrez un fichier .zip à télécharger appelé PDFToolsSDK-JavaSamples.zip qui peut être enregistré sur votre système de fichiers local.

## Étape 2 : Configuration de votre environnement Java

1. Installer [Java 8 ou version ultérieure](https://www.oracle.com/java/technologies/javase-downloads.html) si ce n&#39;est pas déjà fait.
1. Exécuter `javac -version` pour vérifier votre installation.
1. Vérifiez que le dossier bin JDK est inclus dans la variable PATH (la méthode varie selon le système d’exploitation).
1. Installer [Maven](https://maven.apache.org/install.html) en utilisant l’outil de votre choix, si ce n’est pas déjà fait.

Les exemples personnalisés fournissent tout, depuis un exemple de code prêt à l&#39;exécution, un fichier json d&#39;informations d&#39;identification incorporé et des connexions préconfigurées aux dépendances.

1. Télécharger [le projet type](https://github.com/adobe/pdftools-java-sdk-samples).
1. Créez le projet type avec Maven : mvn clean install.
1. Testez l&#39;exemple de code sur la ligne de commande ou dans votre environnement de développement intégré préféré.

## Réflexions finales

L&#39;API PDF Services peut vous aider à éliminer les processus manuels en automatisant les workflows courants et en déplaçant la charge de traitement vers le cloud. Dans un monde où chaque navigateur traite le PDF différemment, l’API Adobe PDF Embed et l’API PDF Services vous permettent de créer des processus simplifiés, fiables et prévisibles qui s’exécutent et s’affichent correctement **chaque fois** quelle que soit la plateforme ou le device.

## Ressources et étapes suivantes

* Pour obtenir de l’aide et une assistance supplémentaires, consultez l’Adobe [[!DNL Acrobat Services] API](https://community.adobe.com/t5/document-cloud-sdk/bd-p/Document-Cloud-SDK?page=1&amp;sort=latest_replies&amp;filter=all) forum de communauté

* API PDF Services [Documentation](https://www.adobe.com/go/pdftoolsapi_doc)

* [FAQ](https://community.adobe.com/t5/document-cloud-sdk/faq-for-document-services-pdf-tools-api/m-p/10726197) pour les questions d’API PDF Services

* [Nous contacter](https://www.adobe.com/go/pdftoolsapi_requestform) pour toute question sur les licences et les tarifs

* Articles connexes

  [La nouvelle API PDF Services offre encore plus de fonctionnalités pour les workflows documentaires](https://community.adobe.com/t5/document-services-apis/new-pdf-tools-api-brings-more-capabilities-for-document-services/m-p/11294170)

  [Version de juillet [!DNL Adobe Acrobat Services]: Services d&#39;intégration et de PDF PDF](https://medium.com/adobetech/july-release-of-adobe-document-services-pdf-embed-and-pdf-tools-17211bf7776d)
