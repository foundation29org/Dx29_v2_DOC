<img align="right" width="100px" src="../_images/Foundation29.png">

# 4. Build & Deploy

The Dx29 application uses a microservices architecture that is deployed in Kubernetes on Azure.
As described so far, it will consist of three environments: development, test and production. In this section we are going to show how the build and deploy tasks would be done for any of them, so we will only focus on an X environment.

For the execution of these tasks, as we work with Gitflow, the only difference at code level will be the branch used for each of the environments. On the other hand, the configuration settings for each one are given by the manifests files, which as we have seen will also be different for each environment and will be located in different projects. 
>- The development environment will use the develop branch of code and the manifest files located in the same repository as the project.
>- The test environment will use the main branch of the code and the manifest files will be located in the same repository as the execution pipeline (Dx29.EnvTEST).
>- The production environment will also use the main branch of the code and the manifest files will be located in the same repository as the execution pipeline (Dx29.EnvPROD).

Taking this into account, in this section we will define the steps in a generic way for the Build and Deploy tasks, that is, these tasks will follow the same steps independently of the environment, only taking into account the different sources of information input.

On the one hand, the **Build tasks** will comprise the creation of the **Docker containers** for each of the projects separately. In this way, we will be able to automate the implementation of the different functionalities as self-sufficient portable containers that can be run in the cloud or in local environments. 

**Kubernetes**, on the other hand, is an open source orchestration software that provides an API to control how and where containers will run. It allows you to run Docker containers and workloads and addresses some of the operational complexities of scaling multiple containers deployed on multiple servers. In other words, **deployment** will take place in a Kubernetes cluster that will orchestrate and organise each of the Docker microservices located in an Azure Container Registry:
>- Deployment of the Docker images
>- Deployment on the kubernetes cluster

While the promise of containers is that they can be scheduled once and run anywhere, Kubernetes allows you to organise and manage all your container resources from a single control plane. It makes it easy to configure networking, load balancing, security, and scaling across all Kubernetes nodes where you run your containers. Kubernetes also has a built-in isolation mechanism, similar to namespaces, that allows you to group container resources by access permission, interim storage environments, and more.

We use Kubernetes with Docker to:
>- Improve the security and high availability of the Dx29 application. It will stay connected, even if some of the nodes go offline.
>- Make Dx29 more scalable. It scales the load horizontally to provide a better user experience and it is easy to launch more containers or add nodes to the Kubernetes cluster.

With all this, we divide this section into sub-sections to:
>- Develop more in depth what the **build and deploy tasks** of the Dx29 application consist of and how they are performed.
>- Expose the **automation** of these tasks using Azure pipelines.
>- Indicate how to **work locally** with the Dx29 application.
>- To give some notes on the **deployment status** of the Dx29 application environments and how to work with them.