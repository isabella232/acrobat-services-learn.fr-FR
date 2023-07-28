---
title: Workflows des documents de RH en Java
description: "[!DNL Adobe Acrobat Services] Les API intègrent facilement des fonctionnalités PDF dans vos applications web de RH."
type: Tutorial
role: Developer
level: Intermediate
feature: Use Cases
thumbnail: KT-7474.jpg
jira: KT-7474
exl-id: add4cc5c-06e3-4ceb-930b-e8c9eda5ca1f
source-git-commit: b65ffa3efa3978587564eb0be0c0e7381c8c83ab
workflow-type: tm+mt
source-wordcount: '1899'
ht-degree: 2%

---

# Workflows de documents RH dans Java

![Utiliser la bannière Case Hero](assets/UseCaseHRHero.jpg)

De nombreuses entreprises ont besoin de documents concernant les nouvelles recrues, comme des ententes de travail pour les employés qui travaillent à domicile. Traditionnellement, les entreprises géraient physiquement ces documents dans des formulaires difficiles à gérer et à stocker. Lorsque vous passez à des documents électroniques, les fichiers de PDF sont un choix idéal car ils sont plus sécurisés et moins modifiables que les autres types de fichiers. et prennent en charge les signatures numériques.

## Ce que vous pouvez apprendre

Dans ce tutoriel pratique, découvrez comment mettre en oeuvre un formulaire RH basé sur le web qui enregistre un accord sur le lieu de travail dans un PDF avec validation dans une application MVC Java Spring simple.

## API et ressources pertinentes

* [API PDF Services](https://opensource.adobe.com/pdftools-sdk-docs/release/latest/index.html)

* [API Adobe Sign](https://www.adobe.io/apis/documentcloud/sign.html)

* [Code du projet](https://github.com/dawidborycki/adobe-sign)

## Génération des identifiants API

Commencez par vous inscrire à la version d’essai gratuite de l’API Adobe PDF Services. Accédez à l’onglet [Adobe](https://www.adobe.io/apis/documentcloud/dcsdk/gettingstarted.html?ref=getStartedWithServicesSDK) [site](https://www.adobe.io/apis/documentcloud/dcsdk/gettingstarted.html?ref=getStartedWithServicesSDK) , puis cliquez sur le bouton *Prise en main* sous le bouton *Créer de nouvelles identifiants*. La version d’essai gratuite offre 1 000 transactions documentaires qui peuvent être utilisées pendant six mois. Sur la page suivante (voir ci-dessous), sélectionnez le service (API PDF Services), définissez le nom des informations d’identification (par exemple, HRDocumentWFCredentials) et entrez une description.

Sélectionnez la langue (Java pour cet exemple) et vérifiez *Création d’exemples de code personnalisés*. La dernière étape garantit que les exemples de code contiennent déjà le fichier prérempli pdftools-api-credentials.json que vous utilisez, ainsi que la clé privée pour authentifier votre application dans l&#39;API.

Enfin, cliquez sur le bouton *Créer des identifiants* du menu. Cela génère les informations d’identification et les exemples commencent automatiquement à être téléchargés.

![Capture D’Écran Créer Des Identifiants](assets/HRWJ_1.png)

Pour vous assurer que les informations d’identification fonctionnent, ouvrez les exemples téléchargés. Ici, vous utilisez IntelliJ IDEA. Lorsque vous ouvrez le code source, l&#39;environnement de développement intégré (IDE) demande le moteur de génération. Maven est utilisé dans cet exemple, mais vous pouvez également utiliser Gradle, selon vos préférences.

Ensuite, exécutez la commande `mvn clean install` Maven a pour but de construire les fichiers jar.

Enfin, exécutez l’exemple CombinePDF, comme indiqué ci-dessous. Le code génère le PDF dans le dossier de sortie.

![Menu pour exécuter la capture d’écran d’exemple CombinePDF](assets/HRWJ_2.png)

## Création de l&#39;application MVC Spring

En fonction des informations d’identification, vous créez ensuite l’application. Cet exemple utilise Spring Initializr.

Tout d’abord, configurez les paramètres du projet pour utiliser le langage Java 8 et le package Jar (voir la capture d’écran ci-dessous).

![Capture d’écran de Spring Initializr](assets/HRWJ_3.png)

Deuxièmement, ajoutez Spring Web (à partir du Web) et Thymeleaf (à partir des moteurs de modèles) :

![Capture d&#39;écran pour ajouter Spring Web et Thymeleaf](assets/HRWJ_4.png)

Après avoir créé le projet, accédez au fichier pom.xml et complétez la section des dépendances avec pdftools-sdk et log4j-slf4j-impl :

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

</dependencies>
```

Ensuite, complétez le dossier racine de votre projet avec deux fichiers téléchargés à l’aide de l’exemple de code :

* pdftools-api-credentials.json

* private.key

## Rendu d’un formulaire web

Pour effectuer le rendu du formulaire web, modifiez l’application avec le responsable du traitement qui effectue le rendu du formulaire de données personnelles et gérez la publication du formulaire. Ainsi, modifiez d&#39;abord l&#39;application avec la classe de modèle PersonForm :

```
package com.hr.docsigning;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

public class PersonForm {
    @NotNull
    @Size(min=2, max=30)
    private String firstName;

    @NotNull
    @Size(min=2, max=30)
    private String lastName;

    public String getFirstName() {
            return this.firstName;
    }


    public void setFirstName(String firstName) {
            this.firstName = firstName;
    }

    public String getLastName() {
           return this.lastName;
    }

    public void setLastName(String lastName) {
            this.lastName = lastName;
    }

    public String GetFullName() {
           return this.firstName + " " + this.lastName;
    }
}
```

Cette classe contient deux propriétés : `firstName` et `lastName`. En outre, utilisez cette validation simple pour vérifier s’ils comptent entre 2 et 30 caractères.

Compte tenu de la classe de modèle, vous pouvez créer le contrôleur (voir PersonController.java à partir du code compagnon) :

```
package com.hr.docsigning;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import javax.validation.Valid;


@Controller
public class PersonController {
    @GetMapping("/")
    public String showForm(PersonForm personForm) {
        return "form";
    }
}
```

Le contrôleur n&#39;a qu&#39;une seule méthode : showForm. Il est responsable du rendu du formulaire à l’aide du modèle de HTML situé dans resources/templates/form.html:

```
<html>
<head>
    <link rel="stylesheet" href="https://www.w3schools.com/w3css/4/w3.css">
</head>
 
<body>
<div class="w3-container">
    <h1>HR Department</h1>
</div>
 
<form class="w3-panel w3-card-4" action="#" th:action="@{/}"
        th:object="${personForm}" method="post">
    <h2>Personal data</h2>
    <table>
        <tr>
            <td>First Name:</td>
            <td><input type="text" class="w3-input"
                placeholder="First name" th:field="*{firstName}" /></td>
            <td class="w3-text-red" th:if="${#fields.hasErrors('firstName')}"
                th:errors="*{firstName}"></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><input type="text" class="w3-input"
                placeholder="Last name" th:field="*{lastName}" /></td>
            <td class="w3-text-red" th:if="${#fields.hasErrors('lastName')}"
                th:errors="*{lastName}"></td>
        </tr>
        <tr>
            <td><button class="w3-button w3-black" type="submit">Submit</button></td>
        </tr>
    </table>
</form>
</body>
</html>
```

Pour effectuer le rendu de contenu dynamique, le moteur de rendu de modèle Thymeleaf est utilisé. Ainsi, après avoir exécuté l’application, vous devriez voir les éléments suivants :

![Capture d’écran du contenu rendu](assets/HRWJ_5.png)

## Génération du PDF avec du contenu dynamique

Maintenant, générez le document de PDF contenant le contrat virtuel en remplissant dynamiquement les champs sélectionnés après avoir rendu le formulaire de données personnelles. Plus précisément, vous devez renseigner les données de personne dans le contrat précréé.

Ici, pour la simplicité, vous avez seulement un en-tête, un sous-en-tête, et une chaîne de lecture constante : &quot;Ce contrat a été préparé pour \&lt;full name=&quot;&quot; of=&quot;&quot; the=&quot;&quot; person=&quot;&quot;>&quot;.

Pour atteindre cet objectif, commencez par l&#39;Adobe [Création d’un PDF à partir du HTML dynamique](https://opensource.adobe.com/pdftools-sdk-docs/release/latest/howtos.html#create-a-pdf-from-dynamic-html) exemple. En analysant cet exemple de code, vous constatez que le processus de remplissage dynamique des champs de HTML fonctionne comme suit.

Vous devez d’abord préparer la page de HTML, qui contient du contenu statique et dynamique. La partie dynamique est mise à jour à l&#39;aide de JavaScript. À savoir, l’API PDF Services injecte l’objet JSON dans votre HTML.

Vous obtenez ensuite les propriétés JSON à l’aide de la fonction JavaScript appelée lorsque le document de HTML se charge. Cette fonction JavaScript met à jour les éléments DOM sélectionnés. Voici l’exemple qui remplit l’élément span, en conservant les données de la personne (voir src\\main\\resources\\contract\\index.html du code compagnon) :

```
<html>
<head>
    <link rel="stylesheet" href="https://www.w3schools.com/w3css/4/w3.css">
</head>
 
<body onload="updateFullName()">
    <script src="./json.js"></script>
    <script type="text/javascript">
        function updateFullName()
        {
            var document = window.document;
            document.getElementById("personFullName").innerHTML = String(
                window.json.personFullName);
        }
    </script>
 
    <div class="w3-container ">
        <h1>HR Department</h1>
 
        <h2>Contract details</h2>
 
        <p>This contract was prepared for:
            <strong><span id="personFullName"></span></strong>
        </p>
    </div>
</body>
</html>
```

Ensuite, vous devez compresser le HTML avec tous les fichiers JavaScript et CSS dépendants. L&#39;API PDF Services n&#39;accepte pas les fichiers de HTML. Au lieu de cela, il nécessite un fichier zip en entrée. Dans ce cas, vous stockez le fichier compressé dans src\\main\\resources\\contract\\index.zip.

Ensuite, vous pouvez compléter la `PersonController` avec une autre méthode qui gère les demandes de POST :

```
@PostMapping("/")
public String checkPersonInfo(@Valid PersonForm personForm,
    BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
        return "form";
    }
 
    CreateContract(personForm);
 
    return "contract-actions";
}
```

Le procédé ci-dessus permet de créer un contrat PDF à l&#39;aide des données personnelles fournies et de restituer la vue des actions de contrat. Ce dernier fournit des liens vers le PDF généré et pour la signature du PDF.

Maintenant, voyons comment le fichier `CreateContract` fonctionne (la liste complète est ci-dessous). La méthode repose sur deux champs :

* `LOGGER`, à partir de log4j, pour déboguer des informations sur les exceptions

* `contractFilePath`, contenant le chemin d’accès au PDF généré

La `CreateContract` configure les informations d&#39;identification et crée le PDF à partir du HTML. Pour transmettre et renseigner les données de la personne dans le contrat, utilisez la boîte de dialogue `setCustomOptionsAndPersonData` aide. Cette méthode récupère les données de la personne à partir du formulaire, puis les envoie au PDF généré via l&#39;objet JSON expliqué ci-dessus.

En outre, `setCustomOptionsAndPersonData` montre comment contrôler l’apparence du PDF en désactivant l’en-tête et le pied de page. Une fois ces étapes terminées, vous enregistrez le fichier du PDF dans output/contract.pdf et supprimez éventuellement le fichier généré précédemment.

```
private static final Logger LOGGER = LoggerFactory.getLogger(PersonController.class);
private String contractFilePath = "output/contract.pdf"; 
private void CreateContract(PersonForm personForm) {
    try {
        // Initial setup, create credentials instance.
        Credentials credentials = Credentials.serviceAccountCredentialsBuilder()
                .fromFile("pdftools-api-credentials.json")
                .build();

        //Create an ExecutionContext using credentials 
       //and create a new operation instance.
        ExecutionContext executionContext = ExecutionContext.create(credentials);
        CreatePDFOperation htmlToPDFOperation = CreatePDFOperation.createNew();

        // Set operation input from a source file.
        FileRef source = FileRef.createFromLocalFile(
           "src/main/resources/contract/index.zip");
       htmlToPDFOperation.setInput(source);

        // Provide any custom configuration options for the operation
        // You pass person data here to dynamically fill out the HTML
        setCustomOptionsAndPersonData(htmlToPDFOperation, personForm);

        // Execute the operation.
        FileRef result = htmlToPDFOperation.execute(executionContext);

        // Save the result to the specified location. Delete previous file if exists
        File file = new File(contractFilePath);
        Files.deleteIfExists(file.toPath());

        result.saveAs(file.getPath());

    } catch (ServiceApiException | IOException | 
             SdkException | ServiceUsageException ex) {
        LOGGER.error("Exception encountered while executing operation", ex);
    }
}
 
private static void setCustomOptionsAndPersonData(
    CreatePDFOperation htmlToPDFOperation, PersonForm personForm) {
    //Set the dataToMerge field that needs to be populated 
    //in the HTML before its conversion
    JSONObject dataToMerge = new JSONObject();
    dataToMerge.put("personFullName", personForm.GetFullName());
 
    // Set the desired HTML-to-PDF conversion options.
    CreatePDFOptions htmlToPdfOptions = CreatePDFOptions.htmlOptionsBuilder()
        .includeHeaderFooter(false)
        .withDataToMerge(dataToMerge)
        .build();
    htmlToPDFOperation.setOptions(htmlToPdfOptions);
}
```

Lors de la génération du contrat, vous pouvez également fusionner les données dynamiques spécifiques à la personne avec les conditions contractuelles fixes. Pour ce faire, suivez la [Création d’un PDF à partir d’un HTML statique](https://opensource.adobe.com/pdftools-sdk-docs/release/latest/howtos.html#create-a-pdf-from-dynamic-html) exemple. Vous pouvez également : [fusion de deux PDF](https://opensource.adobe.com/pdftools-sdk-docs/release/latest/howtos.html#create-a-pdf-from-static-html).

## Présentation du fichier de PDF pour le téléchargement

Vous pouvez maintenant présenter le lien au PDF généré pour que l’utilisateur puisse le télécharger. Pour ce faire, créez d’abord le fichier contract-actions.html (voir resources/templates contract-actions.html du code d’accompagnement) :

```
<html>
<head>
    <link rel="stylesheet" href="https://www.w3schools.com/w3css/4/w3.css">
</head>
 
<div class="w3-container ">
    <h1>HR Department</h1>
 
    <h2>Contract file</h2>
 
    <p>Click <a href="/pdf">here</a> to download your contract</p>
</div>
</body>
</html>
```

Ensuite, vous implémentez l’attribut `downloadContract` dans le panneau `PersonController` classe comme suit :

```
@RequestMapping("/pdf")
public void downloadContract(HttpServletResponse response)
{
    Path file = Paths.get(contractFilePath);
 
    response.setContentType("application/pdf");
    response.addHeader(
        "Content-Disposition", "attachment; filename=contract.pdf");

    try
    {
        Files.copy(file, response.getOutputStream());
        response.getOutputStream().flush();
    }
    catch (IOException ex) 
    {
        ex.printStackTrace();
    }
}
```

Après avoir exécuté l’application, vous obtenez le flux suivant. Le premier écran affiche le formulaire de données personnelles. Pour tester, remplissez-le avec des valeurs comprises entre deux et 30 caractères :

![Capture d’écran des valeurs de données](assets/HRWJ_6.png)

Après avoir cliqué sur le bouton *Envoyer* , le formulaire est validé et le PDF est généré en fonction du HTML (resources/contract/index.html). L’application affiche une autre vue (détails du contrat), dans laquelle vous pouvez télécharger le PDF :

![Capture d’écran dans laquelle vous pouvez télécharger le PDF](assets/HRWJ_7.png)

Le PDF, après le rendu dans le navigateur Web, se présente comme suit. À savoir, les données personnelles que vous avez saisies sont transmises au PDF :

![Capture d&#39;écran du PDF avec données personnelles](assets/HRWJ_8.png)

## Activation des signatures et de la sécurité

Une fois l’accord prêt, Adobe Sign peut ajouter des signatures numériques représentant l’approbation. L’authentification Adobe Sign fonctionne un peu différemment d’OAuth. Voyons maintenant comment intégrer l’application à Adobe Sign. Pour ce faire, vous devez préparer le jeton d’accès pour votre application. Vous rédigez ensuite le code client à l’aide du kit SDK Java d’Adobe Sign.

Pour obtenir un jeton d’autorisation, vous devez effectuer plusieurs étapes :

Commencez par enregistrer un fichier [compte développeur](https://acrobat.adobe.com/fr/fr/sign/developer-form.html).

Créez l’application CLIENT dans le [Portail Adobe Sign](https://www.adobe.io/apis/documentcloud/sign/docs.html#!adobedocs/adobe-sign/master/gstarted/create_app.md).

Configurez OAuth pour l’application comme décrit [ici](https://www.adobe.io/apis/documentcloud/sign/docs.html#!adobedocs/adobe-sign/master/gstarted/configure_oauth.md) et [ici](https://secure.eu1.adobesign.com/public/static/oauthDoc.jsp). Notez votre identifiant client et votre secret client. Ensuite, vous pouvez utiliser `https://www.google.com` comme URI de redirection et les portées suivantes :

* user_login : self

* agreement_read: compte

* agreement_write : compte

* agreement_send : compte

Préparez une URL comme suit en utilisant votre ID client à la place de \&lt;client_id>:

```
https://secure.eu1.adobesign.com/public/oauth?redirect_uri=https://www.google.com
&response_type=code
&client_id=<CLIENT_ID>
&scope=user_login:self+agreement_read:account+agreement_write:account+agreement_send:account
```

Saisissez l’URL ci-dessus dans votre navigateur web. Vous êtes redirigé vers google.com et le code s’affiche dans la barre d’adresse sous la forme code=\&lt;your_code>, par exemple :

```
https://www.google.com/?code=<YOUR_CODE>&api_access_point=https://api.eu1.adobesign.com/&web_access_point=https://secure.eu1.adobesign.com%2F
```

Notez les valeurs données pour \&lt;your_code> et api_access_point.

Pour envoyer une demande de mot de POST HTTP vous fournissant le jeton d’accès, utilisez l’ID client, \&lt;your_code>et api_access_point. Vous pouvez utiliser [Postman](https://helpx.adobe.com/sign/kb/how-to-create-access-token-using-postman-adobe-sign.html) ou cURL :

```
curl --location --request POST "https://**api.eu1.adobesign.com**/oauth/token"
\\

\--data-urlencode "client_secret=**\<CLIENT_SECRET\>**" \\

\--data-urlencode "client_id=**\<CLIENT_ID\>**" \\

\--data-urlencode "code=**\<YOUR_CODE\>**" \\

\--data-urlencode "redirect_uri=**https://www.google.com**" \\

\--data-urlencode "grant_type=authorization_code"
```

L’exemple de réponse se présente comme suit :

```
{
    "access_token":"3AAABLblqZhByhLuqlb-…",
    "refresh_token":"3AAABLblqZhC_nJCT7n…",
    "token_type":"Bearer",
    "expires_in":3600
}
```

Notez votre access_token. Vous en avez besoin pour autoriser votre code client.

## Utilisation du kit SDK Java d’Adobe Sign

Une fois que vous disposez du jeton d’accès, vous pouvez envoyer des appels d’API REST à Adobe Sign. Pour simplifier ce processus, utilisez le kit SDK Java d’Adobe Sign. Le code source est disponible dans le [Référentiel GitHub Adobe](https://github.com/adobe-sign/AdobeSignJavaSdk).

Pour intégrer ce package à votre application, vous devez cloner le code. Ensuite, créez le pack Maven (pack mvn) et installez les fichiers suivants dans le projet (vous les trouverez dans le code complémentaire du dossier adobe-sign-sdk) :

* target/swagger-java-client-1.0.0.jar

* target/lib/gson-2.8.1.jar

* target/lib/gson-fire-1.8.0.jar

* target/lib/hamcrest-core-1.3.jar

* target/lib/junit-4.12.jar

* target/lib/logging-interceptor-2.7.5.jar

* target/lib/okhttp-2.7.5.jar

* target/lib/okio-1.6.0.jar

* target/lib/swagger-annotations-1.5.15.jar

Dans IntelliJ IDEA, vous pouvez ajouter ces fichiers en tant que dépendances en utilisant *Structure du projet* (Structure de fichier/projet).

## Envoi du PDF pour signature

Vous êtes maintenant prêt à envoyer l’accord pour signature. Pour ce faire, commencez par ajouter un autre lien hypertexte à la demande d’envoi :

```
<html>
<head>
    <link rel="stylesheet" href="https://www.w3schools.com/w3css/4/w3.css">
</head>
 
<div class="w3-container ">
    <h1>HR Department</h1>
 
    <h2>Contract file</h2>
 
    <p>Click <a href="/pdf"> here</a> to download your contract</p>
 
    
</div>
</body>
</html>
```

Ensuite, vous ajoutez un autre contrôleur, `AdobeSignController`, dans lequel vous implémentez `sendContractMethod` (voir le code du compagnon). La méthode fonctionne comme suit :

Premièrement, il utilise `ApiClient` pour obtenir le point de terminaison API.

```
ApiClient apiClient = new ApiClient();

//Default baseUrl to make GET /baseUris API call.
String baseUrl = "https://api.echosign.com/";
String endpointUrl = "/api/rest/v6";
apiClient.setBasePath(baseUrl + endpointUrl);

// Provide an OAuth Access Token as "Bearer access token" in authorization
String authorization = "Bearer ";

// Get the baseUris for the user and set it in apiClient.
BaseUrisApi baseUrisApi = new BaseUrisApi(apiClient);
BaseUriInfo baseUriInfo = baseUrisApi.getBaseUris(authorization);
apiClient.setBasePath(baseUriInfo.getApiAccessPoint() + endpointUrl);
```

Ensuite, la méthode utilise le fichier contract.pdf pour créer le document temporaire :

```
// Get PDF file
String filePath = "output/";
String fileName = "contract.pdf";
File file = new File(filePath + fileName);
String mimeType = "application/pdf";
 
//Get the id of the transient document.
TransientDocumentsApi transientDocumentsApi =
    new TransientDocumentsApi(apiClient);
TransientDocumentResponse response = transientDocumentsApi.createTransientDocument(authorization,
    file, null, null, fileName, mimeType);
String transientDocumentId = response.getTransientDocumentId();
```

Vous devez ensuite créer un accord. Pour ce faire, utilisez le fichier contract.pdf et définissez l’état de l’accord sur EN_COURS pour envoyer le fichier immédiatement. Vous pouvez également choisir la signature électronique :

```
// Create AgreementCreationInfo
AgreementCreationInfo agreementCreationInfo = new AgreementCreationInfo();
 
// Add file
FileInfo fileInfo = new FileInfo();
fileInfo.setTransientDocumentId(transientDocumentId);
agreementCreationInfo.addFileInfosItem(fileInfo);
 
// Set state to IN_PROCESS, so the agreement is be sent immediately
agreementCreationInfo.setState(AgreementCreationInfo.StateEnum.IN_PROCESS);
agreementCreationInfo.setName("Contract");
agreementCreationInfo.setSignatureType(AgreementCreationInfo.SignatureTypeEnum.ESIGN);
```

Ensuite, ajoutez les destinataires de l’accord comme suit. Vous ajoutez ici deux destinataires (voir les sections Employé et Responsable) :

```
// Provide emails of recipients to whom agreement is be sent
// Employee
ParticipantSetInfo participantSetInfo = new ParticipantSetInfo();
ParticipantSetMemberInfo participantSetMemberInfo = new ParticipantSetMemberInfo();
participantSetMemberInfo.setEmail("");
participantSetInfo.addMemberInfosItem(participantSetMemberInfo);
participantSetInfo.setOrder(1);
participantSetInfo.setRole(ParticipantSetInfo.RoleEnum.SIGNER);
agreementCreationInfo.addParticipantSetsInfoItem(participantSetInfo);
 
// Manager
participantSetInfo = new ParticipantSetInfo();
participantSetMemberInfo = new ParticipantSetMemberInfo();
participantSetMemberInfo.setEmail("");
participantSetInfo.addMemberInfosItem(participantSetMemberInfo);
participantSetInfo.setOrder(2);
participantSetInfo.setRole(ParticipantSetInfo.RoleEnum.SIGNER);
agreementCreationInfo.addParticipantSetsInfoItem(participantSetInfo);
```

Enfin, envoyez l’accord à l’aide de la boîte de dialogue `createAgreement` à partir du SDK Adobe Sign Java :

```
// Create agreement using the transient document.
AgreementsApi agreementsApi = new AgreementsApi(apiClient);
AgreementCreationResponse agreementCreationResponse = agreementsApi.createAgreement(
    authorization, agreementCreationInfo, null, null);
 
System.out.println("Agreement sent, ID: " + agreementCreationResponse.getId());
```

Après avoir exécuté ce code, vous recevez un courrier électronique (à l&#39;adresse indiquée dans le code comme `<email_address>)` avec la demande de signature d’accord. L’e-mail contient le lien hypertexte, qui dirige les destinataires vers le portail Adobe Sign pour effectuer la signature. Le document s’affiche dans votre portail Adobe Sign Developer (voir la figure ci-dessous) et vous pouvez également suivre le processus de signature par programme à l’aide de la boîte de dialogue [getAgreementInfo](https://github.com/adobe-sign/AdobeSignJavaSdk/blob/master/docs/AgreementsApi.md#getAgreementInfo) méthode.

Enfin, vous pouvez également protéger votre mot de PDF par mot de passe à l’aide de l’API des services de mot de PDF, comme indiqué dans ces [exemples](https://github.com/adobe/pdfservices-java-sdk-samples/tree/master/src/main/java/com/adobe/pdfservices/operation/samples/protectpdf).

![Capture d’écran des détails du contrat](assets/HRWJ_9.png)

## Marche à suivre

Comme vous pouvez le constater, en tirant parti des démarrages rapides, vous pouvez implémenter un formulaire web simple pour créer un PDF approuvé dans Java avec l’API Adobe PDF Services. Les API Adobe PDF s’intègrent facilement dans vos applications client.

En poussant l’exemple plus loin, vous pouvez créer des formulaires que les destinataires peuvent signer à distance en toute sécurité. Lorsque vous avez besoin de plusieurs signatures, vous pouvez même router automatiquement des formulaires vers une série de personnes dans un workflow. L’intégration de vos collaborateurs a été améliorée et votre service RH sera ravi de vous compter parmi ses collaborateurs.

Extraire [[!DNL Adobe Acrobat Services]](https://www.adobe.io/apis/documentcloud/dcsdk/) pour ajouter une multitude de fonctionnalités de PDF à vos applications aujourd&#39;hui.
