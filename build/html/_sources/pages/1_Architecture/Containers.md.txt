<img align="right" width="100px" src="../../_images/Foundation29.png">

## 1.2. Level 2: Containers

Dx29 consists of several connected containers. We have divided them according to their functionality and use within the platform.Thus, we will have:
>- Backend containers:
>>- Manage data containers
>>- Algorithms and functionalities containers
>>- Expose algorithms and functionalities container
>- Frontend containers:
>>- Webapp container
>- External containers

TODO:image

The webapp will be the core of the Dx29 application, from where the frontend will be developed and the communications with different services to provide functionalities.

The backend will however be divided into three types of containers: those in charge of managing data (databases and blobs), those that will perform the necessary calculations (algorithms) to expose the functionality to users, and finally there will be a container in charge of exposing the methods of the previous ones so that they can be used independently of Dx29.

Finally, some external services will also be used to simplify the implementation, development and maintenance of the product.
