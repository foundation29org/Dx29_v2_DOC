<img align="right" width="100px" src="../../_images/Foundation29.png">

## 1.3. Level 3: Component

Taking into account the division by containers exposed in the previous point, we are now going to study in detail the internal architecture and the components that make up each one of them.
These components are intercommunicated using a [REST](https://restfulapi.net/) interface, that is, the communication is established according to the [HTTP protocol](https://restfulapi.net/http-methods/). 

### 1.3.1. Backend containers
#### 1.3.1.1. Manage data containers
These containers will handle the management of user data, being able to read, save, update and delete this information from the Dx29 blobs and databases.
In addition, they will also expose the methods for the application to access the files necessary for its configuration.

How is this information organised?
>- For the **databases**:
>>- **SQL** database: It is controlled from the frontend directly, but it is important to comment at this point that there is a database of this type that stores the information for the login of the users in Dx29. That is to say, it will only contain the user's information, not the patient's information.
>>- **CosmosDB** databases: We will have several databases where the patient information is stored. For security reasons, personal data will not be stored in the same location as medical information, to prevent them from being related to each other. In other words, Dx29 ensures anonymity.

>- In the **blobs**, an input will be created for each medical case or medical history where the files that have been attached (uploaded) will be stored, as well as others necessary for the functioning of the application: literals, documents,...

Thus:
>- On one side we will have the components in charge of database management: It will be the microservice of Dx29.MedicalHistory.
>- On the other hand, we will have the components in charge of blob management: Dx29.FileStorage, Dx29.Localization and Dx29.Documents microservices.

Each microservice will expose within the Kubernetes Cluster the methods to be able to work and will be programmed in the programming language that optimises the development (This will be detailed later for each of them, but the languages used are: NodeJS, C#, and Python).

#### 1.3.1.2. Algorithms and functionalities containers
The Dx29 tool provides a list of functionalities for users. The execution of the algorithms or calculations necessary to achieve this will be divided into different components. 

Thus, we will have different components for:
>- The extraction of information from medical documents, including genotyping.
>- The calculation of disease suggestions according to patient data.
>- Obtaining information on diseases and symptoms for the user's language.
>- Offering a differential diagnosis based on the phenotypic comparison of the patient with a disease.
>- Obtaining more information on a disease such as: documentation, patient groups and clinical trials.
>- The search for symptoms and diseases with natural language.
>- The ability to send emails.

Each of these functionalities will be implemented using one or several microservices that, as before, will expose their methods. These will also be programmed in the programming language that optimises their development, taking into account that the ones we use are: NodeJS, C# and Python.


#### 1.3.1.3. Expose algorithms and functionalities container
In addition to the above, Dx29 offers the possibility for third parties to use its algorithms or calculations outside the environment, i.e. without using the application data or having to use the user interface.

This has been achieved by implementing a dedicated component: Dx29.APIGateway. It will be in charge of exposing all the methods of the algorithms and functionalities containers.

Therefore, this component is programmed in C#.

### 1.3.2. Frontend containers
#### 1.3.2.1. Webapp container
This container is where you will place the Web application component. That is, the frontend.

For this, [Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor) and [C# - ASP.NET](https://docs.microsoft.com/en-GB/visualstudio/get-started/csharp/tutorial-aspnet-core?view=vs-2022) has been used as programming languages.

This application communicates with the different microservices to display the relevant information. 

To use it, it is necessary to be registered, this was implemented using the [Identity](https://docs.microsoft.com/en-GB/aspnet/core/security/authentication/identity?view=aspnetcore-6.0&tabs=visual-studio) service that will access the SQL database.

### 1.3.3. External containers
Finally, as mentioned above, in order to simplify the architecture, implementation, development and maintenance of the application, external services are used.

All of them will be accessed using [REST](https://restfulapi.net/).

The list of these is as follows:
>- [Exomiser](https://github.com/exomiser/Exomiser). Foundation29 has also implemented its own Kubernetes cluster to manage this resource, therefore, the Dx29 application accesses this which in turn runs Exomiser and performs a management and administration of this job, allowing it to be used asynchronously.
>- [F29API](https://f29api.northeurope.cloudapp.azure.com/index.html). It exposes different functionalities, in particular the Dx29 application uses the methods of the Documents driver to prepare and configure the inputs to the services for extracting information from the user's documents.
>- [Azure Microsoft translator](https://docs.microsoft.com/en-GB/azure/cognitive-services/translator/translator-overview).