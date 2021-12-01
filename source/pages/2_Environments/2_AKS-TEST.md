<img align="right" width="100px" src="../../_images/Foundation29.png">

## 2.2. Test environment

This is where application testing is conducted to find and fix errors. All the work done in the development environment is merged into the built system before it is moved into the production environment.

A test environment is where the testing teams analyze the quality of the application/program. This also allows computer programmers to identify and fix any bugs that may impact smooth functioning of the application or impair user experience.
A test environment allows software developers to check how a code/program will behave in a live environment. The testing environment should closely resemble the production environment since it is one of the last safe places to find and fix environment-related bugs before moving into production.

The Dx29 application test environment will consist of the following components:

|  Component               |   Azure   														     |		  Configuration                                         | Description                                                              |
|--------------------------|---------------------------------------------------------------------|--------------------------------------------------------------|--------------------------------------------------------------------------|
| Azure Kubernetes Service | [link](https://docs.microsoft.com/en-us/azure/aks/)                 | One instance for each deployment and service                 | To manage and operate the containers orchestrated on Kubernetes, which contain the microservices necessary for the deployment of the Dx29 application in the test environment. |                                                                     
| Container Registry       | [link](https://docs.microsoft.com/en-GB/azure/container-registry/)  | List of Docker images for each microservice with versions (tags) | Azure Container Registry (ACR) is a private registry for container images. A private container registry allows you to securely build and deploy custom code and applications. It is associated with an Azure resource cluster. The Kubernetes cluster is associated with this resource to use the images contained therein for deployments.|
| Public IP and DNS		   | [link](https://docs.microsoft.com/en-us/azure/aks/static-ip)        | https://dx29test.northeurope.cloudapp.azure.com/              | To access Dx29 application from test environment |
| SQL database			   | [link](https://docs.microsoft.com/en-GB/azure/azure-sql/database/)  | Information about the user: email, encrypted password, First name, Last name, Language and role: Physician or Patient | For managing user accounts of Dx29 application in test environment |
| CosmosDB databases       | [link](https://docs.microsoft.com/en-gb/azure/cosmos-db/introduction) | With [SQL API](https://docs.microsoft.com/en-gb/azure/cosmos-db/choose-api) | There are now two such databases. A separation of data is starting to be made, although it is not entirely correct in terms of security (as it is a test environment this is not a problem as in development). Now we have one for the user data and its associated medical cases: caseRecords and medicalCases, and another one for the resources of each c: resourceGroups. |
| Blob storage             | [link](https://docs.microsoft.com/en-GB/azure/storage/blobs/)      | One container for the application literals, another for the documents, another for communication with openDx29 and finally a private container for each Dx29 medical case. | It is organised in different containers for the following functionalities: translation of Dx29 in English or Spanish, access to documents required by the application such as the privacy policy or emails, or access to a user's data in a private and secure way (medical reports, genotypes, results of Dx29 application processes, etc |

In addition, it uses some components of the development environment:

|  Component               |   Azure   														                                     | Configuration  | Description                                                       |
|--------------------------|-----------------------------------------------------------------------------------------------------|----------------|-------------------------------------------------------------------|
| Application Insights     | [link](https://docs.microsoft.com/en-GB/azure/azure-monitor/app/app-insights-overview)              | -              | For the logging of the Dx29 application                           |
| SignalR                  | [link](https://docs.microsoft.com/en-GB/azure/azure-signalr/signalr-overview)                       | -              | Service for the management of real-time application notifications |  
| ServiceBus               | [link](https://docs.microsoft.com/en-GB/azure/service-bus-messaging/service-bus-messaging-overview) | Queue          | Dx29 messaging service                                            |
