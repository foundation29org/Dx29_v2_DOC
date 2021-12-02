<img align="right" width="100px" src="../_images/Foundation29.png">

# 1. Architecture

Dx29 uses a **microservices-based architecture**. 

This architecture, also known by its acronym MSA (Micro Services Architecture), is a form of software development that is based on the independent implementation of each of the application's services. 

This allows us to partition the services so that development, communications, deployment and production can be carried out without dependency, deployment and production can be carried out without the dependency that exists in the monolithic architecture.

Microservices architecture is an application of distributed programming, which allows the development of open, scalable, transparent and fault-tolerant systems, thus achieving a more continuous production.

Another characteristic of this architecture is that it allows us to develop our application in different programming languages, since our services will be isolated and will communicate with global mechanisms. As we can deduce, this makes the services more specialised, i.e., by being able to choose different types of languages, we will be able to solve more specific problems more efficiently.

Finally, it should be noted that maintenance is faster and more durable.


On the other hand, to manage and administer these services, we are going to use **Kubernetes**. 

The physical Kubernetes cluster can be partitioned into several virtual clusters, called namespaces. This is one of the objects that help us in the security and connection between containers.

Kubernetes divides its architecture into two nodes. Focusing on the "worker nodes", let's describe the main components:

>- **PODS**: These nodes, will be composed of different PODS: Set of containers running sharing the same network stack, volumes and memory. connection between containers. 
>- **REPLICA CONTROLLERS**: Abstraction mechanism over a set of pods, whose unique functionality is to keep a specific number of pod instances active. They scale up or down these instances and replace them if it is detected that they are not working properly.
>- **DEPLOYMENT**: Abstraction on top of the Replica Controller, and most commonly used component for the deployment of the pods, provides continuous updates to the pods. Deployment -> Deployment -> Abstraction on top of the Replica Controller, and most used component for pod deployment, provides rolling updates and rollbacks as it keeps a historical record of the versions that keep a historical record of the container versions.
>- **SERVICES**: Due to the continuous creation and destruction of pods, a mechanism is needed that allows us to define an endpoint, with the objective of interruptively connecting containers that remain in different pods.
>- **SECRETS**: Mechanism that allows us to define keywords or sensitive words, so that they are not revealed. 
>- **VOLUMES**: Mechanism that allows us to store persistent information.


The containers will be those that contain each of the microservices that have been implemented. To create these containers we use **Docker**.
Docker is an open source software that allows you to virtualise at the operating system level using containers, which allows you to have an additional layer of automation of multiple applications on different operating systems.

Therefore, we need to look at the difference between virtualisation and containers. 
>- To virtualise is to create a virtual translation of software, such as an operating system or any type of server, among others, by means of a program. 
>- With Docker containers, not all calls to the operating system are emulated. system and this leads to better performance, as it takes advantage of the kernel of the host machine. kernel of the host machine.

The main components are:
>- **IMAGE**: Template where the application and its different libraries are included, which are used to create the containers.
>- **DOCKERFILE**: It is a file where the configuration that allows the creation of the images is included. This file indicates the operating system to be executed and the different commands that allow the necessary tools to be installed.
>- **Container**: These are instances that are produced when executing an image containing the application that we have developed.
>- **VOLUMES**: They are used to store the information of the containers, so that the data is persistent. data are persistent. When a container is deleted, its data is deleted.

Having clarified the chosen architecture and the concepts necessary for its implementation, we now proceed to detail the components that make it up. To do so, we are going to use the C4 model. Thus, in the description of this section we will differentiate between:

- Level 1: A **System Context** diagram provides a starting point, showing how the software system in scope fits into the world around it.
- Level 2: A **Container** diagram zooms into the software system in scope, showing the high-level technical building blocks.
- Level 3: A **Component** diagram zooms into an individual container, showing the components inside it.
- Level 4: A **code** diagram with the technical explanation about the programming of each one of the modules or components.

<p style="text-align: center;">
	<img width="100%" src="../_images/SW_Architecture_C4.png">
</p>