---
title: Prise en main de l’API Acrobat Sign
description: Découvrez comment inclure l’API Acrobat Sign dans votre application pour collecter des signatures et d’autres informations
role: Developer
level: Intermediate
type: Tutorial
thumbnail: KT-8089.jpg
jira: KT-8089
exl-id: ae1cd9db-9f00-4129-a2a1-ceff1c899a83
source-git-commit: 2d1151c17dfcfa67aca05411976f4ef17adf421b
workflow-type: tm+mt
source-wordcount: '2058'
ht-degree: 2%

---

# Prise en main de l’API Adobe Sign

![Utiliser la bannière Case Hero](assets/UseCaseStartedHero.jpg)

[API Acrobat Sign](https://www.adobe.io/apis/documentcloud/sign.html) est un excellent moyen d’améliorer la gestion des accords signés. Les développeurs peuvent facilement intégrer leurs systèmes avec l’API Sign, qui constitue un moyen fiable et facile de télécharger des documents, de les envoyer pour signature, d’envoyer des rappels et de collecter des signatures électroniques.

## Ce que vous pouvez apprendre

Ce tutoriel pratique explique comment les développeurs peuvent utiliser l’API Sign pour améliorer les applications et les workflows créés avec [!DNL Adobe Acrobat Services]. [!DNL Acrobat Services] inclut [API Adobe PDF Services](https://www.adobe.io/apis/documentcloud/dcsdk/pdf-tools.html), [API Adobe PDF Embed](https://www.adobe.io/apis/documentcloud/viesdk) (gratuit), et [API de génération de documents Adobe](https://www.adobe.io/apis/documentcloud/dcsdk/doc-generation.html).

Découvrez plus précisément comment inclure l’API Acrobat Sign dans votre application pour collecter des signatures et d’autres informations, telles que les informations sur les employés d’un formulaire d’assurance. Les étapes génériques avec demandes et réponses HTTP simplifiées sont utilisées. Vous pouvez implémenter ces demandes dans votre langue préférée. Vous pouvez créer un PDF à l’aide de [[!DNL Acrobat Services] API](https://www.adobe.io/apis/documentcloud/dcsdk/), chargez-le dans l’API Sign en tant que fichier [transitoire](https://www.adobe.io/apis/documentcloud/sign/docs.html#!adobedocs/adobe-sign/master/overview/terminology.md) et demander la signature de l’utilisateur final à l’aide de l’accord ou [widget](https://www.adobe.io/apis/documentcloud/sign/docs.html#!adobedocs/adobe-sign/master/overview/terminology.md) workflow.

## Création d’un document PDF

Commencez par créer un modèle Microsoft Word et enregistrez-le en tant que PDF. Vous pouvez également automatiser votre pipeline à l’aide de l’API de génération de document pour télécharger un modèle créé dans Word, puis générer un document PDF. L’API de génération de documents fait partie de [!DNL Acrobat Services], [gratuit pendant six mois, puis payez au fur et à mesure pour seulement ou 0,05 $ par transaction.](https://www.adobe.io/apis/documentcloud/dcsdk/pdf-pricing.html).

Dans cet exemple, le modèle est simplement un document avec quelques champs de signataire à remplir. Nommez les champs pour l’instant, puis insérez-les ultérieurement dans ce tutoriel.

![Capture d&#39;écran du formulaire d&#39;assurance avec quelques champs](assets/GSASAPI_1.png)

## Découverte du point d’accès API valide

Avant d’utiliser l’API Sign, [créer un compte développeur gratuit](https://acrobat.adobe.com/ca/en/sign/developer-form.html) pour accéder à l’API, testez l’échange et l’exécution de vos documents, ainsi que la fonction d’e-mail.

Adobe distribue l’API Acrobat Sign dans le monde entier dans de nombreuses unités de déploiement appelées &quot;shards&quot;. Chaque partition sert le compte d&#39;un client, tel que NA1, NA2, NA3, EU1, JP1, AU1, IN1, etc. Les noms des partitions correspondent à des emplacements géographiques. Ces partitions composent l’URI de base (points d’accès) des points d’entrée de l’API.

Pour accéder à l’API Sign, vous devez d’abord découvrir le point d’accès correct pour votre compte, qui peut être api.na1.adobesign.com, api.na4.adobesign.com, api.eu1.adobesign.com ou autre, selon votre emplacement.

```
  GET /api/rest/v6/baseUris HTTP/1.1
  Host: https://api.adobesign.com
  Authorization: Bearer {YOUR-INTEGRATION-KEY-HERE}
  Accept: application/json

  Response Body (example):

  {
    "apiAccessPoint": "https://api.na4.adobesign.com/", 
    "webAccessPoint": "https://secure.na4.adobesign.com/" 
  }
```

Dans l’exemple ci-dessus, est une réponse avec la valeur en tant que point d’accès.

>[!IMPORTANT]
>
>Dans ce cas, toutes les demandes ultérieures que vous adressez à l’API Sign doivent utiliser ce point d’accès. Si vous utilisez un point d’accès qui ne dessert pas votre région, vous obtenez une erreur.

## Chargement d’un document temporaire

Adobe Sign vous permet de créer différents flux qui préparent les documents pour les signatures ou la collecte de données. Quel que soit le flux de votre application, vous devez d’abord télécharger un document, qui reste disponible pendant seulement sept jours. Les appels API suivants doivent alors faire référence à ce document temporaire.

Le document est chargé à l’aide d’une demande de POST dans le `/transientDocuments` point de terminaison. La demande en plusieurs parties se compose du nom du fichier, d&#39;un flux de fichiers et du type MIME (média) du fichier de document. La réponse du point de terminaison contient un ID qui identifie le document.

En outre, votre application peut spécifier une URL de rappel pour qu’Acrobat Sign envoie un ping, afin d’informer l’application lorsque le processus de signature est terminé.


```
  POST /api/rest/v6/transientDocuments HTTP/1.1
  Host: {YOUR-API-ACCESS-POINT}
  Authorization: Bearer {YOUR-INTEGRATION-KEY-HERE}
  x-api-user: email:your-api-user@your-domain.com
  Content-Type: multipart/form-data
  File-Name: "Insurance Form.pdf"
  File: "[path]\Insurance Form.pdf"
  Accept: application/json

  Response Body (example):

  {
     "transientDocumentId": "3AAA...BRZuM"
  }
```

## Création d’un formulaire web

Les formulaires web (auparavant appelés widgets de signature) sont des documents hébergés auxquels toute personne ayant accès peut apposer sa signature. Les formulaires web incluent, par exemple, des formulaires d’inscription, des dérogations et d’autres documents auxquels de nombreuses personnes accèdent et qu’elles signent en ligne.

Pour créer un nouveau formulaire web à l’aide de l’API Sign, vous devez d’abord télécharger un document temporaire. La demande de POST au `/widgets` le point de terminaison utilise le `transientDocumentId` .

Dans cet exemple, le formulaire web est `ACTIVE`, mais vous pouvez le créer dans l’un des trois états suivants :

* BROUILLON : pour créer le formulaire web de manière incrémentielle

* CRÉATION : pour ajouter ou modifier des champs de formulaire dans le formulaire web.

* ACTIVE : pour héberger immédiatement le formulaire web.

Les informations sur les participants du formulaire doivent également être définies. La `memberInfos` contient des données sur les participants, telles que l’adresse électronique. Actuellement, ce jeu ne prend pas en charge plus d&#39;un membre. Toutefois, étant donné que l’adresse e-mail du signataire du formulaire web est inconnue au moment de la création du formulaire web, l’adresse e-mail doit rester vide, comme dans l’exemple suivant. La `role` définit le rôle assumé par les membres dans la propriété `memberInfos` (par exemple, SIGNATAIRE et APPROBATEUR).

```
  POST /api/rest/v6/widgets HTTP/1.1
  Host: {YOUR-API-ACCESS-POINT}
  Authorization: Bearer {YOUR-INTEGRATION-KEY-HERE}
  x-api-user: email:your-api-user@your-domain.com
  Content-Type: application/json

  Request Body:

  {
    "fileInfos": [
      {
      "transientDocumentId": "YOUR-TRANSIENT-DOCUMENT-ID"
      }
     ],
    "name": "Insurance Form",
      "widgetParticipantSetInfo": {
          "memberInfos": [{
              "email": ""
          }],
      "role": "SIGNER"
      },
      "state": "ACTIVE"
  }

  Response Body (example):

  {
     "id": "CBJ...PXoK2o"
  }
```

Vous pouvez créer un formulaire web en tant que `DRAFT` ou `AUTHORING`, modifiez ensuite son état au fur et à mesure que le formulaire passe par votre pipeline d’application. Pour modifier un état de formulaire web, consultez la [PUT /widgets/{widgetId}/state](https://secure.na4.adobesign.com/public/docs/restapi/v6#!/widgets/updateWidgetState) point de terminaison.

## Lecture de l’URL d’hébergement du formulaire web

L’étape suivante consiste à découvrir l’URL hébergeant le formulaire web. Le point de terminaison /widgets récupère une liste de données de formulaire web, y compris l’URL hébergée du formulaire web que vous transférez à vos utilisateurs, pour collecter des signatures et d’autres données de formulaire.

Ce point de terminaison renvoie une liste, de sorte que vous pouvez localiser le formulaire spécifique par son ID dans la boîte de dialogue `userWidgetList` avant d’obtenir l’URL hébergeant le formulaire web :

```
  GET /api/rest/v6/widgets HTTP/1.1
  Host: {YOUR-API-ACCESS-POINT}
  Authorization: Bearer {YOUR-INTEGRATION-KEY-HERE}
  Accept: application/json

  Response Body:

  {
    "userWidgetList": [
      {
        "id": "CBJCHB...FGf",
        "name": "Insurance Form",
        "groupId": "CBJCHB...W86",
        "javascript": "<script type='text/javascript' ...
        "modifiedDate": "2021-03-13T15:52:41Z",
        "status": "ACTIVE",
        "Url":
        "https://secure.na4.adobesign.com/public/esignWidget?wid=CBFCIB...Rag*",
        "hidden": false
      },
      {
        "id": "CBJCHB...I8_",
        "name": "Insurance Form",
        "groupId": "CBJCHBCAABAAyhgaehdJ9GTzvNRchxQEGH_H1ya0xW86",
        "javascript": "<script type='text/javascript' language='JavaScript'
        src='https://sec
        "modifiedDate": "2021-03-13T02:47:32Z",
        "status": "ACTIVE",
        "Url":
        "https://secure.na4.adobesign.com/public/esignWidget?wid=CBFCIB...AAB",
        "hidden": false
      },
      {
        "id": "CBJCHB...Wmc",
```

## Gestion de votre formulaire web

Ce formulaire est un document PDF que les utilisateurs peuvent remplir. Cependant, vous devez toujours indiquer à l’éditeur du formulaire les champs que les utilisateurs doivent remplir et l’emplacement où ils se trouvent dans le document :

![Capture d&#39;écran du formulaire d&#39;assurance avec quelques champs](assets/GSASAPI_1.png)

Le document ci-dessus n’affiche pas encore les champs. Ils sont ajoutés lors de la définition des champs qui collectent les informations du signataire, ainsi que leur taille et leur position.

Maintenant, accédez à la [Formulaires web](https://secure.na4.adobesign.com/public/agreements/#agreement_type=webform) sur la page &quot;Vos accords&quot; et recherchez le formulaire que vous avez créé.

![Capture d’écran de l’onglet Gérer dans Acrobat Sign](assets/GSASAPI_2.png)

![Capture d’écran de l’onglet Gérer dans Acrobat Sign avec les formulaires web sélectionnés](assets/GSASAPI_3.png)

Cliquez **Modifier** pour ouvrir la page de modification du document. Les champs prédéfinis disponibles se trouvent dans le panneau de droite.

![Capture d’écran de l’environnement de création de formulaires Acrobat Sign](assets/GSASAPI_4.png)

L’éditeur vous permet de faire glisser et de déposer des champs de texte et de signature. Après avoir ajouté tous les champs nécessaires, vous pouvez les redimensionner et les aligner pour peaufiner votre formulaire. Enfin, cliquez sur **Enregistrer** pour créer le formulaire.

![Capture d’écran de l’environnement de création de formulaires Acrobat Sign avec ajout de champs](assets/GSASAPI_5.png)

## Envoi d’un formulaire web pour signature

Après avoir terminé le formulaire web, vous devez l’envoyer afin que les utilisateurs puissent le remplir et le signer. Une fois le formulaire enregistré, vous pouvez afficher et copier l’URL et le code incorporé.

**Copier l’URL du formulaire web**: utilisez cette URL pour envoyer des utilisateurs vers une version hébergée de cet accord pour révision et signature. Par exemple :

[https://secure.na4.adobesign.com/public/esignWidget?wid=CBFCIBAA3...babw\*](https://secure.na4.adobesign.com/public/esignWidget?wid=CBFCIBAA3AAABLblqZhCndYscuKcDMPiVfQlpaGPb-5D7ebE9NUTQ6x6jK7PIs8HCtTzr3HOx8U6D5qqbabw*)

**Copier le code incorporé du formulaire web**: ajoutez l’accord à votre site web en copiant ce code et en le collant dans votre HTML.

Par exemple :

```
<iframe
src="https://secure.na4.adobesign.com/public/esignWidget?wid=CBFC
...yx8*&hosted=false" width="100%" height="100%" frameborder="0"
style="border: 0;
overflow: hidden; min-height: 500px; min-width: 600px;"></iframe>
```

![Capture d’écran du formulaire web final](assets/GSASAPI_6.png)

Lorsque vos utilisateurs accèdent à la version hébergée de votre formulaire, ils passent en revue le document temporaire d’abord chargé avec les champs positionnés comme spécifié.

![Capture d’écran du formulaire web final](assets/GSASAPI_7.png)

L’utilisateur remplit ensuite les champs et signe le formulaire.

![Capture d’écran de l’utilisateur sélectionnant le champ Signature](assets/GSASAPI_8.png)

Ensuite, l’utilisateur signe le document avec une signature stockée précédemment ou avec une nouvelle signature.

![Capture d’écran de l’expérience de signature](assets/GSASAPI_9.png)

![Capture d&#39;écran de signature](assets/GSASAPI_10.png)

Lorsque l’utilisateur clique sur **Appliquer**, Adobe leur demande d’ouvrir leur courrier électronique et de confirmer la signature. La signature reste en attente jusqu’à ce que la confirmation arrive.

![Capture d&#39;écran d&#39;une étape supplémentaire](assets/GSASAPI_11.png)

Cette authentification ajoute plusieurs facteurs et renforce la sécurité du processus de signature.

![Capture d&#39;écran du message de confirmation](assets/GSASAPI_12.png)

![Capture d&#39;écran du message de fin](assets/GSASAPI_13.png)

## Lecture des formulaires web terminés

Il est maintenant temps d’obtenir les données de formulaire que les utilisateurs ont remplies. La `/widgets/{widgetId}/formData` le point de terminaison récupère les données saisies par l’utilisateur dans un formulaire interactif lorsqu’il a signé le formulaire.

```
GET /api/rest/v6/widgets/{widgetId}/formData HTTP/1.1
Host: {YOUR-API-ACCESS-POINT}
Authorization: Bearer {YOUR-INTEGRATION-KEY-HERE}
Accept: text/csv
```

Le flux de fichiers CSV obtenu contient des données de formulaire.

```
Response Body:
"Agreement
name","completed","email","role","first","last","title","company","agreementId",
"email verified","web form signed/approved"
"Insurance Form","","myemail@email.com","SIGNER","John","Doe","My Job Title","My
Company Name","","","2021-03-07 19:32:59"
```

## Création d’un accord

Vous pouvez créer des accords à la place des formulaires web. Les sections suivantes présentent quelques étapes simples pour gérer des accords à l’aide de l’API Sign.

L’envoi d’un document aux destinataires spécifiés pour signature ou approbation crée un accord. Vous pouvez suivre l’état et l’achèvement d’un accord à l’aide des API.

Vous pouvez créer un accord à l’aide d’un fichier [document temporaire](https://helpx.adobe.com/sign/kb/how-to-send-an-agreement-through-REST-API.html), [document de bibliothèque](https://www.adobe.io/apis/documentcloud/sign/docs.html#!adobedocs/adobe-sign/master/samples/send_using_library_doc.md)ou URL. Dans cet exemple, l’accord est basé sur le fichier `transientDocumentId`, tout comme le formulaire web créé précédemment.

```
POST /api/rest/v6/agreements HTTP/1.1
Host: {YOUR-API-ACCESS-POINT}
Authorization: Bearer {YOUR-INTEGRATION-KEY-HERE}
x-api-user: email:your-api-user@your-domain.com
Content-Type: application/json
Accept: application/json
Request Body:
{
    "fileInfos": [
      {
      "transientDocumentId": "{transientDocumentId}"
      }
     ],
    "name": "{agreementName}",
    "participantSetsInfo": [
      {
      "memberInfos": [
          {
          "email": "{signerEmail}"
          }
        ],
        "order": 1,
        "role": "SIGNER"
      }
    ],
    "signatureType": "ESIGN",
    "state": "IN_PROCESS"
  }
```

Dans cet exemple, l’accord est créé en_COURS, mais vous pouvez le créer dans l’un des trois états suivants :

* VERSION PRÉLIMINAIRE : pour créer progressivement l’accord avant de l’envoyer.

* CRÉATION : pour ajouter ou modifier des champs de formulaire dans l’accord

* EN_COURS : pour envoyer immédiatement l’accord.

Pour modifier l’état d’un accord, utilisez la boîte de dialogue `PUT /agreements/{agreementId}/state` pour effectuer l’une des transitions d’état autorisées ci-dessous :

* VERSION PRÉLIMINAIRE POUR CRÉATION

* CRÉATION DANS EN_COURS

* EN_COURS à ANNULÉ

La `participantSetsInfo` la propriété ci-dessus fournit les e-mails des personnes qui doivent participer à l’accord et l’action qu’elles effectuent (signer, approuver, reconnaître, etc.). Dans l’exemple ci-dessus, il n’y a qu’un seul participant : le signataire. Les signatures manuscrites sont limitées à quatre par document.

Contrairement aux formulaires web, lorsque vous créez un accord, Adobe l’envoie automatiquement pour signature. Le point de terminaison renvoie l’identificateur unique de l’accord.


```
  Response Body:

  {
     id (string): The unique identifier of the agreement
  }
```

## Récupération des informations sur les membres d’accord

Une fois que vous avez créé un accord, vous pouvez utiliser la boîte de dialogue `/agreements/{agreementId}/members` pour récupérer des informations sur les membres de l’accord. Par exemple, vous pouvez vérifier si un participant a signé l’accord.

```
GET /api/rest/v6/agreements/{agreementId}/members HTTP/1.1
Host: {YOUR-API-ACCESS-POINT}
Authorization: Bearer {YOUR-INTEGRATION-KEY-HERE}
Accept: application/json
```

Le corps de réponse JSON résultant contient des informations sur les participants.

```
  Response Body:

  {
     "participantSets":[
        {
           "memberInfos":[
              {
                 "id":"CBJ...xvM",
                 "email":"participant@email.com",
                 "self":false,
                 "securityOption":{
                    "authenticationMethod":"NONE"
                 },
                 "name":"John Doe",
                 "status":"ACTIVE",
                 "createdDate":"2021-03-16T03:48:39Z",
                 "userId":"CBJ...vPv"
              }
           ],
           "id":"CBJ...81x",
           "role":"SIGNER",
           "status":"WAITING_FOR_MY_SIGNATURE",
           "order":1
        }
     ],
```

## Envoi de rappels d’accord

En fonction des règles métier, une échéance peut empêcher les participants de signer l’accord après une date spécifique. Si l’accord a une date d’expiration, vous pouvez rappeler aux participants que cette date approche.

En fonction des informations des membres de l’accord que vous avez reçues après l’appel au `/agreements/{agreementId}/members` dans la dernière section, vous pouvez envoyer des rappels par e-mail à tous les participants qui n’ont pas encore signé l’accord.

Une demande de POST au `/agreements/{agreementId}/reminders` crée un rappel pour les participants spécifiés d’un accord identifié par le `agreementId` paramètre.

```
POST /agreements/{agreementId}/reminders HTTP/1.1
Host: {YOUR-API-ACCESS-POINT}
Authorization: Bearer {YOUR-INTEGRATION-KEY-HERE}
x-api-user: email:your-api-user@your-domain.com
Content-Type: application/json
Accept: application/json
  Request Body:

  {
    "recipientParticipantIds": [{agreementMemberIdList}],
    "agreementId": "{agreementId}",
    "note": "This is a reminder that you haven't signed the agreement yet.",
    "status": "ACTIVE"
  }

  Response Body:

  {
     id (string, optional): An identifier of the reminder resource created on the
     server. If provided in POST or PUT, it will be ignored
  }
```

Une fois que vous avez publié le rappel, les utilisateurs reçoivent un e-mail contenant les détails de l’accord et un lien vers celui-ci.

![Capture d&#39;écran du message de rappel](assets/GSASAPI_14.png)

## Lecture des accords terminés

À l’instar des formulaires web, vous pouvez lire les détails des accords signés par les destinataires. La `/agreements/{agreementId}/formData` Le point de terminaison récupère les données saisies par l’utilisateur lorsqu’il a signé le formulaire web.

```
GET /api/rest/v6/agreements/{agreementId}/formData HTTP/1.1
Host: {YOUR-API-ACCESS-POINT}
Authorization: Bearer {YOUR-INTEGRATION-KEY-HERE}
Accept: text/csv
Response Body:
"completed","email","role","first","last","title","company","agreementId"
"2021-03-16 18:11:45","myemail@email.com","SIGNER","John","Doe","My Job Title","My
Company Name","CBJCHBCAABAA5Z84zy69q_Ilpuy5DzUAahVfcNZillDt"
```

## Marche à suivre

L’API Acrobat Sign vous permet de gérer des documents, des formulaires web et des accords. Les workflows simplifiés mais complets créés à l’aide de formulaires web et d’accords sont réalisés d’une manière générique qui permet aux développeurs de les mettre en oeuvre à l’aide de n’importe quel langage.

Pour une présentation du fonctionnement de l’API Sign, vous trouverez des exemples dans la [Guide du développeur d’utilisation API](https://www.adobe.io/apis/documentcloud/sign/docs.html#!adobedocs/adobe-sign/master/api_usage.md). Cette documentation contient de courts articles sur de nombreuses étapes suivies tout au long de l’article et sur d’autres sujets connexes.

L’API Acrobat Sign est disponible via plusieurs niveaux de [formules de signature électronique pour un ou plusieurs utilisateurs](https://acrobat.adobe.com/fr/fr/sign/pricing/plans.html), ce qui vous permet de choisir le modèle de tarification le mieux adapté à vos besoins. Maintenant que vous savez à quel point il est facile d’intégrer l’API Sign dans vos applications, d’autres fonctionnalités telles que [Webhooks Acrobat Sign](https://www.adobe.io/apis/documentcloud/sign/docs.html#!adobedocs/adobe-sign/master/webhooks.md), un modèle de programmation basé sur le push. Au lieu d’exiger que votre application effectue des vérifications fréquentes dans les événements Acrobat Sign, les webhooks vous permettent d’enregistrer une URL HTTP pour laquelle l’API Sign exécute une demande de rappel de POST chaque fois qu’un événement se produit. Les webhooks permettent une programmation robuste en alimentant votre application avec des mises à jour instantanées et en temps réel.

Consultez la page [tarification à l&#39;utilisation des services](https://www.adobe.io/apis/documentcloud/dcsdk/pdf-pricing.html), par exemple, lorsque votre version d’essai de six mois de l’API Adobe PDF Services prend fin, et que l’API Adobe PDF Embed est gratuite.

Pour ajouter des fonctionnalités intéressantes, telles que la création automatique de documents et la signature de documents dans votre application, commencez par [[!DNL Adobe Acrobat Services]](https://www.adobe.io/apis/documentcloud/dcsdk/gettingstarted.html).
