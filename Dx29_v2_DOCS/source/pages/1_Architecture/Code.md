<img align="right" width="100px" src="../../_images/Foundation29.png">

## 1.4. Level 4: Code
This section will briefly present the code-level components of the Dx29 application.
As it consists of several projects that are necessary for its execution, they will be detailed in a more specific way in the following section: **2. Code repositories and projects**.

In general, with the chosen architecture (this applies to all projects) it will be necessary for each project here to be compiled using Docker, and the created image uploaded to the corresponding Kubernetes cluster. This will be covered in section **6.Build and Deploy**, of this document.

As we have done in the previous section on components, here again the code projects will be presented according to whether they belong to the frontend, backend or external containers.

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

### 1.4.2. Backend
#### 1.4.2.1. Dx29.Annotations
The annotation service is used to extract information from medical documents. 

It offers the following methods for this purpose:
>-Methods for performing extraction operations asynchronously (Text Analytics). Baseline request: ```/api/v1/Annotations/```
>>-Process. 
>>>- To make external queries to Dx29, that is to say, with documents that are not associated to any user profile of the application.
>>>- POST request: ```/api/v1/Annotations/process```
>>>- Body request: Stream document.
>>>- Response: Job Status
>>>>- Name
>>>>- Token: Job identifier. This will be needed later for job status and/or result queries.
>>>>- Date: Date when the query is performed
>>>>- Status: String indicating the status of the job. Can be: Failed, Succeeded, Running, Preparing, Pending, Created or Unknown.
>>>>- CreatedOn: Date when the operation has made.
>>>>- LastUpdate
>>-Process/userId/caseId/reportId. 
>>>- For internal Dx29 queries, i.e. on users, cases and existing reports. This request will query the blobs to perform the relevant operations on the indicated document (For the user and the case, the report with identifier reportId is searched, and processed).
>>>- POST request: ```/api/v1/Annotations/process/{userId}/{caseId}/{reportId}```
>>>- Body request: Stream document.
>>>- Response: Job status. An object like the one in the request described above is returned.
>>-Status. 
>>>- To request the status of a given job. The identifier of the job corresponds to the token returned by any of the previous requests.
>>>- GET request: ```/api/v1/Annotations/status?params=<token>```
>>>- Response: Job status. An object like the one in the request described above is returned.
>>-Results.
>>>- To obtain the results of a given job. The job identifier corresponds to the token returned by any of the previous requests. 
>>>- GET request: ```/api/v1/Annotations/results?params=<token>```
>>>- Object with the results of the annotation process:
>>>>- Analyzer: If the document wa analyze with OCR or not.
>>>>- Segments List. List of the segments extracted as a result, composed by objects with:
>>>>>- Id or identifier of the segment
>>>>>- Language of the results
>>>>>- String Text
>>>>>- Source composed by the Language and the string text sources (from the document).
>>>>>- Annotations list or the items that are recognised in the result text:
>>>>>>- Id or identifier
>>>>>>- Text of the annotation
>>>>>>- Offset
>>>>>>- Length
>>>>>>- Category
>>>>>>- CondifenceScore
>>>>>>- IsNegated
>>>>>>- IsDiscarded
>>>>>>- List of related links composed by the data of the source and an Id.

>-Methods to perform extraction operations synchronously (NCR). Baseline request:```/api/v1/SyncAnnotations/```
>>-Process. 
>>>- To make both external and internal queries to Dx29.
>>>- POST request: ```/api/v1/SyncAnnotations/```
>>>- Body request: string or text of the document to be extracted.
>>>- Response: Object with the results of the annotation process line the one of the asynchronous results request.

It is programmed in C#, and two Docker images will result from this project: Annotations and AnnotationsJobs. The first one will contain the extraction functionality, while the second one will be used to manage the asynchronous jobs.
To configure it, it is necessary to create a ServiceBus, where the asynchronous jobs will be sent.
And finally, it should be mentioned that it depends on two external services: NCR and TextAnalytics.

The structure of the project is as follows:
>- **Dx29.Annotations.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>- **Dx29.Annotations**. It is this project that contains the logic to perform the relevant operations: services and communication with the external NCR and Text Analytics apis.
>- **Dx29.Annotations.Worker**. This project manages the Dispatcher required for the asynchronous functionalities. The administration and communication with the ServiceBus is carried out in this project.
>- **Dx29**, **Dx29.Azure** and **Dx29.Jobs** used as libraries to add the common or more general functionalities used in Dx29 projects programmed in C#.


#### 1.4.2.2. Dx29.APIGateway
Dx29 exposes an API for third parties to use the tool's algorithms independently of the tool. That is, they will be able to run the calculations they need but not access the specific user and patient information stored in the databases or blobs.

It therefore offers the methods of the microservices that perform these functions, and simply acts as a link between an external application and the internal microservices located in the application cluster. Thus, the responses obtained in each case will be of the same type as the one described in the specific section for each microservice.
The difference is that now the base address will be: "http://dx29-api.northeurope.cloudapp.azure.com/api/" and then the controller where the desired requests are located and the path of the request must be specified. 
The APIs that he explains can be consulted in this [link](http://dx29-api.northeurope.cloudapp.azure.com/index.html)

It is programmed in C#, and a Docker image will result from this project.

The structure of the project is as follows:
>- **Dx29.APIGateway**. Where the controllers and methods exposed by this API are implemented. In addition, it will contain the necessary functionality to connect the API controllers and methods to the corresponding microservice.
>- It also includes the generic projects or considered as a library, as it is programmed in C#: **Dx29** and **Dx29.Data**.

It is important to mention that for now this project includes the code of two microservices that are no longer used (and therefore not necessary): Dx29.Bioenties and Dx29.TASearch. These functionalities have already been migrated to other microservices in use.

#### 1.4.2.3. Dx29.Bioentity
The Bioentity microservice is used to obtain symptom and disease information.

For this purpose, it offers the following methods organised according to the type of search in different controllers:
>- Conditions Controller: To search for diseases or conditions. Baseline request: ```api/v1/Conditions/```
>>-Describe.
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
>>>- Result: An object like the one in the request of describe diseases method is returned.
>- Terms Controller: To search for diseases or symptoms regardless of type, i.e. a search for terms in all files.
>>-Describe.
>>>- To obtain information on terms from a list of IDs and in a given language.
>>>- Both GET and POST request are available.
>>>>- GET Request: ```api/v1/Terms/describe?id=<List<string>>&lang=<lang>```
>>>>- POST Request:```api/v1/Terms/describe?lang=<lang>```
>>>>- POST body request: List of ids (strings).
>>>- Result: An object like the one in the request of describe diseases method is returned.

It is programmed in C#, and a Docker image will result from this project.

The structure of the project is as follows:
>-**Dx29.Bioentity.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>-**Dx29.Bioentity**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29**. Used as library to add the common or more general functionalities used in Dx29 projects programmed in C#.

Please note that it uses Orphanet, Mondo, Omim and HPO files as sources of information. Please note the updates of these files for this project.

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
>-**Dx29.BioNET.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>-**Dx29.BioNET**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29**. Used as library to add the common or more general functionalities used in Dx29 projects programmed in C#.
>- **Dx29.Bioentity**. It uses its own implementation of bioentity methods.


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
>-**Dx29.Documents.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>-**Dx29.Documents**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29** and **Dx29.Azure**. Used as libraries to add the common or more general functionalities used in Dx29 projects programmed in C#.

#### 1.4.2.6. Dx29.FileStorage
For the management and administration of user documents for the correct functioning of the Dx29 application.

This microservice will therefore access the corresponding blob and folder (specific user), and will work on the requested document.

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
>-**Dx29.FileStorage.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>-**Dx29.FileStorage**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29** and **Dx29.Azure**. Used as libraries to add the common or more general functionalities used in Dx29 projects programmed in C#.

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
>-**Dx29.Localization.Web.API**. In this project is the implementation of the controllers that expose the aforementioned methods.
>-**Dx29.Localization**. It is this project that contains the logic to perform the relevant operations.
>- **Dx29** and **Dx29.Azure**. Used as libraries to add the common or more general functionalities used in Dx29 projects programmed in C#.


#### 1.4.2.8. Dx29.Mailing
Para que sirve:
Como se utiliza: Métodos que expone
En que lenguaje esta prorgamado
Organizacion general del proyecto:
Si requiere de algun servicio externo.


#### 1.4.2.9. Dx29.MedicalHistory
Para que sirve:
Como se utiliza: Métodos que expone
En que lenguaje esta prorgamado
Organizacion general del proyecto:
Si requiere de algun servicio externo.


#### 1.4.2.10. Dx29.PhenSimmilarity
Para que sirve:
Como se utiliza: Métodos que expone
En que lenguaje esta prorgamado
Organizacion general del proyecto:
Si requiere de algun servicio externo.


#### 1.4.2.11. Dx29.Segmentation
Para que sirve:
Como se utiliza: Métodos que expone
En que lenguaje esta prorgamado
Organizacion general del proyecto:
Si requiere de algun servicio externo.


#### 1.4.2.12. Dx29.TermSearch2
Para que sirve:
Como se utiliza: Métodos que expone
En que lenguaje esta prorgamado
Organizacion general del proyecto:
Si requiere de algun servicio externo.


#### 1.4.2.13. Dx29.WebManagement
Para que sirve:
Como se utiliza: Métodos que expone
En que lenguaje esta prorgamado
Organizacion general del proyecto:
Si requiere de algun servicio externo.


### 1.4.3. External containers
Para que sirve:
Como se utiliza: Métodos que expone
En que lenguaje esta prorgamado
Organizacion general del proyecto:
Si requiere de algun servicio externo.




