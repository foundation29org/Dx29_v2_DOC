<img align="right" width="100px" src="../../_images/Foundation29.png">

## 2.3. Prod environment

The last environment in software development, this is where new builds/updates are moved into production for end users. 

The Dx29 application test environment will consist of the following components:

|  Component               |   Azure   														     |		  Configuration                                         | Description                                                              |
|--------------------------|---------------------------------------------------------------------|--------------------------------------------------------------|--------------------------------------------------------------------------|
| Azure Kubernetes Service | [link](https://docs.microsoft.com/en-us/azure/aks/)                 | Two instances for the most critical microservices and one for those that do not handle so much traffic. | To manage and operate the containers orchestrated on Kubernetes, which contain the microservices necessary for the deployment of the Dx29 application in the production environment. |                                                                     
| Container Registry       | [link](https://docs.microsoft.com/en-GB/azure/container-registry/)  | List of Docker images for each microservice with versions (tags) | Azure Container Registry (ACR) is a private registry for container images. A private container registry allows you to securely build and deploy custom code and applications. It is associated with an Azure resource cluster. The Kubernetes cluster is associated with this resource to use the images contained therein for deployments.|
| Public IP and DNS		   | [link](https://docs.microsoft.com/en-us/azure/aks/static-ip)        | https://dx29.ai/              | To access Dx29 application from production environment |
| SQL database			   | [link](https://docs.microsoft.com/en-GB/azure/azure-sql/database/)  | Information about the user: email, encrypted password, First name, Last name, Language and role: Physician or Patient | For managing user accounts of Dx29 application in production environment |
| CosmosDB databases       | [link](https://docs.microsoft.com/en-gb/azure/cosmos-db/introduction) | With [SQL API](https://docs.microsoft.com/en-gb/azure/cosmos-db/choose-api) | There are now three such databases. The patient's personal information is separated from the patient's medical data. Thus, in each of the databases will be stored: the user's data (caseRecord), their associated medical cases (medicalCases), and the resources or information of each case (resourceGroups), and all of them will be independent of each other. |
| Blob storage             | [link](https://docs.microsoft.com/en-GB/azure/storage/blobs/)      | One container for the application literals, another for the documents, another for communication with openDx29 and finally a private container for each Dx29 medical case. | It is organised in different containers for the following functionalities: translation of Dx29 in English or Spanish, access to documents required by the application such as the privacy policy or emails, or access to a user's data in a private and secure way (medical reports, genotypes, results of Dx29 application processes, etc |
| ServiceBus              | [link](https://docs.microsoft.com/en-GB/azure/service-bus-messaging/service-bus-messaging-overview) | Queue | Dx29 messaging service |

In addition, it uses some components of the development environment:

|  Component               |   Azure   														                                     | Configuration  | Description                                                       |
|--------------------------|-----------------------------------------------------------------------------------------------------|----------------|-------------------------------------------------------------------|
| Application Insights     | [link](https://docs.microsoft.com/en-GB/azure/azure-monitor/app/app-insights-overview)              | -              | For the logging of the Dx29 application                           |
| SignalR                  | [link](https://docs.microsoft.com/en-GB/azure/azure-signalr/signalr-overview)                       | -              | Service for the management of real-time application notifications |  
