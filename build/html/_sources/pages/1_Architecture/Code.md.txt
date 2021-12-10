<img align="right" width="100px" src="../../_images/Foundation29.png">

## 1.4. Level 4: Code
This section will briefly present the code-level components of the Dx29 application.

As we have done in the previous section on components, here again the code projects will be presented according to whether they belong to the frontend, backend or external containers.

In addition, a new classification appears: operations code projects. The latter will contain the projects and code for working with and deploying Dx29 environments. That is to say, they will not be components of the application architecture as such, but they will expose functionalities that allow us to work in a more comfortable, simple and fast way with the application. At this point we are only going to present what they are, later in this document we will detail where they are going to be used, so it is important to mention that they exist and what each of them contains.

Before going into detail in each of the projects, it is important to note that the deployment of this project has been done in Azure, using AKS, container registry, different databases,...  Thus, each of these projects will generate the corresponding Docker images that will be uploaded to the appropriate container registry (this will depend on the environment) and will also contain files for the Deployment configuration in the AKS (also depends on the environment).
The operations container will have different elements that will facilitate these tasks and each of the projects will follow the structure:
>- Folder **src**: The one containing the source code.
>>- In turn, inside this folder we will find the Docker files to compile the necessary images. These images will then be uploaded to the corresponding azure container registry.
>>- Here we will also find the secrets folder, which contains the appsettings.secrets.json file, where the keys and values of the secrets needed to use the project are defined.
>>- Some projects also include the spec folder, which contains the unit tests of the project.
>>- Finally, the rest of the folders that appear will depend on the programming language used to implement the microservice.
>- **Manifests** folder: which has the necessary files for the deployment in the cluster: Deployment Configuration and Kubernetes Service.
>- **Azure-pipeline.yaml** file, which will be used for the automatic deployment.

In general, with the chosen architecture (this applies to all projects) it will be necessary for each project here to be compiled using Docker, and the created image uploaded to the corresponding Kubernetes cluster. This will be covered in section **Build and Deploy**, of this document.

### 1.4.1. Frontend
#### 1.4.1.1. Webapp
The Dx29 frontend has been implemented in the **Dx29.Web project**.

This project consists of several subprojects, mainly:
>- **Dx29.Web.UI**, includes all the elements to build the user interface of the application. It has been programmed using [Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor).
>- **Dx29.Web**. It can be considered as the server of the application, since it contains all the functions and methods necessary to offer the functionalities of Dx29 to the graphical view or client (Dx29.Web.UI). It has been implemented using [C# - ASP.NET](https://docs.microsoft.com/en-GB/visualstudio/get-started/csharp/tutorial-aspnet-core?view=vs-2022).
>- The two previous projects communicate with each other using the **Dx29.Web.Client** project. This includes the services necessary to expose the server functions to the client using a [REST](https://restfulapi.net/) interface, that is, the communication is established according to the [HTTP protocol](https://restfulapi.net/http-methods/). 
>- As this project also communicates with the Dx29 Backend, the server must have access to the functionality exposed by the latter. For this purpose, the **Dx29.Services** project is used.
>- The **Dx29.Data** project will define all the models, extensions, resources and data needed by the previous ones.

Finally, the Dx29 and Dx29.Azure projects are also added. These two are generic and are used as a library to add the common or more general functionalities used in Dx29 projects programmed in C#.

In this repository you can also find the **Dx29.Web.Management** project which will be explained later as it is considered a backend of Dx29.

As it has been programmed with C#, the Dx29.Web project must include a configuration file: appsettings.json. This includes the dependencies with other microservices:

|  Key           | Value     |		                                                         |
|----------------|-----------|---------------------------------------------------------------|
| FileStorage    | Endpoint  |http://dx29-filestorage/api/v1/                                |
| BioEntity      | Endpoint  |http://dx29-bioentity/api/v1/                                  |
| Localization   | Endpoint  |http://dx29-localization/api/v1/                               |
| TermSearch     | Endpoint  |http://dx29-termsearch2:8080/api/v1/                           |
| Diagnosis      | Endpoint  |http://dx29-bionet/api/v1/                                     |
| Management     | Endpoint  |http://dx29-medicalhistory/api/v1/                             |
| MedicalHistory | Endpoint  |http://dx29-medicalhistory/api/v1/                             |
| ResourceGroup  | Endpoint  |http://dx29-medicalhistory/api/v1/                             |
| Annotations    | Endpoint  |http://dx29-annotations/api/v1/                                |
| Exomiser       | Endpoint  |http://dx29-exomiser.northeurope.cloudapp.azure.com/api/v1/    |
| Documents      | Endpoint  |http://dx29-documents/api/v1/                                  |
| Mailing        | Endpoint  |http://dx29-mailing:8080/api/                                  |
| Legacy         | Endpoint  |http://dx29-legacy:8080/api/                                   |
| OpenDx29       | Endpoint  |https://dx29.ai/api                                            |
| PhenSimilarity | Endpoint  |http://dx29-phensimilarity:8080/api/v1/                        |

In addition, this project accesses the user database (SQL) and the blob that openDx29 uses to exchange information with Dx29 v2. It also uses SignalR for notification management and AppInsights for logging. Therefore, in order to run it, the file appsettings.secrets.json must be added to the secrets folder with the following information:

|  Key                 | Value               |		                                                                                |
|----------------------|---------------------|--------------------------------------------------------------------------------------|
| ConnectionStrings    | IdentityConnection  |SQL database endpoint and credentials                                                 |
| ConnectionStrings    | OpenDataBlobStorage |OpenDx29 blob endpoint and credentials                                                |
| SignalR              | ConnectionString    |SignalR connection string & credentials                                               |
| SignalR              | HubName             |SignalR Hub HubName                                                                   |
| AppInsights          | Key                 |Secret key for connecting with AppInsights                                            |
| Account              | Key                 |Secret from SQL database (encrypt)                                                    |
| Account              | Inx                 |Secret from SQL database (encrypt)                                                    |
| Records              | Key                 |Secret from SQL database (encrypt)                                                    |
| Records              | Inx                 |Secret from SQL database (encrypt)                                                    |
| IdentityServer       | Clients             |"Dx29.Web.UI": {"Profile": "IdentityServerSPA"}                                       |
| IdentityServer       | Key                 |"Key": {"Type": "File","FilePath": Path certificate,"Password": Password certificate} |


### 1.4.2. Backend
#### 1.4.2.1. Dx29.Annotations
The annotation service is used to extract information from medical documents. 

It offers the following methods for performing extraction operations asynchronously (Text Analytics):
>- Process. 
>>- To make external queries to Dx29, that is to say, with documents that are not associated to any user profile of the application.
>>- POST request: ```/api/v1/Annotations/process```
>>- Body request: Stream document.
>>- Response: Job Status
>>>- Name
>>>- Token: Job identifier. This will be needed later for job status and/or result queries.
>>>- Date: Date when the query is performed
>>>- Status: String indicating the status of the job. Can be: Failed, Succeeded, Running, Preparing, Pending, Created or Unknown.
>>>- CreatedOn: Date when the operation has made.
>>>- LastUpdate
>- Process/userId/caseId/reportId. 
>>- For internal Dx29 queries, i.e. on users, cases and existing reports. This request will query the blobs to perform the relevant operations on the indicated document (For the user and the case, the report with identifier reportId is searched, and processed).
>>- POST request: ```/api/v1/Annotations/process/{userId}/{caseId}/{reportId}```
>>- Body request: Stream document.
>>- Response: Job status. An object like the one in the request described above is returned.
>- Status. 
>>- To request the status of a given job. The identifier of the job corresponds to the token returned by any of the previous requests.
>>- GET request: ```/api/v1/Annotations/status?params=<token>```
>>- Response: Job status. An object like the one in the request described above is returned.
>- Results.
>>- To obtain the results of a given job. The job identifier corresponds to the token returned by any of the previous requests. 
>>- GET request: ```/api/v1/Annotations/results?params=<token>```
>>- Object with the results of the annotation process:
>>>- Analyzer: If the document wa analyze with OCR or not.
>>>- Segments List. List of the segments extracted as a result, composed by objects with:
>>>>- Id or identifier of the segment
>>>>- Language of the results
>>>>- String Text
>>>>- Source composed by the Language and the string text sources (from the document).
>>>>- Annotations list or the items that are recognised in the result text:
>>>>>- Id or identifier
>>>>>- Text of the annotation
>>>>>- Offset
>>>>>- Length
>>>>>- Category
>>>>>- CondifenceScore
>>>>>- IsNegated
>>>>>- IsDiscarded
>>>>>- List of related links composed by the data of the source and an Id.

It is programmed in C#, and two Docker images will result from this project: Annotations and AnnotationsJobs. The first one will contain the extraction functionality, while the second one will be used to manage the asynchronous jobs.

The structure of the project is as follows:
>- **Dx29.Annotations.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>- **Dx29.Annotations**. It is this project that contains the logic to perform the relevant operations: services and communication with the external NCR and Text Analytics apis.
>- **Dx29.Annotations.Worker**. This project manages the Dispatcher required for the asynchronous functionalities. The administration and communication with the ServiceBus is carried out in this project.
>- **Dx29**, **Dx29.Azure** and **Dx29.Jobs** used as libraries to add the common or more general functionalities used in Dx29 projects programmed in C#.

The project must include a configuration file: appsettings.json. This includes the dependencies with other microservices:

|  Key           | Value     |		                                                         |
|----------------|-----------|---------------------------------------------------------------|
| FileStorage    | Endpoint  |http://dx29-filestorage/api/v1/                                |
| MedicalHistory | Endpoint  |http://dx29-medicalhistory/api/v1/                             |
| Segmentation   | Endpoint  |http://dx29-segmentation:8080/api/v1/                          |
| DocConverter   | Endpoint  |https://f29api.northeurope.cloudapp.azure.com/api/             |

And finally, it should be mentioned that:
>- It depends on a external service: TextAnalytics
>- It accesses the blob 
>- It is necessary to create a ServiceBus, where the asynchronous jobs will be sent, SignalR for notification management, and AppInsights for logging. 

Therefore, in order to run it, the file appsettings.secrets.json must be added to the secrets folder with the following information:

|  Key                 | Value               |		                                                                                |
|----------------------|---------------------|--------------------------------------------------------------------------------------|
| ConnectionStrings    | BlobStorage         |Blob endpoint and credentials                                                         |
| ServiceBus           | ConnectionString    |Connection string service bus                                                         |
| ServiceBus           | QueueName           |Queue configured name                                                                 |
| SignalR              | ConnectionString    |SignalR connection string & credentials                                               |
| SignalR              | HubName             |SignalR Hub HubName                                                                   |
| AppInsights          | Key                 |Secret key for connecting with AppInsights                                            |
| CognitiveServices    | Endpoint            |Endpoint Azure cognitive service configured                                           |
| CognitiveServices    | Authorization       |Authorization key                                                                     |
| CognitiveServices    | Region              |Azure cognitive service region configured                                             |
| TAHAnnotation        | Endpoint            |Endpoint Azure cognitive service configured  for Text Analytics                       |
| TAHAnnotation        | Path                |text/analytics/v3.1/entities/health/jobs                                              |
| TAHAnnotation        | Authorization       |Authorization key                                                                     |
| TAHAnnotation        | BlackList           |User ids in black list                                                                |

#### 1.4.2.2. Dx29.APIGateway
Dx29 exposes an API for third parties to use the tool's algorithms independently of the tool. That is, they will be able to run the calculations they need but not access the specific user and patient information stored in the databases or blobs. 

It therefore offers the methods of the microservices that perform these functions, and simply acts as a link between an external application and the internal microservices located in the application cluster. Thus, the responses obtained in each case will be of the same type as the one described in the specific section for each microservice.
The difference is that now the base address will be: ```http://dx29-api.northeurope.cloudapp.azure.com/api/ ``` and then the controller where the desired requests are located and the path of the request must be specified. 
The APIs that he explains can be consulted in this [link](http://dx29-api.northeurope.cloudapp.azure.com/index.html)

It is programmed in C#, and a Docker image will result from this project.

The structure of the project is as follows:
>- **Dx29.APIGateway**. Where the controllers and methods exposed by this API are implemented. In addition, it will contain the necessary functionality to connect the API controllers and methods to the corresponding microservice.
>- It also includes the generic projects or considered as a library, as it is programmed in C#: **Dx29** and **Dx29.Data**.

It is important to mention that for now this project includes the code of two microservices that are no longer used (and therefore not necessary): Dx29.Bioenties and Dx29.TASearch. These functionalities have already been migrated to other microservices in use.

This project doesn't have any dependency and doesn't need any secret value.

#### 1.4.2.3. Dx29.Bioentity
The Bioentity microservice is used to obtain symptom and disease information.

For this purpose, it offers the following methods organised according to the type of search in different controllers:
>- Conditions Controller: To search for diseases or conditions. Baseline request: ```api/v1/Conditions/```
>>- Describe.
>>>- To obtain information on diseases from a list of IDs and in a given language.
>>>- GET request: ```api/v1/Conditions/describe?id=<List<string>>&lang=<lang>```
>>>- Result: Dictionary with key equal to the identifier of the request and value the list of results obtained. This list of results will be composed of objects with the items:
>>>>- Disease Id (string)
>>>>- Disease Name (string)
>>>>- Disease Description (string)
>>>>- Disease comment (string)
>>>>- Alternates: List of strings of the alternates of this disease
>>>>- Synonyms: list of synonyms with this information: Label, Scope,Type and Xrefs (all strings)
>>>>- Categories, Parents, Children and Consider: List of related items with Id and Name.
>>>>- PubMeds and XRefs (List strings)
>>>>- IsObsolete (bool) and ReplacedBy (Reference of the new disease with its Id and Name).
>>>>- ToString method for printing the disease as: "Id: Name".
>- Phenotype Controller: To search for phenotypes. Baseline request: ```api/v1/Phenotype/```
>>- Describe.
>>>- To obtain information on phenotypes from a list of IDs and in a given language.
>>>- GET request: ```api/v1/Phenotypes/describe?id=<List<string>>&lang=<lang>```
>>>- POST request:  ```api/v1/Phenotypes/describe?lang=<lang>```
>>>- Body request: List symptom identifiers (strings).
>>>- Result: Dictionary with key equal to the identifier of the request and value the list of results obtained. This list of results will be composed of objects with the items:
>>>>- Disease Id (string)
>>>>- Disease Name (string)
>>>>- Disease Description (string)
>>>>- Disease comment (string)
>>>>- Alternates: List of strings of the alternates of this disease
>>>>- Synonyms: list of synonyms with this information: Label, Scope,Type and Xrefs (all strings)
>>>>- Categories, Parents, Children and Consider: List of related items with Id and Name.
>>>>- PubMeds and XRefs (List strings)
>>>>- IsObsolete (bool) and ReplacedBy (Reference of the new disease with its Id and Name).
>>>>- ToString method for printing the disease as: "Id: Name".
>>- Predecessors.
>>>- Get the predeccesors of a symptom with configurable tree depth (-1 is equal to ALL predecessors of a symptom in the tree).
>>>- GET request: ```api/v1/Phenotypes/predecessors?<List<string>>&depth=<int_depth>```
>>>- POST request: ```api/v1/Phenotypes/predecessors?depth=<int_depth>```
>>>- Body request: List symptom identifiers (strings).
>>>- Result request: Dictionary with key equal to the identifier of the request and value is an object with the predecessors information.
>- Terms Controller: To search for diseases or symptoms regardless of type, i.e. a search for terms in all files.
>>- Describe.
>>>- To obtain information on terms from a list of IDs and in a given language.
>>>- Both GET and POST request are available.
>>>>- GET Request: ```api/v1/Terms/describe?id=<List<string>>&lang=<lang>```
>>>>- POST Request:```api/v1/Terms/describe?lang=<lang>```
>>>>- POST body request: List of ids (strings).
>>>- Result: An object like the one in the request of describe diseases method is returned.

It is programmed in C#, and a Docker image will result from this project.

The structure of the project is as follows:
>- **Dx29.Bioentity.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>- **Dx29.Bioentity**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29**. Used as library to add the common or more general functionalities used in Dx29 projects programmed in C#.

Please note that it uses Orphanet, Mondo, Omim and HPO files as sources of information. Please note the updates of these files for this project.
This project doesn't need any secret value.


#### 1.4.2.4. Dx29.BioNET
This microservice contains the Dx29 algorithm for the calculation and suggestion of diseases for a patient from his list of symptoms and/or genes.

In addition, it works with its own BioEntity to obtain the disease information, i.e. it does not access the microservice dedicated to this (in the future this will be changed to use the same as the rest of the microservices and applications).

For this purpose, it offers the following methods organised in different controllers:
>- Diagnosis controller: Baseline request: ``` api/v1/Diagnosis/ ```
>>- Describe. 
>>>- To obtain information on diseases from a list of IDs and in a given language.
>>>- GET request: ```api/v1/Diagnosis/describe?ids=<List<string>>&lang=<lang>```
>>>- Result: Dictionary with the diseases ids requested as keys, and the value is an object with the information of each disease: Id, Name and Description.
>>- Calculate.
>>>- To get the list of Dx29 suggested diseases for a patient from his symptoms and/or genes
>>>- POST request: ```api/v1/Diagnosis/calculate?lang=<lang>&source=<source>&count=<number of results>```
>>>- Body request: DataAnalysis object with the items extracted of the result of the Exomiser execution on the genotype files of the patient (VCFs):
>>>>- symptoms. List of symptoms identifiers (strings).
>>>>- Genes. List of gene items:
>>>>>- Name of the gen
>>>>>- Score of the gen
>>>>>- Combined score
>>>>>- List of diseases identifiers asociated (strings)
>>>- Result:List of objects that contains the information about the diseases results:
>>>>- Id of the disease
>>>>- Name of the disease
>>>>- Description of the disease
>>>>- ScoreDx29, score calculated as total of application Dx29
>>>>- ScoreGenes, score only from genes information
>>>>- ScoreSymptoms, score only from symptoms information
>>>>- Symptoms: The differential diagnosis between disease symptoms and patient symptoms. Is a list of objects with the information of: frecuency of the symptom, if the patient and the disease has it (booleans), if has any related symptom and its relationship.
>>>>- Matches genes. A list of objects with the label of the genes that has de patient and the disease in common.
>>- Phrank.
>>>- To get the list of Phrank suggested diseases for a patient from his symptoms 
>>>>- POST request: ```api/v1/Diagnosis/phrank?lang=<lang>&source=<source>&count=<number of results>```
>>>>- Body request: The same as the previous method (calculate with Dx29 algorithm).
>>>>- Results: The same as the previous method (calculate with Dx29 algorithm).
>- Search controller: Baseline request: ``` api/v1/Search/ ```
>>- Wihout path
>>>- To get the list of Dx29 suggested diseases for a patient ONLY from his symptoms and with another format results
>>>- POST request: ``` api/v1/Search?skip<int>&count=<int>&lang=<lang>&source=<source> ```
>>>- Body request: The same as the previous methods for calculate with Dx29 algorithm or Phrank algorithm.
>>>- Result: Object with the information about Count, total and BestTotal integers and a list of diseases information objects with:
>>>>- The Id, Name and description of the disease
>>>>- The position, Score (total), phenotype score and genes score values.
>>>>- The differential diagnosis between disease symptoms and patient symptoms. Is a list of objects with the information of the frequency, the Id and the name of the symptom, if has related symptom its Id, name and relationship too, the IC and the Score.


It is programmed in C#, and a Docker image will result from this project.

The structure of the project is as follows:
>- **Dx29.BioNET.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>- **Dx29.BioNET**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29**. Used as library to add the common or more general functionalities used in Dx29 projects programmed in C#.
>- **Dx29.Bioentity**. It uses its own implementation of bioentity methods.

This project doesn't have any dependency and doesn't need any secret value.


#### 1.4.2.5. Dx29.Documents
For the management and administration of documents necessary for the correct functioning of the Dx29 application.
For now, only the application's terms and conditions and privacy policy documents have been included, as well as the texts that are sent in the emails.

This microservice will therefore access the corresponding blob and folder (Documents), and will obtain the requested document taking into account that:
>- It may have several versions, therefore a specific one may be requested or it may return the most recent one (it does this if no version is specified).
>- It has documents in different languages and therefore the user must choose which one he/she wants (English or Spanish).

It offers the following methods for this purpose in Documents controller (baseline request: ``` api/v1/Documents/ ```)
>- For specific document and language (and optional version)
>>- Returns the latest or the specified version of the document requested in the choosen language.
>>>- GET request: ``` api/v1/Documents/<document type>/<document name>/<lang>?version=<optional version> ```
>>>- Result: Stream of the document
>- Index.
>>- To get the list of all documents and its versions available.
>>- GET request: ``` api/v1/Documents/index```
>>>- Result: Stream of the document with the versions.

It is programmed in C#, and a Docker image will result from this project.

The structure of the project is as follows:
>- **Dx29.Documents.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>- **Dx29.Documents**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29** and **Dx29.Azure**. Used as libraries to add the common or more general functionalities used in Dx29 projects programmed in C#.

This project doesn't need any dependency but it accesses tthe blob. Therefore, in order to run it, the file appsettings.secrets.json must be added to the secrets folder with the following information:

|  Key                 | Value               |		                                                                                |
|----------------------|---------------------|--------------------------------------------------------------------------------------|
| ConnectionStrings    | BlobStorage         |Blob endpoint and credentials                                                         |


#### 1.4.2.6. Dx29.FileStorage
For the management and administration of user documents for the correct functioning of the Dx29 application.

This microservice will therefore access the corresponding blob and folder (specific user), and will work on the requested document. Therefore needs the configuration file with the secrets: keys and values.

It offers the following methods for this purpose in FileStorage controller (FileStorage2 controller is obsolete):
>- FileStorage controller (baseline request: ``` api/v1/FileStorage/ ```)
>>- File for user, case, resource and name specified.
>>>- To upload a specific document or file of a Dx29 user. 
>>>- POST request: ``` api/v1/FileStorage/file/<userId>/<caseId>/<resourceId>/<name>```
>>>- Body request: Stream of the document.
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- File for user, case, resource and name specified.
>>>- To download a specific document or file of a Dx29 user. 
>>>- GET request: ``` api/v1/FileStorage/file/<userId>/<caseId>/<resourceId>/<name>```
>>>- Result request: The stream of the document.
>>- File for user, case, resource and name specified.
>>>- To delete a specific document or file of a Dx29 user. 
>>>- DELETE request: ``` api/v1/FileStorage/file/<userId>/<caseId>/<resourceId>/<name>```
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Copy file for user, case, resource and name specified.
>>>- To copy a specific document or file of a Dx29 user. 
>>>- PATCH request: ``` api/v1/FileStorage/copy/<userId>/<caseId>/<resourceId>/<name>```
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Move file for user, case, resource and name specified.
>>>- To move a specific document or file of a Dx29 user. 
>>>- PATCH request: ``` api/v1/FileStorage/move/<userId>/<caseId>/<resourceId>/<name>```
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Share file for user, case, resource and name specified.
>>>- To share a specific document or file of a Dx29 user with another.
>>>- PATCH request: ``` api/v1/FileStorage/share/<userId>/<caseId>/<resourceId>/<name>```
>>>- Result request: The new URI of the file shared.

It is programmed in C#, and a Docker image will result from this project.

The structure of the project is as follows:
>- **Dx29.FileStorage.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>- **Dx29.FileStorage**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29** and **Dx29.Azure**. Used as libraries to add the common or more general functionalities used in Dx29 projects programmed in C#.

This project doesn't need any dependency but it accesses the blob. Therefore, in order to run it, the file appsettings.secrets.json must be added to the secrets folder with the following information:

|  Key                 | Value               |		                                                                                |
|----------------------|---------------------|--------------------------------------------------------------------------------------|
| ConnectionStrings    | BlobStorage         |Blob endpoint and credentials                                                         |


#### 1.4.2.7. Dx29.Localization
For the translation of the Dx29 application into English or Spanish. 

This microservice will therefore access the corresponding blob and folder (Literals), and will obtain the requested document taking into account that:
>- It has documents in different languages and therefore the application according to the language of the user profile will load English or Spanish.

It offers the following methods for this purpose in Localization controller  (baseline request: ``` api/v1/Localization/ ```):
>- literal
>>- Get a specific literal in a language
>>- GET request:  ``` api/v1/Localization/literal?key=<key>&lang=<lang> ```
>>- result: String with the value of the literal in the specified language.
>- literals
>>- Get all literals for a language
>>- GET request:  ``` api/v1/Localization/literals?&lang=<lang> ```
>>- result: Dictionary with literals keys and values for the specified language.
>- literal
>>- Delete a literal translation.
>>- DELETE request:  ``` api/v1/Localization/literals?key=<key>&lang=<lang> ```
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>- literal
>>- Update a literal translation.
>>- PUT request:  ``` api/v1/Localization/literals?literal=<Dictionary with key and previous value>&lang=<lang> ```
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>- register
>>- save new literal in specific language.
>>- PUT request:  ``` api/v1/Localization/register?literal=<Dictionary with key and value>&lang=<lang> ```
>>>- Result request: Ok if all is ok, or bad request if any error occurs.

It is programmed in C#, and a Docker image will result from this project.

The structure of the project is as follows:
>- **Dx29.Localization.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>- **Dx29.Localization**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29** and **Dx29.Azure**. Used as libraries to add the common or more general functionalities used in Dx29 projects programmed in C#.

This project doesn't need any dependency but it accesses the blob. In addition, it uses cognitive service for translations [Microsoft translation](https://docs.microsoft.com/en-GB/azure/cognitive-services/translator/translator-overview). Therefore, in order to run it, the file appsettings.secrets.json must be added to the secrets folder with the following information:

|  Key                 | Value               |		                                                                                |
|----------------------|---------------------|--------------------------------------------------------------------------------------|
| ConnectionStrings    | BlobStorage         |Blob endpoint and credentials                                                         |
| CognitiveServices    | Endpoint            |Endpoint Azure cognitive service configured                                           |
| CognitiveServices    | Authorization       |Authorization key                                                                     |
| CognitiveServices    | Region              |Azure cognitive service region configured                                             |

#### 1.4.2.8. Dx29.Mailing
It is used to send emails: with content, subject and attachments, indicating who is sending the email, to whom it is being sent and if there are others in copy.

To perform these tasks it uses the [SendGrid] library (https://www.npmjs.com/package/@sendgrid/mail).

It offers the following methods for this purpose in Sendgrid controller (baseline request: ``` api/ ```)
>- Send email.
>>- This method is the one that performs the function of sending an email.
>>- POST request: 
``` 
api/sendEmail?params={ To: <destination address, From: <request address>, Subject: <subject of the email>, 
CC: <copy>, BCC: <blid copy>, ReplyTo: <To whom the email can be replied to> }
``` 
>> (\*CC, BCC and ReplyTo are optional)
>>- Atachments are added in the header of the request: **dx-attachments** header.
>>>- POST body: Html content of the email
>>>- Result: Data about the events that occur as Twilio SendGrid processes your email, with status code (200 OK).

It is programmed in NodeJS, and a Docker image will result from this project.
The structure of the project is as follows:
>- The **controllers folder** contains the functionality to work with the previous methods. 
>- In the **routes folder**, all the routes of the API appear. It contains the file index.js that links with the controllers, defining the requests that will be available to the external clients. 
>- In the **services folder** the functionality is implemented.
>- In the **secrets folder** we save the keys and values of some variables that must not be public.
>- **Some files**:
>>- index.js: file where the app.js is loaded. It establish the port for connections and listens to requests.
>>- Config.js: configuration file. It contains the keys and values that can be public.
>>- App.js: the crossdomain is established.

This project doesn't need any dependency but it uses Sengrid and AppInsights. Therefore, in order to run it, the file appsettings.secrets.json must be added to the secrets folder with the following information:

|  Key                 | Value               |		                                                                                |
|----------------------|---------------------|--------------------------------------------------------------------------------------|
| SendGrid             | APIKey              |Secret Key for connect SendGrid                                                       |
| AppInsights          | Key                 |Secret key for connecting with AppInsights                                            |


#### 1.4.2.9. Dx29.MedicalHistory
It is used for the management of the Dx29 databases, i.e. it is the microservice that works with the data of each Dx29 user and patient. Therefore, it is not exposed to third parties but only the Dx29 application will have permissions to work with it. Furthermore, it ensures that only the user of the case (or with whom it has been shared) has access to its information.

It has access to the databases and therefore needs the configuration file with the secrets: keys and values.

For this purpose, it offers the following methods organised in different controllers:
>- Medical cases controller: ``` api/v1/MedicalCases/ ```
>>- Get by userId:
>>>- To get the cases for an specific user.
>>>- GET request: ```api/v1/MedicalCases/<userId>?includeDeleted=<bool> ``` (includeDeleted is optional, for getting the deleted cases or not. Default value = false)
>>>- Result: List of medical case objects, with:
>>>>- Id
>>>>- User id owner
>>>>- Status of the case
>>>>- Patient info: an object with:Name, Gender, birdthdate and list of diagnosis diseases (string ids).
>>>>- Resource groups associated with this case. Dictionary with identfiers of the resource group as keys, and a object data as the value with: Type, Name, last update time and a dictionary with the resources of this group (with resource identifier as key, and string with the state or information as value).
>>>>- SharedBy and SharedWith: Who shared it with me and who I shared it with. First with the information of: userId, caseId, Status and last update date. Second with userId, status and last update date.
>>>>- Dates of created on and updated on.
>>>>- Method ToString() to return the userId and medical case id in string format: userId-id.
>>- Get by userId and caseId:
>>>- To get the specific case for an specific user.
>>>- GET request: ```api/v1/MedicalCases/<userId>/<caseId>?includeDeleted=<bool> ``` (includeDeleted is optional, for getting the deleted cases or not. Default value = false)
>>>- Result: Medical case object like the one in the request of get medical cases from userId.
>>- New medical case for userId
>>>- Create new medical case for userId
>>>- POST request: ```api/v1/MedicalCases/<userId>```
>>>- Body request: Object with patient information: Name, gender, birthdate and list of diagnosed diseases.
>>>- Result: Medical case object like the one in the request of get medical cases from userId.
>>- Update medical case:
>>>- Update information of an existing medical case.
>>>- PATCH request: ```api/v1/MedicalCases/<userId>/<caseId>```
>>>- Body request: Object with patient information: Name, gender, birthdate and list of diagnosed diseases.
>>>- Result: Medical case object like the one in the request of get medical cases from userId.
>>- Delete by userId:
>>>- To delete all the cases for an specific user.
>>>- GET request: ```api/v1/MedicalCases/<userId>``` (includeDeleted is optional, for getting the deleted cases or not. Default value = false)
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Delete medical case:
>>>- Delete an existing medical case.
>>>- DELETE request: ```api/v1/MedicalCases/<userId>/<caseId>```
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Share medical case:
>>>- Share information of an existing medical case with another user.
>>>- POST request: ```api/v1/MedicalCases/share/<userId>/<caseId>```
>>>- Body request: String. Email of the user we want to share with.
>>>- Result: Medical case object like the one in the request of get medical cases from userId.
>- Medical cases shared controller: ``` api/v1/MedicalCaseShared/ ```
>>- Medical case shared by:
>>>- Get medical case shared by
>>>- GET request: ```api/v1/MedicalCaseShared/<userId>/<caseId>```
>>>- Result request:
>>- Share medical case:
>>>- To share an specific medical case (caseId) of an user (userId) with another user (email).
>>>- POST request: ```api/v1/MedicalCaseShared/<userId>/<caseId>```
>>>- Body request: Object with the email string and the action (accept, revoke, delete or created)
>>>- Result request: Medical case object like the one in the request of get medical cases from userId.
>>- Stop sharing:
>>>- Stop sharing medical case with a user.
>>>- PATCH request: ```api/v1/MedicalCaseShared/<userId>/<caseId>```
>>>-  Body request: Object with the email string and the action (accept, revoke, delete or created)
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>- Resource groups controller: ``` api/v1/ResourceGroups/ ```
>>- Get resource group
>>>- Get Resource Groups by userId, caseId. Filter by type or name
>>>- GET request: ``` api/v1/ResourceGroups/<userId>/<caseId>?type=<resource type>&name=<resource name> ``` (type and name are optionals, if indicated the method will only return the resources that meets this filters).
>>>- Result request: Resource group object with:
>>>>- Identifier, name and type of the resource group
>>>>- The userId and the caseId associated with this resource group.
>>>>- Created on and updated on dates.
>>>>- A dictionary with the resources associated with resource identifier as key, and string with the state or information as value.
>>- Get resource group by id
>>>- Get Resource Group by Id
>>>- GET request: ``` api/v1/ResourceGroups/<userId>/<caseId>/<groupId>``` 
>>>- Result request: Resource group object with the same items of get resource group previous request.
>>- New resource group:
>>>- Create Resource Group
>>>- POST request: ``` api/v1/ResourceGroups/<userId>/<caseId>?type=<resource type>&name=<resource name> ``` 
>>>- Body request: A list of resources objects with:
>>>>- Identifier and name of the resource
>>>>- Status: undefined, selected, unselected
>>>>- Dictionary of strings with the properties of the resource: path, score, error, errorMessage.
>>>>- The created on and updated on dates.
>>>- Result request: Resource group object with the same items of get resource group previous request.
>>- Update resource group.
>>>- Update an existing resource Group
>>>- PUT request: ``` api/v1/ResourceGroups/<userId>/<caseId>?type=<resource type>&name=<resource name> ``` 
>>>- Body request: A list of resources objects with the same items as the body request of create new resource.
>>>- Result request: Resource group object with the same items of get resource group previous request.
>>- Delete
>>>- Delete Reource Group by groupId
>>>- DELETE request: ``` api/v1/ResourceGroups/<userId>/<caseId>/<groupId>``` 
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>- Resources controller: ``` api/v1/Resources/ ```
>>- Get resources 1
>>>- Get Resources by type?, name?, resourceId?
>>>- GET request:
``` 
api/v1/Resources/<userId>/<caseId>?groupType=<group type>&groupName=<group name>&
<resourceId=<resource id> 
``` 
>>>	(All queries are optional).
>>>- Result response: A dictionary with resourceGroupId.resourceGroupName as keys and list of resource objects as values with
>>>>- Identifier and name of the resource
>>>>- Status: undefined, selected, unselected
>>>>- Dictionary of strings with the properties of the resource: path, score, error, errorMessage.
>>>>- The created on and updated on dates.
>>- Get resources 2.
>>>- Get Resource by groupId and resourceId
>>>- GET request: ``` api/v1/Resources/<userId>/<caseId>/<groupId>/<resourceId> ```
>>>- Result response: A resource object with:
>>>>- Identifier and name of the resource
>>>>- Status: undefined, selected, unselected
>>>>- Dictionary of strings with the properties of the resource: path, score, error, errorMessage.
>>>>- The created on and updated on dates.
>>- Get resources 3
>>>- Get Resource by type, name and resourceId
>>>- GET request: ``` api/v1/Resources/<userId>/<caseId>/<groupType>/<groupName>/<resourceId> ```
>>>- Result response: A resource object with:
>>>>- Identifier and name of the resource
>>>>- Status: undefined, selected, unselected
>>>>- Dictionary of strings with the properties of the resource: path, score, error, errorMessage.
>>>>- The created on and updated on dates.
>>- Upsert resources 1
>>>-  Upsert Resources grouped by groupId
>>>- PUT request: ``` api/v1/Resources/<userId>/<caseId>/```
>>>- Body request: A dictionary with resourceGroupId.resourceGroupName as keys and list of resource objects as values with
>>>>- Identifier and name of the resource
>>>>- Status: undefined, selected, unselected
>>>>- Dictionary of strings with the properties of the resource: path, score, error, errorMessage.
>>>>- The created on and updated on dates.
>>>- Result request: The same object send in body updated.
>>- Upsert resources 2:
>>>- Upsert Resources by groupId
>>>- PUT request  ``` api/v1/Resources/<userId>/<caseId>/<resourceGroupId>/```
>>>- Body request: List of resource objects with:
>>>>- Identifier and name of the resource
>>>>- Status: undefined, selected, unselected
>>>>- Dictionary of strings with the properties of the resource: path, score, error, errorMessage.
>>>>- The created on and updated on dates.
>>>- Result request: Resource group object with:
>>>>- Identifier, name and type of the resource group
>>>>- The userId and the caseId associated with this resource group.
>>>>- Created on and updated on dates.
>>>>- A dictionary with the resources associated with resource identifier as key, and string with the state or information as value.
>>- Upsert resources 3:
>>>- Upsert Resources by type, name
>>>- PUT request: ``` api/v1/Resources/<userId>/<caseId>/<groupType>/>groupName>```
>>>- Body request: List of resource objects with:
>>>>- Identifier and name of the resource
>>>>- Status: undefined, selected, unselected
>>>>- Dictionary of strings with the properties of the resource: path, score, error, errorMessage.
>>>>- The created on and updated on dates.
>>>- Result request: Resource group object with:
>>>>- Identifier, name and type of the resource group
>>>>- The userId and the caseId associated with this resource group.
>>>>- Created on and updated on dates.
>>>>- A dictionary with the resources associated with resource identifier as key, and string with the state or information as value.
>>- Delete resources 1:
>>>- Delete Resources by groupId, resourceId
>>>- DELETE request: ``` api/v1/Resources/<userId>/<caseId>/<groupId>?resourceId=<list of resource ids>```
>>>- Result request:Resource group object with:
>>>>- Identifier, name and type of the resource group
>>>>- The userId and the caseId associated with this resource group.
>>>>- Created on and updated on dates.
>>>>- A dictionary with the resources associated with resource identifier as key, and string with the state or information as value.
>>- Delete resources 2:
>>>- Delete Resources by group type, group name, resourceId
>>>- DELETE request: ``` api/v1/Resources/<userId>/<caseId>/<groupType>/<groupName>?resourceId=<list of resource ids>```
>>>- Result request: Resource group object with:
>>>>- Identifier, name and type of the resource group
>>>>- The userId and the caseId associated with this resource group.
>>>>- Created on and updated on dates.
>>>>- A dictionary with the resources associated with resource identifier as key, and string with the state or information as value.

It is programmed in C#, and a Docker image will result from this project.
 
The structure of the project is as follows:
>- **Dx29.MedicalHistory.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>- **Dx29.MedicalHistory**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29** and **Dx29.Azure**. Used as libraries to add the common or more general functionalities used in Dx29 projects programmed in C#.

This project doesn't need any dependency but it accesses the data bases. Therefore, in order to run it, the file appsettings.secrets.json must be added to the secrets folder with the following information:

|  Key                 | Value               |		                                                                                |
|----------------------|---------------------|--------------------------------------------------------------------------------------|
| CaseRecords          | AppName             |Appplication name                                                                     |
| CaseRecords          | DatabaseName        |Database Name where case records are saved                                            |
| CaseRecords          | ConnectionString    |Connection string to case records database                                            |
| MedicalCase          | AppName             |Appplication name                                                                     |
| MedicalCase          | DatabaseName        |Database Name where medical cases are saved                                           |
| MedicalCase          | ConnectionString    |Connection string to medical cases database                                           |
| ResourceGroups       | AppName             |Appplication name                                                                     |
| ResourceGroups       | DatabaseName        |Database Name where resource groups are saved                                         |
| ResourceGroups       | ConnectionString    |Connection string to resource groups database                                         |
| Account              | Key                 |Secret from SQL database (encrypt)                                                    |
| Account              | Inx                 |Secret from SQL database (encrypt)                                                    |
| Records              | Key                 |Secret from SQL database (encrypt)                                                    |
| Records              | Inx                 |Secret from SQL database (encrypt)                                                    |

#### 1.4.2.10. Dx29.PhenSimilarity
It is used to obtain the differential diagnosis at the phenotypic level between a patient's symptoms and those of a disease.

For this purpose, it offers the following method: 
>- Calculate
>>- Getting the differential diagnosis between two list of symptoms
>>- POST request: ```api/v1/calculate```
>>- Body request: 
```
{"list_reference":<list of the patient symptoms ids (string)>, 
"list_compare":<list of the disease symptoms ids (string)>]}
```
>>- Response: List of objetcs with:
>>>- Symptom identifier (HP)
>>>- HasPatient and HasDisease booleans, indicating if the patient and if the disease has or not this symptom.
>>>- RelatedId: if the symptom is present in patient or disease by inheritance, here appears the related symptom identifier that has been used to assume the value of the previous booleans.
>>>- Relationship: Relationship between symptom and related symptom (Equals, Predeccessor or Successor)
>>>- Score: Relationship type score

It is programmed in NodeJS, and a Docker image will result from this project.
The structure of the project is as follows:
>- The **controllers folder** contains the functionality to work with the previous method. 
>- In the **routes folder**, all the routes of the API appear. It contains the file index.js that links with the controllers, defining the requests that will be available to the external clients. 
>- In the **services folder** the functionality is implemented.
>- **Some files**:
>>- index.js: file where the app.js is loaded. It establish the port for connections and listens to requests.
>>- Config.js: configuration file. It contains the keys and values that can be public.
>>- App.js: the crossdomain is established.

This project deppends on Dx29.Bioentity, so in config file we must include:

|  Key                 | Value                                             |		                                                                              |  
|----------------------|---------------------------------------------------|--------------------------------------------------------------------------------------|
| f29bio               | http://dx29-bioentity                             | API for get symptoms information (predecessors method)                               |


This project doesn't need any secret value.

#### 1.4.2.11. Dx29.Segmentation
It is used to divide a text into paragraphs, lines, sections,... before sending it to the annotation service.

For this purpose, it offers the following method: 
>- POST request: ```document/segmentation?lan=<language>```
>- Body request: An object like: ``` {"Text":<string>}``` 
>- Result request: An object with: 
>>- 'language_source': The language detected in the source document or selected by the query.
>>- 'segments': A list of objects with a identifier and source of the segment.

It is programmed in Python, and a Docker image will result from this project.
The structure of the project is as follows:
>- **RunWeb.py** file: Is the main file, that access the aforementioned method.
>- **WebAPI** folder: With the files to expose the method funcionality.
>- **Lib** folfer: with the files to tah contains the logic to perform the relevant operations.

This project doesn't have any dependency and doesn't need any secret value.

#### 1.4.2.12. Dx29.TermSearch2
This microservice allows the searches described in the Dx29 application to be carried out: both for symptoms and diseases. That is, it will be the one called by the search boxes.

For this purpose, it offers the following method: 
>- Search symptoms:
>>- GET request: ``` api/v1/search/symptoms?q={query}&lang={lang}&rows={rows} ```
>>- Result request: List of Terms (objects) with: identifier, name and description strings, and a list of strings with the identifiers of the synonyms.
>- Search diseases:
>>- GET request: ``` api/v1/search/diseases?q={query}&lang={lang}&rows={rows} ```
>>- Result request: List of Terms (objects) with: identifier, name and description strings, and a list of strings with the identifiers of the synonyms.

It is programmed in Python, and a Docker image will result from this project.

The structure of the project is as follows:
>- **app.py** file: Is the main file, that access the aforementioned methods.
>- **WebAPI** folder: With the files to expose the method funcionality.
>- **Lib** folfer: with the files to tah contains the logic to perform the relevant operations.


Please note that it uses diseases-terms-lang.json and symptom-terms-lang.json files as sources of information. Please note the updates of these files for this project.
This project doesn't need any secret value.


#### 1.4.2.13. Dx29.WebManagement
This project will allow the use of methods in the Dx29 web application or the frontend (Dx29.Web) that require authentication and authorisation. For example, those related to the databases: Medical History.
It will be used on demand, in order to be able to perform automated actions.

It shall not be exposed for use by third parties. It is an internal management tool for Fundation 29 programmers.

It will present the following methods organised in different controllers: Requests baseline: ```management/api/v1/```
>- Analysis controller: ```management/api/v1/DataAnalysis ```
>>- Get analysis list:
>>>- GET request: ```analysis/{CaseId} ```
>>>- Result request: List of analysis yet executed for a medical case. It uses the Dx29.BioNET service, saving the results in the corresponding medical history blob (the ones to be read with this method) and therefore its result will be the object returned by this microservice.
>>- Get specific analysis:
>>>- GET request: ```management/api/DataAnalysis/analysis/{CaseId}/{resourceId} ```
>>- Result request: Analysis execution speficic result. Now only the result of the execution of Dx29.BioNET with the identifier of the specified resource will be read, and therefore the result will be only an object with the structure returned by the microservice.
>>- Process new analysis
>>>- POST request: ```analysis/{CaseId} ```
>>>- Body request: List of the symptoms identifiers of this case (string).
>>>- Result request: It uses the Dx29.BioNET service, therefore its result will be the object returned by this microservice and the resource identifier asociated that has been created in the databases.
>>- Calculate new analysis.
>>>- POST request:```calculateDiagnosis ```
>>>- Body request: The expected body of Dx29.BioNET microservice, with the information of the symptoms and the genes of a medical case.
>>>- Result request:It uses the Dx29.BioNET service and not created any new resource in Dx29 databases, so its result will be the object returned by this microservice and the resource identifier asociated.
>>- Obtain gene information in th format that Dx29.BioNET expects:
>>>- POST request: ```geneInput ```
>>>- Body request: Exomiser JSON result.
>>>- Result request: Genes information in the format that Dx29.BioNET expects as body input.
>>- Delete a specific analyisis:
>>>- DELETE request: ```analysis/{CaseId}/{resourceId} ```
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Get gene report used in a specific analysis:
>>>- GET request:  ```geneReport/{CaseId}/{resourceId} ```
>>>- Result request: The identifiers if the gene reports used in that analysis.
>- File storage controller, exponse the methods for communicate with this microservice.```management/api/v1/FileStorage ```
>>- Upload multipart files
>>>- POST request: ```{caseId} ```
>>>- Data request: Content-type in headers (max 128) and string text in body.
>>>- Data validation: This methods thorws error if content is empty or if there's a file present without form data. If form data is present, this method immediately fails and returns the model error.
>>>- This method doesn't trust the file name sent by the client. To display the file name, HTML-encode the value.
>>>- Result request: The resource identifier created (string)
>>- Upload simple file: 
>>>- POST request: ```{caseId}/{name}```
>>>- Body request: Content of the file.
>>>- Data validation: this methods throw error if the name is too large (max 128) or file size is zero or file size is too large (2Gb).
>>>- Result request: A file model object with: resource identifier, file name, size, thresold, extension, file type (Medical or Genetic), if is phenotype or of is genotype booleans, and two methods for get file type and file extension.
>>- Get or download a file content 1:
>>>- GET request: ```{caseId}/{resourceId}/{name}```
>>>- Data validation: this methods throws a exception if the name is too large (max 128).
>>>- Result request: A File stream object with the stream, content-type = application/octet-stream, and the file name.
>>- Get or download a file content 2:
>>>- GET request: ```{caseId}/{groupType}/{groupName}/{resourceId}```
>>>- Data validation: this methods throws a exception if resource not found.
>>>- Result request: A File stream object with the stream, content-type = application/octet-stream, and the file name.
>>- Delete file:
>>>- DELETE request:```{caseId}/{name}```
>>>- Data validation: this methods throws a exception if the name is too large (max 128).
>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>- Some controllers that access data bases and manage the information about different types of resources: Notes, Patients, Reports and Symptoms. And a controller for manage medical cases and resource groups associated with a case: Medical History controller.
>>- Notes controller: ```management/api/v1/Notes ```
>>>- Get all notes for a case:
>>>>- GET request: ```Notes/all/{caseId} ```
>>>>- Result request: List of note objects, with the dentifier, name and content of the note, and the created on and updated on dates.
>>>- Get specific note for a case:
>>>>- GET request: ```get/{caseId}/{noteId} ```
>>>>- Result request: A note object, with the dentifier, name and content of the note, and the created on and updated on dates.
>>>- Create new note for a case:
>>>>- POST request: ```create/{caseId} ```
>>>>- Body request: A note object, with the name and content of the note.
>>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>>- Update note for a case:
>>>>- POST request: ```update/{caseId}?noteId=<note identifier> ```
>>>>- Body request: A note object, with the name and content of the note.
>>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>>- Delete note for a case:
>>>>- POST request: ```delete/{caseId}/{noteId}```
>>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Patients controller: ```management/api/v1/Patients ```
>>>- Get patients for a medical case:
>>>>- GET request: ```{caseId}```
>>>>- Result request: A patient model object with:
>>>>- Identifier
>>>>- Status of the case
>>>>- Patient info: an object with:Name, Gender, birdthdate and list of diagnosis diseases (string ids).
>>>>- The number of symptoms, medical reports and genotype reports.
>>>>- SharedWith: who shared it with. With userId, status and last update date.
>>>>- Dates of created on and updated on.
>>>>- Method IsCaseOwner: if this patient is the owner of the case.
>>>>- Methods IsShared and IsSharing: status of sharing this case, and method CanBeShared for determine if this case can be or not shared with others.
>>>- Get summary information:
>>>>- GET request: ```summary/{caseId}```
>>>>- Result request: A patient model object with:
>>>>>- Symptoms summary: Object with the status and the last update of each symptom of the case.
>>>>>- Genes summary: Object with the status and the last update of each symptom of the case.
>>>>>- Reports summary: Object with the status and the last update of each medical report of the case.
>>>>>- Genotype summary: Object with the status and the last update of each genotype report of the case.
>>>>>- DataAnalysis summary: Object with the status and the last update of each data analysis execution of the case.
>>>>>- Last update date.
>>>- Create new patient:
>>>>- POST request: Empty path
>>>>- Body request: Create patient model object with: Patient info (name, gender, birthdate and diseases ids), the reports (File model) and the list of symptoms identifiers (string).
>>>>- Result request: Patient model object.
>>>- Update patient: 
>>>>- PATCH request: ```{caseId}```
>>>>- Body request: Patient info model object with: name, gender, birthdate and diseases ids.
>>>>- Result request: Patient model object.
>>>- Delete patient:
>>>>- DELETE request: ```{caseId}```
>>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>>- Delete all patients for the actual user:
>>>>- DELETE request: Empty path
>>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Gene reports controller: ```management/api/v1/GeneReports ```
>>>- Get all genotype reports for a case:
>>>>- GET request: ```all/{caseId}```
>>>>- Result request: List of report model objects with: identifier, name, status, Error description (if any error occurs it will be described in this field), Error code (if any error occurs the code will be appear in this field), size, the created on date, and two methods for get the extension of the file and if is ready (Processing on this file is either finished or not.)
>>>- Get specific genotype results:
>>>>- GET request: ```results/{caseId}/{resourceId}```
>>>>- Result request: List of genotype info objects, with: 
>>>>- Name
>>>>- Whitelisted, with: Name, Color and Text color.
>>>>- List of clinVar items, with: Name, Color, Text color and Link.
>>>>- Score with: Value, Color and TextColor.
>>>>- List of gen info with:
>>>>>- Whitelisted, with: Name, Color and Text color.
>>>>>- List of clinVar items, with: Name, Color, Text color and Link.
>>>>>- ModeOfInheritance, with: Name, Name short and Link.
>>>>>- List of Mutation objects with: Name and Link.
>>>>>- The Chromosome
>>>>>- VariantEffect with: Name, Color, Text color and Link.
>>>>>- The ExomiserScore, PhenotypeScore and VariantScore, each one with: Value, Color and TextColor.
>>>>>- List of Literature items with: Name and Link.
>>>>>- List of Frequency items with: Name, Value and Link.
>>>>>- List of PredictedPathogenicityScores_ with: Name and Value.
>>>- Get specific genotype results in Exomiser result format:
>>>>- GET request: ```exomiser/{caseId}/{resourceId}```
>>>>- Result request: Exomiser JSON object.
>>>- Process genotype file:
>>>>- PUT request: ```process/{caseId}/``` or only ```{caseId}```
>>>>- Body request: File model object.
>>>>- Result request: Job status object of the proccess, with: Name, Toke, Status, Created on and updated on dates, ErrorCode (if any error occurs the code will be appear in this field), Message (describe the execution), Details, and logs object (with date time, status and message).
>>>- Delete genotype file:
>>>>- DELETE request: ```delete/{caseId}/{reportId}```
>>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>>- Manage notifications of Genotype file process:
>>>>- POST request: ```Notification```
>>>>- Body request: Exomiser notification object with: userId, CaseId, ResourceId and token.
>>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Medical reports controller: ```management/api/v1/PhenReports ```
>>>- Get all medical reports for a case:
>>>>- GET request: ```all/{caseId}```
>>>>- Result request: List of report model objects with: identifier, name, status, Error description (if any error occurs it will be described in this field), Error code (if any error occurs the code will be appear in this field), size, the created on date, and two methods for get the extension of the file and if is ready (Processing on this file is either finished or not.)
>>>- Get specific medical results:
>>>>- GET request: ```annotations/{caseId}/{reportId}```
>>>>- Result request: List of DocAnnotations objects, with: Analyzer string and segment objects: with identifier, text, source, language and list of annotations (Yet described in Dx29.Annotations microservice).
>>>- Process a medical file:
>>>>- PUT request:```process/{caseId}/```
>>>>- Body request: This uses the microservice of Dx29.Annotations, so the body must be the same as the one expected by the microservice to process a file: an object of type FileModel.
>>>>- Result request: Job status object of the proccess, with: Name, Toke, Status, Created on and updated on dates, ErrorCode (if any error occurs the code will be appear in this field), Message (describe the execution), Details, and logs object (with date time, status and message).
>>>- Delete medical file
>>>>- DELETE request: ```delete/{caseId}/{reportId}```
>>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Reports controller: ```management/api/v1/Reports ```. It includes on the one hand the requests to obtain the results of the medical and genotypic files.
In addition to this, it adds two new methods: 
>>>- To obtain all the files of a case independently of the type (looking for both medical and genetic).
>>>>- GET request: ```{caseId}```
>>>>- Result request: List of report model objects with: identifier, name, status, Error description (if any error occurs it will be described in this field), Error code (if any error occurs the code will be appear in this field), size, the created on date, and two methods for get the extension of the file and if is ready (Processing on this file is either finished or not.)
>>>- To save in a case the files that we want independently of the type (classifying them in medical or genetic automatically).
>>>>- PUT request: ```{caseId}```
>>>>- Body request: File model object.
>>>>- Result request: Job status object of the proccess, with: Name, Toke, Status, Created on and updated on dates, ErrorCode (if any error occurs the code will be appear in this field), Message (describe the execution), Details, and logs object (with date time, status and message).
>>- Symptoms controller: ```management/api/v1/Symptoms ```
>>>- Get symptoms of a case:
>>>>- GET request: ```{caseId}?name=<group name>&lang=<language>```. Resource group name and language are optional.
>>>>- Result request: 
>>>>>- If language is null List of symptom objects with:  Identifier, Status, LastUpdate and source of the symptom (Identifier, name and segments). It includes some methods for each symptom too: IsKeySymptom, IsSelected, IsRemoved,IsUnselected, IsUndefined, SelectionOrder, HasDocument, MergLastUpdate and MergeStatus.
>>>>>- If language is NOT null List of symptom description objects with: The same items as a symptom type object, plus: Name, Desc and the categories (identifier, name). 
>>>- Update symptoms of a case:
>>>>- PUT request: ```{caseId}```
>>>>- Body request: Dictionary with symptom identifier as key and symptom status as value.
>>>>- Result request: Ok if all is ok, or bad request if any error occurs.
>>- Medical History controller: ```management/api/v1/MedicalHistory ```
>>>- Get a specific medical case:
>>>>- GET request: ```medicalcase/{caseId} ```
>>>>- Result request: 
>>>>>- Id
>>>>>- User id owner
>>>>>- Status of the case
>>>>>- Patient info: an object with:Name, Gender, birdthdate and list of diagnosis diseases (string ids).
>>>>>- Resource groups associated with this case. Dictionary with identfiers of the resource group as keys, and a object data as the value with: Type, Name, last update time and a dictionary with the resources of this group (with resource identifier as key, and string with the state or information as value).
>>>>>- SharedBy and SharedWith: Who shared it with me and who I shared it with. First with the information of: userId, caseId, Status and last update date. Second with userId, status and last update date.
>>>>>- Dates of created on and updated on.
>>>- Get resource groups for a medical case:
>>>>- GER request: ```resourcegroups/{caseId} ```
>>>>- Result request: List of Resource group items:
>>>>>- Identifier, name and type of the resource group
>>>>>- The userId and the caseId associated with this resource group.
>>>>>- Created on and updated on dates.
>>>>>- A dictionary with the resources associated with resource identifier as key, and string with the state or information as value.
>- Users management controllers:
>>- Users controller: ```management/api/v1/Users ```
>>>- Create user:
>>>>- POST request: Empty path
>>>>- Body request: Create user model object with: mail, password,First name, Last name,Language and role (Patient or Physician).
>>>>- Result request:  Ok if all is ok, or bad request if any error occurs.
>>>- Get user by email:
>>>>- GET request: Empty path and email in query
>>>>- Result request: string user identifier.
>>>- Delete user:
>>>>- DELETE request: Empty path and email in query
>>>>- Result request: Ok if all is ok, Not found if specified user is not registered, or bad request if any error occurs.
>>- Login controller: ```management/api/v1/Login ```
>>>- Authenticate
>>>>- POST request: Empty path
>>>>- Body request: User model object with: Email and password.
>>>>- Result request: string token.

This project consists of several subprojects, mainly:
>- **Dx29.WebManagement**, it contains all the functions and methods necessary to offer the functionalities of Dx29. It has been implemented using [C# - ASP.NET](https://docs.microsoft.com/en-GB/visualstudio/get-started/csharp/tutorial-aspnet-core?view=vs-2022). 
>- Finally, the **Dx29.Data** and **Dx29.Azure** projects are also added. These two are generic and are used as a library to add the common or more general functionalities used in Dx29 projects programmed in C#.

It has the same dependencies and needs the same secrets as Dx29.Web:
As it has been programmed with C#, the project must include a configuration file: appsettings.json. This includes the dependencies with other microservices:

|  Key           | Value     |		                                                         |
|----------------|-----------|---------------------------------------------------------------|
| FileStorage    | Endpoint  |http://dx29-filestorage/api/v1/                                |
| BioEntity      | Endpoint  |http://dx29-bioentity/api/v1/                                  |
| Localization   | Endpoint  |http://dx29-localization/api/v1/                               |
| TermSearch     | Endpoint  |http://dx29-termsearch2:8080/api/v1/                           |
| Diagnosis      | Endpoint  |http://dx29-bionet/api/v1/                                     |
| Management     | Endpoint  |http://dx29-medicalhistory/api/v1/                             |
| MedicalHistory | Endpoint  |http://dx29-medicalhistory/api/v1/                             |
| ResourceGroup  | Endpoint  |http://dx29-medicalhistory/api/v1/                             |
| Annotations    | Endpoint  |http://dx29-annotations/api/v1/                                |
| Exomiser       | Endpoint  |http://dx29-exomiser.northeurope.cloudapp.azure.com/api/v1/    |
| Documents      | Endpoint  |http://dx29-documents/api/v1/                                  |
| Mailing        | Endpoint  |http://dx29-mailing:8080/api/                                  |
| Legacy         | Endpoint  |http://dx29-legacy:8080/api/                                   |
| OpenDx29       | Endpoint  |https://dx29.ai/api                                            |
| PhenSimilarity | Endpoint  |http://dx29-phensimilarity:8080/api/v1/                        |

In addition, this project accesses the user database (SQL) and the blob that openDx29 uses to exchange information with Dx29 v2.. It also uses SignalR for notification management and AppInsights for logging. Therefore, in order to run it, the file appsettings.secrets.json must be added to the secrets folder with the following information:

|  Key                 | Value               |		                                                                                |
|----------------------|---------------------|--------------------------------------------------------------------------------------|
| ConnectionStrings    | IdentityConnection  |SQL database endpoint and credentials                                                 |
| ConnectionStrings    | OpenDataBlobStorage |OpenDx29 blob endpoint and credentials                                                |
| SignalR              | ConnectionString    |SignalR connection string & credentials                                               |
| SignalR              | HubName             |SignalR Hub HubName                                                                   |
| AppInsights          | Key                 |Secret key for connecting with AppInsights                                            |
| Account              | Key                 |Secret from SQL database (encrypt)                                                    |
| Account              | Inx                 |Secret from SQL database (encrypt)                                                    |
| Records              | Key                 |Secret from SQL database (encrypt)                                                    |
| Records              | Inx                 |Secret from SQL database (encrypt)                                                    |
| IdentityServer       | Clients             |"Dx29.Web.UI": {"Profile": "IdentityServerSPA"}                                       |
| IdentityServer       | Key                 |"Key": {"Type": "File","FilePath": Path certificate,"Password": Password certificate} |


### 1.4.3. External containers
In the previous subsections it has been indicated when a project had external dependencies and which were these. In summary, Dx29 requires [Exomiser](https://github.com/exomiser/Exomiser), [Microsoft translator](https://docs.microsoft.com/en-GB/azure/cognitive-services/translator/translator-overview) and [F29API](https://f29api.northeurope.cloudapp.azure.com/index.html).

The documentation of each of these projects is not the subject of this manual. It is only necessary to understand that they expose APIs, with the endpoints that have been indicated here, and that their methods will be used, so you only need to understand the type of request, the input and output of each one.

### 1.4.4. Operation containers
#### 1.4.4.1. Pull images and Docker compose.
The Dx29.Compose project includes two functionalities:
> 1. Obtain the images of an environment from the Dx29 application. To do this you must have permissions in Azure in the Foundation29 subscription (therefore it is internal, for the programmers of the foundation). It is in the **PullImages** folder of the project and is programmed in Python. With its execution, the latest versions of the images of the development environment are downloaded locally (with a few code modifications, changing the endpoints, it can be quickly adapted to download those of another environment).
> 2. This project also includes the **DockerCompose** file. Once the images are downloaded, this can be used to build the local container with the Docker images and ports that Foundation29 developers work with.

This project therefore accesses the Azure container registry of a selected environment. This will be covered in more depth in the **Build & Deploy** section of this document.

#### 1.4.4.2. Dx29.ENVTEST and Dx29.ENVPROD
As will be explained later in the **Build & Deploy** section of this document, the automation of microservices has been implemented in each of the application environments in Azure. So far we have seen that each project has its own manifest files and azure-pipeline.yaml, that will be useful to deploy the development environment to the programmers. To deploy the test and production environments, these two projects will be used, in each of them defining the manifest and the YAML of each microservice that we want to use.

Again, these projects will access the container registry of the test and production environments, as well as the clusters (AKS). This will be explained later in the **Build & Deploy** section, where we will see the true usefulness and necessity of these.

At this point, we are going to describe and explain in detail what each of these projects contains. 
Both projects have the following structure in common:
>- **azure-pipelines.yam** file. It contains the main definition of the pipeline to be executed for automatic deployment. It contains the main definition of the pipeline to be executed for the automatic deployment. In it, the project directories are accessed and reference is made to the different templates for each of the microservices.
>- **Templates** folder. A template is defined individually for each microservice with the jobs to be performed by the pipeline to compile and deploy that microservice. In other words, the pipelines for each microservice are found here.
>- **manifests** folder. As in each of the projects, this folder includes the files necessary for the configuration of the deployment and service of the microservice in the corresponding cluster. 
>- **vars** folder. The files with the variables and values necessary for the correct execution of the pipeline will be located in this folder. In both projects there is a file vars_pipeline.yaml where the variables related to the container registry, the virtual machine on which the pipeline will be executed and the name of the images to be compiled and deployed are defined. 

The main difference comes in the variable folder, 
> 1. Dx29.ENVTEST. It only contains the file vars_pipeline.yaml, so that the version of the images is configured as an input parameter to the pipeline: in each execution a value is chosen and it is common for all the images to be compiled and deployed in that execution.
> 2. Dx29.ENVPROD. In addition to containing the vars_pipeline.yaml file, it also includes: vars_tag.yaml. The tags or versions of each of the images to be deployed will be defined in this file. This pipeline does not have input parameters but it will read this file and according to what has been specified it will compile and deploy the images in such a way that each microservice can have a different one.


#### 1.4.4.3. Environments
During the implementation of the application, as we will see in the Environments section, three environments have been deployed: DEV, TEST and PROD. The previous projects help us to: generate the images, upload them to the corresponding container registry and deploy them in the cluster already created in the corresponding environment.
This project shows how these environments have been created in Azure: Creation of the AKS, cluster, Association of a container registry, etc. and also the configuration to expose different IPs with DNS and HTTPs communication and the inclusion of the file with the secrets.

So, here are presented the different scripts to perform these functions in each of the environments.

These actions will only be necessary for the creation of new environments, therefore, although they are related to the Build and deploy of the application, they do not influence the designed automation and they will only have to be executed once. For this reason, we have left them as scripts that indicate the commands to be executed on the command line.
