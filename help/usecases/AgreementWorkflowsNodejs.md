---
title: Workflows d’accord dans Node.js
description: "[!DNL Adobe Acrobat Services] Les API intègrent facilement des fonctionnalités de PDF dans vos applications web."
type: Tutorial
role: Developer
level: Beginner
thumbnail: KT-7473.jpg
jira: KT-7473
keywords: En vedette
exl-id: 44a03420-e963-472b-aeb8-290422c8d767
source-git-commit: 2d1151c17dfcfa67aca05411976f4ef17adf421b
workflow-type: tm+mt
source-wordcount: '2182'
ht-degree: 1%

---

# Workflows des accords dans Node.js

![Utiliser la bannière Case Hero](assets/UseCaseAgreementHero.jpg)

De nombreuses applications et processus métier nécessitent une documentation, comme les propositions et les accords. Les documents en PDF garantissent la protection et la personnalisation des fichiers. Ils prennent également en charge les signatures numériques, ce qui permet à vos clients de compléter facilement leurs documents. [!DNL Adobe Acrobat Services] Les API intègrent facilement des fonctionnalités de PDF dans vos applications web.

## Ce que vous pouvez apprendre

Dans ce tutoriel pratique, découvrez comment ajouter des services de PDF à une application Node.js pour numériser un processus d’accord.

## API et ressources pertinentes

* [API PDF Services](https://opensource.adobe.com/pdftools-sdk-docs/release/latest/index.html)

* [API PDF Embed](https://www.adobe.com/devnet-docs/dcsdk_io/viewSDK/index.html)

* [API Adobe Sign](https://www.adobe.io/apis/documentcloud/sign.html)

* [Code du projet](https://github.com/adobe/pdftools-node-sdk-samples)

## Configuration [!DNL Adobe Acrobat Services]

Pour commencer, configurez les informations d’identification à utiliser [!DNL Adobe Acrobat Services]. Enregistrez un compte et utilisez le [Node.js Quickstart](https://opensource.adobe.com/pdftools-sdk-docs/release/latest/index.html#node-js) pour vérifier que vos informations d’identification fonctionnent avant d’intégrer la fonctionnalité dans une application plus grande.

Commencez par obtenir un compte développeur Adobe. Ensuite, dans le [Prise en main](https://www.adobe.io/apis/documentcloud/dcsdk/gettingstarted.html?ref=getStartedWithServicesSDK) , sélectionnez l&#39;élément *Prise en main* sous Créer de nouvelles informations d’identification. Vous pouvez vous inscrire pour leur essai gratuit qui fournit 1 000 transactions de document qui peuvent être utilisées sur six mois.

![Image de création de nouvelles informations d’identification](assets/AWNjs_1.png)

Sur la page Créer des informations d’identification et de connexion, vous êtes invité à choisir entre l’API PDF Embed et l’API PDF Services.

Sélectionner *API PDF Services*.

Saisissez le nom de l’application et cochez la case intitulée *Créer un exemple de code personnalisé*. Cochez cette case pour incorporer automatiquement vos informations d’identification dans l’exemple de code. Si vous ne cochez pas cette case, vous devez ajouter manuellement vos informations d’identification à l’application.

Sélectionner *Node.js* pour le type d’application et cliquez sur *Créer des identifiants*.

Quelques instants plus tard, un fichier .zip commence à être téléchargé avec un exemple de projet comprenant vos informations d’identification. Le package Node.js pour [!DNL Acrobat Services] est déjà inclus dans l’exemple de code de projet.

![Image de la sélection des identifiants d&#39;API PDF Services](assets/AWNjs_2.png)

## Configuration manuelle du projet d&#39;exemple

Si vous choisissez de ne pas télécharger un exemple de projet à partir de la page Créer de nouvelles informations d’identification et de connexion, vous pouvez également configurer le projet manuellement.

Téléchargez le code (sans vos informations d’identification incorporées) à partir de [GitHub](https://github.com/adobe/pdftools-node-sdk-samples). Si vous utilisez cette version du code, vous devez ajouter vos informations d’identification au fichier pdftools-api-credentials.json avant de l’utiliser :

```
{
  "client_credentials": {
    "client_id": "<client_id>",
    "client_secret": "<client_secret>"
  },
  "service_account_credentials": {
    "organization_id": "<organization_id>",
    "account_id": "<technical_account_id>",
    "private_key_file": "<private_key_file_path>"
  }
}
```

Pour votre propre application, vous devez copier le fichier de clé privée et les fichiers d’informations d’identification vers la source de votre application.

Vous devez installer le package Node.js pour [!DNL Acrobat Services]. Pour installer le pack, utilisez la commande suivante :

```
npm install --save @adobe/documentservices-pdftools-node-sdk
```

## Configuration de la journalisation

Les exemples ici utilisent Express pour le cadre d’application. Ils utilisent également log4js pour la journalisation des applications. Avec log4js, vous pouvez facilement diriger la connexion vers la console ou vers un fichier :

```
const log4js = require('log4js');
const logger = log4js.getLogger();
log4js.configure( {
    appenders: { fileAppender: { type:'file', filename: './logs/applicationlog.txt'}},
    categories: { default: {appenders: ['fileAppender'], level:'info'}}
});
 
logger.level = 'info';
logger.info('Application started')
```

Le code ci-dessus écrit les données enregistrées dans un fichier dans ./logs/applicationlog.txt. Si vous souhaitez qu&#39;il écrive sur la console à la place, vous pouvez commenter l&#39;appel à log4js.configure.

## Conversion de fichiers Word en PDF

Les accords et les propositions sont souvent rédigés dans une application de productivité, comme Microsoft Word. Pour accepter les documents dans ce format et les convertir en PDF, vous pouvez ajouter des fonctionnalités pour votre application. Voyons comment télécharger et enregistrer un document dans une application Express et l’enregistrer dans le système de fichiers.

Dans le HTML de l’application, ajoutez un élément fichier et un bouton pour démarrer le téléchargement :

```
<input type="file" name="source" id="source" />
<button onclick="upload()" >Upload</button>
```

Dans le code JavaScript de la page, chargez le fichier de manière asynchrone à l’aide de la fonction fetch :

```
function upload()
{
  let formData = new FormData();
  var selectedFile = document.getElementById('source').files[0];
  formData.append("source", selectedFile);
  fetch('documentUpload', {method:"POST", body:formData});
}
```

Choisissez un dossier dans lequel accepter les fichiers téléchargés. L’application a besoin d’un chemin d’accès à ce dossier. Recherchez le chemin absolu en utilisant un chemin relatif joint à \_\_dirname :

```
const uploadFolder = path.join(__dirname, "../uploads");
```

Comme le fichier est envoyé par courrier, vous devez répondre à un message de publication côté serveur :

```
router.post('/', (req, res, next) => {
  console.log('uploading')
  if(!req.files || Object.keys(req.files).length === 0) {
  return res.status(400).send('No files were uploaded');
  }
    
  const uploadPath = path.join(uploadFolder, req.files.source.name);
  var buffer = req.files.source.data;
  var result = {"success":true};
  fs.writeFile(uploadPath, buffer, 'binary', (err)=> {
    if(err) {
      result.success = false;
    }
    res.json(result);
  });       
});
```

Une fois cette fonction exécutée, le fichier est enregistré dans le dossier de téléchargement des applications et peut être traité ultérieurement.

Ensuite, convertissez le fichier de son format natif en PDF. L’exemple de code que vous avez téléchargé précédemment contient un script nommé `create-pdf-from-docx.js` pour convertir un document en PDF. La fonction suivante : `convertDocumentToPDF`, prend un document chargé et le convertit en PDF dans un autre dossier :

```
function convertDocumentToPDF(sourcePath, destinationPath)
{    
  try {   
    const credentials = PDFToolsSDK.Credentials
    .serviceAccountCredentialsBuilder()
    .fromFile("pdftools-api-credentials.json")
    .build();
 
    const executionContext = 
      PDFToolsSDK.ExecutionContext.create(credentials),
    createPdfOperation = PDFToolsSDK.CreatePDF.Operation.createNew();
 
    const docxReadableStream = fs.createReadStream(sourcePath);
    const input = PDFToolsSDK.FileRef.createFromStream(
      docxReadableStream, 
      PDFToolsSDK.CreatePDF.SupportedSourceFormat.docx);
    createPdfOperation.setInput(input);
 
    createPdfOperation.execute(executionContext)
    .then(result => result.saveAsFile(destinationPath))
    .catch(err => {        
      logger.erorr('Exception encountered while executing operation');        
    })
  }
  catch(err) {        
    logger.error(err);
  }
}
```

Vous pouvez remarquer un schéma général avec le code :

Le code crée un objet d&#39;informations d&#39;identification et un contexte d&#39;exécution, initialise une opération, puis exécute l&#39;opération avec le contexte d&#39;exécution. Vous pouvez le constater dans l&#39;exemple de code.

En apportant quelques ajouts à la fonction de téléchargement afin qu&#39;il appelle cette fonction, les documents Word que les utilisateurs téléchargent sont maintenant automatiquement convertis en PDF.

Le code suivant crée le chemin de destination du PDF converti et lance la conversion :

```
const documentFolder = path.join(__dirname, "../docs");
var extPosition = req.files.source.name.lastIndexOf('.') - 1;
if(extPosition < 0 ) {
  extPosition = req.files.source.name.length
}
const destinationName = path.join(documentFolder,  
  req.files.source.name.substring(0, extPosition) + '.pdf');
console.log(destinationName);
 
logger.info('converting to ${destinationName}')
  convertDocumentToPDF(uploadPath, destinationName);
```

## Conversion d’autres types de fichiers en PDF

Le toolkit de documents convertit d’autres formats en PDF, tels que les HTMLS statiques, un autre type de document courant. Le toolkit accepte les documents HTML compressés sous forme de fichier .zip avec toutes les ressources référencées par le document (fichiers CSS, images et autres fichiers) dans le même fichier .zip. Le document de HTML doit être nommé index.html et placé à la racine du fichier .zip.

Pour convertir un fichier .zip contenant HTML, utilisez le code suivant :

```
//Create an HTML to PDF operation and provide the source file to it
htmlToPDFOperation = PDFToolsSdk.CreatePDF.Operation.createNew();     
const input = PDFToolsSdk.FileRef.createFromLocalFile(sourceZipFile);
htmlToPDFOperation.setInput(input);
 
// custom function for setting options
setCustomOptions(htmlToPDFOperation);
 
// Execute the operation and Save the result to the specified location.
htmlToPDFOperation.execute(executionContext)
  .then(result => result.saveAsFile(destinationPdfFile))
  .catch(err => {
    logger.error('Exception encountered while executing operation');
});
```

La fonction `setCustomOptions` spécifie d’autres paramètres du PDF, tels que le format de page. Ici, vous pouvez voir la fonction définit le format de la page sur 11,5 x 11 pouces :

```
const setCustomOptions = (htmlToPDFOperation) => {    
  const pageLayout = new PDFToolsSdk.CreatePDF.options.PageLayout();
  pageLayout.setPageSize(11.5, 8);

  const htmlToPdfOptions = 
    new PDFToolsSdk.CreatePDF.options.html.CreatePDFFromHtmlOptions.Builder()
    .includesHeaderFooter(true)
    .withPageLayout(pageLayout)
    .build();
  htmlToPDFOperation.setOptions(htmlToPdfOptions);
};
```

Lorsque vous ouvrez un document de HTML contenant certains termes, les éléments suivants s’affichent dans le navigateur :

![Image des termes informatiques](assets/AWNjs_3.png)

La source de ce document se compose d’un fichier CSS et d’un fichier de HTML :

![Image du fichier CSS et HTML](assets/AWNjs_4.png)

Une fois le fichier de HTML traité, le même texte s’affiche au format PDF :

![Fichier PDF des termes de l&#39;ordinateur](assets/AWNjs_5.png)

## Pages en attente

Une autre opération courante avec les fichiers de PDF consiste à ajouter à la fin des pages pouvant contenir du texte standard, tel qu’une liste de termes. La boîte à outils de document permet de regrouper plusieurs documents PDF dans un seul document. Si vous disposez d’une liste de chemins d’accès au document (ici dans `sourceFileList`), vous pouvez ajouter les références de fichier de chaque fichier à un objet pour une opération de combinaison.

Lorsque l’opération de combinaison s’exécute, elle fournit un fichier unique avec le contenu source. Vous pouvez utiliser `saveAsFile` sur l&#39;objet pour conserver le fichier dans le stockage.

```
const executionContext = PDFToolsSDK.ExecutionContext.create(credentials);
var combineOperation = PDFToolsSDK.CombineFiles.Operation.createNew();
 
sourceFileList.forEach(f => {
  var combinedSource = PDFToolsSDK.FileRef.createFromLocalFile(f);
  console.log(f);
  combineOperation.addInput(combinedSource);
});
    
 
combineOperation.execute(executionContext)
  .then(result=>result.saveAsFile(destinationFile))
  .catch(err => {
    logger.error(err.message);
});    
```

## Affichage de documents PDF

Vous avez effectué plusieurs opérations sur des fichiers de PDF, mais en fin de compte, votre utilisateur doit afficher les documents. Vous pouvez incorporer le document dans une page Web à l’aide de l’API PDF Embed d’Adobe.

Sur la page qui affiche le PDF, ajoutez un élément `<div />` pour contenir le document et lui attribuer un ID. Vous utiliserez cet ID sous peu. Dans la page Web, incluez un élément `<script />` référence à la bibliothèque JavaScript de l’Adobe :

```
<script src="https://documentcloud.adobe.com/view-sdk/main.js"></script>
```

Le dernier bit de code dont vous avez besoin est une fonction qui affiche le document une fois le code JavaScript de l’API Adobe PDF Embed chargé. Lorsque vous recevez une notification indiquant que le script est chargé via un événement adobe_dc_view\_sdk.ready, créez un nouvel objet AdobeDC.View. Cet objet a besoin de votre ID client et de l’ID de l’élément créé précédemment. Recherchez votre ID client dans le [Console Adobe Developer](https://console.adobe.io/). Lorsque vous affichez les paramètres de l&#39;application que vous avez créée lors de la génération des informations d&#39;identification, l&#39;ID client s&#39;affiche ici.

![Image de la clé client API](assets/AWNjs_6.png)

## Autres options du PDF

La [Démonstration de l’API Adobe PDF Embed](https://documentcloud.adobe.com/view-sdk-demo/index.html#/view/FULL_WINDOW/Bodea%20Brochure.pdf) vous permet de prévisualiser les différentes autres options d’incorporation de documents PDF.

![Image des options du PDF intégré ](assets/AWNjs_7.png)

Vous pouvez activer et désactiver diverses options et voir immédiatement comment elles s’affichent. Lorsque vous trouvez une combinaison qui vous convient, cliquez sur l’icône *\&lt;/\> Générer le code* pour générer le code de HTML réel à l’aide de ces options.

![Image d&#39;aperçu du code](assets/AWNjs_8.png)

## Ajout de signatures numériques et de sécurité

Une fois le document prêt, vous pouvez apposer des signatures numériques pour validation dans Adobe Sign. Cette fonctionnalité fonctionne un peu différemment de celle que vous avez utilisée jusqu’à présent. Pour les signatures numériques, une application doit être configurée pour utiliser OAuth pour l’authentification des utilisateurs.

La première étape de la configuration de votre application consiste à [enregistrer votre demande](https://www.adobe.io/apis/documentcloud/sign/docs.html#!adobedocs/adobe-sign/master/gstarted/create_app.md) pour utiliser OAuth pour Adobe Sign. Une fois connecté, accédez à l’écran de création d’applications en cliquant sur *Compte*, puis ouvrez la boîte de dialogue *API Adobe Sign* , puis cliquez sur *Applications API* pour ouvrir la liste des applications enregistrées.

![Image de la première étape de l&#39;enregistrement de votre demande](assets/AWNjs_9.png)

Pour créer une nouvelle entrée d’application, cliquez sur l’icône plus dans le coin supérieur droit.

![Image de l’icône plus dans l’angle supérieur droit de l’écran](assets/AWNjs_10.png)

Dans la fenêtre qui s’ouvre, saisissez un nom d’application et un nom d’affichage. Sélectionner *Client* pour le domaine, puis cliquez sur *Enregistrer*.

![Image de l’emplacement où entrer le nom de l’application et le nom complet](assets/AWNjs_11.png)

Une fois l’application créée, vous pouvez la sélectionner dans la liste et cliquer sur *Configurer OAuth pour l’application*. Sélectionnez les options. Dans le champ URL de redirection, entrez l’URL de votre application. Vous pouvez entrer plusieurs URL ici. Pour l’application que vous testez, la valeur est :

```
http://localhost:3000/signed-in 
```

Le processus d’utilisation d’OAuth pour obtenir un jeton est standard. Votre application dirige un utilisateur vers une URL de connexion. Une fois connecté, l’utilisateur est redirigé vers l’application avec des informations supplémentaires dans les paramètres de requête de la page.

Pour l’URL de connexion, votre application doit transmettre votre ID client, l’URL de redirection et une liste des portées nécessaires.

Le modèle de l’URL est le suivant :

```
https://secure.adobesign.com/public/oauth?
  redirect_uri=&
  response_type=code&
  client_id=&
  scope=
```

L’utilisateur est invité à se connecter avec son ID Adobe Sign. Une fois connectés, ils sont invités à indiquer s’ils souhaitent accorder des autorisations pour l’application.

![Image de l&#39;écran Confirmer l&#39;accès](assets/AWNjs_12.png)

Si l’utilisateur clique sur *Autoriser l’accès* sur l&#39;URL de redirection, un paramètre de requête nommé code transmet le code d&#39;autorisation :

https://YourServer.com/?code=**\&lt;authorization_code>**\&amp;api_access_point=https://api.adobesign.com&amp;web_access_point=https://secure.adobesign.com

La publication de ce code sur le serveur Adobe Sign avec votre ID client et votre secret client fournit un jeton d’accès pour accéder au service. Enregistrez les valeurs dans les paramètres `api_access_point` et `web_access_point`. Ces valeurs sont utilisées pour les demandes ultérieures.

```
var requestURL = ' ${api_access_point}oauth/token?code=${code}'
  +'&client_id=${client_id}'
  +'&client_secret=${client_secret}&'
  +'redirect_uri=${redirect_url}&'
  +'grant_type=authorization_code';
request.post(requestURL, {form: { }
}, (err,response,body)=>{                
    var token_response = JSON.parse(body)
    var access_token = token_response.access_token;
    console.log(access_token);
});
```

Lorsqu’un document nécessite une signature, celui-ci doit d’abord être téléchargé. Votre application peut télécharger le document dans le dossier `api_access_point` valeur reçue lors de la demande du jeton OAUTH. Le point de terminaison est `/api/rest/v6/transientDocuments`. Les données de la demande sont les suivantes :

```
POST /api/rest/v6/transientDocuments HTTP/1.1
Host: api.na1.adobesign.com
Authorization: Bearer MvyABjNotARealTokenHkYyi
Content-Type: multipart/form-data
Content-Disposition: form-data; name=";File"; filename="MyPDF.pdf"
<PDF CONTENT>
```

Dans votre application, créez la demande avec le code suivant :

```
var uploadRequest = {
  'method': 'POST',
  'url': '${oauthParameters.signin_domain}/api/rest/v6/transientDocuments',
  'headers': {
    'Authorization': 'Bearer  ${auth_token}'
  },
  formData: {
    'File': {
      'value': fs.createReadStream(documentPath),
      'options': {
        'filename': fileName,
        'contentType': null
      }
    }
  }
};
 
request(uploadRequest, (error, response) => {
  if (error) throw new Error(error);
  var jsonResponse = JSON.parse(response.body);
  var transientDocumentId = jsonResponse.transientDocumentId;
  logger.info('transientDocumentId:', transientDocumentId)
});
```

La demande renvoie un `transientID` valeur. Le document a été chargé, mais il n&#39;est pas encore envoyé. Pour envoyer le document, utilisez la boîte de dialogue `transientID` pour demander l’envoi du document.

Commencez par créer un objet JSON contenant les informations relatives au document à signer. Dans l’exemple suivant, la variable `transientDocumentId` contient l’ID du code ci-dessus et `agreementDescription` contient un texte décrivant l’accord nécessitant une signature. Les personnes qui doivent signer le document sont répertoriées dans la section `participantSetsInfo` par leur adresse e-mail et leur rôle.

```
var requestBody = {
  "fileInfos":[
    {"transientDocumentId":transientDocumentId}],
    "name":agreementDescription,
    "participantSetsInfo":[
      {"memberInfos":[{"email":"user@domain.com"}],
       "order":1,"role":"SIGNER"}
    ],
    "signatureType":"ESIGN","state":"IN_PROCESS"
};
```

L’envoi de cette demande web génère la demande de signature et renvoie un objet JSON avec un ID pour la demande d’accord :

```
request(requestBody, function (error, response) {
  if (error) throw new Error(error);
  var JSONResponse = JSON.parse(response.body);
  var requestId = JSONResponse.id;
});
```

Si les signataires oublient de signer et ont besoin d’un autre e-mail de notification, envoyez à nouveau les notifications à l’aide de l’ID reçu précédemment. La seule différence est que vous devez également ajouter les ID des participants des parties. Vous pouvez obtenir les ID des participants en envoyant une demande de GET à `/agreements/{agreementID}/members`.

Pour demander l’envoi du rappel, créez d’abord un objet JSON décrivant la demande. L’objet minimal nécessite une liste d’ID de participant et un statut pour le rappel (&quot;ACTIF&quot;, &quot;TERMINÉ&quot; ou &quot;ANNULÉ&quot;).

La demande peut éventuellement comporter des informations supplémentaires, telles qu&#39;une valeur de &quot;note&quot; qui s&#39;affichera pour l&#39;utilisateur. Ou un délai (en heures) pour attendre l’envoi du rappel (en `firstReminderDelay`), et une fréquence de rappel (dans le champ &quot;frequency&quot;), qui accepte des valeurs telles que DAILY_UNTIL_SIGNED, EVERY_THIRD_DAY_UNTIL_SIGNED ou WEEKLY_UNTIL_SIGNED.

```
var requestBody = {
  //participantList is an array of participant ID strings
  "recipientParticipantIds":participantList
  ,"status":"ACTIVE",
  "note":"This is a reminder to sign out important agreement."
}
 
var reminderRequest = {
  'method': 'POST',
  'url': '${oauthParameters.signin_domain}/api/rest/v6/agreements/${agreementID}/reminders',
  'headers': {
    'Authorization': `Bearer ${access_token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(requestBody)
 
};

request(reminderRequest, function (error, response) {
});
```

Et c&#39;est tout ce qu&#39;il faut pour envoyer une demande de rappel.

![Image d’un formulaire web](assets/AWNjs_13.png)

## Création de formulaires web

Vous pouvez également utiliser l’API Adobe Sign pour créer des formulaires web. Les formulaires web vous permettent d’incorporer un formulaire dans une page web ou d’y créer un lien direct. Une fois un formulaire web créé, il s’affiche également dans les formulaires web de votre console Adobe Sign. Vous pouvez créer des formulaires web avec l’état BROUILLON pour la création incrémentielle, l’état CRÉATION pour la modification des champs de formulaire web et l’état ACTIF pour héberger immédiatement le formulaire.

![Image d’un formulaire web dans l’écran Gérer dans Adobe Sign](assets/AWNjs_14.png)

Pour créer un formulaire web, utilisez le formulaire `transientDocumentId`. Choisissez un titre pour le formulaire et un état pour l&#39;initialiser.

```
var requestBody = {
  "fileInfos": [
    {
      "transientDocumentId": transientDocumentId
    }
  ],
  "name": webFormTitle,
  "state": status,
  "widgetParticipantSetInfo": {
    "memberInfos": [ { "email": "" } ],
    "role": "SIGNER"
  }
}
```

```
var createWebFormRequest = {
  'method': 'POST',
  'url': `${oauthParameters.signin_domain}/api/rest/v6/widgets`,
  'headers': {
    'Authorization': `Bearer ${access_token}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(requestBody)
}
```

```
request(createWebFormRequest, function (error, response) {
  var jsonResp = JSON.parse(response.body);
  var webFormID = jsonResp.id;
});
```

Vous pouvez désormais incorporer votre document ou créer un lien vers celui-ci.

## Marche à suivre

Comme vous pouvez le voir à partir des démarrages rapides et du code fourni, il est facile d&#39;implémenter des processus d&#39;approbation de documents PDF et numériques en utilisant Node avec le [!DNL Adobe Acrobat Services] API. Les API d’Adobe s’intègrent facilement dans vos applications clientes.

Pour découvrir les portées requises pour un appel ou pour voir comment l&#39;appel est créé, vous pouvez créer des exemples d&#39;appels à partir de la boîte de dialogue [Documentation des API Rest](https://secure.na4.adobesign.com/public/docs/restapi/v6). La [Quickstart](https://github.com/adobe/pdftools-node-sdk-samples) montrent également d’autres fonctionnalités et formats de fichier dans la [!DNL Adobe Acrobat Services] Processus API.

Vous pouvez ajouter une multitude de fonctionnalités de PDF à vos applications, permettant à vos utilisateurs d’afficher et de signer rapidement et facilement leurs documents, et bien plus encore. Pour commencer, consultez la page [[!DNL Adobe Acrobat Services]](https://www.adobe.io/apis/documentcloud/dcsdk/) aujourd&#39;hui.
